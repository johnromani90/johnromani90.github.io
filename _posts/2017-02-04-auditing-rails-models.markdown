---
title:  "Auditing models with Ruby on Rails"
date:   2017-02-04 10:18:00
description: adding additional audit lookups from associated foreign key
---

This week I worked on auditing using the [audited gem](https://github.com/collectiveidea/audited){:target="_blank"}. The idea of auditing models is  not new or necessarily complex, however, the implmentation can effect production cost as we are essentially creating new row in our "audits" table every time our logic says "save this."

The gem implements [polymorphism](https://johnromani90.github.io/2017/setting-up-polymorphism-in-ruby-on-rails/){:target="_blank"} and allows you to specify which models to audit. The audits table has an "auditable_id" and an "auditable_type" along with "user_id" which makes for a handy "current_user" helper method that you can use to pull the user on any audit instance.

Now our client has had auditing in place for some time now, but recently they asked for a better way to view the changes on the audits screen. I was unaware at the time, but the way the gem works is that it returns the Model that was changed along with the id of that instance. So from a ui perspective seeing things like this are not very useful...

![alt text](/assets/images/before_screenshot.png "before")

Currently the gem does not have a nice way of [showing related display names from the foreign key](https://github.com/collectiveidea/audited/issues/156){:target="_blank"}.

Because audits are obviously important, I decided to start by test driving this with some requests tests (the audited models were already tested well and we are not actually making any changes to the table or the way that it is written). 

First, the environment

```ruby
  describe "order audit" do

    before :each do

      @patient2 = FactoryGirl.create(:patient)
      @order_outcome = FactoryGirl.create(:order_outcome)
      @prescriber = FactoryGirl.create(:prescriber)
      @product = FactoryGirl.create(:product)
      @activity = FactoryGirl.create(:activity)
      @shipment_tracking_number = Faker::Code.isbn if (1..6).to_a.sample == 1

      #this will show initial creation of my audited model
      @order = FactoryGirl.create(:order, patient: @patient,
                                  prescriber: @prescriber,
                                  activity: @activity,
                                  basket_number: 400,
                                  insurance_type_id: 1
      )

      # this will show the new changes on the audit
      @order.update(
          patient: @patient2,
          order_status_id: 1,
          order_outcome: @order_outcome,
          prescriber: @prescriber,
          product: @product,
          shipment_tracking_number: @shipment_tracking_number

      )

      visit "/orders/#{@order.id}/audit"

    end
```

and the specs

```ruby
    context "when updating with no user" do

      it "shows names of the things that changed" do
        expect(page).to have_content "Changed Activity to #{@activity.display_name}"
        expect(page).to have_content "Changed Basket Number to #{400}"
        expect(page).to have_content "Changed Patient to #{@patient.display_name}"
        expect(page).to have_content "Changed Order Status to #{OrderStatus.find(1).display_name}"
        expect(page).to have_content "Changed Prescriber to #{@prescriber.display_name}"
        expect(page).to have_content "Changed Insurance Type to #{InsuranceType.find(1).display_name}"
      end

      it "shows names that were updated after create" do
        prescriber = FactoryGirl.create(:prescriber)
        product = FactoryGirl.create(:product)
        activity = FactoryGirl.create(:activity)
        @order.update!(
            prescriber: prescriber,
            order_outcome_id: nil,
            activity: activity,
            insurance_type_id: 2,
            product: product,
        )

        visit "/orders/#{@order.id}/audit"

        #first changes
        expect(page).to have_content "Changed Tag Number to #{@shipment_tracking_number}"
        expect(page).to have_content "Changed Activity to #{@activity.display_name}"
        expect(page).to have_content "Changed Basket Number to #{400}"
        expect(page).to have_content "Changed Patient to #{@patient.display_name}"
        expect(page).to have_content "Changed Order Status to #{OrderStatus.find(1).display_name}"
        expect(page).to have_content "Changed Prescriber to #{@prescriber.display_name}"
        expect(page).to have_content "Changed Insurance Type to #{InsuranceType.find(1).display_name}"

        #second changes
        expect(page).to have_content "Changed Activity from #{@activity.display_name} to #{activity.display_name}"
        expect(page).to have_content "Changed Prescriber from #{@prescriber.display_name} to #{prescriber.display_name}"
        expect(page).to have_content "Changed Insurance Type from #{InsuranceType.find(1).display_name} to #{InsuranceType.find(2).display_name}"
      end

    end
```

Basically the above is creating an instance of a model that is audited (order in this case) and makeing some changes to it. We then visit the audit page for that resource and check to see if the actual __names__ are displayed instead of the id (every model in the project responds to #display_name which returns a friendly string with title if necessary)

Now the gem gives you back your audits in an array, along with the foreign key (the "auditable_type") and the id ("auditable_id"). In order to get the display names we need to make a custom method.

```ruby
  def classify_foreign_key(foreign_key)
    foreign_key = "user_id" if foreign_key == "owner_id"
    if foreign_key =~ /_id$/
      attr = foreign_key.sub(/_id$/, '')
      attr = attr.classify.safe_constantize
    else
      attr = nil
    end
    return attr.nil? ? foreign_key : attr
  end
```

and some tests for our method

```ruby
describe AuditsHelper do
  describe "#classify_foreign_key" do
    it "returns class name" do
      p = "patient_id"
      expect(helper.classify_foreign_key(p)).to eq Patient
    end

    it "returns original value if no class is found" do
      p = "patienttt"
      p2 = "patientttt_id"
      expect(helper.classify_foreign_key(p)).to eq p
      expect(helper.classify_foreign_key(p2)).to eq p2
    end

    it "returns original value of there is no _id at end of string" do
      p = "patient"
      expect(helper.classify_foreign_key(p)).to eq p
    end

    it "works for other classes" do
      u = "user_id"
      o = "order_id"
      product = "product_id"

      expect(helper.classify_foreign_key(u)).to eq User
      expect(helper.classify_foreign_key(o)).to eq Order
      expect(helper.classify_foreign_key(product)).to eq Product
    end
  end

end
```

The reason I checked if the key was "owner_id" is because the our user table has a relationship with a separate model that is not thought of as a user, but rather as an "owner". This is not ideal, and I can see this becoming a problem in the future if more relationships like this are created, but for now I am satisfied with the results.

Now we can feel confident that every time we pass in our foreign key, we will either get back a "display_name" or the original value (which is the desired case in certain situations)

End result (all data is fake):

![alt text](/assets/images/end_screen_shot.png "before")

I would love to hear if you have any feedback or have been through a similar experience with auditing.

---
title:  "Setting Up Polymorphish in Ruby On Rails"
date:   2017-01-25 10:18:00
---
Polymorphism can take a little longer to understand conceptually than other topics in OO. It is an abstraction which takes a little more effort for the developer to implement, but makes extending easier later on.
In this example I have three models that I want to add an attachment too: Patient, Prescriber, Order. Each attachment requires the same attributes and instead of adding a new attachment column on each model I am electing to create a new model called "AttachmentObject" and make it polymorphic. I am using the paperclip gem with S3 for attachment upload.

```ruby
class AddAttachmentPolyObject < ActiveRecord::Migration
  def change
    create_table :attachment_objects do |t|
      t.string :title, null: false
      t.references :attachable, polymorphic: true, index: true, null: false
      t.attachment :attachment, null: false

      t.timestamps
    end
  end
end
```

```ruby
class AttachmentObject < ActiveRecord::Base
 belongs_to :attachable, polymorphic: true
 validates :title, :attachable_id, :attachable_type, presence: true
end
```

then for my relationships to my "attachable" models (Patient, Prescriber, Order).

```ruby
class Patient < ActiveRecord::Base
 has_many :attachment_objects, as: :attachable
end


class Prescriber < ActiveRecord::Base
 has_many :attachment_objects, as: :attachable
end


class Order < ActiveRecord::Base
 has_many :attachment_objects, as: :attachable
end
```

It is not necessary to name your "has_many" relationship with a name that uses "able" as a suffix, but you will see that it is a popular amongst rails devs and you should use it as a convention when possible. Now comes the fun parts. With my AttachmentObject I can now use rails helpers to find its related attachable parent. Active Record does this by looking up the 'attachable_type' (which points to either Patient, Prescriber, or Order table) and 'attachable_id' which points to a row on the table. for example:

```ruby
#explicitly
order = Order.create( valid_attributes )
order_attachment = AttachmentObject.create(title: "phish", attachable_type: "order", attachable_id: order.id) 

#or implicitly
order_attachment = AttachmentObject.create(title: "phish", attachable: order) 
```

You now have access to some familiar rails helpers like

```ruby

phish = AttachmentObject.find_by_title("phish") 
phish.attachable #returns the related order_attachment

```


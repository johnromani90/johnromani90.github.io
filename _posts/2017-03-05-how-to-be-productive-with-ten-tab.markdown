---
title:  "How to be productive with the Ten Tab method"
date:   2017-03-05 10:18:00
description: using Tamper Monkey and a simple JavaScript to learn more efficiently
---
I'm a developer and by far the most valuable tool that I use everyday is the Google search engine. In today's world it matters less about what you know at this moment in time and more about how efficient you are in learning new things.

One mistake that I see developers making is rushing into an issue, wanting to solve the problem with time still left on the clock, without researching some of the basics first. I am guilty of this myself, and I have found that more times than not this attitude can ultimately slow down progress on an issue.

Even when feeling confident about implementing a feature, it still helps taking a few minutes to brush up on some of the things you may have overlooked.

Some of the best advice that I have received is when ever you don't know how to do something (which is at least ten times daily) go to your favorite search engine, ask your question, and then open every link on the first page in a new tab. Now, before making any assumptions about what you do or do not know about what ever it is you are asking, skim through every tab, closing the ones that seem irrelevant, and leaving open the ones you find helpful.

While skimming through all the tabs a couple of things happen:

***

1. You will notice common terms or patterns that have been proven to work.
2. You will see the problem from multiple perspectives.
3. You will extinguish your own misconceptions about the problem.

***

More than a couple of times I have found myself wishing I had spent 5 more minutes researching before choosing a [design pattern](https://johnromani90.github.io/2016/blog-post/).

To encourage this technique, I made a JavaScript with Tamper Monkey to open up first page results in new tabs. [Tamper Monkey](https://tampermonkey.net/){:target="_blank"} is a Google Chrome extension that allows you to write your own custom JavaScripts. You can find the code here:


```javascript

$(document).ready( function(){
  function add_google_button( div ){
    if (div.length){
      list = $("h3");
      google_button = $('<input id="ten-tab-button" type="button" value="Ten Tab">');
      div.append(google_button);
      google_button.css({"font-size": "16px",
                     padding: "8px",
                     "background-color": "black",
                     color: "white",
                     border: "none",
                     display: "block",
                     "font-weight": "bold"});
        google_button.click(function(){
            list.each(function(){
                if ($(this).find('a').attr("href")){
                 console.log($(this).find('a').attr("href"));
                  window.open($(this).find('a').attr("href"));
                }
            });
        });
    }
  }

    var test = $('#topabar');
    add_google_button( $(test) );
});

```

A simple implementation but it is a friendly reminder to take more time learning and less time assuming.
---
layout: post
title:      "JS Off The Rails"
date:       2019-01-04 18:41:31 +0000
permalink:  js_off_the_rails
---


Alright, so, man, here's the thing..... JS is complicated. AJAX is complicated.  Learning both of those things together.... Yep, complicated.

But we're Flatiron students. We live for complicated.

I remember starting this project, staring at a blank file, utterly stumped by the list of requirements in front of me. I remember thinking to myself, "Man, I have no idea what I'm doing."

Lots of hours later, well, I still feel like I don't know what I'm doing, but my code sure does, so let's look at it!

# SPA, Anyone?

Nah, I didn't take a break to get massages and wear bath robes and steam in rooms (Though that would have helped my stress a lot!).

SPA stands for Single Page Application, a web application where everything the user interacts with happens on one browser page. We accomplish this as developers with, you guessed it, lots of JS and AJAX. [Medium](https://medium.com/@NeotericEU/single-page-application-vs-multiple-page-application-2591588efe58) has a good article explaining some of the pros and cons of single page vs multi-page apps.

I decided that [GunSafe](http://github.com/almighty-mose/gunsafe) would benefit from being a SPA. This led to lots of head scratching, cursing, and moaning, and has generally been regarded by historians as a bad move.

Okay, so it wasn't that bad, but the switch did present a lot of unique challenges that really served to solidify my understanding of JS and the DOM.

So, let's check some code!

I decided that I would use the index of a user's firearms (the firearms_path in Rails, firearms/index.html) as the foundation of my redesign, as a list of a user's firearms seems the most logical access point. 

First task: Display that list of firearms using JS and AJAX.

I need a container to insert that list into:

```
<div id="firearmList">
  <button class="accordion">Rifles</button>
  <div class="panel">
    <ul id="rifle-list">
    </ul>
  </div>
  <button class="accordion">Pistols</button>
  <div class="panel">
    <ul id="pistol-list">
    </ul>
  </div>
  <button class="accordion">Shotguns</button>
  <div class="panel">
    <ul id="shotgun-list">
    </ul>
  </div>
</div>
```

Lookin' good! Okay, now how do we populate that list? Enter JS:

First, a function **populateFirearmsIndex()** (first one to guess what that one does wins a prize!)

```
function populateFirearmsIndex() {
  //We need to grab all the user's firearms
  $.get("/firearms.json", function(firearmData) {
    //Iterate over the JSON response, which is an array of firearms
    firearmData.forEach(function(firearm) {
      //Push the ID of each firearm into the firearmIds const
      firearmIds.push(firearm.id);
      //lines 39-41 create new objects and containers for each firearm
      let f = new Firearm(firearm);
      let li = document.createElement("li");
      let a = document.createElement("a");
      //set the required attributes for the firearm's <a> tag
      a.setAttribute('href', "javascript:void(0)");
      a.setAttribute("data-id", f.id);
      a.innerHTML = f.name;
      //SORT!
      if (f.category === "Rifle") {
        var $list = $("#rifle-list")
      } else if (f.category === "Pistol") {
        var $list = $("#pistol-list")
      } else if (f.category === "Shotgun") {
        var $list = $("#shotgun-list")
      };
      //Add the <li> to the proper list, then add in the formatted <a>
      $list.append(li);
      li.appendChild(a);
    });
  });
};
```


> Comment. Your. Code.

Mostly self-explanatory, I fire an AJAX GET request to my API via the /firearms path, which returns a JSON object containing the current user's firearms, then simply create a new local Firearm object for each one, prepare a list item and link tag for each one, then sort them into categories and append them into specific named containers.

One thing to note, I create a global const called firearmIds, which is an array of ONLY the current user's firearm IDs. This is so my "next" and "previous" buttons traverse only that user's firearms, instead of the entire database, cause my app doesn't allow users to see other's firearms.

So this isn't too crazy. What's crazy is that I decided, "Hey, wouldn't it be cool if when you clicked on a firearm, instead of rendering a new page, a little drawer popped open with the info and you never left the index page? Yeah, that'd be cool, right? Shouldn't be too hard."

Famous last words.

Turns out, nearly all of my project requirements ended up stuffed in this drawer, which presented some issues.

I'll start with the base container firearmDrawer:

```
<div id="firearmDrawer" class="drawer">
  <button class="js-prev">Previous</button>
  <button class="js-next" data-id="">Next</button>
  <a href="javascript:void(0)" class="closebtn" onclick="closeFirearmDrawer()">&times;</a>
  <h2 class="firearmName"></h2>

  <hr>

  <p>Caliber: <span class="firearmCaliber"></span></p>
  <p>Serial Number: <span class="firearmSerial"></span></p>
  <p>Purchase Price: $<span class="firearmPrice"></span></p>
  <p>Purchase Date: <span class="firearmPurchase"></span></p>
```

But wait! I want to put a firearm's accessories in there too! Gotta satisfy those nested requirements!

```
<div id="accessory-container">
    <h4>Accessories</h4>
    <button id="js-accessory-add">Add an Accessory</button>
    <div id="new-accessory-form">
      <!-- Handlebars form template accessories/new renders here --> 
    </div>
    <ul id="accessoryList">
    </ul>
  </div>
</div>

<div id="accessoryDrawer" class="drawer">
  <a href="javascript:void(0)" class="closebtn" onclick="closeAccessoryDrawer()">&times;</a>
  <h2 class="accessoryName"></h2>

  <hr>

  <p>Purchase Price: $<span class="accessoryPrice"></span></p>
  <p>Purchase Date: <span class="accessoryPurchase"></span></p>
</div>
```

Eagle eyed readers will notice the Handlebars comment hiding in there. I elected to try something new and built the accessories/new form into the firearmDrawer using a Handlebars template. We'll talk about that later. Spoiler alert: It was a pain in the butt.

*On to the JS!*

So, we assign a click event listener to each entry in the firearms list:

```
$("#firearmList").on("click", "a", function(e) {
    e.preventDefault();
    let id = $(this).data("id");
    insertFirearm(id);
    // Opens the firearmDrawer element which contains firearms info
    $("#firearmDrawer").css('width', '500px');
  });
```

When a firearm link is clicked (see the [event delegation](https://learn.jquery.com/events/event-delegation/) up there?), the function **insertFirearm(id)** is fired, passing through an ID parameter grabbed from the data-id attribute of the firearm link. That function looks like this:

```
function insertFirearm(id) {
  $.get("/firearms/" + id + ".json", function (data) {
    var firearm = new Firearm(data);
    //this will insert the data response into the firearmDrawer element of firearms/index.html
    firearm.insertIntoDrawer();
    let accessories = data["accessories"];
    insertAccessories(accessories);
  });
}
```

Here the actual AJAX request is made, firing a GET to "/firearms/:id.json". That data response (which is a JSON object representing a firearm) is passed to the firearm class constructor, which instantiates a new Firearm object, then uses the prototype method **insertIntoDrawer()** to inject that information into the DOM.

```
class Firearm {
  constructor(data) {
    this.name = `${data["make"]} ${data["model"]}`;
    this.caliber = data["caliber"];
    this.price = data["price"];
    this.serial = data["serial_number"];
    this.purchaseDate = data["purchase_date"];
    this.id = data["id"];
    this.category = data["category"]
  }
}

Firearm.prototype.insertIntoDrawer = function() {
  $(".js-next").attr("data-id", this.id);
  $("#edit-button").attr("href", `firearms/${this.id}/edit`)
  $(".firearmName").text(this.name);
  $(".firearmCaliber").text(this.caliber);
  $(".firearmSerial").text(this.serial);
  $(".firearmPrice").text(this.price);
  $(".firearmPurchase").text(this.purchaseDate);
}
```

But wait! We also wanted accessories! NESTED RESOURCES!!! HAS-MANY RELATIONSHIPS!!!!!!!

**insertAccessories(accessories)**

```
function insertAccessories(accessories) {
  let $list = $("#accessoryList");
  $($list).empty();
  accessories.forEach(function (accessory) {
    //create an <li> for each accessory
    let li = document.createElement("li");
    let a = document.createElement("a");
    //set the required attributes for the accessory's <a> tag
    a.setAttribute('href', "javascript:void(0)");
    a.setAttribute("data-id", accessory.id);
    a.setAttribute("class", "js-accessory")
    a.innerHTML = accessory.name;
    $list.append(li);
    li.appendChild(a);
  });
}
```

This function is passed its "accessories" parameter and invoked inside **insertFirearm()**. It then iterates over each accessory, stuffs them into a list and also makes them clickable links (which opens another drawer with the accessory info, basically just the same thing as **insertFirearm()**).

So, a firearm gets clicked, which fires a whole lot of JS to open a drawer and dynamically inject information about that firearm, including its accessories.

<iframe src="https://giphy.com/embed/4cUCFvwICarHq" width="240" height="180" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

# Dynamic Form Rendering!
This requirement taught me many things. All I need to do is generate a form to dynamically submit a resource via AJAX. I decided to use the accessories form, and have it create a new accessory that was already associated with its parent firearm. Double trouble!

Again, start as before with a container for the form:

```
<div id="accessory-container">
    <h4>Accessories</h4>
    <button id="js-accessory-add">Add an Accessory</button>
    <div id="new-accessory-form">
      <!-- Handlebars form template accessories/new renders here --> 
    </div>
    <ul id="accessoryList">
    </ul>
  </div>
```

Remember the Handlebars template I mentioned before? I needed a way to tell this form I was going to render both A) which firearm the accessory should be associated with and B) what the CSRF token my Rails app was, so the form could actually submit to my app.

I could no longer rely on implicit rendering and having access to Ruby objects passed in from the controller. I needed a better way.

So I built a Handlebars template.

```
<form class="new_accessory" id="new_accessory" action="/accessories" accept-charset="UTF-8" method="post">
    <input type="hidden" name="authenticity_token" value="{{csrf}}">
    <input value="{{firearm-id}}" type="hidden" name="accessory[firearm_id]" id="accessory_firearm_id" />
    <input placeholder="Name" type="text" name="accessory[name]" id="accessory_name" />
    <input placeholder="Price" type="number" name="accessory[price]" id="accessory_price" />
    <input placeholder="Date Purchased - MM/DD/YYYY" type="text" name="accessory[purchase_date]" id="accessory_purchase_date" />
    <select name="accessory[category]" id="accessory_category">
        <option value="Optic">Optic</option>
        <option value="Sling">Sling</option>
        <option value="Light">Light</option>
        <option value="Trigger">Trigger</option>
        <option value="Other">Other</option>
    </select>
    <input type="submit" name="commit" value="Create Accessory" data-disable-with="Create Accessory" />
</form>
```

Looks just like a form, but those values in {{}} are Handlebars expressions, which will have values passed to them at execution.

Those values are the csrf token, and the firearm-id, the two pieces of information I used to get from implicit rendering via my application layout, and from a Firearm instance passed to me by the controller.

How does the template know what those values will be? The magic of JavaScript, of course!

There's a button in firearmDrawer called "Add an Accessory". When that gets clicked, the following function fires:

```
$("#js-accessory-add").on("click", function(e) {
    e.preventDefault()
    let $container = $("#new-accessory-form")
    let firearmId = parseInt($(".js-next").attr("data-id"));
    let token = $('meta[name="csrf-token"]').attr('content');
    let context = {
      "firearm-id": firearmId,
      "csrf": token
    }
    let template = HandlebarsTemplates['accessories/new'](context);
    $container.html(template);
  });
```

So the template gets compiled inside firearmDrawer with the information it needs to submit a new accessory associated to a parent firearm. Neat! 

>This took me AGES to figure out.

Now, to actually submit the form, we need some more AJAX.

```
$("#new-accessory-form").on("submit", function(e) {
    e.preventDefault();
    let values = $("#new_accessory").serialize();
    let posting = $.post('/accessories', values);
    posting.done(function(accessory_response) {
      let $list = $("#accessoryList");
      //create an <li> for the accessory
      let li = document.createElement("li");
      let a = document.createElement("a");
      //set the required attributes for the accessory's <a> tag
      a.setAttribute('href', "javascript:void(0)");
      a.setAttribute("data-id", accessory_response.id);
      a.setAttribute("class", "js-accessory")
      a.innerHTML = accessory_response.name;
      $list.append(li);
      li.appendChild(a);
      $("#new-accessory-form").empty();
    });
  });
```

Let's break it down. First, serialize the values of the form inputs into JSON. Fire an AJAX POST request to "/accessories" with those values. When the controller finishes chatting with the models and database to create that new accessory, take the response (Which is the newly created accessory in JSON format) and inject it into the accessory list in firearmDrawer. Oh, and make the form disappear for good measure.

And that's it!!
# Step 4: Profit
<iframe src="https://giphy.com/embed/57UyiRANUopju1lnrQ" width="480" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

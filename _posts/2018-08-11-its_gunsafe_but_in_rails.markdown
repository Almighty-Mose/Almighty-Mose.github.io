---
layout: post
title:      "It's GunSafe, but in Rails!"
date:       2018-08-11 23:12:04 +0000
permalink:  its_gunsafe_but_in_rails
---


Wow, what a framework Rails is. Tough, complex, magical, frustrating... The road to this point was long and full of obstacles, but what an achievement to have built a full, functioning web application!  Many lessons were learned along the way, not only about coding in Rails, but about motivation, persistence, and dedication as well. Allow me to share some of the major things I learned during my journey to get GunSafe off the ground.

# Filtering Made Easy
At its heart, GunSafe is a Content Management System, and what's a CMS without a way to filter and sort its data? Firearms come in three major types, pistols, rifles, and shotguns; I wanted my users to be able to sort their firearms using this filter.  Achieving this goal was much easier than I initially expected, thanks to **scope**.

Scopes in Rails allow you to query your database tables for records that match a specific criteria. In my case, I used a scope to query my Firearms table by the category attribute, which looks like this:

```scope :category, -> (category) {where category: category}```

Defined in the Firearm class, this gives me access to a method .category(category), which I can use to pull all the firearm records whose category attribute matches the argument passed in.  This, coupled with a small form:

```
Filter Firearms:
<%= form_tag("/firearms", method: "get") do %>
  <%= select_tag :category, options_for_select(@current_user.firearms.pluck(:category).uniq), include_blank: true %>
  <%= submit_tag "Filter" %>
<% end %>
```

gives my users the ability to select a category based on the categories that already exist in the database (which also allows the filter form to dynamically grow as new categories are added).  Finally, a simple logic flow in the controller action will check for the presence of ```params[:category]``` and either display all firearms or the firearms which have a matching ```:category```.

```
if !params[:category].blank?
  @firearms = @current_user.firearms.category(params[:category])
else
  @firearms = @current_user.firearms
end
```

With that, GunSafe's users can now filter their firearms by category!
# Form Partials
Let's face it, I hate repeating code. That's why Ctrl+C is a thing, but even then, as a lazy ...ahem.... *efficient* programmer, I don't want to be repeating code *at all* if I can help it. Rails has a great way of handling this (because the nice Rails developers basically thought of everything) through a feature called partials. [Partials](https://guides.rubyonrails.org/layouts_and_rendering.html#using-partials) are small files that contain code which can be re-used throughout views, and can be accessed by using the render method, e.g., ``` <%= render 'partial' &> ```.

> Partials are small files that contain code you plan to re-use in views.

This mechanic is especally useful in the case of "new" and "edit" forms, which generally are very similar, if not outright identical in construction.  GunSafe uses form partials for most of its rendered forms. The form partial itself looks like this (using Rails' [form_for](https://api.rubyonrails.org/v5.1/classes/ActionView/Helpers/FormHelper.html#method-i-form_for) helper, which is also awesome)

```
<%= f.text_field(:make, placeholder: "Make") %>
<%= f.text_field(:model, placeholder: "Model") %>
<%= f.select :category, ['Pistol', 'Rifle', 'Shotgun', 'NFA', 'Other'] %>
<%= f.text_field(:caliber, placeholder: "Caliber") %>
<%= f.number_field(:price, placeholder: "Price") %>
<%= f.text_field(:serial_number, placeholder: "Serial Number") %>
<%= f.text_field(:purchase_date, placeholder: "Purchase Date MM/DD/YYYY") %>
<h3>Add an Accessory or Select From Existing Accessories</h3>
<%= f.collection_check_boxes :accessory_ids, Accessory.all, :id, :name %>
```

Notice there's no opening ```form_for``` tag or closing ```end```! Those are contained in the respective view, since the full form for a new firearm looks different than the form to edit a firearm.  This exposes another benefit of using partials, in that it frees up my views to only contain the unique code they require.  The new and edit views then look like this:

*New*

```
<%= form_for(@firearm) do |f| %>
  <%= render 'form', f: f %>
    <fieldset>
      <legend>New Accessory</legend>
        <%= f.fields_for :accessories do |a| %>
          <%= a.text_field(:name, placeholder: "Accessory Name") %>
          <%= a.number_field(:price, placeholder: "Accessory Price") %>
          <%= a.text_field(:purchase_date, placeholder: "Date Purchased - MM/DD/YYYY") %>
          <%= a.select :category, ['Optic', 'Sling', 'Light', 'Trigger', 'Other'] %>
        <% end %>
    </fieldset>
  <%= f.submit %>
<% end %>
```

(The extra form fields are a nested form allowing users to add an optional Accessory to the firearm they're creating)

*Edit*

```
<%= form_for(@firearm) do |f| %>
  <%= render 'form', f: f %>
  <%= f.submit %>
<% end %>
```

Inside both views is the line ```<%= render 'form', f: f %>``` wrapped in the opening and closing ```form_for``` tags I need to complete the form. Rails will go looking for a partial named ```_form.html.erb``` (partial names in Rails always start with an underscore), and injects whatever code is in the partial into the view. Voila! Now my form is defined once, meaning if I need to make a change later, it's as simple as editing the partial, and both views will reflect the change. Awesome!

# Authentication with Google

Let's face it, designing a secure login process is extremely difficult, time consuming, and fraught with peril.  Done improperly, it could expose user's data to malicious persons (which we never, ever, EVER want to happen).

>So let's leverage third party authentication and let someone else handle the hard work!

OmniAuth is a popular Ruby gem which allows me to do just that, and supports a variety of popular providers; Facebook, Google, Twitter, and Github being some examples.

After adding the ```omniauth``` and ``` omniauth-google-oauth2``` gems to GunSafe's Gemfile, I need to define a few new routes, Sessions Controller actions, and a User model class method.

```
# routes.rb

get 'auth/:provider/callback' => 'sessions#create_google'
get 'auth/failure' => redirect('/')

# In the Sessions Controller

def create_google
  user = User.from_omniauth(request.env["omniauth.auth"])
  session[:user_id] = user.id
  redirect_to firearms_path
end

# and the User class method

  def self.from_omniauth(auth)
    where(provider: auth.provider, uid: auth.uid).first_or_initialize.tap do |user|
      user.provider = auth.provider
      user.uid = auth.uid
      user.username = auth.info.name
      user.password = Sysrandom.base64(32)
      user.email = auth.info.email
      user.oauth_token = auth.credentials.token
      user.oauth_expires_at = Time.at(auth.credentials.expires_at)
      user.save!
    end
  end
```

One last thing, I needed to tell Rails what exactly to do with all this new code, so I created a new file in config/initializers called omniauth.rb, which contains this:

```
OmniAuth.config.logger = Rails.logger

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :google_oauth2, 
  'Client ID',
  'Client Secret',
  {client_options: {ssl: {ca_file: Rails.root.join("cacert.pem").to_s}}}
end
```

```Client ID``` and ```Client Secret``` are two very important pieces of information which Google uses to authenticate and process any requests coming from GunSafe. Obviously, I'm not actually showing what those are. Don't want to compromise my own app's security!

All this adds up to Google doing the heavy lifting of authentication and information security! Fantastic!

# Yer a Wizard, Rails

<iframe src="https://giphy.com/embed/7hnNiwfkYsPFC" width="480" height="245" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/hagrid-7hnNiwfkYsPFC">via GIPHY</a></p>

Rails is often described as "magical", and I feel like I definitely learned some spells while working through this project!


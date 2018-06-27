---
layout: post
title:      "GunSafe - A Sinatra Web App"
date:       2018-06-27 20:06:09 +0000
permalink:  gunsafe_-_a_sinatra_web_app
---


As a responsible gun owner, it's important to keep a record of all of your purchases in the event your firearms are ever lost or stolen. Memory is faulty, and paper can be lost, stolen, or destroyed, making these methods insufficient. 

Enter GunSafe.

GunSafe is a small web application allowing users to securely store a list of their firearms -- achieving this by signing up users and giving them forms to enter their firearm's information. Let's take a look.

**Users**

First, we have a users_controller, which facilitates the signing up and logging in of new and existing users. This is all fairly basic Sinatra routes, aside from one key feature important to GunSafe. Most responsible gun owners are fairly private, as such, I wanted each firearm list to be secure and private to each user. I have designed the app to only show a user their own information, and disallow them from viewing other users or their firearms, making each list private. I facilitated this by adding some checks to users_controller, and also to firearms_controller, which we will see in a moment.

```
#shows a User's homepage
  get '/users/:slug' do
    @user = User.find_by_slug(params[:slug])
    if logged_in? && current_user.id == @user.id
      erb :'users/show'
    elsif logged_in? && current_user != @user.id
      flash[:message] = "You Do Not Have Permission to View This User"
      redirect '/firearms'
    else
      flash[:message] = "Please Log In To Continue"
      redirect '/login'
    end
  end ```
	
> 	As you can see, the users_controller will only allow you to view your own profile page, not the pages of other users.

This is the key functionality of GunSafe - the ability to keep your sensitive information private. 

**Firearms**

We have users, and those users have firearms. As the purpose of GunSafe is to store firearm information, the firearms_controller is where most of the meat of the app lies. As I mentioned before, the firearms_controller prevents a user from viewing any firearms but their own. This is accomplished in two routes, ``` get '/firearms' ``` and ```get '/firearms/:id```.

*get '/firearms'*

```
# GET: /firearms
  get "/firearms" do
    if logged_in?
      @user = current_user
      if @user.firearms.count == 0
        redirect 'firearms/new'
      else
        @firearms = @user.firearms.all
        erb :"/firearms/index"
      end
    else
      flash[:message] = "Please Log In To Continue"
      redirect '/login'
    end
  end
	```
	
Here, the value of @firearms is always set to a list of *only* the current user's firearms, ensuring the associated view will only ever have a single user's Firearm objects to render in browser.

*get '/firearms/:id*

```
# GET: /firearms/5
  get "/firearms/:id" do
    @firearm = Firearm.find_by_id(params[:id])
    if logged_in? && @firearm.user_id == session[:user_id]
      erb :"/firearms/show"
    elsif logged_in? && @firearm.user_id != session[:user_id]
      flash[:message] = "You Do Not Have Permission to View This Firearm"
      redirect '/firearms'
    else
      flash[:message] = "Please Log In To Continue"
      redirect '/login'
    end
  end
	```
	
Likewise, if a user attempts to navigate directly to a firearm's listing that is not their own, the controller will prevent this by checking to see if the current user's :id matches the :user_id of the firearm being accessed. If not, the controller will redirect back to the current user's firearms list with the message "You Do Not Have Permission To View This Firearm."

> This ensures privacy for GunSafe users.

Creating this app gave me a solid grasp of Sinatra, RESTful routes, user authentication, and controlling user access. I look forward to moving on to Rails!

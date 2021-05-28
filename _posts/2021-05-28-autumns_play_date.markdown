---
layout: post
title:      "Autumn's Play Date"
date:       2021-05-28 23:40:01 +0000
permalink:  autumns_play_date
---

![](http://i.imgur.com/GqlrSOL.jpg)

My Sinatra app was a labor of love inspired by my daughter Autumn. I wanted to create an app that enabled parents to connect outside of Facebook's world of political posts and memes.

When my daughter was 3, she developed a habit of marching up to parents and demanding their phone number in hopes of setting up play dates with their kids. It was usually met with an awkward response. It rarely resulted in an exchange of phone numbers. When numbers were exchanged, they were stored in a wasteland of contacts with names like "Matthew's mom (park)" or "girl from library". Who? I told her about my idea for autumnsplaydate.com. She started yelling "autumnsplaydate.com!!!" at people instead of asking them for their phone numbers. I don't know how they felt about it but I thought it was super cute. Only issue was that it didn't exist. Until now! I don't know if anyone will use it, but seeing her pass out autumnsplaydate.com cards at her pre-k graduation made it so worth it!

I think the best way to go about explaining my app is through it's model's associations.  

# Associations:

**Parents & Children**


**Parent**

`has_many :relationships`

`has_many :children, through: :relationships`


**Child**

`has_many :relationships`

`has_many :parents, through: :relationships`


**Relationship**  

`belongs_to :child`

`belongs_to :parent`

A Parent and Child share a many to many relationship through a join table that is created by the Relationship model when a Parent signs up or adds a sibling to their account.

```
post '/parents/signup' do
    @name = "#{params[:first_name]} #{params[:last_name]}"
    @parent = Parent.create(name: @name, email: params[:email], password: params[:password])
    @child = Child.create(name: params[:child], age: params[:childs_age], gender: params[:gender])
    Relationship.create(parent_id: @parent.id, child_id: @child.id)
    session[:id] = @parent.id
    redirect to :"/parents/home"
  end
```

When a parent signs up, a `Parent` and `Child` is created using Active Record's `create` method that automatically saves each instance to the DB. A `Relationship` is also created which stores the `parent_id` and the `child_id` in a join table linking the Parent to it's children through their associations.



**Playdate and Participant**


**Playdate**

`has_many :relationships`

`'has_many :children, through: :participants`

`has_many :parents, through: :participants`



**Participant**

`has_many :participants`

`has_many :children, through: :participants`

`has_many :parents, through: :participants`



**Parent**

`has_many :participants`

`has_many :playdates, through: :participants`



**Child**

`has_many :participants`

`has_many :playdates, through: :participants`


When a play date is created, a Participant is also created and stores the `playdate_id`, `child_id` and `parent_id`.

I was able to iterate `@playdate.children` to show the user what children are participating in the play date.  

```
  <%@playdate.children.each do |child| %>
    <a href="/children/<%=child.slug%>"> <%=child.name%> </a></br>
  <%end%>
```


I noticed a pattern with each feature I added. The code was getting repetitious with slight variations. For example:

Play dates are usually well organized and planned ahead of time. I wanted a feature that would allow something a bit more impromptu. A list of locations that parents could check into.  Playgrounds, Chuck E Cheese etc...  

I created a Location model as well as a Checkin model (check in)


**Location**

`has_many :checkins`

`has_many :children, through: :checkins`


**Checkins**


`belongs_to :location`

`belongs_to :child`



```
post "/locations/checkin" do
  @parent = Parent.find_by_id(session[:id])
  @location = Location.find_by_id(params[:location_id])
  params[:children].each do |child|
    Checkin.create(location_id: params[:location_id], child_id: child[:child_id], parent_id: session[:id])
  end
  redirect to :"/locations/#{@location.id}/checked_in"
end

```

No surprises there. Similar concept and pattern as Playdate yet it was still a completely different feature for the user to use.



**Some other features I added to my app:**

* Comment section for /playdates/show so parents to interact with each other

* Jquery datetime picker for /playdates/new to make it easier to to save a datetime data type to the DB

* Bootstrap navbar and a custom CSS file for additional styles

* a second layout for my about page




**Some features I plan on adding:**

* At the moment, play dates are open to all users. I want to set up a private play date option where      people can invite specific members.

*  A best friends list that is only seen by the user as opposed to sending friend requests like facebook. It would be more like an address book to quickly invite the child's closest friends.

* E-mail notifications

* A message board for parents to communicate with each other

* Profile pics

* maps and photos for Check in locations

Please feel free to check it out at autumnsplaydate.com

Please don't hate me for the hot pink! I had a very demanding 5 year old boss looking over my shoulder when it was time to style it!

Here's some screen shots:

![](http://i.imgur.com/IAiJMIV.png/)

![](http://i.imgur.com/FGdWgbi.png)

![](http://i.imgur.com/Yrl7NT9.png)

![](http://i.imgur.com/3GUNrCx.png)

![](http://i.imgur.com/H1vodiL.png)

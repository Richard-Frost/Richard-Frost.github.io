---
layout: post
title:      "That Family App"
date:       2021-06-01 12:18:01 -0400
permalink:  that_family_app
---


<a href="https://imgur.com/19Uwbaq"><img src="https://i.imgur.com/19Uwbaq.jpg" title="source: imgur.com"style="max-width: 40%;" /></a>

For my Rails project, I attempted to create an app I wish I had over 9 years ago when my daughter was born.** That Family App** is a tool to help like-minded familes connect via a social network void of politics, drama and personal information. This app was inspired by each chapter of my family's adventures as we moved to various locations  throughout the country but most importantly, Brooklyn, NY. We couldn't walk more than 100 feet through Prospect Park without running into a family that would be happy to meet us. I tried to make this app look and feel like a walk through Prospect Park. I hope That Family App can one day bring those magical Brooklyn vibes to families far beyond the scope of city life.

<a href="https://imgur.com/sDfHkc6"><img src="https://i.imgur.com/sDfHkc6.jpg" title="source: imgur.com" style="max-width: 100%;" /></a>



Like family life, most things in this app revolves around the Family model.

<div style="background-color: #00c858;color:white;text-shadow: 1px 1px black;padding: 5px;">
**app/models/family.rb**
</div>

```
has_many :users
has_many :children
```

<div style="background-color: #00c858;color:white;text-shadow: 1px 1px black;padding: 5px;">
**app/models/user.rb**
</div>

```
belongs_to :family
has_many :children, through: :families
```

<div style="background-color: #00c858;color:white;text-shadow: 1px 1px black;padding: 5px;">
**app/models/child.rb**
</div>

```
belongs_to :famiy
has_many :users, :through family
```

<br>
The **User** model has a **title** attribute that is assigned when the user signs up. The user can be a parent, grandparent, aunt, uncle or a babysitter.


After the first User signs up, additional users can sign up and join a family via a family code that the original user creates. Once a user joins a family, they have access to all the features that the original user has.

If a child is with Granda for the afternoon, she can check the schedule to see if there are any play dates scheduled or find one if the child is bored and looking for something to do.





The next step was to enable families to find other famies. **FamilyConnection** is the join table that connects a **many to many** relationship between families through the use of an alias.  I created a **Request** model that acts like a friend request on other social media sites.

<div style="background-color: #00c858;color:white;text-shadow: 1px 1px black;padding: 5px;">
**app/models/family_connection.rb**
</div>

```
belongs_to :family
belongs_to :famconnect, class_name: "Family"
```

<div style="background-color: #00c858;color:white;text-shadow: 1px 1px black;padding: 5px;">
**app/models/family.rb**
</div>

```
has_many :requests
has_many :family_connections
has_many :famconnects, through: :family_connections
```

<div style="background-color: #00c858;color:white;text-shadow: 1px 1px black;padding: 5px;">
**app/models/request.rb**
</div>

```
belongs_to :family
```

When a user goes to the families index page, a **ruby gem** called <a href="https://rubygems.org/gems/geocoder/versions/1.3.7"> Geocoder </a> is used to find all families within a certain amount of miles from the user's zipcode.

As a user searches through the familes index page they see each family seperated into it's on div. Each **Child** is listed along with it's age attrubute. I also created a **Tag** model that a **Family** uses to display their* family interests*.  This is included on the familes index page as well so families can connect with like minded familes. I should mention that the process of a Family creating a Tag utilizes nested resources in routes.rb.

```
resources :families do
resources :tags
end
```

which provided me with the following routes and path helpers:

```
family_tags_path(@family.id)
new_family_tag_path(@family.id)
edit_family_tag_path(@family.id, tag.id)
```
&nbsp;

At this point, a family is created and have connected with other families. Next step is to create and join some** play dates**!  The Child and Playdate models are part of a many to many relationship. Participant is the join table.
&nbsp;


  <div style="background-color: #00c858;color:white;text-shadow: 1px 1px black;padding: 5px;">
**app/models/child.rb**
</div>


```
scope :i_am_bored, -> { where(bored: true) }

has_many :participants
has_many :playdates, through: :participants
```
&nbsp;


  <div style="background-color: #00c858; color:white;text-shadow: 1px 1px black;padding: 5px;">
     **app/models/playdate.rb**
  </div>

```
has_many :participants  
has_many :children, through: :participants
has_many :comments
```
&nbsp;


  <div style="background-color: #00c858;color:white;text-shadow: 1px 1px black;padding: 5px;">
     **app/models/participant.rb**
  </div>

```
belongs_to :child
belongs_to :playdate
```
<br>
<br>
There are 2 **class methods** on **Child** that reders `#playdate_today` and `#upcoming_playdates` on the user's home page.  

```
def playdate_today
self.playdates.where(datetime: Date.today.beginning_of_day..Date.today.end_of_day).order(datetime: :asc)
  end

  def upcoming_playdates
    self.playdates.where("datetime > ?", Date.today.end_of_day).order(datetime: :asc)
  end

```

I also included a scope method on the Playdate model. It works with the bored attribute which is a boolean reflecting a child's current state. It let's other parents know that you're child is bored and looking for a friend to play with. The scope method #

*app/models/child.rb*
```
scope :i_am_bored, -> { where(bored: true) }
```
*app/views/users/home.html.erb*
```
<% Child.i_am_bored.each do |child| %>
  <%= link_to child.first_name, child_path(child.id) %></br>
<%end%>
```

<figure>
<a href="https://imgur.com/vXv8iSe"><img src="https://i.imgur.com/vXv8iSe.jpg" title="source: imgur.com" style="max-width:70%;"/></a>
<figcaption>    The purple button sends out a bored alert and sets the User.bored to true</figcaption>
</figure>
<br>


<figure>
<a href="https://imgur.com/QTNcxal"><img src="https://i.imgur.com/QTNcxal.jpg" title="source: imgur.com" style="max-width:70%;" /></a>

<figcaption>   #i_am_bored helps display all children where child.bored == true</figcaption>
</figure>

My Sinatra project was an early version of this app. It was called Autumn's Playdate. The idea for the app was inspired by my daughter. When she was a little girl, she would march up to parents and ask them for their phone numbers. She was quite good at scheduling play dates. Often better than the parents. I wanted to create a tool to help them out. That Family App takes the concept a  bit closer to the functionality I want the app to have. I hope to keep building this app with the skills I learn in the upcoming modules.

---
layout: post
title:      "Sinatra CRUD Application build and learning more about Join tables"
date:       2020-10-01 20:07:18 +0000
permalink:  sinatra_crud_application_build_and_learning_more_about_join_tables
---


In finding inspiration for this project, admittedly I just pulled from what I've been watching most with my family lately. My family is outright OBSESSED with ghost adventures and the wacky stories from Zak Bagans, so I thought it might be fun and interesting to build an app where users can submit hauntings, track how many ghost sightings they've had, and compare the amount of sightings they've had with other users for some friendly competition. 

Admittedly, it was my idea that multiple users should be able to say they've been to the same spot that caused the most trouble for me during this project. For example, if one user submits the Ryman Auditorium as a haunted spot, then I believed another user should also be able to say they say a ghost at the Ryman Auditorium, without creating another haunting object for the Ryman, because then the views on this application could get repetitive fast and just be filled with many different users corroborating they went to the same place and saw the same ghost. I wanted to have one sighting, that could say "sighed by x amount of users."

Originaly, when I set up my project, I set up three classes. The City class, which held city objects with a name and state. The Ghost class, which was an object with a name and content that would basically serve as the haunting/sighting information. And of course, a User class with a name, email, and password. I originally used the ghosts table as my join table, the code as follows: 

```
class CreateGhosts < ActiveRecord::Migration
  def change
    create_table :ghosts do |t|
      t.string :name
      t.string :content
      t.integer :city_id
      t.integer :user_id

      t.timestamps null: false
    end
  end
end
```

City_id and user_id acted as my foreign keys, and the Ghost model class was set up with belongs_to relationships through Active Record.
```
class Ghost < ActiveRecord::Base
    belongs_to :user
    belongs_to :city
  end
  
```

This code worked in I was able to create ghosts/hauntings that happened in a city such as Nashville or Atlanta and a User object would be associated with the ghost so that user could edit or delete their ghost/haunting. However, this setup isn't conducive to my idea that a ghost should have multiple users that could claim it as sighted, I would need to make a separate join table and stop using my ghosts table as a join, I needed another table that joined the three classes, something I hadn't done before. After research and consulting, I found making a join table joining three classes was the same as making a join table joining two classes. it just required three foreign keys. After some experimentation, this is what I came up with for my join table:

```
class CreateUserGhosts < ActiveRecord::Migration
  def change 
    create_table :user_ghosts do |t|
      t.integer :user_id
      t.integer :ghost_id
      t.integer :city_id
      t.timestamps null: false
    end
  end
end

```

This would allow me to create a relationship in the Ghost model that the object has many users through user_ghosts. The ghost table I changed to be:

```
class CreateGhosts < ActiveRecord::Migration
  def change
    create_table :ghosts do |t|
      t.string :name
      t.string :content
      t.string :user_id

      t.timestamps null: false
    end
  end
end

```

I kept a foreign key for user_id in the ghosts table, so user and users could be separate in a ghost object. The user will be the creator of the ghost, what the ghost belongs to and capable of editing and deleting the ghost. The users a ghost has many through user_ghosts, are associated with the ghost but should not have the capability to edit or delete these ghost objects if they weren't the creator, if they just sighted them. 

```
class Ghost < ActiveRecord::Base
    has_many :user_ghosts
    has_many :users, through: :user_ghosts
    has_many :cities, through: :user_ghosts
    belongs_to :user
  end
  
```

My ghost model exemplifies this differentiation, and this difference is what drives the ability of my app to allow for many users to say they have sighted the same ghost object, without many users trying to change the name or content, which would become confusing fast. 

Overall, this project was a really enjoyable experience in learning more about how database tables work and how ActiveRecord relationships can lead to some really incredible functionality. I also learned not to overthink things too much, because making a join table between three classes seemed so daunting to me, and then I realized the difference was really just one more foreign key and wondered why I ever felt it was daunting to begin with. Each bit of code knowledge can be built upon, and if you know how to join two things, you know how to join more. If you know how to build one class, you can build more and so one. I learned not to get daunted by making things more complex, because I've really been given the tools and great building blocks to build up the more complex codes.

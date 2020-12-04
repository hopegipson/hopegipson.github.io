---
layout: post
title:      "Fantasy Football and Rails"
date:       2020-12-04 16:20:44 +0000
permalink:  fantasy_football_and_rails
---


To find inspiration for this project, I admittedly went with something I already consider to be one of my favorite hobbies. I'm a fantasy football enthusiast, and ever since I started this journey into coding, I couldn't help but constantly wonder, "How do these fantasy football websites work?" Obviously, at this stage of coding I can't create something that updates a player's score the second they score a touchdown like NFL and ESPN have the capability of doing, but I figured I could make something that could at least perform the basic fantasy football app abilities. The ability for a user to create a team, add players to that team, and play another team resulting in a  win or loss based on how many points those players score.

My first hurdle when I started to plan out this app was "How am I going to get players into my database?" The app needed players so the teams could add, drop, and use them in their lineups in matchups. I thought about manually entering a bunch of player information, but to be honest that just sounded like a horrible use of my time. So, I decided to use a lesson from our very first module, and use an API to get information that could be used in my application. My first instinct was to use exactly the same gems we used in the first module, but the lesson of Rails seems to me - if you're doing a ton of work, you're probably doing something wrong. It seems like Rails has an intuitive way to do everything, so I researched applications that played best with Rails and making HTTP calls to an API. I eventually settled on Faraday. 

To create my ability to search for a player, I added Faraday to my gemfile, and then created a search controller, as well as a route that would associate with the index of that search controller. I also created a corresponding view.
```
  get '/search' => 'search#index', :as => 'search'

```

In the view, I created a form intending for the user to be able to search for an NFL player, and the database would create a Player object based on that NFL player's real information from the database. I used a form_with and disabled remote submits.

```
<%= form_with(url: search_path, method: "get", local: true) do %>
  <%= label_tag(:player_name, "Enter Player name:") %>
  <%= text_field_tag(:player_name) %>
  <%= submit_tag("Search") %>
<% end %>
<% if @player %>
  <div>
    <%= link_to @player.name, player_path(@player) %> was found in this league database.
  </div>
<% end %>
```

When this form submits, it gives the index url parameters of the player_name. In my search controller index action, if player_name exists, then Player.find_or_create_from_api is called, which is a class method for the Player class I have created to first attempt to find the player in the database, and if it is not there, it creates one from the API. If the Player is not found, or cannot be created from the API due to no match in the API data, then a flash error is generated.
```
def index
    @players = Player.all
    player_name = params['player_name']
    if player_name
      @player = Player.find_or_create_from_api(player_name.to_s)
      if !@player
      flash[:errors] = ['Error finding your submitted player name, please try another']
      redirect_to search_path
      end
    end
  end
end
```

The next step in the chain is the Player.find_or_create_from_api(name) method 
```
def self.find_or_create_from_api(name)
        find_by_name(name) || create_from_api(name)
    end
```

This method itself is pretty standard from what a programmer would usually see in a find or create method, however, the create_from_api is not your standard create method.

```
def self.create_from_api(name)
        @response = SportsData::Search.by_player(name)
        if @response
            player = Player.new
            player.name = @response["Name"]
            player.nfl_team = @response["Team"]
            player.position = @response["Position"]
            player.projected_points = @response["FantasyPoints"]
            player.team_id = 1
            player.save
            player
        end
    end
```

In the first line, a module SportsData and a class Search within that module are invoked using a name parameter. The module, class and method I created in the services folder. This Module handles the communication with the API and sends a get request using the Faraday gem. This returns a Faraday::Response object, that can be parsed. I am also using the Dotenv gem to protect my API_KEY in this instance. Once the response object is found, JSON is used to parse through it and make a hash I can access with keys.
```
module SportsData
  class Search
    def self.by_player(player_name)
      @player_name = player_name
      response = Faraday.get 'https://api.sportsdata.io/v3/nfl/projections/json/PlayerSeasonProjectionStats/2020REG?key=' + ENV['API_KEY']
      response =JSON.parse(response.body)
      foundplayer = response.detect {|result| result["Name"].to_s.downcase == @player_name.downcase}
      foundplayer
    end
  end
end
```

After the player is found using this module, the hash of information is picked through and assigned to attributes of a Player object, then the object is saved to the database. 

Using this basic flow, I created a great deal of players for my application, without googling their attributes, but rather just searching for them and saving them to the database. Faraday is an incredibly versatile gem, and though I used a pretty simple get request from Faraday, Faraday can also handle post, put, and patch requests as well as create a Faraday::Connection object in which a common URL base path or HTTP headers are stored and applied to every request. As predicted, Rails did have an easier way than the former NET/HTTP of Module 1 to get information from an API and sent HTTP requests. And, I think using this information really helped take my fantasy football application from a basic battle between teams, to allowing me to create realistic player objects that belong to those teams with actual data that could be used throughout the application. For example, using the database to get the positions and NFL teams besides just the player names was very relevant in my application as I made it possible with scope to sort by position and by the NFL team. 

```
    scope :filter_by_position, -> (position1) { where(position: position1)}
    scope :filter_by_nfl_team, -> (nflteam) { where(nfl_team: nflteam)}

```
These are attributes and abilities that just wouldn't have been possible for me to use for my application, without getting this information for each player from a data source.

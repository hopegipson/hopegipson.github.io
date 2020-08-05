---
layout: post
title:      "Superhero Battle Simulator CLI"
date:       2020-08-05 04:40:08 +0000
permalink:  superhero_battle_simulator_cli
---


For this project, I delved into some of my admittedly "nerdy" interests, and I tried to create a pseudo command line interface game that a user could play. With "Injustice" in mind (an excellent Xbox game) I sought out to create a game where you can pick two heros, battle them, and the program will judge if one hero would win, lose, or if the two heroes would eventually just tie. 

I searched for API's that would give me not only facts about superheros, but stats, attributes of their strength and other common characteristics that might matter in a fighting arena. I found what I needed in https://www.superheroapi.com/ , an API which provides powerstats of superheroes including attributes like strength, combat, intelligence and other stats that might be relevant in a battle. With this API in mind, I set out to create my battle simulator.

I was aware three classes would be the crux of my application. A hero object, that would be created and initialized with the states that were retrieved from the API database. To do this, I would need an APIService class, to communicate and send requests to the API for information. Finally, I needed a CLI class that the user could interact with to be a part of the battles, a class where the user could pick a hero to battle as, pick another hero to fight, and the program would decide the victor.

I sketched out my main menu first, and came up with the idea that this superhero battle simulator would have a menu similar to this:

- Main Menu:
    - Choose a Superhero to Learn about
    - Will ask for hero input
        - Will print Superhero facts/data 
        - If hero input isn't valid prints couldn't find a character with that name
    - Pick a hero for Battle (input hero to save as user hero)
        - will ask for hero input
            - prints what hero you are - main menu
            - if hero isn'tt valid prints can't find hero with name
    - What is my current hero for battle? (learn about user hero)
        - prints hero facts/data user currently is
        - tells you if hero is unassigned to pick a hero
    - Battle a different hero
        - If you don't have a hero picked as user will tell you to do so
        -  What hero do you want to battle (name)?
            - You have chosen __ battle now?
                - Yes
                    - Win main menu
                    - Lose main menu
                - No
                    - "Coward" Main menu
                -  Invalid input
    - How many battles has each hero in battleground won/lost
        -prints list of heros and wins/losses
        -prints no heroes in battleground yet
				

After I had these basic ideas of how the CLI menu would progress, I set to building my CLI. The most difficult part of this process, was the actual creation of the battle. Ultimately when you select the option to battle a different hero, the first thing the program is going to need to do is check to see if you've selected a hero to play as. If you haven't, then you can't march nothing into battle and try to fight Superman. Originally, I used an if statement to check this, but I started to nest if statements within if statements and the code just wasn't that easy to follow or clean anymore. So, I separated the battle into several different methods, responsible for doing different actions of the battle.

First, I have a method that checks if the user hero is assigned. If it responds and there is no user hero assigned to the instance variable @userhero (which initializes as nil) then the program will responds with an error message.
	
```	nouserhero? || make_enemy_user ```


 I used boolean logic so if nouserhero? returns true, then the user is brought back to the main menu. If it returns false, the or logical operator will make_enemy user. This is another method I created to simplify the battle into components.
	
The make_enemy_user method will receive an input from the user with gets.chomp (as we've received every input in this program) and will first compare to see if the enemy user the user is inputting is the same as the stored user hero. If they are, you can't battle yourself. These are both effected by the downcase operation for sake of using the == to accurately compare them even if capitalizations differ. If the enemy the user has inputted is not the same as the user hero, then the program will find or create the hero by name using the find_or_create_by_name(name) class method of Hero. It will use another method I created that checks if the hero was actually created or if it wasn't found in the API. If that method returns true and no hero exists so it will puts out an error message. If that hero does exist and an enemy hero was created, the program will tell you what hero you're fighting.
``` (noheroexists?(hero) || (@enemyhero = hero)    ```

The next piece is back in the original battle method, where if the @enemyhero variable is initialized and no longer nil, the user will be asked if they're sure about fighting, yes or no. If @enemyhero is nil the user is taken back to the main menu. If the user selects no in this next menu, the program calls them a coward and sends them to the main menu. A reference to a video game I used to play that made me laugh when it called you a coward for not undertaking a mission. If you put in random text that isn't yes or no, it will puts out an error message. If you select yes, then the program will start battling the two heros in the testusers method.

The testusers method was a long one to create, and I picked one of two ways to do it I discovered while creating this program. First the method puts out a message that will appear supposedly while the heroes are fighting. I used the sleep method to suspend the thread for a little while this message appeared, to bulid suspense and make it feel like the heroes were battling real quick. Two counter variables are initialized in this method, @enemyheropoints and @userheropoints. As the method compares their attributes, I wanted to increment these point values for each time one character had a higher attribute than the other. Then, at the end of the program, it would decide the winner based on which point value was higher.  

When I first wrote the code to compare their variables, I wrote far too much unnecessary code. There are six attributes we'll be comparing, intelligence, strength, speed, durability, power, and combat.
My first code was this, for every single one of those six attributes.
```
def test_intelligence;
      if @enemyhero.intelligence.to_i > @userhero.intelligence.to_i;
              @enemyheropoints += 1;
      elsif @enemyhero.intelligence.to_i < @userhero.intelligence.to_i;
              @userheropoints += 1;
      end ;
end;
```

Six paragraphs of methods for each of the attributes and calling test_intelligence, test_strength, etc was too redundant. I did some research and some playing around in pry, and figured out there were two ways I could simplify my code. One way was by adding an array of attributes to be initialized in every hero instance, and this array would contain @strength, @speed, etc. Then, a code such as this could be implemented in the CLI class.
```
def test_attributes;
      @userhero.attributes.each_index do |i|;
      if @enemyhero.attributes[i].to_i > @userhero.attributes[i].to_i;
              @enemyheropoints += 1;
      elsif @enemyhero.attributes[i].to_i < @userhero.attributes[i].to_i;
              @userheropoints += 1;
              end ;
      end;
end;
```

Iterating through the attribute arrays for the Hero instance of the user hero and the instance of the enemy hero and comparing each every time. Then, in the testusers method, test_attributes would only need to be called. I ended up doing it an alternate way though, because I wanted to practice metaprogramming, and using the .send method. Using .send, you can call methods by using symbols as the arguments. Passing a symbol into the .send method is the same as calling that method you're passing into send. 

```
def test_attribute(attribute)   ;
       if @enemyhero.send(attribute).to_i > @userhero.send(attribute).to_i;
              @enemyheropoints += 1;
       elsif @enemyhero.send(attribute).to_i < @userhero.send(attribute).to_i;
              @userheropoints += 1;
       end ;
end;
```

This method I chose to use, asks for the argument of a symbol, and it will call the method based on that symbol and compare them. So essentially doing the same thing I did in the test_intelligence method, but you don't have to write 6 different methods, you can just call this method test_attribute(strength) and test_attribute(speed). The array of attributes and iterating through that array is less code, but I just like this method because it gave me a better understanding of metaprogramming and using .send. 

Finally, after points have been incremented and attributes have been compared, testusers uses an if statement to puts out a message based on if the user hero won, lost or tied. 

Overall, this was such a fun experience, and learning different ways of how to do something and how to make code easier to read and simplify code, is a really rewarding process. It's so fun making clunky long working code, breaking that code, making more concise better code that works, breaking that again, and making the best code. It just shows the old addage there's lots of ways to do one thing, and really exercises you mind in learning the best way to do something.



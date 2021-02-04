---
layout: post
title:      "Breaking Down the Magic of A Hogwarts Quiz"
date:       2021-02-04 09:51:30 +0000
permalink:  breaking_down_the_magic_of_a_hogwarts_quiz
---

Every year, my friends and I have a Harry Potter themed Halloween party.  At this party, we all take a house test and get sorted into one of four Hogwarts Houses: Gryffindor, Slytherin, Ravenclaw, and Hufflepuff. Then, we play board games all night. Each one of the games allows a chance for house members to score points for their house, and at the end of the night, one of the four houses takes home the house cup. It gets quite competitive if I'm being honest, and when I was brainstorming an idea for a Javascript app I thought: "Hey, why don't you make something you and your friends can actually use?" We can't get together for any Harry Potter parties during COVID because we are all cognizant of safety, so I thought I could replicate the magic of sorting into houses and playing games to score points for a house online. Thus, the Hogwarts Sorting Cup App was born.

The most important component of the app is the quiz class, so that is what I will be primarily discussing in this post. Understanding the quiz class is the crux of how this applet works. First of all, there are two buttons appended to the DOM that have event listeners that serve to create instances of the quiz class. 

```
renderQuiz(quizid){  
      api.getQuiz(quizid).then(function(quiz){new Quiz(quiz)})
      }   
```
 A fetch request to the API using an instance of the APIAdapter class we defined in index.js as a const allows us to access the JSON data rendered by the serializer and create an instance of our Quiz class using that data. The quiz instance is created with the constructor, and in the constructor `this.pickTenQuestions(quiz)` is called. Because this was used, making the context of the function the instance of the Quiz class that this was referring to rather than a global variable, inside the function the variables of the Quiz class instance are accessible. 
 
 `pickTenQuestions = (quiz) => {
      this.selectedQuestions = []
      for (let i = 0; i < 10; i++) {
        let removedItem = quiz.questions.splice(this.getRandomIndex(quiz.questions), 1);
        this.selectedQuestions.push(removedItem[0])
      }
    }`
		
	The purpose of this method is to take the quiz element passed into the Quiz constructor and now into the function and pick ten questions that will be displayed in this quiz. In the seed data, I have around 30 questions prepared for Harry Potter trivia, and 20 questions for the Sorting Hat, but to keep users guessing, I wanted to try to randomize which questions would be shown each time so it wasn't the same quiz over and over again. Which is why, I wanted to select ten questions and make them an attribute of the Quiz instance so that I could use just ten of them in the display and results for this instance of teh Quiz class. Originally, when trying to make this method I was just using `let selected = quiz.questions[Math.floor(Math.random() * quiz.questions.length)];` to generate a random quiz item and adding that to the selectedQuestions array. I ran into some problems with that, because by using that method of randomization, there was a possibility of picking the same element twice, which wouldn't work in my quiz as when I created an instance of the Question class with that same question, the two instances would be identical in id and other attributes which caused a host of problems with my radio buttons. The solution, to remove the random element from the array every time I selected one so I wouldn't get it twice, hence the use of splice.
	
	After I had my ten questions retrieved, the createQuestions method is the next stop, in which I created instances of the Question class for every question I had selected and appended them to slides, so I could have the ability of showing one slide at a time rather than showing a quiz of many slides all at once.
	`createQuestions = () => {
      console.log(this.selectedQuestions)

        this.selectedQuestions.forEach((currentQuestion) => {
            const question1 = new Question(currentQuestion)
           this.slides.push(question1.slide)
           this.questions.push(question1)
           this.quizElement.appendChild(question1.slide)
        })
    }`
		
I created buttons with a createButtons function, the previous button with the function of showing the previous slide upon click, the next button with the function of showing the next slide upon click, and the submit button that would call my resultQuiz function. 

 `resultQuiz = () => {
      this.submitBtn.disabled = true
        if (this.id === 1){
            this.renderResults()
        }
        else if (this.id === 2){
            this.renderHouseResults()
        }
    }
`


The resultQuiz function  is unique in it first disables the submitBtn so it can't continually be pressed and the user can't change their answers without restarting the quiz. Then, it directs the submission to one of two functions, based on the id of the quiz. The Trivia Quiz has the ID of 1 and it is going to be scored differently than the house quiz, scored on a basis of "did you get the correct answer or not?" The other button directs to a scoring system that is more complex, and assigns points to the variables :

` this.gryffindorCount = 0  
this.slytherinCount = 0 
this.hufflepuffCount = 0;
  this.ravenclawCount = 0;`
	
	A function then determines how many answers you selected that have to do with each House, calculates a points total, and wherever you have the most House points, is now your assigned House.
	
	`calculateHouseResults = () =>{
     let largestNumber = Math.max(this.gryffindorCount, this.slytherinCount, this.hufflepuffCount, this.ravenclawCount)
      let house = []
      console.log(largestNumber)
      if (this.gryffindorCount === largestNumber){ house.push("Gryffindor")}
      if (this.slytherinCount === largestNumber){ house.push("Slytherin")}
      if (this.ravenclawCount === largestNumber){ house.push("Ravenclaw")}
      if (this.hufflepuffCount === largestNumber){ house.push("Hufflepuff")}
      if (house.length === 1){
        return house[0]
      }
      else {
        return house[Math.floor(Math.random()*house.length)]
      }
      }`
			
Because finding the results of the House test is not as cut and dry as "did you get the right answer or not" it is possible to tie and have multiple houses with the same point values. Say you got 5 points of Ravenclaw, and 5 points in Slytherin. Which house are you? This function knows you can't be both, so if you have two houses or more in your array house[] which is collecting the names of the House or Houses you have the most points in, it will pick one at Random - a true sorting hat! It does this using the `house[Math.floor(Math.random()*house.length)]`. 

Render results for the trivia quiz works in that it just calculates how many questions you got right - but unlike the renderHouseResults function, the renderResults function is going to create a new instance of the Score class. This is because though the renderHouseResults is going to send a patch request and change the house of a user, this function needs to create a score and append that to the database so it can be referenced in leaderboards and calculating house points for the house cup. 

From renderResults:
```
 api.postScore(this.numberCorrect, game.user.id).then(function(scoreObject){
      api.getUser(game.user.id).then(function(userObject){
        let userUpdated = new User(userObject)
        game.setUser(userUpdated)
        LoginDisplay.resetForm()
        LoginDisplay.login(game.user)
        LeaderboardDisplay.createLDisplay()
```

If the user saved in the Game instance is not unsorted (the instance of the Game class initialized in index.js and the user assigned to that instance is how we keep track of our current user throughout), then we are going to send a request to the API to create a Score. We only need to supply the score with the number_correct and the user_id not the house_points to be saved in the score, because that calculation is actually done in the backend. In the Score model, there is an instance method that will calculate and assign a house_points value to the instance based on the number_correct.

```
 def check_score_for_house_points
    if self.number_correct == 10 
      self.house_points += 10
    elsif self.number_correct > 8 && self.number_correct < 10
      self.house_points += 5
    else
      self.house_points += 0
    end
  end
```

After we have our Score posted, our User object we want to reference has changed because it has a new Score according to the has_many Scores relationship. For that reason we send a fetch call to get our updated User, and set the Game instance attribute of User to the new User. Then we reset the LoginDisplay using a static function and then login using another static function passing the user to that function. We also create another Leaderboard because the Leaderboard will update if the Scores have been updated.

After we finish a House Sorting Quiz, if we have a user logged in that can now have a new House assigned to that User we do a similar workflow:

This is called in renderHouseResults:
```
addHouseToUser = (user, house) => {
      game.user.removeScores()
      
      api.patchUserHouse(user, house)
      .then(function(userObject){
       let userUpdated = new User(userObject)
        game.setUser(userUpdated)
        LoginDisplay.resetForm()
        LoginDisplay.login(game.user)
        LeaderboardDisplay.createLDisplay()
       let results = document.getElementById('results')
        results.innerHTML = `${userObject.house.name} is your new House! ${userObject.house.house_information}`;
    })
    }
```

First we remove the scores the user already has by calling a method that will send a 'DELETE' request to our API. We do this because if our user is switching houses, they shouldn't have some points earned for one house, and some for another. You can only earn points for one house at a time or the leaderboard would get awful messy if one user was listed for scoring 5 points for Gryffindor and 5 points for Slytherin. Then, we send a 'PATCH' request to patch our User and change the House that user is associated with. Then, alike to the renderResults, we take that updated user and set it to our game, reset the login display, and the leaderboard. 

Overall, this application is built upon the core Quiz class, and it provides functionality for Users in two quizzes that will update the DOM. It may not be the experience of playing board games with your friends in the name of earning points for your Harry Potter House, but it comes as close as we can virtually at the moment and I hope this application brings you some fun as it will bring to my friends!




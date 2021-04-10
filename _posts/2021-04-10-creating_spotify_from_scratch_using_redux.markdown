---
layout: post
title:      "Creating Spotify from scratch using Redux"
date:       2021-04-10 05:17:28 +0000
permalink:  creating_spotify_from_scratch_using_redux
---


As programmers, one of our core instincts is the desire to actually create our favorite websites, to know how they were created and build them ourselves - demystifying these programs we use every single day. Personally, the website I spend the most time on is Spotify. As an audio engineer and programmer, I'm constantly using the site to listen to client music, creating playlists that inspire my work, and in general enjoying music as I go about my daily activities. When I first began this journey, I knew I wanted to learn programming to add to my skills that I utililize in the music industry, and it was a perfect final project to finally mesh my love of the music industry and my love of programming. Spotify is a complex website, so I challenged myself to make a similar program using Redux. Though it isn't an exact duplicate, I emulated a lot of the core Spotify features and created a very user friendly interface. I got a little excited working on this so the amount of code that went into this is far too much to explain in a singular blog post, so I'd like to talk about the feature that I think most developers have had issues with when they've tried to create a Spotify application: actually playing music from the application.

A huge resource for me in implementing song playback technology was the information found in https://developer.spotify.com/documentation. I highly encourage any developer who is going to be playing music in their application to read through the documentation on both the web API and SDK which are required to play music from an application. I chose to use the Spotify Player and their API in feeding tracks to play to the Player so I could play any song in my application, not just songs stored on my local machine, I wanted my users to be able to access any song they wanted and play it right in the React application.

The first step to using the Spotify SDK or webplayer is Authentication. Each individual user needs to have a token from Spotify that allows them to access the SDK and the Spotify API. Instead of instructing my users to go get a token and create an environmental variable with it, I decided to make a login page using an authentication endpoint through Spotify. That way, a user could login using their Spotify credentials and not have to go chase down an authentication token on the website, and the token will be stored as the user utilizes the application.

If there is not a valid token stored in the Redux store when the application renders, then it will render the component SpotifyAuthButton.  The Spotify Auth Button when clicked will render this URI:

```
const hrefURI = `${authEndpoint}?client_id=${clientId}&redirect_uri=${redirectUri}&scope=${scopes.join(
            "%20"
          )}&response_type=token&show_dialog=true`
```

In my config.js file and SpotifyAuthButton file, I have configured an authEndpoint with a value of "https://accounts.spotify.com/authorize" as well as some personalized information that can be configured in the Spotify developer portal that entails the clientId, clientSecret, redirectURI, and scopes of the information I sought to use from Spotify in this application. Further reading on the scopes can be found here: https://developer.spotify.com/documentation/general/guides/scopes/.  This comprises of the proper authorization endpoint where a user can login to their Spotify account and give my app permission to access information which generates a token. The token is immediately saved from the window location where Spotify generates the token and is then stored in the Redux state so the token can be utilized in calls to the Spotify API and in configuring the web player.

```
componentDidMount() {
    const hash = window.location.hash
    .substring(1)
    .split("&")
    .reduce(function(initial, item) {
      if (item) {
        var parts = item.split("=");
        initial[parts[0]] = decodeURIComponent(parts[1]);
      }
      return initial;
    }, {});    
    let foundToken = hash.access_token;
    if (foundToken) {
      this.props.setToken(foundToken)
      
			
			
  }}
	
	const mapDispatchToProps = dispatch => {
  return {
    setToken: tokenText => dispatch({type: 'SET_TOKEN', payload: tokenText })
  }
}
	
	
```

After the app finds a valid token in the state, the dashboard is rendered. The Footer container contains our MusicPlayerContainer component, where all the magic that goes into actually playing the music is stored. The first step is loading the Script from the Spotify Player resource. In the documentation about the SDK, Spotify gave this example for loading the Player resource into your project.

```
<!DOCTYPE html>
<html>
<head>
  <title>Spotify Web Playback SDK Quick Start Tutorial</title>
</head>
<body>
  <h1>Spotify Web Playback SDK Quick Start Tutorial</h1>
  <h2>Open your console log: <code>View > Developer > JavaScript Console</code></h2>

  <script src="https://sdk.scdn.co/spotify-player.js"></script>
  <!-- We will insert our code here. -->
</body>
</html>
```

This works great for certain kinds of applications, but this introduced problems in a Redux application where I wanted to load the Script source and then use that script and manipulate it in my application. My solution was to create a function where I would load the Spotify Web Player script, append it to the body of the application and then call a callback function that would serve to create a Player object I could store in the Redux store and access through my application.  The function I used to do this is as follows:

```
export const loadSpotifyScript = (callback) => {
          const existingScript = document.getElementById('spotify');
          if (!existingScript) {
            const script = document.createElement('script');
            script.src = 'https://sdk.scdn.co/spotify-player.js';
            script.id = 'spotify';
            document.body.appendChild(script);
            script.onload = () => { 
              if (callback) callback();
            };
          }
          if (existingScript && callback) callback();
        };
```

When the MusicPlayerContainer component mounts, it calls this application as so, feeding it the callback function of this.spotifySDKcallback:
```
componentDidMount() {
        loadSpotifyScript(this.spotifySDKCallback)
            }
```

In the Spotify SDK documentation, it then described creating a Player object using code such as this: 

```
window.onSpotifyWebPlaybackSDKReady = () => {
  const token = '[My Spotify Web API access token]';
  const player = new Spotify.Player({
    name: 'Web Playback SDK Quick Start Player',
    getOAuthToken: cb => { cb(token); }
  });


```

My code is as follows:

```
spotifySDKCallback = () => {
             window.onSpotifyWebPlaybackSDKReady = () => {

             let { Player } = window.Spotify;        
                const spotifyPlayer = new Player({
                    name: 'React Spotify Player',
                    getOAuthToken: cb => {
                        cb(this.props.state..token);
                    }
                });
                spotifyPlayer.addListener('player_state_changed', ({
                    position,
                    duration,
                    track_window: { current_track }
                  }) => {
                    console.log('Currently Playing', current_track);
                    console.log('Position in Song', position);
                    console.log('Duration of Song', duration);
                  });
                this.setState({
                    loadingState: "Loaded",
                    spotifyPlayer
                }, () => {
                    this.connectToPlayer();
                });
            }
         }
```

Due to the fact this component is connected to the Redux store, I used the stored token and fed it to the Player creation function. I also added some event listeners to help me later in playback tracking, and finally called my function connectToPlayer. In connectToPlayer, I operated on my stored Player instance and sought the deviceID from the Player instance. I did not give it a specific device to connect to, allowing the Player to link to the default playback device in the web browser for each individual user. 

```
connectToPlayer = () => {
            if (this.state.spotifyPlayer) {
                this.state.spotifyPlayer.addListener('ready', ({device_id}) => {
                    console.log('Ready with Device ID', device_id);
                    this.setState({
                        loadingState: "Player Ready",
                        spotifyDeviceId: device_id,
                        spotifyPlayerReady: true
                    }, () => {
                      this.notifyConnected()
                          
                    });
                });
                this.state.spotifyPlayer.connect().then(success => {
                    if (success) {
                      console.log('The Web Playback SDK successfully connected to Spotify!');
                    }})
             }
        }
```

Using a method the Player had in it's documentation I connected after the Device was ready, and then we are officially set up to begin Playback. The Player and DeviceID are also stored in the Redux store: 

```
addPlayer: (player) => dispatch(addPlayer(player)),
    addDeviceID: (deviceid) => dispatch(addDevice(deviceid))
```

Now we have a device ready to play some songs - an instance of the Spotify SDK web player. However, the webplayer alone does not really allow you to do anything but play from a webpage instead of the Spotify webpage and only plays back what you're already playing on Spotify's website alone, just from itself in a different page. I wanted to make it so you could select any song and use the Player to listen to that song, so I also utilized the Spotify API. I utilize Playback Functionality in a few different spots in my application, but we'll go over the most basic one . If you use the search page to look up beyonce, you'll get a list of a few of her songs. You can click on one of these songs and it plays back in the Application. This is done through the Song Result component that houses the child components of Songs. The SongResult component is passing down as a prop to each song the callPlayback function, which is then called on click of the play button on each song. I put the method for the callPlayback in the Parent component of all the Song components because I wanted to manage the Play/Pause images for each of the song objects as a whole - so not all of them would be marked at play if someone clicked all the buttons (only one at a time) and I wanted to manage the requests sent to the Player in a way where only one would be sent at a time.

```
  callPlayback = (event) => {

       if(!this.props.state.playbackOn){
        let selectedElement = this.props.songs.splice(event.target.id, 1)[0]

        startPlayback(event.target.name, this.props.state.deviceID, this.props.state.token).then(this.changeStatesPlay(selectedElement))
          this.props.songs.forEach(function (song) {
            song.open = false;
          })
        selectedElement.open = true;
       this.props.songs.splice(event.target.id, 0, selectedElement)   
       this.setState({songs: this.props.songs, selectedElement: selectedElement})
        }
        else if(!this.props.state.playbackPaused){
           pauseTrack(this.props.state.deviceID, this.props.state.token).then(this.changeStatesPause())
           this.props.songs.forEach(function (song) {
            song.open = false;
          })
       this.setState({songs: this.props.songs, selectedElement: "empty"})  
      }
   }
```

The callPlayback function is not only responsible for sending a request to startPlayback, but it also adjusts the states of the songs - if open is false it will show a play picture, if open is true then it will show a pause picture to give a visual representation of which song is playing one at a time. The startPlayback function is in my actions folder and it serves to send a request to play a song using our already loading Player that is linked to our deviceID. Each song is rendered with the uri stored as such:

```
name={this.props.song.uri} 
```

This allows event.target.name to send the individual uri of the track to the Spotify API in our request. Start playback is a fetch request, as is PauseTrack. 

```
  export const startPlayback = (spotify_uri, deviceID, token) => {
    return fetch(STARTPLAYBACKURL +
          "device_id=" + deviceID, {
          method: 'PUT',
          body: JSON.stringify({uris: [spotify_uri]}),
          headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${token}`
          }})}
					
					export const pauseTrack = ( deviceID, token) => {
    return fetch(PAUSEURL +
          "device_id=" + deviceID, {
          method: 'PUT',
          headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${token}`
          }})}
```
These two requests using the deviceID and token stored in the Redux state allow the Player to start playing any song rendered, as songs will always be rendered with their track uri which is what the API requires to load the playback information of the song. In addition, two dispatches are also used to send currentlyPlaying information to the songTracker component which provides a visual representation of how far along you are in the song within the Footer.

```
changeTrackerSong: (song) => dispatch(changeTrackerSong(song)),
   eraseTrackerSong: () => dispatch(eraseTrackerSong())  
```


This Song Tracker works using this method to get the progress and duration from the song that is currently playing:

```
getCurrentlyPlayingS = (token) => {
          getCurrentlyPlaying(token).then((data) => {
                    this.setState({
                        data: data,
                      item: data.item,
                      duration_ms: data.item.duration_ms,
                      is_playing: data.is_playing,
                      width: ((data.progress_ms * 100 / data.item.duration_ms)*6),
                      progress_ms: data.progress_ms
                    })
          
            }
          )
					
					componentDidMount() {
       this.interval = setInterval(() => this.tick(), 1000);
      }

      tick = () => {
         if(this.props.state.playbackOn === true){
        this.getCurrentlyPlayingS(this.props.state.token);  
         }
         
      }
```

Every second tick is called, and tick get the information of the currently playing song again including where exactly the user is in the progress of listening. This value of how far along the user is in listening to the song is then added to the state, where that value is used to calculate the width of a progress bar in pixels in relation to the entirety of the song to give a visual representation of how far along the user is in listening to the song.

```
 const progressBarStyles =  {
            width: (this.state.width) + 'px'
            
          };

          const progressBarStyles2 = {
            width: 600 + 'px'
          };
					
					   <div className="progress__bar2" style={progressBarStyles2} />
            <div className="progress__bar" style={progressBarStyles} />
```


All of this combined allows for playback, as well as some visual representation of that playback. By far, figuring out how to adapt the documentation for the SDK and the Spotify API to be easily workable in the React Redux format was the toughest part of the project, but the toughest part always yields the best part - and by far I think the best part of this project, is the ability to listen to tunes as you interact with it.


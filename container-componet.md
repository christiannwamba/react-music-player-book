# Container Component

Previously, we discussed React's presentation components with few examples backing it up. This is the last section of the "Build a Music Player with React & Electron" series and we will discuss container components while completing our Scotch Player Electron app.


## Revisit Our `app` Directory Structure

Just as we did in the previous section, let us have another look at the directory structure:

```bash
|---app #All React projects goes here
|----components # Presentation Component Directory
|------details.component.js # Completed
|------footer.component.js # Completed
|------player.component.js # Completed
|------progress.component.js # Completed
|------search.component.js # Completed
|----containers # Container Component Directory
|------app.container.js # WHERE WE ARE
|----app.js # SOME CHANGES  WILL GO HERE
```

The comments will give you an idea of where we are headed and what we have achieved (if you stumbled on this article first). Our focus is to flesh out the `app/containers/app.container.js` and make some changes to `app/app.js`. Let's begin!

## Our Container Component

Just as a recap, this is the type of component that handles the dirty jobs which we abstracted from presentation component. These jobs include:

- state mutation
- event handling
- composing different presentation components together

For easy understanding, we will talk about each UI component as we import them one after the other into the container component. We also define the states as we meet them in the short journey. Below is a skeleton of `app.container.js`:

```javascript
// React library
import React from 'react';

// Axios of Ajax
import Axios from 'axios';

// AppContainer class
class AppContainer extends React.Component {
	// AppContainer constructor
	constructor(props) {
	super(props);
	}
	 
	// componentDidMount lifecycle method. Called once a component is loaded
	componentDidMount() {
		this.randomTrack();
	}
	  
	// Render method
	render () {
		return (
		<div className="scotch_music">

		</div>
		);
	}
}

// Export AppContainer Component
export default AppContainer
```

>__Life cycle methods__
> For every given time in a component's life cycle, a given method is provided to handle certain tasks. Starting from before the component even loads to when it is completely loaded and [some others](https://facebook.github.io/react/docs/component-specs.html). One life cycle method is `componentDidMount()` and is called after it is loaded 

When the component is fully loaded we want to pick a track from Soundcloud to play. `componentDidMoint` is already calling a member method which returns a random track using `Ajax`. We can make `Ajax` request with [Axios](https://github.com/mzabriskie/axios):

```javascript
randomTrack () {
	let _this = this;
	
	//Request for a playlist via Soundcloud using a client id
	Axios.get(`https://api.soundcloud.com/playlists/209262931?client_id=${this.client_id}`)
		.then(function (response) {
			// Store the length of the tracks
			const trackLength = response.data.tracks.length;

			// Pick a random number
			const randomNumber = Math.floor((Math.random() * trackLength) + 1);

			// Set the track state with a random track from the playlist
			_this.setState({track: response.data.tracks[randomNumber]});
		})
		.catch(function (err) {
			// If something goes wrong, let us know
			console.log(err);
		});
}
```

**What have we done?** Request for a particular playlist on Soundcloud, use a random number to pick a track from the returned array, set the track to the `track` state. The `track` state initial state needs to be defined:

```javascript
// ...
constructor(props) {
	super(props);
	
	// Client ID
	this.client_id = 'YOUR_CLIENT_ID';
	
	// Initial State
	this.state = {
		// What ever is returned, we just need these 3 values
		track: {stream_url: '', title: '', artwork_url: ''}
	}
}
//...
```

## React Sound

[React Sound](https://github.com/leoasis/react-sound) is a component that wraps around [Sound Manager 2](http://www.schillmania.com/projects/soundmanager2/) and expose common sound APIs like

- play
- pause
- stop
- etc

We already installed it using `package.json`, so let's start with the import:

```javascript
// React library
import React from 'react';

// Axios of Ajax
import Axios from 'axios';

// Sound component
import Sound from 'react-sound';

// ...
```

... then include the component in render's return like so:

```javascript
//...
render () {
	return (
		<div className="scotch_music">
		  <Sound
		   url={this.prepareUrl(this.state.track.stream_url)}
		   playStatus={this.state.playStatus}
		   onPlaying={this.handleSongPlaying.bind(this)}
		   playFromPosition={this.state.playFromPosition}
		   onFinishedPlaying={this.handleSongFinished.bind(this)}/>
		</div>
	);
}
//...
```

From just adding that, you have a top level overview of the properties that we need to supply `Sound`. We already started by supplying undefined values and callbacks. Let us take them one after the other:

__`url`__: This is the stream URL of the sound. We are using Soundcloud stream URLs in this case which requires a client id else you will get a `401`. We will use a utility function (`prepareUrl`) to attach the client id to the URL:

```javascript

//...

// A method in the AppContainer class
prepareUrl(url) {
	// Attach client id to stream url
	return `${url}?client_id=${this.client_id}`
}
  
//...

```

... then define the client id in the class constructor:

```javascript
// ...

constructor(props) {
	super(props);
	this.client_id = 'YOUR_CLIENT_ID';
}

// ...
```

The URL is also gotten from a `track` state which we already defined.

__`playStatus`__ has the value `this.state.playStatus` therefore will have an initial state too:

```javascript
//...
constructor(props) {
     this.state = {
       // What ever is returned, we just need these 3 values
       track: {stream_url: '', title: '', artwork_url: ''},
       playStatus: Sound.status.STOPPED,
     }
}
//...
```

React Sound has 3 play statuses:

- `Sound.status.STOPPED`
- `Sound.status.PAUSED`
- `Sound.status.PLAYING`

We want the sound player to stay stopped even when the URL is available so the user can manually play it. That is why the initial state is `Sound.status.STOPPED`. Keep in mind that this is the state that we change when play or pause is clicked.

__`onPlaying`__:  This one does not require a value but a method to be called while the track is playing. This method is handy when we need to update elapsed time of the sound. The value is `this.handleSongPlaying.bind(this)` which is a `AppContainer` member method:

```javascript
handleSongPlaying(audio) {
	this.setState({  elapsed: this.formatMilliseconds(audio.position),
		total: this.formatMilliseconds(audio.duration),
		position: audio.position / audio.duration })
}
```

It receives the current status of the audio instance which gives us access to the audio properties like `position` and `duration` 

We are also updating 3 more states here which needs to be set in the initial state:

```javascript
// ...

constructor(props) {
	this.state = {
		// What ever is returned, we just need these 3 values
		track: {stream_url: '', title: '', artwork_url: ''},
		playStatus: Sound.status.STOPPED,
		elapsed: '00:00',
		total: '00:00',
		position: 0
	}
}

// ...
```

- `elapsed` state is set with the current position of the audio.
- `total` state is set with the estimated total play time of the audio.
- `position` state is used for the progress bar (yet to be seen)

These 3 states are handy for the `Progress` component which we will use later.

We are using another utility method `formatMilliseconds` to format time:

```javascript
formatMilliseconds(milliseconds) {
	// Format hours
	var hours = Math.floor(milliseconds / 3600000);
	milliseconds = milliseconds % 3600000;
	
	// Format minutes
	var minutes = Math.floor(milliseconds / 60000);
	milliseconds = milliseconds % 60000;
	
	// Format seconds
	var seconds = Math.floor(milliseconds / 1000);
	milliseconds = Math.floor(milliseconds % 1000);
	
	// Return as string
	return (minutes < 10 ? '0' : '') + minutes + ':' +
	(seconds < 10 ? '0' : '') + seconds;
}
```

__`playFromPosition`__:  We would always have a reason to update the play position. Reasons like rewind (backward) or fast-forward. Takes another state which will be updated when the rewind or forwad buttons are clicked. Let us include in our initial state definition:

```javascript
//...
constructor(props) {
	this.state = {
		// What ever is returned, we just need these 3 values
		track: {stream_url: '', title: '', artwork_url: ''},
		playStatus: Sound.status.STOPPED,
		elapsed: '00:00',
		total: '00:00',
		position: 0,
		playFromPosition: 0
	}
}
//...
```

We just initialize it with the value of 0.

__`onFinishedPlaying`__: This receives a callback also which is called after a particular track is done playing. We passed in `handleSongFinished`, let's define that:

```javascript
handleSongFinished () {
	// Call random Track
	this.randomTrack();
}
```

Quite straight-forward! Call the `randomTrack()` member method after a track is done playing. This means another track from the playlist will begin to play.

## Searching for Music

This component wraps our auto-complete functionality. Let us start with importing it:

```javascript
// ...

import Search from '../components/search.component';

// ...
```

... then include the component in render's return like so:

```javascript
//...

render () {
	return (
		<div className="scotch_music">
		<Search
		autoCompleteValue={this.state.autoCompleteValue}
		tracks={this.state.tracks}
		handleSelect={this.handleSelect.bind(this)}
		handleChange={this.handleChange.bind(this)}/>
		{/*...*/}
		</div>
	);
}

//...
```

Recall that our UI component uses props to communicate with the container component. Both values and callbacks are passed in as props:

__`autoCompleteValue`__: Simply the value of the auto-complete input/search box. It receives a state and the state needs to be defined:

```javascript
constructor(props) {
	//...
	
	this.state = {
		// ...other states
		autoCompleteValue: ''
	};
}
```

__`tracks`__: The auto-complete items shown in the drop-down has to be a list of tracks we is an array of Soundcloud tracks. It's initial state is an empty array:

```javascript
constructor(props) {
	//...
	
	this.state = {
		// ...other states
		autoCompleteValue: '',
		tracks: []
	};
}
```

__`handleSelect`__: Receives a callback which is called when an item on the auto-complete's drop-down is clicked. The value is a member method `handleSelect`:

```javascript
handleSelect(value, item){
	this.setState({ autoCompleteValue: value, track: item });
}
```

We are updating the state of two values, `autoCompleteValue` which will just set the input box with what we are entering and track which will change the current song that is playing to what we selected.

__`handleChange`__:  Also receives a callback which is called when the value of the auto-complete's input box changes. The value is a member method `handleChange`:

```javascript
handleChange(event, value) {

    // Update input box
    this.setState({autoCompleteValue: event.target.value});
    let _this = this;
	
    //Search for song with entered value
    Axios.get(`https://api.soundcloud.com/tracks?client_id=${this.client_id}&q=${value}`)
      .then(function (response) {
        // Update track state
        _this.setState({tracks: response.data});
      })
      .catch(function (err) {
        console.log(err);
      });
  }
```

First we update the value in the input box and then use Axios to search for songs on Soundcloud based on what is entered in the input box.

## Music Details

This component as we already saw, expects just the track title which it will display with `h3`:

```javascript
//... other imports
//Details components
import Details from '../components/details.component';

class AppContainer extends React.Component {

  constructor(props) {
     super(props);
     this.state = {
       track: {stream_url: '', title: '', artwork_url: ''},
       //... other states
     };
   }

  render () {
    return (
      <div className="scotch_music">
        //... other components
        <Details
          title={this.state.track.title}/>
        //...other components
      </div>
    );
  }
}

export default AppContainer
```

The title property is supplied with the title of the playing track.

## Music Player Controls

The player component is filled with series of callbacks because it is made up of just control buttons:

```javascript
//... other imports
//Details components
import Player from '../components/player.component';

class AppContainer extends React.Component {

  //... other members

  render () {
    return (
      <div className="scotch_music">
        //... other components
        <Player
          togglePlay={this.togglePlay.bind(this)}
          stop={this.stop.bind(this)}
          playStatus={this.state.playStatus}
          forward={this.forward.bind(this)}
          backward={this.backward.bind(this)}
          random={this.randomTrack.bind(this)}/>
        //...other components
      </div>
    );
  }
}

export default AppContainer
```

__`togglepplay`__:  Receives a member method as callback, `togglePlay` :

```javascript
togglePlay(){
    // Check current playing state
    if(this.state.playStatus === Sound.status.PLAYING){
      // Pause if playing
      this.setState({playStatus: Sound.status.PAUSED})
    } else {
      // Resume if paused
      this.setState({playStatus: Sound.status.PLAYING})
    }
  }
```

Simply runs a check on the current play status, if it is playing, the sound will be paused when the button is clicked and vice versa.

__`stop`__: Just as the name goes, halts a current sound and resets it's position to 0 when played again. Receives a callback, `stop`:

```javascript
stop(){
    // Stop sound
   this.setState({playStatus: Sound.status.STOPPED});
  }
```

We are just updating the `playStatus` state.

__`forward`__: You can also refer to it as `fast-forward` We want to push the song +10 sec when it is clicked. Receives `forward` method as callback:

```javascript
forward(){
    this.setState({playFromPosition: this.state.playFromPosition+=1000*10});
  }
```

__`backward`__: Just like forward but -10sec:
```javascript
backward(){
    this.setState({playFromPosition: this.state.playFromPosition-=1000*10});
  }
```

__`random`__: We also want our app to have a cool feature of manually picking a random track from the playlist we are using. To do so we just call the `randomTrack` method when the button is clicked. The method is available above as we have used it before in `componentDidMount`.

## Song Progress Bar

The progress component's properties only receive states. States which previously, we have seen how and why they exist. Just as a recap, they include: `elapsed`, `total` and `position`:

```javascript
<Progress
          elapsed={this.state.elapsed}
          total={this.state.total}
          position={this.state.position}/>
```

## Footer

This _guy_ does not even deserve a subtitle because there is nothing to discuss about it. It is just a static component with no state or behavior:

```javascript
<Footer />
```

## Background Image Artwork Trick

From the peek of how our app looks which was show in the first section, the background of the app changes with each track's artwork. Colors in such images are unpredictable and might conflict with the components' colors. The trick is to use [background overlay](https://css-tricks.com/forums/topic/css-background-image-color-overlay/):

```javascript
// app.component.js
// ...other members
  render () {
    const scotchStyle = {
      width: '500px',
      height: '500px',
      backgroundImage: `linear-gradient(
      rgba(0, 0, 0, 0.7),
      rgba(0, 0, 0, 0.7)
    ),   url(${this.xlArtwork(this.state.track.artwork_url)})`
    }
    return (
      <div className="scotch_music" style={scotchStyle}>
        {/*other components*/}
//...
```

Soundcloud returns a small (though Soundcloud calls it large) artwork URL so we needed a util method to replace it with a large one (Soundcloud calls it `t500`):

```javascript
xlArtwork(url){
    return url.replace(/large/, 't500x500');
  }
```

Long story short, this is how our `app.component.js` looks:

```javascript
//React library
import React from 'react';

//Sound component
import Sound from 'react-sound';

//Axios for Ajax
import Axios from 'axios';

//Custom components
import Details from '../components/details.component';
import Player from '../components/player.component';
import Progress from '../components/progress.component';
import Search from '../components/search.component';
import Footer from '../components/footer.component';

class AppContainer extends React.Component {

  constructor(props) {
     super(props);
     this.client_id = '2f98992c40b8edf17423d93bda2e04ab';
     this.state = {
       track: {stream_url: '', title: '', artwork_url: ''},
       tracks: [],
       playStatus: Sound.status.STOPPED,
       elapsed: '00:00',
       total: '00:00',
       position: 0,
       playFromPosition: 0,
       autoCompleteValue: ''
     };
   }

 componentDidMount() {
    this.randomTrack();
  }

  prepareUrl(url) {
    //Attach client id to stream url
    return `${url}?client_id=${this.client_id}`
  }

  xlArtwork(url){
    return url.replace(/large/, 't500x500');
  }

  togglePlay(){
    // Check current playing state
    if(this.state.playStatus === Sound.status.PLAYING){
      // Pause if playing
      this.setState({playStatus: Sound.status.PAUSED})
    } else {
      // Resume if paused
      this.setState({playStatus: Sound.status.PLAYING})
    }
  }

  stop(){
    // Stop sound
   this.setState({playStatus: Sound.status.STOPPED});
  }

  forward(){
    this.setState({playFromPosition: this.state.playFromPosition+=1000*10});
  }

  backward(){
    this.setState({playFromPosition: this.state.playFromPosition-=1000*10});
  }

  handleSelect(value, item){
    this.setState({ autoCompleteValue: value, track: item });
  }

  handleChange(event, value){
    // Update input box
    this.setState({autoCompleteValue: event.target.value});
    let _this = this;
    //Search for song with entered value
    Axios.get(`https://api.soundcloud.com/tracks?client_id=${this.client_id}&q=${value}`)
      .then(function (response) {
        // Update track state
        _this.setState({tracks: response.data});
      })
      .catch(function (err) {
        console.log(err);
      });
  }


  formatMilliseconds(milliseconds) {
     var hours = Math.floor(milliseconds / 3600000);
     milliseconds = milliseconds % 3600000;
     var minutes = Math.floor(milliseconds / 60000);
     milliseconds = milliseconds % 60000;
     var seconds = Math.floor(milliseconds / 1000);
     milliseconds = Math.floor(milliseconds % 1000);

     return (minutes < 10 ? '0' : '') + minutes + ':' +
        (seconds < 10 ? '0' : '') + seconds;
  }

  handleSongPlaying(audio){
     this.setState({  elapsed: this.formatMilliseconds(audio.position),
                      total: this.formatMilliseconds(audio.duration),
                      position: audio.position / audio.duration })
   }

  handleSongFinished () {
    this.randomTrack();
   }

  randomTrack () {
    let _this = this;
    //Request for a playlist via Soundcloud using a client id
    Axios.get(`https://api.soundcloud.com/playlists/209262931?client_id=${this.client_id}`)
      .then(function (response) {
        // Store the length of the tracks
        const trackLength = response.data.tracks.length;
        // Pick a random number
        const randomNumber = Math.floor((Math.random() * trackLength) + 1);
        //Set the track state with a random track from the playlist
        _this.setState({track: response.data.tracks[randomNumber]});
      })
      .catch(function (err) {
        //If something goes wrong, let us know
        console.log(err);
      });
   }

  render () {
    const scotchStyle = {
      width: '500px',
      height: '500px',
      backgroundImage: `linear-gradient(
      rgba(0, 0, 0, 0.7),
      rgba(0, 0, 0, 0.7)
    ),   url(${this.xlArtwork(this.state.track.artwork_url)})`
    }
    return (
      <div className="scotch_music" style={scotchStyle}>
        <Search
          clientId={this.state.client_id}
          autoCompleteValue={this.state.autoCompleteValue}
          tracks={this.state.tracks}
          handleSelect={this.handleSelect.bind(this)}
          handleChange={this.handleChange.bind(this)}/>
        <Details
          title={this.state.track.title}/>
        <Sound
           url={this.prepareUrl(this.state.track.stream_url)}
           playStatus={this.state.playStatus}
           onPlaying={this.handleSongPlaying.bind(this)}
           playFromPosition={this.state.playFromPosition}
           onFinishedPlaying={this.handleSongFinished.bind(this)}/>
        <Player
          togglePlay={this.togglePlay.bind(this)}
          stop={this.stop.bind(this)}
          playStatus={this.state.playStatus}
          forward={this.forward.bind(this)}
          backward={this.backward.bind(this)}
          random={this.randomTrack.bind(this)}/>
        <Progress
          elapsed={this.state.elapsed}
          total={this.state.total}
          position={this.state.position}/>
        <Footer />
      </div>
    );
  }
}

export default AppContainer
```

Note that we export `AppContainer` so it can be imported in `app.js`

## Refactoring `app.js`

 In the previous article, our `app.js`'s App component is the guy importing and using our UI components. Now we have shifted that responsibility to `AppContainer`, therefore, our `app.js` will end up looking like this:

```javascript
//React libraries
import React from 'react';
import ReactDOM from 'react-dom';

//Import Container component
import AppContainer from './containers/app.container'

class App extends React.Component {
  render () {
    return (
      <AppContainer />
    );
  }
}

// Render to index.html
ReactDOM.render(
  <App />,
  document.getElementById('content')
);
```

![React and Electron Music Player](https://cdn.scotch.io/10/Z1uDIk7HQyuooL13Smm6_scotch-final.png)

The [Codepen demo](http://codepen.io/christiannwamba/full/aNjjPE/) is not for code perusing as I just dumped the bundle.js into it. Rather, it is a good play guide of what we built

## Conclusion

We had a bit of a lot of things: Electron, React, Babel, Browserify, Soundcloud, Axios, etc. Just one matters a lot on the long run and that is React. This is the end of Journey for this series and hopefully you understand what React is, why it exists, its fundamentals, and its use cases. There are a lot to still learn about React  (React Router, Redux, etc) which Scotch will be bringing to your table in no distant time.

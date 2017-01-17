# Making The UI

In the previous chapter, we setup an Electron project, discussed the basic concepts of React and created a _useless_ example to get us started. Now we will start making something - a music app.


We also discussed about the types of components which are presentation and container components. In this tutorial we will focus on our presentation component and talk about container in the next.

## Revisit `app` Directory Structure

```bash
|---app #All React projects goes here
|----components # Presentation Component Directory
|------details.component.js
|------footer.component.js
|------player.component.js
|------progress.component.js
|------search.component.js
|----containers # Container Component Directory
|------app.container.js
|----app.js
```

The `app` folder has two sub folders and an `app.js` at its root. Our only concern in this section of the series is `app/components` where our presentation component will live and `app.js` which will put the components together and tie them to the DOM.

## Components
We will build the presentation component one after the other. The idea of components makes it easier for designers to work with wireframes and transform the wire-frames into components. Below is a wireframe for Scotch Player:

![React.js and Electron Music Player Wireframe](https://cdn.scotch.io/10/MvP4eIbSkWEyKnH8sxZc_Scotch_Player_Wireframe.png)

With the above wireframe, it becomes really simple to fish out the components and reason around them. We have 5 UI (presentation) components:

1. Search
2. Details
3. Player
4. Progress
5. Footer

> A presentation component can also be referred to as UI component

## Search (UI) Component

This component lives in `app/components/search.component.js` and has the same basic React component skeleton:

```javascript
// Import React
import React from 'react';

// Create Search component class
class Search extends React.Component{

  render() {
    // Return JSX via render()
    return (
      <div className="search">

      </div>
    );
  }
  
}

// Export Search
export default Search
```

What will be cool is that instead of using the usual search form with a submit button, we could make use of an auto-complete component. The React team has an awesome [auto-complete component](https://github.com/reactjs/react-autocomplete?ref=scotch.io) which we aready included in the `package.json`; no need to re-invent the wheel:

```javascript
// Import React
import React from 'react';

// Import React's Autocomplete component
import Autocomplete from 'react-autocomplete';

// Create Search component class
class Search extends React.Component{

  render() {
    // Return JSX via render()
    return (
      <div className="search">
        {/*Autocomplete usage with value and behavior handled via this.props*/}
        <Autocomplete
         ref="autocomplete"
         inputProps={{title: "Title"}}
         value={this.props.autoCompleteValue}
         {/*Array of tracks is passed in to items*/}
         items={this.props.tracks}
         {/*Single value selected*/}
         getItemValue={(item) => item.title}
         {/*What happens when an item is selected*/}
         onSelect={this.props.handleSelect}
         {/*What happens when keystrokes are received*/}
         onChange={this.props.handleChange}
         {/*How items are redered.*/}
         renderItem={this.handleRenderItem.bind(this)}
       />
      </div>
    );
  }
  
}

// Export Search
export default Search
```

> Although `{/* */}` is the proper way to comment in JSX, React will not allow those comments between properties, therefore, remove in your usage.

`renderitem` is the only property that does not receive props but value which is `handleRenderitem` and is created as a method in the `Search` class:

```javascript
// ...
//class Search extends React.Component{
  handleRenderItem(item, isHighlighted){
    // Some basic style
    const listStyles = {
      item: {
        padding: '2px 6px',
        cursor: 'default'
      },

      highlightedItem: {
        color: 'white',
        background: '#F38B72',
        padding: '2px 6px',
        cursor: 'default'
      }
    };
	
    // Render list items
    return (
      <div
        style={isHighlighted ? listStyles.highlightedItem : listStyles.item}
        key={item.id}
        id={item.id}
      >{item.title}</div>
    )
  }
//  render() {
// ...
```
Notice how styles are pased in too. In React there is room for global styles and inline dynamic styles as seen above.

> Note that the Search component has no constructor method. If all a component does is receive props (which is exactly what UI components do), then the constructor is not needed.

To see what we have done so far, update the `app.js` with the following:

```javascript
// ES6 Component
// Import React and ReactDOM
import React from 'react';
import ReactDOM from 'react-dom';

// Import Search Component
import Search from './components/search.component';

// Component Class
class App extends React.Component {
    // render method is most important
    // render method returns JSX template
    render() {
        return (
          <Search />
        );
    }
}

// Render to ID content in the DOM
ReactDOM.render(
    <App/ >,
    document.getElementById('content')
);
```

See how `Search` is imported and used inside `App`. No functionality attached  yet, remember this section is just about UI

![React Electron Music Player Search](https://cdn.scotch.io/10/WLaiEUHYTVeBhLLnnLBY_scotch-player-search.png)


Yeah! So darn ugly! We will fix that later.

## Details (UI) Component

This is the simplest component we will create as it just has a `h3` tag with the track title:
```javascript
// Import React
import React from 'react';

class Details extends React.Component {
  // Render
  render(){
    return(
      <div className="details">
        <h3>{this.props.title}</h3>
      </div>
    )
  }

}
// Export
export default Details
```
We update `app.js` with `Details`:
```javascript
// ES6 Component
// Import React and ReactDOM
import React from 'react';
import ReactDOM from 'react-dom';

// Import Search Component
import Search from './components/search.component';

// Import Details Component
import Details from './components/details.component';

// Component Class
class App extends React.Component {
    // render method is most important
    // render method returns JSX template
    render() {
        return (
          <div>
            <Search />
            {/* Added Details Component */}
            <Details title={'Track title'} />
          </div>
        );
    }
}

// Render to ID content in the DOM
ReactDOM.render(
    <App/ > ,
    document.getElementById('content')
);

```
We import the `Details` component and add it to the `App`.

>JSX will fail to compile if we do not wrap and expose the content using only one tag. That's why we wrapped `Search` and `Details` with `div`

![React Electron Music Player Details](https://cdn.scotch.io/10/qHyLcmbaR9SJRbvhKCzA_scotch-player-details.png)

## Player (UI) Component

The player component is made of the controls for our music player:
```javascript
// Import React
import React from 'react';

// Import ClassNames
import ClassNames from 'classnames';

// Player component class
class Player extends React.Component {

  render(){
    // Dynamic class names with ClassNames
    const playPauseClass = ClassNames({
      'fa fa-play': this.props.playStatus == 'PLAYING' ? false : true,
      'fa fa-pause': this.props.playStatus == 'PLAYING' ? true : false
    });
	
    // Return JSX
    return(
      <div className="player">
        {/*Rewind Button*/}
        <div className="player__backward">
          <button onClick={this.props.backward}><i className="fa fa-backward"></i></button>
        </div>
        <div className="player__main">
          {/*Play/Pause Button*/}
          <button onClick={this.props.togglePlay}><i className={playPauseClass}></i></button>
          {/*Stop Button*/}
          <button onClick={this.props.stop}><i className="fa fa-stop"></i></button>
          {/*Random Track Button*/}
          <button onClick={this.props.random}><i className="fa fa-random"></i></button>
        </div>
        {/*Forward Button*/}
        <div className="player__forward">
          <button onClick={this.props.forward}><i className="fa fa-forward"></i></button>
        </div>
      </div>
    )
  }

}

// Export Player
export default Player
```

The new stuff to observe in this component is that we imported a `classnames` utility library which helps us dynamically set classes. It is useful here to swith the `play/pause` button to `pause` or `play` depending on the current state of the track.

The `events` callbacks will be handled in the container component which we will discuss in the next section. Next, include `Player` component in `app.js`:

```javascript
// ES6 Component
// Import React and ReactDOM
import React from 'react';
import ReactDOM from 'react-dom';

// Import Search Component
import Search from './components/search.component';

// Import Details Component
import Details from './components/details.component';

// Import Player Component
import Player from './components/player.component';

// Import Progress Component
// Component Class
class App extends React.Component {

    // render method is most important
    // render method returns JSX template
    render() {
        return (
          <div>
            <Search />
            <Details title={'Track title'} />
            {/* Added Player component*/}
            <Player  />
          </div>
        );
    }
	
}

// Render to ID content in the DOM
ReactDOM.render(
    <App/ > ,
    document.getElementById('content')
);
```

We need to add `font-awesome` to `index.html` for the control icons to look as expected:
```markup
<!-- ... -->
  <title>Scotch Player</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.6.1/css/font-awesome.min.css">
<!-- ... -->
```

![React Electron Music Player Play Buttons](https://cdn.scotch.io/10/9NNnfredRUOEzcyzhZgH_scotch-player-player.png)

You will end up dissapointed if you try clicking them - no behavior or state yet. Relax!

## Progress (UI) Component

This guy is a simple one also - just a:

- progress bar
- total play time
- elapsed time:

```javascript
// Import React
import React from 'react';

// Create Progress component class
class Progress extends React.Component {

  // Render method
  render() {
  
    return(
      <div className="progress">
        {/* Elapsed time */}
        <span className="player__time-elapsed">{this.props.elapsed}</span>
        {/* Progress Bar */}
        <progress
           value={this.props.position}
           max="1"></progress>
         {/* Total time */}
         <span className="player__time-total">{this.props.total}</span>
      </div>
    )
  }

}

//Export Progress
export default Progress
```

And now to include this component in `app.js` like we did for the others:

```javascript
// ES6 Component
// Import React and ReactDOM
import React from 'react';
import ReactDOM from 'react-dom';

// Import Search Component
import Search from './components/search.component';

// Import Details Component
import Details from './components/details.component';

// Import Player Component
import Player from './components/player.component';

// Import Progress Component
import Progress from './components/progress.component';

// Component Class
class App extends React.Component {

    // render method is most important
    // render method returns JSX template
    render() {
        return (
          <div>
            <Search />
            <Details title={'Track title'} />
            <Player  />
            {/* Added Progress component*/}
            <Progress
	          position={'0.3'}
              elapsed={'00:00'}
              total={'0:40'}/>
          </div>
        );
    }
	
}

// Render to ID content in the DOM
ReactDOM.render(
    <App/ > ,
    document.getElementById('content')
);

```

After updating the `app.js` as we have been doing, you should have something like the image below:

![React Electron Music Player Progress Bar](https://cdn.scotch.io/10/ZiQVdq4eTT6V7ViVtJ3U_scotch-player-progress.png)

## Footer (UI) Component

The last is the `footer` component. This is a completely _dumb_ static component:

```javascript
import React from 'react';

class Footer extends React.Component {
  render(){
    return(
      <div className="footer">
        <p>Love from <img src="public/img/logo.png" className="logo"/>
            & <img src="public/img/soundcloud.png" className="soundcloud"/></p>
      </div>
    )
  }

}

export default Footer
```

Include in `app.js`, add the images in the `public/img` directory and the app should be looking like the image below:

![React Electron Music Player Logo](https://cdn.scotch.io/10/IIg0q4cRSlmFjBgGSb4D_scotch-player-footer.png)

Awful look, right? Let's fix that.

## Global Styles 

React supports [inline styles](https://facebook.github.io/react/tips/inline-styles.html) which we have seen. Inline styles are handy for dynamic functionality and to compose reusable components. This does not mean that we cannot use our global style and at the time of this writing, React has no rigid recommendation for how styles are added.

We won't spend much time talking about the styles as it is out of scope. Just dump the following in `public/css/globals.css` and wait for the _magic_:

```css
/* Box sizing resets*/
*, *:before, *:after {
  box-sizing: border-box;
}

/* Body resets and fancy font*/
body{
  margin: 0;
  font-family: 'Exo 2', sans-serif;
}

.scotch_music{
  position: relative;
}

.search div, .search input {
  width: 100%;
}

.search input {
  border: none;
  border-bottom: 2px solid rgb(243, 139, 114);
  outline: none;
  background: rgba(255, 255, 255, 0.8);
  padding: 5px;
}

.details h3{
  text-align: center;
  padding: 50px 10px;
  margin: 0;
  color: white;
}

.player{
  text-align: center;
  margin-top: 60px;
}

.player div{
  display: inline-block;
  margin-left: 10px;
  margin-right: 10px;
}


.player .player__backward button, 
.player .player__forward button{
  background: transparent;
  border: 1px solid rgb(243, 139, 114);
  color: rgb(243, 139, 114);
  width: 75px;
  height: 75px;
  border-radius: 100%;
  font-size: 35px;
  outline: none;
}

.player .player__backward button{
  border-left: none;
}

.player .player__forward button{
  border-right: none;
}

.player .player__main button:hover, 
.player .player__backward button:hover, 
.player .player__forward button:hover{
  color: rgba(243, 139, 114, 0.7);
  border: 1px solid rgba(243, 139, 114, 0.7);
}

.player .player__main {
  border: 1px solid rgb(243, 139, 114);
}

.player .player__main button {
  color: rgb(243, 139, 114);
  background: transparent;
  width: 75px;
  height: 75px;
  border: none;
  font-size: 35px;
  outline: none;
}

.progress{
  text-align: center;
  margin-top: 100px;
    color: white;
}

.progress progress[value] {
  /* Reset the default appearance */
  -webkit-appearance: none;
   appearance: none;

  width: 390px;
  height: 20px;
  margin-left: 4px;
  margin-right: 4px;
}

.progress progress[value]::-webkit-progress-bar {
  background-color: #eee;
  border-radius: 2px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.25) inset;
}

.progress progress[value]::-webkit-progress-value {
    background-color: rgb(243, 139, 114);
    border-radius: 2px;
    background-size: 35px 20px, 100% 100%, 100% 100%;
}

.footer{
  color: white;
  position: absolute;
  bottom: 0px;
  width: 100%;
  background: #524C4C;
}

.footer p{
  text-align: center;
}

.footer .logo{
  height: 25px;
  width: auto;
}
.footer .soundcloud{
  height: 25px;
  width: auto;
}
```

We need to import the `global.css` and `Exo 2` font in the `index.html`:

```markup
<link rel="stylesheet" href="public/css/global.css">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.6.1/css/font-awesome.min.css">
    <link href='https://fonts.googleapis.com/css?family=Exo+2:500' rel='stylesheet' type='text/css'>
```

![React Electron Music Player Styled](https://cdn.scotch.io/10/dlvLYAKQScC9KHqr2H71_scotch-player-global-style.png)

Looks better, right? We're another big step closer to achieving our fully functional React and Electron music player!

## Up Next...

Our presentation components are ready and styled (though needs some more touches). In the next section we will create our container component.
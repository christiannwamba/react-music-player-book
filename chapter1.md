# Component Basics

## Required Knowledge

A basic knowledge of HTML and JavaScript (ES6) is enough for you to understand this tutorial. On the other hand the only fairly advanced topic you need is an [understanding of the _mystries_ of `this` keyword](http://www.sitepoint.com/inner-workings-javascripts-this-keyword/).

## Setup and Prerequisites

### Browserify and Babel:
[Browserify](http://browserify.org/) helps us use client JS libraries like React and jQuery with Node's `require()` and makes bundling easy:
```bash
npm install -g browserify
```

If you want to learn more abou Browserify, check out [Peleke's tutorial](https://scotch.io/tutorials/getting-started-with-browserify)

Not all browsers have EcmaScript 2015 (ES6) support, therefore a transpile tool will be needed in such case. JavaScript transpilers are important. [here is an article](https://scotch.io/tutorials/transpilers-what-they-are-why-we-need-them) on when and why we use them. 

To install [Babel](https://babeljs.io) (which is the transformer) and its squad of tools called [`presets`](https://babeljs.io/docs/plugins/), we will include them in our `package.json` which we will address soon. 

We also need to create `./.babelrc` file to inform babal which presets we are using:

```bash
"presets": ["es2015", "react"]
```

### Electron
Electron is a tool for building cross platform desktop apps with web technologies. This means that you do not have to learn an OS native language so as to build an app that runs natively on computers. The amazing aspect actually is that you write ones and build the same code for different platforms (OSX, Windows, Linux). We have written an article on Angular and Electron and can learn more from [this Jasim's tutorial](https://scotch.io/tutorials/creating-desktop-applications-with-angularjs-and-github-electron)

Our app is expected to run as a standalone app and not in a browser.
Electron is in my opinion the most popular tool for building desktop apps with web technologies (HTML, CSS, JS):

```bash
# Clone the Quick Start repository
$ git clone https://github.com/electron/electron-quick-start scotch-player

# Go into the repository
$ cd scotch-player
```

The starter has two important files: `main.js` and `index.html`. These files serve as entry to an Electron project.

To see the expected blank workspace, run:
```bash
# Launch the App with Electron
npm start
```

![](https://cdn.scotch.io/10/cmgKLXmT9KEucgRClfHO_scotch-player-first-run.png)


### Directory Structure
Below is the directory structure of what we are building and will serve as guide down the journey:

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
|---public # Client Files here
|----css
|------global.css
|----img
|------logo.png
|------soundcloud.png
|----js
|------bundle.js
|---index.html # Electron Default View
|---main.js # Electron entry point
|---package.json
|---.babelrc # Babal's Configurations
```

### Configure `package.json`
```json
{
  "name": "scotch-player",
  "productName":"Scotch Player",
  "version": "1.0.0",
  "description": "Scotch Demo Player",
  "main": "main.js",
  "scripts": {
    "start": "electron main.js",
    "watch": "watchify app/app.js -t babelify -o public/js/bundle.js --debug --verbose"
  },
  "author": "Scotch",
  "license": "MIT",
  "dependencies": {
    "axios": "^0.9.1",
    "babel-preset-es2015": "^6.6.0",
    "babel-preset-react": "^6.5.0",
    "babelify": "^7.2.0",
    "classnames": "^2.2.3",
    "electron-prebuilt": "^0.36.0",
    "electron-reload": "^0.2.0",
    "jquery": "^2.2.3",
    "react": "^0.14.8",
    "react-autocomplete": "^1.0.0-rc2",
    "react-dom": "^0.14.7",
    "react-sound": "^0.4.0",
    "soundmanager2": "^2.97.20150601-a"
  }
}
```

Our concerns are the `scripts` and `dependencies` section. The `scripts` has two commands, the firsts (`start`) starts the app and the second (`watch`) tells browserify to watch the app folder and bundle the JS content to `public/js/bundle.js`

Go ahead and install the dependencies:

```bash
npm install
```

## Enter React: Presentation vs Container Components

React is really a simple and small library. It only echoes one word, `Components`. React is simply a UI library that helps web designers/developers build reusable UI components. Reusable from as small as a button to as complex as navigation menus. See [Ken's article](https://scotch.io/tutorials/learning-react-getting-started-and-concepts) for more on getting started with React.

A simple component could be:

```javascript
// ES6 Component
// Import React
import React  from 'react';
// Search component created as a class
class Search extends React.Component {

// render method is most important
// render method returns JSX template
    render() {
	return (
            <form>
		        <input type="text" />
        		<input type="submit" />
            </form>
        );
    }
}

// Export for re-use
export default Search
```
 We will see more of that while we build the app. The only weird thing here is the XML-like content in our JavaScript. It is called JSX and just a convenient way to write HTML in JavaScript. You have the option to go hardcore with `document.createElement`.

A little exception could be made though - components are not just for UIs, they can be used to manage state of UI component. Let's have a deeper look:

### Presentation Components
This components are simple and very straightforward. They just present UI details and nothing much. It should never manage state but should only receive properties to be bound to its UI. 

Events also are handled as callbacks via properties and not in components.

One more thing, for no reason should a presentation component be aware of how data sent to it came about. It should be possible to isolate it.

A simple example: 

```javascript
import React  from 'react';

class Search extends React.Component {
// Props is received via the constructor
  constructor(props) {
  //...and props is sent back to the parent Component
  //class using super()
    super(props);
    // Initial State of the component is defined
    // in the constructor also
    this.state = {
      value:''
    };
  }
  
  handleSubmit() {
	//postRequest is assumed to be the function
	// that makes an Ajax request
    postRequest(this.state.value);
  }
  
  handleChange(e) {
  // Bind value state with current input
    this.setState({value: e.target.value});
  }
  
  render() {
  	return (
        <form 
          onSubmit={this.handleSubmit.bind(this)}>
            <input type="text" 
              value={this.state.value} 
              onChange={this.handleChange.bind(this)}/>
    		<input type="submit"/>
        </form>
      );
    }
}

export default Search
```

The above example is doing exactly what we do not want a presentation component to do. It knows how values are manipulated and sent to the server. Let's refactor:

```javascript
import React  from 'react';
// Simplified component
class Search extends React.Component {
  render() {
  	return (
        <form 
          onSubmit={this.props.handleSubmit}>
          {/* Notice how values and callbacks are passed in using props */}
           <input type="text" 
             value={this.props.searchValue} 
             onChange={this.props.handleChange}/>
    	   <input type="submit"/>
        </form>
      );
    }
}
```

Now the component is extra lean and is focused on presenting content depending on the values passed to it via `this.props`. It also handles events with functions passed to it via `this.props` too. So where do the states and those event handlers go to? They are moved to the container components.

> __Note on `props` and `states`__
> 
> Properties and states join forces to make React an amazing and awesome tool. Properties are just read only values passed from one component to another or a component to its UI elements. Props are accessed with just `this.props` because they are read only. 
>
> States on the other hand are writable and that is the way we update our application data (state). States are accessible via `this.state` and because they are writable, can be updated with `this.setState({value: 'value'})`.

### Container Components
This works with the concept of `service provider`. They tend to expose APIs for the presentation components. This is where you can get your hands dirty and do the dirty jobs of updating states and defining application behaviors. This is the _guy_ that takes care of all those jobs we asked presentation components to drop.

To better understand container components, let us have a look at one that compliments our above `Search Component`:

```javascript
import React  from 'react';
import Search from './search';

class AppContainer extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value:''
    };
  }
  
  // Makes a request to the server (simulated for this tutorial)
  handleSubmit() {
    postRequest(this.state.value);
  }
  
  // React input update (binding) is manual which
  // makes it rubust. This is how you keep the input box
  // in sync with keystroke inputs
  handleChange(e) {
    // New values are availbale from the event object
    this.setState({value: e.target.value});
  }
  
  // Container components wrap presentation component
  render() {  
  	return (
        <Search
          handleSubmit={this.handleSubmit.bind(this)}
          handleChange={this.handleChange.bind(this)}
          searchValue={this.state.value}/>
      );
    }
}

export default AppComponent
```

See how the above component abstracts the state update and event handling from the presentation component. Once an event is called on the presentation component, it asks its parent container component to handle the event and if any change in state, pass it back.

Do not panic, we learn better with examples and there are few examples in the demo we will talk about.

## React in Electron
It is very simple to use React in an Electron project. You can bundle with Browserify or Webpack but in our case we will use browserify. The following checklist is good for setting React in any platform including Electron:

 1. Install Browserify: [✔]
 2. Install Babel: [✔]
 3. Configure Babel presets using `package.json` or `.babelrc`: [✔]
 4. Add a `watch` script to `package.json`:  [✔]
 5. Create a component and render it to the index.html
 6. Run `start` and `watch` scripts to start Electron and bundling respectively

As you can see, we have accomplished 1 to 4 and that is why they are checked. Let us have a look at 5 and 6.

Create `app.js` in the `app` folder as shown in the directory structure with the following:

```javascript
// ES6 Component
// Import React and ReactDOM
import React from 'react';
import ReactDOM from 'react-dom';
// Search component created as a class
class Search extends React.Component {

    // render method is most important
    // render method returns JSX template
    render() {
        return (
          <form>
            <input type = "text" />
            <input type = "submit" />
          </form>
        );
    }
}

// Render to ID content in the DOM
ReactDOM.render( < Search / > ,
    document.getElementById('content')
);
```
 Something new is ReactDOM which is used to render components to the DOM. The `index.html` in our case:

```markup
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Scotch Player</title>
  </head>
  <body>

    <div id="content"></div>

  <script src="public/js/bundle.js"></script>
  </body>
</html>
```

We are pointing to a non existing file, `bundle.js`.  It will contain our bundled app therefore create the file and leave it empty then run:

```bash
npm run watch
```

> Browserify comes with two flavors: `browserify` and `watchify`. The difference is that watchify just waites for change and re-creates the bundle while `browserify` bundles only when you issue the command.

It is painful to continue running `npm start`  for every change just to update Electron with the changes. We can automate it by adding the following in the `main.js`:

```javascript
require('electron-reload')(__dirname);
```

We already install the package using `package.json`.

![React and Electron Music Player](https://cdn.scotch.io/10/aBiSChdhSBungjClzJm8_scotch-player-first-component.png)

## Up Next...

Hopefully, you have a fair knowledge of React, how it is useful, and the two types components. If you need more on component types [Dan's Medium post](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.j0gtmis7e) will help.  In the next chapter, we will design our presentation components.
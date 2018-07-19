
## Django with React 
![](https://codyparker.com/wp-content/uploads/2016/10/dj-react.png)

### Setup:

 - Create Django project.

 - Setup django app, views,urls

 - Install django-weback-loader: Run `pip install django-webpack-loader` in ProjectContainer directory

 - Add webpack_loader to the INSTALLED_APPS in settings.py
```python
INSTALLED_APPS = [
	...
	'webpack_loader',
]
```
Also add this
```python
WEBPACK_LOADER = {
    	'DEFAULT': {
        	'BUNDLE_DIR_NAME': 'bundles/local/',  # end with slash
        	'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats-local.json'),
     }
}
```
Also add this and comment it for now
```python
WEBPACK_LOADER = {
    	'DEFAULT': {
        	'BUNDLE_DIR_NAME': 'bundles/stage/',  # end with slash
        	'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats-stage.json'),
     }
}
```
Also add this
```python
STATICFILES_DIRS = [
	   os.path.join(BASE_DIR, 'project/assets'),
]
```
 - Create folder `assets` in `project` folder like below
```
project  
│
└───assets
│   └───local
│   └───stage
```



	
 - Create package.json file in ProjectContainer
```python
{
  "name": "djreact",
  "version": "0.0.1",
  "devDependencies": {
    "babel": "^6.5.2",
    "babel-core": "^6.6.5",
    "babel-eslint": "^5.0.0",
    "babel-loader": "^6.2.4",
    "babel-plugin-transform-decorators-legacy": "^1.3.4",
    "babel-preset-es2015": "^6.6.0",
    "babel-preset-react": "^6.5.0",
    "babel-preset-stage-0": "^6.5.0",
    "eslint": "^2.2.0",
    "react": "^0.14.7",
    "react-hot-loader": "^1.3.0",
    "redux-devtools": "^3.1.1",
    "webpack": "^1.12.13",
    "webpack-bundle-tracker": "0.0.93",
    "webpack-dev-server": "^1.14.1"
  },
  "dependencies": {
    "es6-promise": "^3.1.2",
    "isomorphic-fetch": "^2.2.1",
    "lodash": "^4.5.1",
    "radium": "^0.16.6",
    "react-cookie": "^0.4.5",
    "react-dom": "^0.14.7",
    "react-redux": "^4.4.0",
    "redux": "^3.3.1",
    "redux-thunk": "^1.0.3"
  }
}
```


 - Run `npm install`

 - Create webpack.base.config.js
```javascript
var path = require("path")
var webpack = require('webpack')
module.exports = {
  context: __dirname,
  entry: {
    // Add as many entry points as you have container-react-components here
    App1: './reactjs/App1',
    App2: './reactjs/App2',
    App3: './heatmap/static/js/App3',
    vendors: ['react'],
  },
  output: {
      path: require('path').resolve('./project/assets/bundles/local/'), //static-->assets
      filename: "[name]-[hash].js"
  },
  externals: [
  ], // add all vendor libs
  plugins: [
    new webpack.optimize.CommonsChunkPlugin('vendors', 'vendors.js'),
  ], // add all common plugins here
  module: {
    loaders: [] // add all common loaders here
  },
  resolve: {
    modulesDirectories: ['node_modules', 'bower_components'],
    extensions: ['', '.js', '.jsx']
  },
}
```


 - Create webpack.local.config.js
```javascript
var path = require("path")
var webpack = require('webpack')
var BundleTracker = require('webpack-bundle-tracker')

var config = require('./webpack.base.config.js')
var ip = 'localhost'
config.ip = ip


config.devtool = "#eval-source-map"

config.entry = {
  App1: [
    'webpack-dev-server/client?http://' + ip + ':3000',
    'webpack/hot/only-dev-server',
    './reactjs/App1',
  ],
  App2: [
    'webpack-dev-server/client?http://' + ip + ':3000',
    'webpack/hot/only-dev-server',
    './reactjs/App2',
  ],
  App3: [
    'webpack-dev-server/client?http://' + ip + ':3000',
    'webpack/hot/only-dev-server',
    './heatmap/static/js/App3',
  ],
}

config.output.publicPath = 'http://' + ip + ':3000' + '/assets/bundles/'

config.plugins = config.plugins.concat([
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NoErrorsPlugin(),
  new BundleTracker({filename: './webpack-stats-local.json'}),
  new webpack.DefinePlugin({
    'process.env': {
      'NODE_ENV': JSON.stringify('development'),
      'BASE_API_URL': JSON.stringify('https://'+ ip +':8000/api/v1/'),
  }}),
])

config.module.loaders.push(
  { test: /\.jsx?$/, exclude: /node_modules/, loaders: ['react-hot', 'babel'] }
)

module.exports = config
```

 - Create webpack.stage.config.js
```javascript
var webpack = require('webpack')
var BundleTracker = require('webpack-bundle-tracker')

var config = require('./webpack.base.config.js')

config.output.path = require('path').resolve('./project/assets/bundles/stage/')
config.output.publicPath = '/static/bundles/stage/'


config.plugins = config.plugins.concat([
  new BundleTracker({filename: './webpack-stats-stage.json'}),

  // removes a lot of debugging code in React
  new webpack.DefinePlugin({
    'process.env': {
      'NODE_ENV': JSON.stringify('staging'),
      'BASE_API_URL': JSON.stringify('https://sandbox.example.com/api/v1/'),
  }}),

  // keeps hashes consistent between compilations
  new webpack.optimize.OccurenceOrderPlugin(),

  // minifies your code
  new webpack.optimize.UglifyJsPlugin({
    compressor: {
      warnings: false
    }
  })
])

// Add a loader for JSX files
config.module.loaders.push(
  { test: /\.jsx?$/, exclude: /node_modules/, loader: 'babel' }
)

module.exports = config
```

- Create server.js
```javascript
var webpack = require('webpack')
var WebpackDevServer = require('webpack-dev-server')
var config = require('./webpack.local.config')

new WebpackDevServer(webpack(config), {
  publicPath: config.output.publicPath,
  hot: true,
  inline: true,
  historyApiFallback: true,
  headers: { "Access-Control-Allow-Origin": "*" }

}).listen(3000, config.ip, function (err, result) {
  if (err) {
    console.log(err)
  }

  console.log('Listening at ' + config.ip + ':3000')
})

```

 - Create .babelrc
```javascript
{
  "presets": ["es2015", "react", "stage-0"],
  "plugins": [
    ["transform-decorators-legacy", "react-hot-loader/babel"],
  ]
}
```

 - Create fabfile_.py
```python
from subprocess import call
cmd1 = 'rm -rf project/assets/bundles/local/*'.split(" ")
cmd2 = 'rm -rf project/assets/bundles/stage/*'.split(" ")
cmd4 = 'webpack --config webpack.local.config.js --progress --colors'.split(" ")
cmd5 = 'webpack --config webpack.stage.config.js --progress --colors'.split(" ")
cmd7 = 'python manage.py collectstatic --noinput'.split(" ")
call(cmd1)
call(cmd2)
call(cmd4)
call(cmd5)
call(cmd7)
```


- Use the react bundle like this

	##### index.html 
```html
{% extends "base.html" %}
{% load render_bundle from webpack_loader %}
{% block main %}
<div id="App1"></div>
{% render_bundle 'vendors' %}
{% render_bundle 'App1' %}
{% endblock %}

```	

	##### App1.jsx
```JavaScript
import React from "react"
import { render } from "react-dom"
import App1Container from "./containers/App1Container"
class App1 extends React.Component {
  render() {
    return (
      <App1Container />
    )
  }
}
render(<App1/>, document.getElementById('App1'))
```
	

 - For Development with hot reloading
	In `settings.py` use local WEBPACK_LOADER
```python
WEBPACK_LOADER = {
    	'DEFAULT': {
        	'BUNDLE_DIR_NAME': 'bundles/local/',  # end with slash
        	'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats-local.json'),
     }
}

```
	Run `python fabfiles_.py`

	Run `node server.js`

	Edit...save...edit..

 - For Production
 	In `settings.py` use stage WEBPACK_LOADER
```python
WEBPACK_LOADER = {
    	'DEFAULT': {
        	'BUNDLE_DIR_NAME': 'bundles/stage/',  # end with slash
        	'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats-stage.json'),
     }
}

```

	Run `python fabfiles_.py`, assets folder will be updated

	Commit changes

	Also commit `assets` folder

	Push changes


### In web-server

Pull changes from git

Run collectstatic

Restart server

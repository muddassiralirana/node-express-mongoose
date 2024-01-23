This wiki describes the architecture of the [node-express-mongoose](https://github.com/muddassiralirana/node-express-mongoose) boilerplate.

* [server.js](#serverjs)
* app
  * [models](#models)
  * [views](#views)
  * [controllers](#controllers)
  * mailers
* config
  * [routes](#routes)
  * express
  * passport
  * environment
  * using middlewares
  * using route middlewares
* tests
* [Applications built using this approach](https://github.com/muddassiralirana/node-express-mongoose/wiki/Apps-built-using-this-approach)

<a name="serverjs" />

## server.js

This is the main file which gets executed when you run `npm start`. It bootstraps models, configurations and routes.

<a name="models" />

## models

Simply drop a mongoose model file in the `app/models/` directory, make sure the schema is defined

```js
var mongoose = require('mongoose')
var Schema = mongoose.Schema

var UserSchema = new Schema({
  name: { type: String, default: '' },
  email: { type: String, default: '' },
  hashed_password: { type: String, default: '' },
  salt: { type: String, default: '' }
})
```

and is exposed using

```js
// after schema declaration
mongoose.model('User', UserSchema)
```

As you can see in the [user model](https://github.com/muddassiralirana/node-express-mongoose/blob/master/app/models/user.js) file, we are using [mongoose-user](https://github.com/muddassiralirana/mongoose-user) plugin - which does the validations and provides some generic statics like `.list()` and `.load()`.

Further down, you can use these list and load methods to write your own statics. For example

somewhere in your admin or controller

```js
User.deleted(function (err, users) {
  // do your stuff
  // `users` is an array of deleted users
})
```

in app/models/user.js, you define your static like this

```js
UserSchema.static({

  // Lists all the deleted users

  deleted: function (cb) {
    this.list({
      criteria: { deleted: true },
      sort: { name: 'desc' },
      limit: 20,
      populate: [{
        path: 'company', select: 'name', match: { deleted: false }
      }],
      select: 'name email'
    }, cb)
  }
})
```

<a name="views" />

## views

The views are located in `app/views/` directory. The boilerplate uses `swig` as the templating system. If you want to use [ejs](http://github.com/visionmedia/ejs) (also use [ejs-locals](https://github.com/RandomEtc/ejs-locals/)) or any other templating system, rename the `.html` files to `.ejs` or whatever extension the templating system provides.

The views directory contains

1. `includes/` - contains
  * `head.html` - meta information, scripts, stylesheets
  * `header.html` - contains the logo and the navigation links
  * `foot.html` - usually javascript
  * `footer.html` - footer links
2. `layouts/` - contains the layout files.
3. 404 and 500

We are using the [view-helpers](https://github.com/muddassiralirana/node-view-helpers) middleware, which provides some generic methods. If any request comes from a mobile device, then the app automatically tries to look for a `.mobile.html` file and renders it.

For example

```js
res.render('home/index', {
  title: 'Welcome to our app'
})
```

If the request is coming from a mobile device then, the the app tries to look for a `index.mobile.html`, if its there, it tries to render that, otherwise the `index.html` is rendered.

<a name="controllers" />

## controllers

Controllers are in the `app/controllers/` directory. A simple controller method

```js
exports.index = function (req, res) {
  Post.list({}, function (err, posts) {
    res.render('posts', {
      title: 'List of posts',
      posts: posts
    })
  })
}
```

Its always better to follow conventions. For CRUD stuff, I usually name the controller methods like

```js
// load a post
exports.load = function (req, res, next, id) {
  // set req.post
  next()
}

// list posts
// GET /posts
exports.index = function (req, res) {
  // render posts/index
}

// new post
// GET /posts/new
exports.new = function (req, res) {
  // render posts/new
}

// create post
// POST /posts
exports.create = function (req, res) {
  // render posts/new if err
  // redirect to posts/:id if created
}

// show post
// GET /posts/:id
exports.show = function (req, res) {
  // render posts/show
}

// edit post
// GET /posts/:id/edit
exports.edit = function (req, res) {
  // render posts/edit
}

// update post
// PUT /posts/:id
exports.update = function (req, res) {
  // render posts/edit if err
  // redirect to /posts/:id if updated
}

// delete post
// DEL /posts/:id
exports.destroy = function (req, res) {
  // redirect to /posts
}
```

<a name="routes" />

## routes

```js
// since NODE_PATH=./app/controllers is set, you don't have to use the
// whole path
var posts = require('posts')

// posts crud routes
app.get('/posts', posts.index)
app.get('/posts/new', posts.new)
app.post('/posts', posts.create)
app.get('/posts/:id', posts.show)
app.get('/posts/:id/edit', posts.edit)
app.put('/posts/:id', posts.update)
app.del('/posts/:id', posts.destroy)
```

<a name="express-config" />

## express config

<a name="passport-config" />

## passport config

<a name="environments" />

## environment config

<a name="app-middlewares" />

## using middlewares

<a name="route-middlewares" />

## using route middlewares

<a name="mailers" />

## mailers

<a name="tests" />

## tests
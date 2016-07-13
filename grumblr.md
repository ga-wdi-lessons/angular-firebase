# Grumblr and Firebase

Your task for the rest of the lesson is to integrate Firebase intro Grumblr. The functionality will be exactly the same -- we're just persisting data differently.

## Starter Code

We'll be starting where the `ui-router` class left off. Clone down the below code and switch to the appropriate branch...

```bash
$ git clone git@github.com:ga-wdi-exercises/grumblr_angular.git
$ cd grumblr_angular
$ git checkout -b grumblr-firebase 2.0.0
```

> If you already have this repo cloned down, you can just run the third line in the above instructions.

## Setup CDNs

Begin by including the Firebase and AngularFire CDNs in your main `index.html` file...

```html
<!-- index.html -->

<!DOCTYPE html>
<html data-ng-app="grumblr">
  <head>
    <title>Angular</title>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.15/angular.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-router/0.2.15/angular-ui-router.min.js"></script>
    <script src="https://www.gstatic.com/firebasejs/3.0.3/firebase.js"></script>
    <script src="https://cdn.firebase.com/libs/angularfire/2.0.0/angularfire.min.js"></script>

    <script src="js/app.js"></script>
    <script src="js/grumbles/grumbles.js"></script>
    <script src="js/grumbles/index.controller.js"></script>
  </head>
  <body>
    <h1>Grumblr</h1>
    <main data-ui-view></main>
  </body>
</html>
```

> You can get rid of the `ngResource` CDN. We don't need it today.

## Create Firebase DB

In order for our application to interact with a Firebase database, we need to provide it with the proper credentials. We can do this by visiting the Firebase Console -- [https://console.firebase.google.com/](https://console.firebase.google.com/) -- in the browser and creating a DB.

Click "Create New Project". Give it a name and then click "Create Project". From there you should see three options, each represented by a circle icon. Select the right-most one, `Add Firebase to your web app`.

You should now get a notification with a code snippet that looks something like this...

```js
<script>
  // Initialize Firebase
  var config = {
    apiKey: "API-KEY-GOES-HERE",
    authDomain: "AUTH-DOMAIN-GOES-HERE",
    databaseURL: "https://database-url-goes-here.firebaseio.com",
    storageBucket: "storage-bucket-goes-here.appspot.com",
  };
  firebase.initializeApp(config);
</script>
```

> You should see actual values for all the keys in the `config` object.

Go ahead and place that `<script>` directly in your HTML like so...

```html
<!-- index.html -->

<html lang="en" data-ng-app="sampleApp">
<head>
  <meta charset="UTF-8">
  <title>Firebase Practice</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.5/angular.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-router/0.2.15/angular-ui-router.min.js"></script>
  <script src="https://www.gstatic.com/firebasejs/3.0.3/firebase.js"></script>
  <script src="https://cdn.firebase.com/libs/angularfire/2.0.0/angularfire.min.js"></script>
  <script>
    // Initialize Firebase
    var config = {
      apiKey: "API-KEY-GOES-HERE",
      authDomain: "AUTH-DOMAIN-GOES-HERE",
      databaseURL: "https://database-url-goes-here.firebaseio.com",
      storageBucket: "storage-bucket-goes-here.appspot.com",
    };
    firebase.initializeApp(config);
  </script>
  <script src="app.js"></script>
</head>
```

## Update Firebase Permissions

Because we are not using Firebase's auth functionality in this example, we need to update our database permissions so that anybody can read or write do it. We can do this by returning to the page for our database in the Firebase Console and clicking the "Database" option in the left sidebar. Then, click the `Rules` tab under the `Realtime Database` header.

Once there, modify the JSON you see so that it looks like this...

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

When that's done, click "Publish".

> You'll get a warning message about your database being public. You can "Dismiss" it.

## Import JSON

Let's add some data to our DB so that there's something to read. Firebase allows us to seed the database by importing a JSON file.

Create a `data.json` file containing the following...

```json
[
  {
    "authorName": "Jesse",
    "content": "There is no reason at all to believe that White Wine is any different to water when it comes to removing Red Wine stains.",
    "title": "10 skateboard tricks writing rails apps",
    "photoUrl": "https://splashbase.s3.amazonaws.com/splitshire/regular/SplitShire-85962-384x253.jpg"
  },
  {
    "authorName": "Adam",
    "content": "It's wrong to be right.",
    "title": "32 panda bears taking selfies",
    "photoUrl": "https://splashbase.s3.amazonaws.com/unsplash/regular/tumblr_n6es0tRk5w1st5lhmo1_1280.jpg"
  },
  {
    "authorName": "Andy",
    "content": "It always seems impossible, until it's done.",
    "title": "24 shocking T-shirts",
    "photoUrl": "https://splashbase.s3.amazonaws.com/lifeofpix/regular/Life-of-Pix-free-stock-photos-ocean-wood-orange-geoffroy-1440x960.jpg"
  }
]
```

Then go back to the `Data` tab under `Realtime Database`. Click the three vertical buttons towards the right of the screen and you should see an option to "Import JSON". Click that and upload `data.json`. If successful, your DB should now be populated with some data.

## Inject Firebase

Let's inject firebase as a dependency into our `grumbles` sub-module so we can use it throughout our application.

```js
// js/grumbles/grumbles.js

"use strict";

(function(){
  angular
  .module("grumbles", [
    "firebase"
  ]);
}());
```

## Inject $firebaseArray

Now we can inject the services we need to interact with our database -- in this case, just `$firebaseArray` -- into the controller. Let's begin with `index.controller.js`...

```js
// js/grumbles/index.controller.js

angular
  .module("grumbles")
  .controller("GrumbleIndexController", [
    "$firebaseArray",
    GrumbleIndexControllerFunction
  ]);

function GrumbleIndexControllerFunction($firebaseArray){
  this.grumbles = [] // Delete the hard-coded grumbles. We'll replace this in a bit.
}
```

## Synchronize with Database / Index

Next let's get those grumbles stored in our database to appear on the page. That means we need to establish a link between our controller and the database. We'll do that by defining a reference to the "grumbles" section of our database. We will then generate a `$firebaseArray` using that reference.

```js
// js/grumbles/index.controller.js

angular
  .module("grumbles")
  .controller("GrumbleIndexController", [
    "$firebaseArray",
    GrumbleIndexControllerFunction
  ]);

function GrumbleIndexControllerFunction($firebaseArray){
  var vm = this;
  var ref = firebase.database().ref().child("grumbles");
  vm.grumbles = $firebaseArray(ref);
}
```

> Let's also store a reference to `this` in a `vm` variable so we don't have to worry about any scope issues moving forward.

When you visit `http://localhost:3000/#/grumbles`, you should now see a list of all the Grumbles in the database.

## New

### View

Let's give the user the ability to create a new Grumble. We'll add this functionality to our index (i.e., we will not be creating a separate new state). Let's go ahead and add a form to our index view...

```html
<h2>These are all the Grumbles</h2>

<div data-ng-repeat="grumble in GrumbleIndexViewModel.grumbles">
  <p>{{grumble.title}}</p>
</div>

<h2>Create Grumble</h2>

<form data-ng-submit="GrumbleIndexViewModel.create()">
  <input placeholder="Title" data-ng-model="GrumbleIndexViewModel.newGrumble.title">
  <input placeholder="Author" data-ng-model="GrumbleIndexViewModel.newGrumble.authorName">
  <input placeholder="Content" data-ng-model="GrumbleIndexViewModel.newGrumble.content">
  <input placeholder="Photo URL" data-ng-model="GrumbleIndexViewModel.newGrumble.photoUrl">
  <button type="submit">Create Grumble</button>
</form>
```

> What's `newGrumble`? It's an empty object that we're initializing here and will be using as a staging area for new Grumbles that will be sent to the database.

Note that we are adding a `ng-submit` directive to the form. Whenever the form is submitted, it will trigger a `.create` method that will be defined in our controller.

### Controller

Let's add that `.create` method to our index controller and fill it with some code that will add the new Grumble to our Firebase DB.

```js
// /js/grumbles/index.controller.js

angular
  .module("grumbles")
  .controller("GrumbleIndexController", [
    "$firebaseArray",
    GrumbleIndexControllerFunction
  ]);

function GrumbleIndexControllerFunction($firebaseArray){
  var vm = this;
  var ref = firebase.database().ref().child("grumbles");
  vm.grumbles = $firebaseArray(ref);
  vm.create = function(){
    vm.grumbles.$save(vm.newGrumble);
  }
}
```

> Remember, we initialized `newGrumble` in our view so you won't see any mention of `vm.newGrumble = ...` in the controller.

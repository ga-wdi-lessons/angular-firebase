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

Now let's import this JSON into our database. We want to import it to a section of our DB dedicated to Grumbles. We can do this by adding the word "grumbles" to the end of our DB's root URL. It would look something like this...

```
https://console.firebase.google.com/project/grumblr-47bcc/database/data/grumbles
```

> `data` is the root directory of our database. `grumbles` is where we want to store our Grumbles.

By visiting this URL, we are actually creating a "reference" to "grumbles" (i.e., creating a "grumbles" section in our database).

On the resulting page, click the `Data` tab under `Realtime Database`. Then click the three vertical buttons towards the right of the screen and you should see an option to "Import JSON". Click that and upload `data.json`. If successful, your DB should now be populated with some data.

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

When you visit `http://localhost:8080/#/grumbles`, you should now see a list of all the Grumbles in the database.

## New

### View

Let's give the user the ability to create a new Grumble. We'll add this functionality to our index (i.e., we will not be creating a separate new state). Let's go ahead and add a form to our index view...

```html
<!-- js/grumbles/index.html -->

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

  // This method is triggered whenever the user clicks "Create Grumble".
  vm.create = function(){
    vm.grumbles.$add(vm.newGrumble).then(function(){
      vm.newGrumble = {}; // Once the Grumble has been created, clear the contents of vm.newGrumble.
    });
  }
}
```

> Remember, we initialized `newGrumble` in our view so you won't see any mention of `vm.newGrumble = ...` in the controller.

## Delete

Before we move onto our show state, let's add some delete functionality to our index page.

### View

Each grumble should have a delete button next to it. When it is clicked, it should trigger a delete method defined in our controller that removes the Grumble from our view and database. The method takes the Grumble in question as an argument.

```html
<!-- js/grumbles/index.html -->

<h2>These are all the Grumbles</h2>

<div data-ng-repeat="grumble in GrumbleIndexViewModel.grumbles">
  <p>{{grumble.title}}</p>
  <button data-ng-click="GrumbleIndexViewModel.delete(grumble)">Delete Grumble</button>
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

### Controller

Now let's define that `delete` method...

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
    vm.grumbles.$add(vm.newGrumble).then(function(){
      vm.newGrumble = {};
    });
  }

  // This method is triggered whenever the user clicks "Delete Grumble".
  // It takes a grumble as an argument.
  vm.delete = function(grumble){
    vm.grumbles.$remove(grumble)
  }
}
```

## Show

### Add Show State

Onto show. First, let's round out the show state in `app.js` by adding `controller` and `controllerAs` values...

```js
// app.js

angular
  .module("grumblr", [
    "ui.router",
    "grumbles"
  ])
  .config([
    "$stateProvider",
    RouterFunction
  ]);

  function RouterFunction($stateProvider){
    $stateProvider
    .state("grumbleIndex", {
      url: "/grumbles",
      templateUrl: "js/grumbles/index.html",
      controller: "GrumbleIndexController",
      controllerAs: "GrumbleIndexViewModel"
    })
    .state("grumbleShow", {
      url: "/grumbles/:id",
      templateUrl: "js/grumbles/show.html",
      controller: "GrumbleShowController",
      controllerAs: "GrumbleShowViewModel"
    });
  }
```

### Update `js/grumbles/index.html`

Let's update each Grumble in our index so that it is a link to its respective show page. The important thing to note is that, since we're using Firebase, we need to write out `$id` when accessing the id of a given Grumble.

```html
<!-- js/grumbles/index.html -->

<h2>These are all the Grumbles</h2>

<div data-ng-repeat="grumble in GrumbleIndexViewModel.grumbles">
  <a data-ui-sref="grumbleShow({id: grumble.$id})">{{grumble.title}}</a>
  <button data-ng-click="GrumbleIndexViewModel.delete(grumble)">Delete Grumble</button>
</div>
```

### Controller

Now create a `GrumbleShowController` to correspond with the show state.

```bash
touch js/grumbles/show.controller.js
```

Don't forget to link to this new controller in `index.html`...

```html
<!-- index.html -->

<script src="js/app.js"></script>
<script src="js/grumbles/grumbles.js"></script>
<script src="js/grumbles/index.controller.js"></script>
<script src="js/grumbles/show.controller.js"></script>
```

Time to add some content to `show.controller.js`, including a `vm.grumbles` Firebase array identical to the one in the index controller...

```js
// js/grumbles/show.controller.js

"use strict";

(function(){
  angular
    .module("grumbles")
    .controller("GrumbleShowController", [
      "$stateParams",
      "$firebaseArray",
      GrumbleShowControllerFunction
    ])

  function GrumbleShowControllerFunction($stateParams, $firebaseArray){
    var vm = this;
    var ref = firebase.database().ref().child("grumbles");
    vm.grumbles = $firebaseArray(ref);
  }
})();
```

Next we need to define the individual grumble -- `vm.grumble` -- that we want to display on the page. In order to do this, we will need to define `vm.grumble` inside of a promise method.

```js
// js/grumbles/show.controller.js

function GrumbleShowControllerFunction($stateParams, $firebaseArray){
  var vm = this;

  // This time, ref contains a reference to a specific grumble.
  var ref = firebase.database().ref().child("grumbles/" + $stateParams.id);

  // Then we retrieve a $firebaseObject based on ref. Once that asynchronous action is done, we save the resulting grumble to `vm.grumble`.
  $firebaseObject(ref).$loaded().then(function(grumble){
    vm.grumble = grumble
  });
}
```

> **`.$loaded`** - We can only chain `.then()` to `$firebaseArray(ref)` if we place `$loaded()` between them. `$loaded()` returns a promise once `$firebaseArray(ref)` is done pulling all the Grumbles from the database.
>
> **`.$getRecord`** - This method allows us to retrieve a particular item in a Firebase array. `$.getRecord` takes a key (in this case, the id stored in `$stateParams.id`) as an argument.

### View

Let's update `show.html` so that we can view this data in the browser...

```html
<!-- js/grumbles/show.html -->

<h2>This is a Grumble</h2>

<p>{{GrumbleShowViewModel.grumble.title}}</p>
<p>BY: {{GrumbleShowViewModel.grumble.authorName}}</p>
<p>{{GrumbleShowViewModel.grumble.content}}</p>
<img data-ng-src="{{GrumbleShowViewModel.grumble.photoUrl}}">
```

## Edit

We're almost there! Last order of business is to add edit functionality. The user should be able to edit a post via a form on a Grumble's show page.

### View

Let's add a form to `show.html`. When submitted, it should trigger a yet-to-be-defined `.update` method in the show controller...

```html
<!-- js/grumbles/show.html -->

<h2>This is a Grumble</h2>

<p>{{GrumbleShowViewModel.grumble.title}}</p>
<p>BY: {{GrumbleShowViewModel.grumble.authorName}}</p>
<p>{{GrumbleShowViewModel.grumble.content}}</p>
<img data-ng-src="{{GrumbleShowViewModel.grumble.photoUrl}}">

<h2>Edit Grumble</h2>

<form data-ng-submit="GrumbleShowViewModel.update()">
  <input placeholder="Title" data-ng-model="GrumbleShowViewModel.grumble.title">
  <input placeholder="Author" data-ng-model="GrumbleShowViewModel.grumble.authorName">
  <input placeholder="Content" data-ng-model="GrumbleShowViewModel.grumble.content">
  <input placeholder="Photo URL" data-ng-model="GrumbleShowViewModel.grumble.photoUrl">
  <button type="submit">Update Grumble</button>
</form>
```

### Controller

Now define an `.update` method in the show controller. This will make use of some terminology we haven't seen yet...

```js
// js/grumbles/show.controller.js

"use strict";

(function(){
  angular
    .module("grumbles")
    .controller("GrumbleShowController", [
      "$stateParams",
      "$firebaseObject",  // We are now using $firebaseObject, not $firebaseArray.
      GrumbleShowControllerFunction
    ]);

  function GrumbleShowControllerFunction($stateParams, $firebaseObject){
    var vm = this;

    var ref = firebase.database().ref().child("grumbles/" + $stateParams.id);

    $firebaseObject(ref).$loaded().then(function(grumble){
      vm.grumble = grumble
    });

    // This method is triggered when the user clicks "Update Grumble".
    vm.update = function(){
      vm.grumble.$save();
    }
  }
})();
```

## Bonuses

Congrats - you've created a version of Grumblr with full Firebase CRUD functionality! If you finish this lab early, we've included some bonuses below for you to try out.

### Deploy Grumblr Using Firebase Hosting

As we mentioned earlier, Firebase is a PaaS, meaning it offers more than just a realtime database. You can also use Firebase to deploy your application. Get started by looking through [this guide](https://firebase.google.com/docs/hosting/quickstart). This will require installing the Firebase CLI Tools.

> You can also use the Firebase CLI Tools to spin up a server like you have been doing with `http-server`. Give this a shot if you're interesting in learning a bit more about Firebase!

### Factories & Services

Create a factory or service that handles all of the Firebase database synchronization for you. You should be able to inject this factory/service into a controller and call methods that allow you execute CRUD functionality on Grumbles from within the controller.

> Start simple and create a factory or service that just returns the `ref` to the database. From there, think about how you could export a `$firebaseArray`, `$firebaseObject` or even the methods that we use to interact with a Firebase DB (e.g., `$add`, $`save`).

### Styling

Grumblr looks pretty boring. Take a break from Angular and Javascript and make it look a little better!

### If You're Sick of Grumblr...

Add Firebase to an app of your own. It can be a previous lab, project, homework assignment or something else entirely!

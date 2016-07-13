# Todo App

> Do not code along. Just sit back and watch. You'll have an opportunity to give Firebase a spin in a little bit.

We'll take a first dive into Firebase by creating a simple todo app. The model here will be a simple one: a todo with a `content` attribute.

## Bootstrap an Angular Application

Let's start by creating an application app with a single module and controller.

```js
// app.js

"use strict";

(function(){
  angular
    .module("todoApp", [])
    .controller("TodoController", [
      TodoControllerFunction
    ]);

  function TodoControllerFunction(){

  }
})
```

Next let's create a corresponding `index.html` file. Note that, along with Angular, we have linked to two new CDNs: Firebase and AngularFire.

```html
<!-- index.html -->

<html lang="en" data-ng-app="sampleApp">
<head>
  <meta charset="UTF-8">
  <title>Firebase Practice</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.5/angular.min.js"></script>
  <script src="https://www.gstatic.com/firebasejs/3.0.3/firebase.js"></script>
  <script src="https://cdn.firebase.com/libs/angularfire/2.0.0/angularfire.min.js"></script>
  <script src="app.js"></script>
</head>
<body ng-controller="TodoController as TodoControllerViewModel">
  <h1>Todos</h1>
</body>
</html>
```

## Add Firebase Credentials

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
    "text": "Gym"
  },
  {
    "text": "Tan"
  },
  {
    "text": "Laundry"
  },
  {
    "text": "Repeat"
  }
]
```

Now let's import this JSON into our database. We want to import it to a section of our DB dedicated to Todos. We can do this by adding the word "todos" to the end of our DB's root URL. It would look something like this...

```
https://console.firebase.google.com/project/todoapp-47bcc/database/data/todos
```

> `data` is the root directory of our database. `todos` is where we want to store our Todos.

By visiting this URL, we are actually creating a "reference" to "todos" (i.e., creating a "todos" section in our database).

On the resulting page, click the `Data` tab under `Realtime Database`. Then click the three vertical buttons towards the right of the screen and you should see an option to "Import JSON". Click that and upload `data.json`. If successful, your DB should now be populated with some data.

## Inject Firebase and AngularFire Dependencies

Let's start using Firebase and AngularFire in our app. We'll start by injecting the proper dependencies into our module...

```js
angular
  .module("todoApp", ["firebase"])
  .controller("TodoController", [
    SampleControllerFunction
  ])
```

> **`firebase`** - We now have access to this module because we linked to the Firebase CDN.

...and then our controller...

```js
angular
  .module("todoApp", ["firebase"])
  .controller("TodoController", [
    "$firebaseArray",
    TodoControllerFunction
  ])

function TodoControllerFunction($firebaseArray){

}
```

> **`$firebaseArray`** - This method creates a synchronized array, binding a collection of data in a Firebase DB to a collection in an Angular application. So think of it as a form of two-way binding between Angular and the DB.

## Synchronize Todos with Database

Just like the rest of the Angular apps we've made in class, we're going to save data to a value in our controller, like so...

```js
angular
  .module("todoApp", ["firebase"])
  .controller("TodoController", [
    "$firebaseArray",
    TodoControllerFunction
  ])

function TodoControllerFunction($firebaseArray){
  this.todos = [] // Collection of todos goes here.
}
```

The plan now is to devote a section of our Firebase DB to storing todos. We'll create this section by adding the below code to our `TodoControllerFunction`...

```js
function TodoControllerFunction($firebaseArray){
  var ref = firebase.database().ref().child("todos");
}
```

> **`firebase.database()`** - Indicates that we're accessing the database portion of the Firebase library. Remember, Firebase offers more than just database services.
>
> **`.ref().`** - Stands for "reference." Everything in a Firebase DB is a reference, whether it's a collection of key-value pairs or just a single key-value pair. When we write out `.ref()` with no argument, it means we are accessing the root directory of our database.
>
> **`child("todos")`** - This method drills down to a particular reference in our Firebase DB. In this case, we are accessing the "todos" reference.

Next we want to set `this.grumbles` to the content of the `todos` section of our Firebase DB.

```js
function TodoControllerFunction($firebaseArray){
  vm = this;
  var ref = firebase.database().ref().child("todos");
  vm.todos = $firebaseArray(ref);
}
```

> **`$firebaseArray(ref)`** - This generates a synchronized array from whatever is passed in as an argument. In this case, it is the "todos" reference in our Firebase DB.

## Add Read, Create and Update Functionality

We can't test our code quite yet because there's nothing in the Firebase DB. Let's give our app the ability to read, create and update todos so we can see Firebase in action...

```html
<h1>Todos</h1>

<div ng-repeat="todo in vm.todos">
  <input type="text" data-ng-model="todo.text" data-ng-change="vm.update(todo)">
</div>
<br/>

<form data-ng-submit="vm.addTodo()">
  <input data-ng-model="vm.newTodoText">
  <button type="submit">Add Todo</button>
</form>
```

```js
function TodoControllerFunction($firebaseArray){
  var vm = this;
  var ref = firebase.database().ref().child("todos");
  vm.todos = $firebaseArray(ref);

  // This is triggered whenever we click on the "Add Todo" button.
  vm.addTodo = function(){
    vm.todos.$add({
      text: vm.newTodoText
    }).then(function(){
      // After we create a new todo, clear the "Add Todo" input field.
      vm.newTodoText = "";
    })
  }

  // This is triggered whenever the content of the input field changes.
  vm.update = function(todo){
    vm.todos.$save(todo);
  }
}
```

> **`$add`** - The AngularFire method used to create something in a Firebase DB.
>
> **`$save`** - The AngularFire method used to update something in a Firebase DB.

## Add Delete Functionality

Let's make it so that each todo has a delete button right next to it. Whenever that button is clicked, it triggers a `delete` method defined in our controller...

```html
<h1>Todos</h1>

<div ng-repeat="todo in vm.todos">
  <input type="text" data-ng-model="todo.text" data-ng-change="vm.update(todo)">
  <button data-ng-click="vm.delete(todo)">Delete Todo</button>
</div>
<br/>

<form data-ng-submit="vm.addTodo()">
  <input data-ng-model="vm.newTodoText">
  <button type="submit">Add Todo</button>
</form>
```

That delete method will remove the todo from the Firebase Array, thus removing it from both the view and the database...

```js
function TodoControllerFunction($firebaseArray){
  var vm = this;
  var ref = firebase.database().ref().child("todos");
  vm.todos = $firebaseArray(ref);

  vm.addTodo = function(){
    vm.todos.$add({
      text: vm.newTodoText
    }).then(function(){
      vm.newTodoText = "";
    })
  }

  vm.update = function(todo){
    vm.todos.$save(todo);
  }

  // This is triggered whenever the delete button is clicked.
  vm.delete = function(todo){
    vm.todos.$remove(todo);
  }
}
```

> **`.$remove`** - The AngularFire method used to delete something from a Firebase DB.

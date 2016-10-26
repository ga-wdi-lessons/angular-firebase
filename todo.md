# Todo App

We'll take a first dive into Firebase by creating a simple todo app. The model here will be a simple one: a todo with a `text` attribute.

## Bootstrap an Angular Application

Let's start by creating an application app with a single module and controller.

```js
// app.js
  angular
    .module("todoApp", [])
    .controller("TodoController", [
      TodoControllerFunction
    ])

  function TodoControllerFunction(){

  }
```

Next let's create a corresponding `index.html` file. Note that, along with Angular, we have linked to two new CDNs: Firebase and AngularFire.

```html
<!-- index.html -->

<html lang="en" data-ng-app="todoApp">
<head>
  <meta charset="UTF-8">
  <title>Todo App</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.8/angular.min.js"></script>
  <script src="https://www.gstatic.com/firebasejs/3.4.1/firebase.js"></script>
  <script src="https://cdn.firebase.com/libs/angularfire/2.1.0/angularfire.min.js"></script>
  <script src="app.js"></script>
</head>
<body ng-controller="TodoController as vm">
  <h1>Todos</h1>
</body>
</html>
```

## Add Firebase Credentials

In order for our application to interact with a Firebase database, we need to provide it with the proper credentials. We can do this by visiting the Firebase Console -- [https://console.firebase.google.com/](https://console.firebase.google.com/) -- in the browser and creating a DB.

Click `"Create New Project"`. Give it a name and then click `"Create Project"`. From there you should see three options, each represented by a circle icon. Select the right-most one, `Add Firebase to your web app`.

You should now get a notification with a code snippet that looks something like this...

```html
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
<html lang="en" data-ng-app="todoApp">
<head>
  <meta charset="UTF-8">
  <title>Todo App</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.8/angular.min.js"></script>
  <script src="https://www.gstatic.com/firebasejs/3.4.1/firebase.js"></script>
  <script src="https://cdn.firebase.com/libs/angularfire/2.1.0/angularfire.min.js"></script>
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

> **Note**: no need to worry about obfuscating these keys/urls as Firebase requires authentication for each access via the GUI console.

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

> **Note**: You'll get a warning message about your database being public. You can "Dismiss" it.

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

By visiting this URL, we are actually creating a "reference" to "todos" (i.e., creating a "todos" section in our database). Firebase saves each piece of data as a reference to a real time url.

On the resulting page, click the `Data` tab under `Realtime Database`. Then click the three vertical buttons towards the right of the screen and you should see an option to "Import JSON". Click that and upload `data.json`. If successful, your DB should now be populated with some data.

## Inject Firebase and AngularFire Dependencies

Let's start using Firebase and AngularFire in our app. We'll start by injecting the proper dependencies into our module...

```js
angular
  .module("todoApp", ["firebase"])
  .controller("TodoController", [
    TodoControllerFunction
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

## Synchronize Todos with Database / Index

Just like the rest of the Angular apps we've made in class, we're going to save data to a value in our controller...

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

 Now we just need to tell our app where it can find the data - the section of our Firebase DB storing todos. We'll connect to our database by adding the below code to our `TodoControllerFunction`...

```js
function TodoControllerFunction($firebaseArray){
  let ref = firebase.database().ref().child("todos");
}
```

> **`firebase.database()`** - Indicates that we're accessing the database portion of the Firebase library. Remember, Firebase offers more than just database services.
>
> **`.ref().`** - Stands for "reference." Everything in a Firebase DB is a reference, whether it's a collection of key-value pairs or just a single key-value pair. When we write out `.ref()` with no argument, it means we are accessing the **root** node of our database.
>
> **`child("todos")`** - This method drills down to a particular reference in our Firebase DB. In this case, we are accessing the "todos" reference.

Great, now that we have reference to where our todos our stored, we need to make them into objects that we can use in our app and that are saved in real time in our database. To do this, we are going to use the `$firebaseArray` method that we injected earlier into this controller/

```js
function TodoControllerFunction($firebaseArray){
  let ref = firebase.database().ref().child("todos");
  this.todos = $firebaseArray(ref);
}
```

> **`$firebaseArray(ref)`** - This generates a synchronized array from whatever is passed in as an argument. In this case, it is the "todos" reference in our Firebase DB.

So that we can see these todos in the browser, let's add an `ng-repeat` directive to our index view...

```html
<h1>Todos</h1>

<div ng-repeat="todo in vm.todos">
  <input type="text" ng-model="todo.text">
</div>
```

## New

Let's start by giving the user the ability to create todos and add them to the database. We'll begin by adding a form to the index view...

```html
<h1>Todos</h1>

<div ng-repeat="todo in vm.todos">
  <input type="text" ng-model="todo.text">
</div>
<br>
<!-- .addTodo() will be triggered whenever a user clicks the submit button or hits enter on the input field. -->
  <form data-ng-submit="vm.create()">
    <input type="text" data-ng-model="vm.newTodo.text">
    <button type="submit">Add Todo</button>
  </form>
```

> `vm.newTodo.text` is a value that we are defining in our view. It acts as a staging area for the text content of a new todo.

```js
function TodoControllerFunction($firebaseArray){
  var ref = firebase.database().ref().child("todos");
  this.todos = $firebaseArray(ref);
  this.newTodo = {}

  this.addTodo = function(){
    // After we create a new todo, clear the "New Todo" input field.
    this.todos.$add(this.newTodo).then( _ => this.newTodo = {} )
  }
}
```

> **`$add`** - The AngularFire method used to create something in a Firebase DB.
>
> Since `$add` is asynchronous, after it completes, we then can reinitialize the value bound to the input field.

## Edit

Next up: edit functionality. Whenever a user makes a change to the input field inside of `ng-repeat`, we want that change to be immediately reflected in the database. Let's implement that by adding a `ng-change` directive to that input. It will trigger an `update` method we will define in our controller.

```html
<h1>Todos</h1>

<div ng-repeat="todo in vm.todos">
  <input type="text" data-ng-model="todo.text" data-ng-change="vm.update(todo)">
</div>
<br>

<form data-ng-submit="vm.create()">
  <input data-ng-model="vm.newTodoText">
  <button type="submit">Add Todo</button>
</form>
```

Now let's define `.update` in our controller. It will make use of AngularFire's `$save` method.

```js
function TodoControllerFunction($firebaseArray){
  var ref = firebase.database().ref().child("todos");
  this.todos = $firebaseArray(ref);
  this.newTodo = {}

  this.create = function(){
    // After we create a new todo, clear the "New Todo" input field.
    this.todos.$add(this.newTodo).then( _ => this.newTodo = {} )
  }

  // This is triggered whenever the content of the input field changes.
  this.update = function (todo) {
    this.todos.$save(todo)
  }
}
```

> **`$save`** - The AngularFire method used to update something in a Firebase DB in realtime.

## Delete

Let's make it so that each todo has a delete button right next to it. Whenever that button is clicked, it triggers a `delete` method defined in our controller...

```html
<h1>Todos</h1>

<div ng-repeat="todo in vm.todos">
  <input type="text" data-ng-model="todo.text" data-ng-change="vm.update(todo)">
  <button data-ng-click="vm.delete(todo)">Delete Todo</button>
</div>
<br>

<form data-ng-submit="vm.create()">
  <input data-ng-model="vm.newTodoText">
  <button type="submit">Add Todo</button>
</form>
```

That delete method will remove the todo from the Firebase Array, thus removing it from both the view and the database...

```js
function TodoControllerFunction($firebaseArray){
  var ref = firebase.database().ref().child("todos");
  this.todos = $firebaseArray(ref);
  this.newTodo = {}

  this.create = function(){
    // After we create a new todo, clear the "New Todo" input field.
    this.todos.$add(this.newTodo).then( _ => this.newTodo = {} )
  }

  // This is triggered whenever the content of the input field changes.
  this.update = function (todo) {
    this.todos.$save(todo)
  }

  // This is triggered whenever the delete button is clicked.
  this.delete = function(todo){
    this.todos.$remove(todo);
  }
}
```

> **`.$remove`** - The AngularFire method used to delete something from a Firebase DB.

Great, we now have full CRUD functionality for our todo app that is using three-way data-binding to persist data in real time!

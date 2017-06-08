# bind-this-basics
Getting to know it better, by writing about it with examples.

#### I've known "this" and "bind" along with "call/apply" for quite some time now and I thought I understood the concept properly since I've been creating projects from scratch that depended on these being used. Then I was asked to explain it along with context (or function scope) in JavaScript and I couldn't even do that intelligibly. So here I am, re-learning the basics and trying to get the most of it by writing about every little step to remember and understand it better.

Let's start with the definition. "this" keyword is used to point to the current context, i.e where the "this" is in the code, so that f.e. we can reuse this code within different context to get different results. By context I mean the place where the function has been invoked, i.e in which object instance. A simple example would be creating a function that introduces specific person:
```
"use strict";

var name = "Matt";

function sayHello() {
  var output = "Hello, I'm " + name;
  console.log(output);
}

sayHello(); // Hello, I'm Matt
```
It works just fine logging the proper greeting. Now, I'd like to make the function work in more than one context. I'll use a constructor function and make a couple of instances of it with different names assinged to it's own variable "name".
```
"use strict";

var name = "Matt";

function sayHello() {
  var output = "Hello, I'm " + name;
  console.log(output);
}

function Person(personsName) {
 this.name = personsName;
 var doubleName = personsName + personsName;
}

var bert = new Person("Bert");
var ernie = new Person("Ernie");

console.log(name); // Matt
console.log(bert.name, bert.doubleName); // Bert, undefined
console.log(ernie.name, ernie.doubleName);  // Ernie, undefined

sayHello(); // Hello, I'm Matt
```
The Person constructor function that I added already uses the "this" keyword. Thanks to "this" the names that we pass as a parameter with ```new Person()``` are stored as keys inside an object and are available for each individual instance of this constructor. The typical declaration with "var" is treated as a private value and can be only accessed by methods defined in the constructor. You can see that because the normal singular name is logged, but the doubleName is undefined. Now, the sayHello function still outputs Matt as it's always referring to name in the global scope. I'll edit that now, so that it uses name key based on where the function has been invoked in.
```
function sayHello() {
  var output = "Hello, I'm " + this.name;
  console.log(output);
}
```
The only difference is the "this" key. Now when we invoke the function sayHello it will give us an error.
```Uncaught TypeError: Cannot read property 'name' of undefined```
That happens thanks to "use strict" context declaration, where we are forbidden from using the global window object scope by mistake, f.e. by declaring a variable without "var" or trying to access global variables, where specific context is expected as indicated by "this" keyword. Without the "use strict" declaration we would still get the usual log ```Hello, I'm Matt``` since the function was invoked in the window object scope. I'll do another example to make sure:
```
"use strict";

var name = "Matt";

function sayHello() {
  var output = "Hello, I'm " + this.name;
  console.log(output);
}

function Person(personsName) {
 this.name = personsName;
 var doubleName = personsName + personsName;
}

var bert = new Person("Bert");
var ernie = new Person("Ernie");

console.log(name);
console.log(bert.name, bert.doubleName);
console.log(ernie.name, ernie.doubleName);

var newObjectScope = {
  name: "Thomas",
  greet: sayHello
}

newObjectScope.greet(); // Hello, I'm Thomas
```
We're back to strict mode. Well, seems like I'm getting it so far, since invoking same function within a new object scope changed the output of the same function as seen on the last line. Now the context is the newObjectScope and it's name value equals "Thomas", therefore the log will say ```Hello, I'm Thomas``` Before we get to "bind" I'll just make sure we can really access the typical "var" declarations of our object instances by invoking an internal method that logs the doubleName.
```
"use strict";

var name = "Matt";

function sayHello() {
  var output = "Hello, I'm " + this.name;
  console.log(output);
}

function Person(personsName) {
 this.name = personsName;
 var doubleName = personsName + personsName;

 this.greet = function() {
   console.log(doubleName);
 }
}

var bert = new Person("Bert");
var ernie = new Person("Ernie");

bert.greet(); // BertBert
```
Even though I made the variable inaccesible from outside the object, I could invoke a method that has access to all variables inside this object. Now, the final explanation of usage of this subject that I'm aware of: bind vs call/apply. The difference between bind and call/apply is that bind changes the context permanently. I'll try to go through it with previous example with newObjectScope object literal, which creates it's own scope and provides the name of its own that is "Thomas".
```
var bertHello = sayHello.bind(bert);

var newObjectScope = {
  name: "Thomas",
  greet: sayHello,
  bertGreet: bertHello
}

newObjectScope.greet(); // Hello, I'm Thomas
newObjectScope.bertGreet(); // Hello, I'm Bert
```
First, I declared a new variable and since we can assign functions to variables, which essentially makes them a function I assigned sayHello function to it with a little twist. Thanks to the bind function with bert as previously created object it will be bound to, bertHello will always use "this" keyword inside the bert object. Since bert's name is Bert it will log ```Hello, I'm Bert``` even though it's been invoked inside newObjectScope object scope. Thanks to this I can create specific functions built on top of generic ones.

Now, the call/apply functions change the target function's context and invoke it immidiately. From what I understand it can't be assigned to a variable as it was already invoked and then if we try to invoke it with the variable's name and it outputs an error ```Uncaught TypeError: newThing is not a function``` . Therefore it can't be used as a more specific function multiple times at a specific time in execution order since it's not a permanent modification to the target function. But we can still achieve the same effect just once like so:
```
sayHello.call(bert); // Hello, I'm Bert
sayHello.apply(ernie); // Hello, I'm Ernie
```
Basically, the same as bind, BUT they were invoked immidiately and we have to use call/apply every single time we'd like to do the same thing. For repetitive use I prefer bind then. I know of a second difference between bind and call/apply and it's that call/apply can also take arguments to immidiately apply them to the specified object. Call, takes a specified amount of arguments separated by commas. Apply takes one array of arguments that gets applied to the specified object. Here are some examples:
```
function createRobot(type, strength) {
  this.type = type;
  this.strength = strength;
  console.log(
    "You've created a "
    + this.type
    + " named "
    + this.name
    + " , which is as strong as a "
    + this.strength
  )
}

createRobot.callbert, "cyborg", "banana"); // You've created a cyborg named Bert , which is as strong as a banana
createRobot.call(ernie, "terminator", "tree"); // You've created a terminator named Ernie , which is as strong as a tree
```
In the example above I've created a new function createRobot which takes two arguments: name and strength. First it adds a this.strength to the specified object and assigns the strength argument to it, and then it logs the message with the specified objects name and strength. With apply it would look like this:
```
createRobot.apply(bert, ["cyborg", "banana"]);
createRobot.apply(ernie, ["terminator", "tree"]);
```
So it turned out that bind can take arguments too. Now, this function could look like this with bind:
```
function createRobot(type, strength) {
  this.type = type;
  this.strength = strength;
  console.log(
    "You've created a "
    + this.type
    + " named "
    + this.name
    + " , which is as strong as a "
    + this.strength
  )
}

var createFlimsyBertRobot = createRobot.bind(bert, "cyborg", "banana");
var createStrongErnieRobot = createRobot.bind(ernie, "terminator", "tree");

createFlimsyBertRobot();
createStrongErnieRobot();
```
Going back then, the only difference between those that I can see is that bind is permanent and can be assigned to new variables as a more specific function and call/apply invoke the function immidiately changing the context just this once, when they are invoked.

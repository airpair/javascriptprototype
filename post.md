## Who this post is for.
This post is for everyone who has been confused with what prototype is.  

Most of the confusion in JavasSript comes from the fact that it overuses the same term to refer to different but somewhat related concept.

For instance, when a person refers to prototype, in JavaScript it could mean numerous things. It could mean Prototype.js which is a javascript library, prototype as a inheritence concept, `prototype` as a name of one of the properties in a function. Maybe what that person really meant to refer to was `__proto__` which links to its prototype's `prototype`.

This post clarifies numerous confusions relating to "prototype", and `__proto__`.

Note colored words in this post are related to variable or attribute names that exists in JavaScript to help distinguish english as a language from JavaScript names which uses english.

## Main concept.
```javascript
function randomFunc () {
  var msg = 'How can a function have a property?';
  return msg;
}
var x = randomFunc.prototype.constructor  
// Expected undefined got 'randomFunc'
```
Every function declared in any way will always have a secret(inenumerable) property called `prototype` which is an object with few properties in it, one of it is called `constructor` which contains a pointer to itself (a function). As can be seen in below image, where I simply created a function named myFunction;
![](https://raw.githubusercontent.com/imskojs/imskojs.github.io/master/constructor.png)

When any kind of object is created in any way (including literal notation), other than with `Object.create`, it is treated as though it is created with a constructor function. This means, when you type `var x = {}` you are really typing `var x = new Object()`.

This implies that every object has `constructor` property. Exception to this rule is when an object is created with `Object.create(<someObj>)`.  `__proto__` of a created object in this way does not have `constructor` property because it was not created with a constructor function which means `prototype` property of a constructor function was not linked to `__proto__` property of an object created with that function. 

However since `__proto__` itself is created with `Object` function, `__proto__`'s `__proto__` eventually points to `Object.prototype` (just before pointing to `null`) which contains a `constructor` property which points to a function `Object`. 

Because of the fact that `__proto__` is itself an "`Object` object" (an object created with `Object` constructor), EVERY object has a `constructor` property in its prototype's `prototype` property that points to `Object` function somewhere in prototypal chain, eventhough function created with methods other than `Object.create()` may not get to the higher point of prototypal chain where `constructor` property points to `Object`.

Because of this behavior people say "Everything is an object in JavaScript", which is not true, it's more like "Everything is a pointer in JavaScript" or more specifically "Everything is a pointer that eventually points to some class's `prototype` property" but if you needed to say that people would not understand it anyway so lets stick with "Um hem, Everything is an object in JavaScript"

## Examples
In this section we will look at examples that would help us solidify our understanding of prototype.


### Create an object that has array methods but is not an array

```javascript
// x is an Object object not an array
var x = Object.create(Array.prototype)

// verify that x is definitely not an array
Array.isArray(x);  // output: false

// but, x has array mehods like push
x.push('apple')

// seems to work just like an array
x[0]  // output: 'apple'
```
"prototype of x object is `Array.prototype`" which means that `__proto__` is linked to `Array.prototype` property.  
Objects created with above code gets access to `Array.prototype` methods before `Object.prototype` methods eventhough the object is "`Object` object". However all the methods are designed for arrays and not "`Object` objects" which means that most of them will not work on objects without numeric key values. Also adding indexes or keys created in this way will not increase `length` property. So don't create an object that is not an array but has array's prototype methods at the lowest point of prototypal chain.

### Create a function that dives up the prototypal chain and print them out.
```javascript
// Returns All prototypical chain of an object.
function protos(obj) {
  var result = [];
  function diver(obj){
    if(obj.__proto__){
      result.push(obj.__proto__);
    } else {
      return false;
    }
    diver(obj.__proto__);
  }
  diver(obj);
  return result;
}

// Example;
var x = [];
var y = protos(x); 
console.dir(y)   // output: [Array, Object]
```
`protos` function returns all prototypes of an object from bottom to top.
You should use this in Dev tools like Google Chrome which can extend what is inside each object, otherwise it will always print "Object", "Array", "String", "Number", or "Boolean" it's what is inside these objects that interests us which can be accessed with `console.dir(<someObj>)`

If you are sure that all the objects are created with methods other than `Object.create()`, and that the object's constructor functions `prototype` property is not overwritten (i.e. there is a `constructor` property within it) you can alter the function to access constructor of the object which is more accurate description of its prototype;
```javascript
// Returns All prototypical chain of an object.
function protos(obj) {
  var result = [];
  function diver(obj){
    if(obj.__proto__){
      result.push(obj.__proto__.constructor);
    } else {
      result.push(null);
      return false;
    }
    diver(obj.__proto__);
  }
  diver(obj);
  return result;
}

```
Note that Object.prototype has a `__proto__` value of `null` otherwise we will recurse infinitely with `protos` function.
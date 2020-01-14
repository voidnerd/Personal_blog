---
title: Introduction to OOP in Javascript
date: "2017-11-17T22:40:32.169Z"
template: "post"
draft: false
slug: "introduction-to-oop-in-javascript"
category: "Javascript"
tags:
  - "Javascript"
  - "OOP"
  - "Beginners"
description: We covered class definition, properties, methods and inheritance.
socialImage: "https://res.cloudinary.com/iamndie/image/upload/v1578776841/Blog/oop_in_Js.jpg"
---

What is this OOP thingy.

"*Object-oriented programming (OOP) is a programming language model organized around objects rather than "actions" and data rather than logic*" - Margeret Rouse.


To better understand let's have a look at Person as an Object, what attributes can a person have? legs, hands, head etc; these are the **properties** of the Person. Okay so what can a person do, run, walk, crawl, talk, sit, stand etc; these are methods of the Person object. Notice that I keep using capital "P" when I refer to the Person object, well that's how class names are written.

*The basic idea of OOP is that we use objects to model real world things that we want to represent inside our programs*  - developer.mozilla.org


let's see some code examples shall we;

#Defining Classes

We define classes using the `class` keyword and name ("Person"). Properties are written in the constructor method. The `this` keyword assigns properties to the class, **this** here refers to an instance of the class, think  of `this` as a pronoun if `class` was a noun.

```javascript
//class declaration

class Person {
     constructor() {
        this.name = "Isaac";
         this.age = 21;
     }
}

let person = new Person();

console.log(person.name); // logs - Isaac


```

This looks fine, but what if we want users of our programme to enter their name and age, just what if, then we have to add *parameters* to our constructor method. Parameters are placeholders use in functions to accept arguments( what are arguments again? just values pal). code below:

```javascript
class Person {
     constructor(name, age) {
        this.name = name;
         this.age = age;
     }
}

let person = new Person("Isaac", 23);

let person1 = new Person("Bob", 28);

console.log(person.name); // logs - Isaac
console.log(person1.name);// logs - Bob

```
# Class Methods
That was cool, now let's look at methods (getters, setters etc), they are not scary at all, let's look at an example:

```javascript
class Person {
    constructor(name, age) {
       this.name = name;
       this.age = age;
    }

   // setter
   setName(name) {
       this.name = name;
   }

   //getter
   bio() {
       return this.description();
   }

   //some method with a lil logic
   description(){
       return this.name + " is " + this.age + "years old.";
   }

}

let person = new Person("Isaac", 23);

person.setName("Joy");

console.log(person.bio()); // logs -  Joy is 23years old.
```

I told you they were not scary, getters just get property values while setters change property values
and I used this opportunity to show you you can return another method with another, note that we can simply do `return this.name + " is " + this.age + "years old.";` in our `bio()` method.

#Inheritance
Now we have a nice  Person class that describes a Person,but as we go down our programme we may have  other classes like Boss, Father, Mother, Worker etc. All this class will have same properties of the Person class and more. Why write same codes over and over again when you can use inheritance.  

Here,  a Father inherits properties/methods of Person. 

```javascript
 //Parent class
class Person {
    constructor(name, age) {
       this.name = name;
       this.age = age;
    }
   setName(name) {
       this.name = name;
   }
   bio() {
       return this.description();
   }
   description(){
       return this.name + " is " + this.age + "years old.";
   }

}

//child class
class Father extends Person {

	bio(){
		return super.bio();
	}
}

var father = new Father("Isaac", 34);

console.log(father.name) //logs - Isaac
console.log(father.bio()); //logs - Isaac is 34years old.

```

We used `extends` to allow Father  access to properties/methods of Person.
Noticed the `super` used to return `bio()`?

We use `super` to access methods of parent-class("Person").

#Conclusion 

We covered class definition, properties, methods and inheritance, if you need more information,[mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) is always there to help;

I will be waiting for your awesome contributions down in the comment section.

Thanks for taking your time to read till the end, I appreciate, Bye.
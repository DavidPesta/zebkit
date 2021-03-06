---
layout: page
parent: docs
notitle: true
title: Zson
---

JSON for ages is widely used format to keep configuration, exchange and store data etc. Zebkit uses JSON:

   * To configure UI components properties
   * As a descriptive language to define UI forms
   * As a form to define coupled hierarchy of objects  

## Zson features

Standard JSON format is quite simple and often cannot cover desired requirements. Zebkit extends JSON interpretation to support number of advanced features. The new JSON interpretation is called __Zson__ (__zebkit.Zson__ class), the number of new JSON features it introduces are the following:
   
   * **Class instantiation.** A JSON field value can be fulfilled with an instance of a class. Imagine we have "a" field. Let's set the field with an instance of JS "RegExp" (```a = new RegExp("[a-z]+","i")```) class:

```json
    { "a" : { "@RegExp" : [ "[a-z]+", "i" ] } }
```

   * **Properties.** Loading a JS object with a JSON takes in account the target object can define setter methods for a dedicated properties. For the following JSON zebkit tries to call __"setTest(100)"__ method of the target object if the method is available: 
   
```json
    { "test": 100 }
```

   * **References.** A value can be set as reference to other JSON value. For instance in example below "b" field value will be set to 12:

```json
    {  "a": 12, "b": "%{a}" } 
```
   
   * **Expressions.** Expression is JS code that is evaluated and then assigned to a JSON field. For instance the value of "a" field below will be resolved as JS expression and set to 20:
   
```json
    {  "a": { ".expr" : "10 + 10" } }
```

   *  **Image, textual file and external JSON loadings.**  JSON values can  refer to external resources like images, JSON. For instance to load image as "a" field value do the following: ```{ "a": "%{<img> picture.jpg}" }```. External files referenced with Zson:
      * Loaded asynchronously. 

      *  Can be loaded recursively. That means loaded files can contain references to external resources and so on. 

      * Zebkit Zson keeps the loadings files in order. That means any subsequent field value can be a reference to a an externally loaded field. 
       
      For instance Zson below loads "embedded.json" and set the loaded content as a value of "e" field. In its turn "f" field is reference to loaded "e":    
   
```json 
    {  "a" : 100,
       "b" : { "c": { "@Date" : [] } }, 
       "d" : "%{a}",
       "e" : "%{<json> http://test.com/embedded.json}",
       "f" : "%{e.m.k}"
    }
```


   Where "embedded.json" is the following:


```json 
{  "m" : {  "k" : "Hello" } }
```

The result of the Zson loading described below:

   - __"a"__ equals 100
   - __"c"__ is an instance of JavaScript "Date" class 
   - __"d"__ is reference to "a" filed. It is resolved to equal 100
   - __"e"__ value is set to loaded "http://test.com/embedded.json". It equals  ```{"m":{  "k": "Hello" }}```
   - __"f"__ value is a reference to JSON content that is loaded with external JSON. In this case "f" equals "Hello".

The code that load the Zson us shown below:

```js
new Zson().then(function("config.json", zson) {
    zson.root // handle fully loaded json 
});  
```


## Configure JS objects with Zson

Zson can be used to configure classes and class instances with required properties. The way to do it looks as follow:

   - Imagine we declare simple class "A": 

```js
var A = zebkit.Class([
    function() {  // constructor
        this.b = this.a = null;
    },

    function setC(c) { // setter of property "c" 
        this.c = c;
    }
]); 
```

   - Then we declare JSON  ("config.json") to configure the class instance:

```json 
{  
    "a" : "value of property 'a'",
    "b" : { "@Date": [] },
    "c" : [ 1,2,3 ]
}
```

  - Then we instantiate the class and configure the instance with JSON:  

```js
var a = new A();
// configure instance of class "A" with JSON
zebkit.Zson.then("config.json", a);  
```

The result is:

   - "a" field of the class instance equals "value of property 'a'"
   - "b" field of the class instance is set to an instance of JS Date class
   - "c" field of the class instance is set to [1,2,3] via calling the property setter method - "setC(c)".

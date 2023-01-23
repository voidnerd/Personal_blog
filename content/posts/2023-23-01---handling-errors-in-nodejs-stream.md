---
title: Handling errors in nodejs stream
date: "2023-01-23T00:18:13.351Z"
template: "post"
draft: false
slug: "handling-errors-in-nodejs-stream"
category: "Nodejs"
tags:
  - "Nodejs"
  - "Error Handling"
description: This brings us back to the relationship between streams and EventEmitter - All streams are instances of EventEmitter.
---


Most people encounter nodejs streams through `fs.createReadStream` or `fs.createWriteStream`. Maybe you did not bother to read the documentation(like me) or the layered information(`fs -> stream  -> EventEmitter`) is why you missed this very important line (All [streams](https://nodejs.org/api/stream.html) are instances of [EventEmitter](https://nodejs.org/api/events.html#class-eventemitter)) in the documentation.

We should discuss the relationship between streams and the EventEmitter class, but first, let’s look at this code that throws an annoying error.

```
const fs = require("fs");
// input.txt does not exist in current directory
reader = fs.createReadStream("input.txt");

// Read and display the file data on console
reader.on("data", function (chunk) {
  console.log(chunk.toString());
});

// Throw error and crashes nodejs

```
The above code throws an error that crashes nodejs. You might be tempted to handle the error by wrapping this code with a try /catch block.

```

try {
  const fs = require("fs");
  // input.txt does not exist in current directory
  reader = fs.createReadStream("input.txt");

  // Read and display each file data chunk on console
  reader.on("data", function (chunk) {
    console.log(chunk.toString());
  });
} catch (error) {
  console.log("Error");
}

// Throws error and still crashes nodejs
```
As you can see, this does not work. 

This brings us back to the relationship between streams and EventEmitter - All streams are instances of EventEmitter. 

When an error occurs within an EventEmitter instance, the typical action is for an 'error' event to be emitted. Unless there is an error listener present on that instance, the node process will crash.

You know the saying, "if it walks like an Eventemitter, talks like an EventEmitter, it probably is an EventEmitter".(Coined from [Duck Test](https://en.wikipedia.org/wiki/Duck_test)) 

With our newfound knowledge, we know that adding an error listener to `reader` will fix our crash problem.

```
const fs = require("fs");

// input.txt does not exist in current directory
reader = fs.createReadStream("input.txt");

// Read and display each file data chunk on console
reader.on("data", function (chunk) {
  console.log(chunk.toString());
});

reader.on("error", function (error) {
  console.log("An error occured");
});

// Logs “An error occurred without crashing nodejs
```

That’s it, we handled our error!!

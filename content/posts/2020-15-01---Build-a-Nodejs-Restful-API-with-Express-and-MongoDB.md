---
title: Build a Nodejs Restful API with Express and MongoDB
date: "2020-01-15T01:40:32.169Z"
template: "post"
draft: false
slug: "build-a-nodejs-restful-api-with-express-and-mongodb"
category: "Nodejs"
tags:
  - "Nodejs"
  - "Express"
  - "JWT"
  - "API"
  - "Web Development"
description: In this tutorial, we are going to build an authenticated nodejs api with express, hash passwords with bcryptjs and secure your API with JWT.
socialImage: "https://res.cloudinary.com/iamndie/image/upload/v1579086997/nodeexpress_cu3syi.png"
---

In this tutorial, we are going to build an authenticated nodejs api with express, hash passwords with bcryptjs and secure your API with JWT.

## Prerequisite

Make sure you have the following installed on your system

- Nodejs >= v12
- Mongo >= v4
- Postman (To test your endpoints)

Now that we have the prerequisites out of the way, let's create a directory for our app, run below this bash command for this.

```bash
$ mkdir node-api && cd node-api
```

NPM utility will walk you through creating a package.json file. Follow the prompt and press `enter` all through to use the defaults.

```bash
$ npm init
```

Create Files and Folders in your `node-api` directory like so:

```
--model
-----user.js
-----index.js
--controllers
-----auth.js
-----user.js
-----index.js
--public
--.env
--middleware.js
--.gitignore
--routes.js
--server.js
```

Notice the `.evn` file; this is where we will store our sensitive values as enviromental variables. <br>
`.gitignore` is where we specify files or folders we don't want git to track; this is so we don't push sensitive or unwaranted data to github (or your prefered source code repository). <br>
`public` folder for serving your static files.

### Install project dependencies

In your project directory run below command to install dependencies.

```bash
$ npm install express mongoose morgan parse-error dotenv bcryptjs jsonwebtoken indicative
```

For further reading on these dependencies

- Express - [https://expressjs.com/](https://expressjs.com/)
- Mongoose -[https://mongoosejs.com/](https://mongoosejs.com/)
- morgan - [https://www.npmjs.com/package/morgan](https://www.npmjs.com/package/morgan)
- indicative - [https://github.com/poppinss/indicative](https://github.com/poppinss/indicative)
- dotenv - [https://www.npmjs.com/package/dotenv](https://www.npmjs.com/package/dotenv)
- jsonwebtoken - [https://www.npmjs.com/package/jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken)
- bcryptjs - [https://www.npmjs.com/package/bcryptjs](https://www.npmjs.com/package/bcryptjs)
- parse-error - [https://www.npmjs.com/package/parse-error](https://www.npmjs.com/package/parse-error)

Moving on...

### Register.

Just like every other thing on earth, we are going to start with creation (pun intended). In this section, we will focus on setting up our app and enabling user creation. Less talk, more code, let's roll :) .

Add needed enviromental variables to your `.env` file like so:

```
MONGO_URL=mongodb://127.0.0.1:27017/nodeApp
JWT_SECRET=thisisasecretlongstring
```

Add paths we would like git to ignore in `.gitignore` file like so:

```
node_modules/
.env
```

Type in the following code in `model/User.js` .

```js
const mongoose = require("mongoose");
const Schema = mongoose.Schema;
const bcrypt = require("bcryptjs");
const JWT = require("jsonwebtoken");

const jwtSecret = process.env.JWT_SECRET;

let userSchema = Schema({
  name: String,
  email: {
    type: String,
    required: true,
    unique: true
  },
  password: String
});

// hash passwords for new records before saving
userSchema.pre("save", function(next) {
  if (this.isNew) {
    var salt = bcrypt.genSaltSync(10);
    var hash = bcrypt.hashSync(this.password, salt);
    this.password = hash;
  }
  next();
});
//validate user password
userSchema.methods.validPassword = function(inputedPassword) {
  return bcrypt.compareSync(inputedPassword, this.password);
};

//sign token for this user
userSchema.methods.getJWT = function() {
  return JWT.sign({ userId: this._id }, jwtSecret);
};
module.exports = mongoose.model("User", userSchema);
```

In the above sample we are defining our user schema and hashing our password for new records(`this.isNew` in pre save hook) . We also made available `validPassword` method for validating passwords and `getJWT` method of retrived user specific token.

Add this to `models/index.js`, this will help us have access to all the modules in models folders when we import just the folder [e.g require('./models').

```js
var normalizedPath = require("path").join(__dirname);
require("fs")
  .readdirSync(normalizedPath)
  .forEach(function(file) {
    if (!file.includes("index")) {
      var moduleName = file.split(".")[0];
      exports[moduleName] = require("./" + moduleName);
    }
  });
```

For our registration logic, add below code to `controllers/auth.js`

```js
const { validate } = require("indicative").validator;
const { User } = require("../models");

exports.register = async (req, res) => {
  //Validate request data
  const rules = {
    name: "required|string",
    email: "required|email",
    password: "required|min:6|max:30"
  };

  validate(req.body, rules).catch(errors => {
    return res.status(422).json(errors[0]);
  });
  try {
    const user = new User(); //initialize mongoose Model
    user.name = req.body.name;
    user.email = req.body.email;
    user.password = req.body.password;
    await user.save(); //save user record to database

    const token = user.getJWT();

    // data { user, token } = data {user: user, token token}
    return res.status(201).json({ data: { user, token } });
  } catch (err) {
    //return error if user unique field already exists
    if (err.name === "MongoError" && err.code === 11000) {
      field = Object.keys(err.keyValue)[0];
      const response = {
        message: `${field} already exists!`,
        field: field
      };
      return res.status(422).json(response);
    }

    return res.status(409).json({ message: "error saving data" });
  }
};
```

In the above code we validated our request with `indicative`, save out user record to mongodb and return the appropriate responses to our users.

Add this to `constrollers/index.js` to have access to all modules in `controllers` folder.

```js
var normalizedPath = require("path").join(__dirname);
require("fs")
  .readdirSync(normalizedPath)
  .forEach(function(file) {
    if (!file.includes("index")) {
      var moduleName = file.split(".")[0];
      exports[moduleName] = require("./" + moduleName);
    }
  });
```

let's work on our routes on `routes.js`

```js
var express = require("express");
var router = express.Router();

/* Import Controllers*/
const controllers = require("./controllers");

/* Define all your routes*/

router.post("/register", controllers.auth.register);
router.post("/login", controllers.auth.login);

/*Export your routes*/
module.exports = router;
```

See how neat our routes are, when we use controllers. I love it :)

Let's get down to our Server Logic. Add below code to `server.js`.

```js
const express = require("express");
const app = express();
const path = require("path"); //native module, no need to install
require("dotenv").config();
const logger = require("morgan");
const mongoose = require("mongoose");
const pe = require("parse-error");
const routes = require("./routes");

const port = 3000; // server starts on this port

//mongoose options
const mongooseOptions = {
  useUnifiedTopology: true,
  useNewUrlParser: true,
  useCreateIndex: true
};

//connect to database
mongoose.connect(process.env.MONGO_URL, mongooseOptions).then(
  () => console.log("Database Connection established!"),
  err => console.log(err)
);

app.use(logger("dev")); // For logging out errors to the console
app.use(express.json()); // for parsing application/json
app.use(express.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded
app.use(express.static(path.join(__dirname, "public"))); // For serving static files
//use routes
app.use("/api", routes);

//handle unhandled error
process.on("unhandledRejection", error => {
  console.error("Uncaught Error", pe(error));
  return;
});

app.listen(port, () => {
  console.log(`Server Stated on http://localhost:${port}`);
});
```

Change directory to your project and run the command

```bash
node server.js
```

Test your api with postman using the register endpoint `http://localhost:3000/api/register` and you should get a successful response like the image below.

![Register Endpoint Test](https://res.cloudinary.com/iamndie/image/upload/v1579011179/Screenshot_2020-01-14_at_3.11.56_PM_k8vkug.png "Example Resgister endpoint Test")

### Login

Our Initial setup is going to be really helpful from here on out.

For our login logic, add code to `controllers/auth.js` just below your register logic

```js
...
exports.login = async (req, res) => {
	const  rules = {
		email:  'required|email',
		password:  'required|min:6|max:30'
	}

	validate(req.body, rules).catch((errors) => {
		return  res.status(422).json(errors[0])
	});

	try {
		const  user = await  User.findOne({email:  req.body.email})

		if(!user) throw  new  Error("Invalid Email or Password")

		if(!user.validPassword(req.body.password) ) {
			throw  new  Error("Invalid Email or Password")
		}

		const  token = user.getJWT();

		return  res.status(200).json({data: { user, token }});

	} catch (err) {
		console.log(err)
		if(err) return  res.status(401).json({message:  err.message})
	}
}
```

In the above code we validated incoming request, checked if it's user has correct credentials and returned the appropriate response.

Now add this to your `routes.js` file just below your register route.

```javascript
...

router.post('/login', controllers.auth.login)

...
```

Ctrl C to stop your app if it is running and re-run it

```bash
$ node server.js
```

Test your api with postman using the login endpoint `http://localhost:3000/api/login` .
![Login Endpoint Test](https://res.cloudinary.com/iamndie/image/upload/v1579013402/Screenshot_2020-01-14_at_3.49.36_PM_c2j6yb.png "Example Login endpoint Test")

### User Profile

For our grand finale, we are going to have one route(user profile) of which users need the right access get a successful response.

Add this to `middleware.js`

```js
const JWT = require("jsonwebtoken");
const { User } = require("./models");

const jwtSecret = process.env.JWT_SECRET;

exports.auth = async (req, res, next) => {
  try {
    //get token from header: Bearer <token>
    const token = req.headers.authorization.split(" ")[1];
    //verify this token was signed by your server
    const decodedToken = JWT.verify(token, jwtSecret);
    ///Get user details
    let user = await User.findById(decodedToken.userId);
    if (!user) throw Error("Unauthenticated");
    //put user in req object; so the controller can access current user
    req.user = user;
    next();
  } catch {
    return res.status(401).json({
      message: "Unauthenticated"
    });
  }
};
```

In the above code, we get the `bearer token` from the authorization header, verify the token, get user details and pass those details to the `next` function(controller).

Add below code to `controllers/users.js`

```js
exports.currentUser = (req, res) => {
  return res.status(200).json(req.user);
};
```

And finally, your `routes.js` should look like the one below

```javascript
var express = require("express");
var router = express.Router();

/* Import Controllers*/
const controllers = require("./controllers");

/* Import Middleware*/
const middleware = require("./middleware");

/* Define all your routes*/
router.post("/register", controllers.auth.register);
router.post("/login", controllers.auth.login);

router.get("/profile", middleware.auth, controllers.users.currentUser);

/*Export your routes*/
module.exports = router;
```

Ctrl C to stop your app if it is running and re-run it

```bash
$ node server.js
```

Test your protected endpoing `http://localhost:3000/api/profile` .
![Auth Endpoint Test](https://res.cloudinary.com/iamndie/image/upload/v1579019612/Screenshot_2020-01-14_at_5.33.00_PM_vmzoax.png "Example Auth endpoint Test")

Tip: Pay attention to the `Headers` section and the Authorization value.
Tip 2: Get your token from a successful login/register response.

Where do I go from here? Well there are a lot more ways we could improve our current API; you could add bonus routes like: `delete user`, `get all users`, `get single user`, `update user` and `search users` . I will leave this to you as a challenge, have fun :) .

Here's a [link to the full code on github](https://github.com/ndiecodes/node-auth-example).

Wow, you got this far!! You are super awesome. As always, I would like to see your contributions down in the comments.

Thanks for taking your time out to read all through :), Adios!

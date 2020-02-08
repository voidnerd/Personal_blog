---
title: Build a JWT Authenticated API with Lumen(v5.8)
date: "2019-08-24T22:40:32.169Z"
template: "post"
draft: false
slug: "build-a-jwt-authenticated-api-with-lumen"
category: "PHP"
tags:
  - "Laravel"
  - "PHP"
  - "API"
  - "JWT"
  - "Lumen"
  - "Web Development"
description: In this tutorial, we will be using lumen; a super-fast micro-framework by laravel to build a simple and secure REST API. At the end of this tutorial, you should be able to build production-ready APIs. Let's get started!
socialImage: "https://res.cloudinary.com/iamndie/image/upload/v1578778835/Blog/lumen.png"
---


In this tutorial, we will be using lumen; a super-fast micro-framework by laravel to build a simple and secure REST API. At the end of this tutorial, you should be able to build production-ready APIs. Let's get started!

## Prerequisite

Make sure you have the essentials, I beg of you.

- PHP >= 7.1.3
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Mysql >= 5.7
- Composer (Dependency Manager for PHP)
- Postman (To test your endpoints)

## Installation

First things first, you need to get lumen's cli.

```
$ composer global require "laravel/lumen-installer"
```

If your download was successful, run below command to confirm you have lumen installed.

```
$ lumen
```
If you encounter an error like  `-bash: lumen: command not found`, you need to  [add composer's vender bin to your path](https://stackoverflow.com/questions/25373188/laravel-installation-how-to-place-the-composer-vendor-bin-directory-in-your).

If all is in order, you should see something like so.

![Lumen](https://res.cloudinary.com/iamndie/image/upload/v1566656870/Screen_Shot_2019-08-24_at_3.16.08_PM_mqbqso.png "Lumen CLI")

Now run this command to create the lumen project

```
$ lumen new auth-app
```

UPDATE: To create a specific  version of lumen project(v5.8), use this command instead: 
```
$ composer create-project laravel/lumen auth-app "5.8.*"
```

Enter the project folder
```
$ cd auth-app
```

Run the app
```
$ php -S localhost:8000 -t public
```
Load `localhost:8000` on your browsers address bar and it should render a result as shown below.

![Lumen response](https://res.cloudinary.com/iamndie/image/upload/v1566658430/Screen_Shot_2019-08-24_at_3.53.20_PM_ivhqpc.png "Lumen response")

Open up the project (auth-app) in your preferred editor. 

Create a .env file, copy all contents in .env.example into the .env file and add your database configurations.

In `boostrap/app.php` uncomment the facades and eloquent method


```php
//before

// $app->withFacades();

// $app->withEloquent();

//after

$app->withFacades();

$app->withEloquent();

```
Turning on withFacades inject the application IoC to Illuminate\Support\Facades\Facade. Without doing so even if you're importing Illuminate\Support\Facades\File it wouldn't work. [Credit](https://github.com/laravel/lumen-framework/issues/143)

The $app->withEloquent() method actually enables the query builder too. It's registering the DatabaseServiceProvider, which is required to use the query builder. [Credit](https://github.com/laravel/lumen-framework/issues/584).


## Create a user

Make user's database migration

```
$ php artisan make:migration create_users_table --create=users
```

Locate the migration file `database/migrations/*_create_users_table.php` and add neede table columns(name, email, password); see code below:

```php
...
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->string('email')->unique()->notNullable();
            $table->string('password');
            $table->timestamps();
        });
    }
...
```

Migrate your database

```
$ php artisan migrate

```

Add register route which as the name implies; register users. Locate `routes/web.php` and insert the needed code as seen below

```php
// API route group
$router->group(['prefix' => 'api'], function () use ($router) {
   // Matches "/api/register
   $router->post('register', 'AuthController@register');

});
```

Since we are going to prefix `api` in all our endpoint, to reduce repetition we will use route grouping to do just that.

This method (`$router->post($uri, $callback);` takes in a $url and a $callback parameter. In the `$callback`, `AuthController` is our controller class (we will create this class in a bit) and `register` is a method in said class.

Let's create our AuthControler.

Create file `app/Http/Controllers/AuthController.php` and populate it with code as seen below.

```php

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use  App\User;

class AuthController extends Controller
{
    /**
     * Store a new user.
     *
     * @param  Request  $request
     * @return Response
     */
    public function register(Request $request)
    {
        //validate incoming request 
        $this->validate($request, [
            'name' => 'required|string',
            'email' => 'required|email|unique:users',
            'password' => 'required|confirmed',
        ]);

        try {
           
            $user = new User;
            $user->name = $request->input('name');
            $user->email = $request->input('email');
            $plainPassword = $request->input('password');
            $user->password = app('hash')->make($plainPassword);

            $user->save();

            //return successful response
            return response()->json(['user' => $user, 'message' => 'CREATED'], 201);

        } catch (\Exception $e) {
            //return error message
            return response()->json(['message' => 'User Registration Failed!'], 409);
        }

    }

    
}
```

Register a user(use POSTMAN) with route `localhost:8000/api/register` and you should get a successful response like so 

![Lumen register example](https://res.cloudinary.com/iamndie/image/upload/v1566663229/Screen_Shot_2019-08-24_at_4.34.01_PM_vnm7zv.png "Lumen register example")

## User sign in

Pull in the JWT authentication package.

```
$ composer require tymon/jwt-auth:dev-develop
```

Generate your API secret

```
$ php artisan jwt:secret
```

create file `config/auth.php` with below config

```php
//config.auth.php

<?php

return [
    'defaults' => [
        'guard' => 'api',
        'passwords' => 'users',
    ],

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => \App\User::class
        ]
    ]
];

```
Make some changes to your `User` model(`app/User.php`) to fit **tymon/jwt-auth's** requirements. Keep your eye out for everything that includes "JWT".

```php
<?php

namespace App;

use Illuminate\Auth\Authenticatable;
use Laravel\Lumen\Auth\Authorizable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
use Illuminate\Contracts\Auth\Access\Authorizable as AuthorizableContract;


use Tymon\JWTAuth\Contracts\JWTSubject;

class User extends Model implements AuthenticatableContract, AuthorizableContract, JWTSubject
{
    use Authenticatable, Authorizable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email'
    ];

    /**
     * The attributes excluded from the model's JSON form.
     *
     * @var array
     */
    protected $hidden = [
        'password',
    ];


  

    /**
     * Get the identifier that will be stored in the subject claim of the JWT.
     *
     * @return mixed
     */
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims()
    {
        return [];
    }
}

```
Make some changes to `bootstrap/app.php`

```php
//before
// $app->routeMiddleware([
//     'auth' => App\Http\Middleware\Authenticate::class,
// ]);

//After
$app->routeMiddleware([
    'auth' => App\Http\Middleware\Authenticate::class,
]);
```

```php
//before
 // $app->register(App\Providers\AppServiceProvider::class);
 // $app->register(App\Providers\AuthServiceProvider::class);
 // $app->register(App\Providers\EventServiceProvider::class);

//After
 // $app->register(App\Providers\AppServiceProvider::class);
 $app->register(App\Providers\AuthServiceProvider::class);
 // $app->register(App\Providers\EventServiceProvider::class);

 // Add this line
 $app->register(Tymon\JWTAuth\Providers\LumenServiceProvider::class);
```

Add login route in `routes/web.php`

```php
// API route group
$router->group(['prefix' => 'api'], function () use ($router) {
     // Matches "/api/register
    $router->post('register', 'AuthController@register');
     
      // Matches "/api/login
     $router->post('login', 'AuthController@login');
});
```

Add a global respondWithToken method to Controller class in  `app/Http/Controllers/Controller.php`. This is so we could access it from any other controller.

```php
   ...
  //import auth facades
  use Illuminate\Support\Facades\Auth;
  

  //Add this method to the Controller class
  protected function respondWithToken($token)
    {
        return response()->json([
            'token' => $token,
            'token_type' => 'bearer',
            'expires_in' => Auth::factory()->getTTL() * 60
        ], 200);
    }

```

Add a login method to your AuthController class in `app/Http/Controllers/AuthController.php`

```php
   ...

   //import auth facades
   use Illuminate\Support\Facades\Auth;

   ...

     /**
     * Get a JWT via given credentials.
     *
     * @param  Request  $request
     * @return Response
     */
    public function login(Request $request)
    {
          //validate incoming request 
        $this->validate($request, [
            'email' => 'required|string',
            'password' => 'required|string',
        ]);

        $credentials = $request->only(['email', 'password']);

        if (! $token = Auth::attempt($credentials)) {
            return response()->json(['message' => 'Unauthorized'], 401);
        }

        return $this->respondWithToken($token);
    }

```

Login a user using route `localhost:8000/api/login` and you should get a successful response like so:

![Lumen login example](https://res.cloudinary.com/iamndie/image/upload/v1566672721/Screen_Shot_2019-08-24_at_7.51.18_PM_srhwrs.png "Lumen login example")


## Authenticated routes


For our grand finale, we are going to make some authenticated routes.


Add a couple of routes to `routes/web.php`

```php
...
// API route group
$router->group(['prefix' => 'api'], function () use ($router) {
    // Matches "/api/register
   $router->post('register', 'AuthController@register');
     // Matches "/api/login
    $router->post('login', 'AuthController@login');

    // Matches "/api/profile
    $router->get('profile', 'UserController@profile');

    // Matches "/api/users/1 
    //get one user by id
    $router->get('users/{id}', 'UserController@singleUser');

    // Matches "/api/users
    $router->get('users', 'UserController@allUsers');
});

...

```

Create a file `app/Http/Controllers/UserController.php` and populate it with this elegant looking code.

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Auth;
use  App\User;

class UserController extends Controller
{
     /**
     * Instantiate a new UserController instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');
    }

    /**
     * Get the authenticated User.
     *
     * @return Response
     */
    public function profile()
    {
        return response()->json(['user' => Auth::user()], 200);
    }

    /**
     * Get all User.
     *
     * @return Response
     */
    public function allUsers()
    {
         return response()->json(['users' =>  User::all()], 200);
    }

    /**
     * Get one user.
     *
     * @return Response
     */
    public function singleUser($id)
    {
        try {
            $user = User::findOrFail($id);

            return response()->json(['user' => $user], 200);

        } catch (\Exception $e) {

            return response()->json(['message' => 'user not found!'], 404);
        }

    }

}


``` 

Below is an example call to one of the three newly added endpoints 

![Lumen users example](https://res.cloudinary.com/iamndie/image/upload/v1566677009/Screen_Shot_2019-08-24_at_8.54.39_PM_vqr7bx.png "Lumen users example")

Here's a [link to the full code on github](https://github.com/ndiecodes/lumen-auth-example).

The end-ish!

I hope this article has helped you in some way, and that you build upon this knowledge to deploy awesome APIs in the nearest future. I would like to see your contributions down in the comments too.

Hasta la vista.

---
title: Deploying Laravel on shared Hosting
date: "2019-08-19T22:40:32.169Z"
template: "post"
draft: false
slug: "deploying-laravel-on-shared-hosting"
category: "Hosting"
tags:
  - "Laravel"
  - "PHP"
  - "Hosting"
  - "Shared Hosting"
description: If you are here, you are probably exploring how laravel works on shared hosting, on some low-budget project or just have reasons you prefer shared hosting. Alright, let's get down to business.
socialImage: "https://res.cloudinary.com/iamndie/image/upload/v1578777965/Blog/laravel_on_sharedhosting.png"
---

If you are here, you are probably exploring how laravel works on shared hosting, on some low-budget project or just have reasons you prefer shared hosting. Alright, let's get down to business.

Here are the steps to get laravel up and running on shared hosting:

(1) Run `php artisan config:clear`; this clears the cache config. Since you are going to be editing `.env` online; you don't need your local configurations cached.

(2) Export Your migrated Database to a file (db.sql) without the `create database` statement, like so:

```sql
DROP TABLE IF EXISTS `users`;

CREATE TABLE `users` (
 `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
 `title` varchar(255) DEFAULT NULL,
 `name` varchar(255) NOT NULL,
 `facebook_id` varchar(255) DEFAULT NULL,
 `google_id` varchar(255) DEFAULT NULL,
 `linkedin_id` varchar(255) DEFAULT NULL,
 `tagline` varchar(255) DEFAULT NULL,
 `email` varchar(255) DEFAULT NULL,
 `username` varchar(255) DEFAULT NULL,
 `email_verification_code` varchar(255) DEFAULT NULL,
 `email_verified_at` timestamp NULL DEFAULT NULL,

 `password` varchar(255) DEFAULT NULL,
 `remember_token` varchar(100) DEFAULT NULL,
 `created_at` timestamp NULL DEFAULT NULL,
 `updated_at` timestamp NULL DEFAULT NULL,
 PRIMARY KEY (`id`)
) ENGINE=InnoDB;

DROP TABLE IF EXISTS `password_resets`;

CREATE TABLE `password_resets` (
 `email` varchar(254) NOT NULL,
 `token` varchar(254) NOT NULL,
 `created_at` timestamp NULL DEFAULT NULL,
 KEY `password_resets_email_index` (`email`)
) ENGINE=InnoDB;
```
(3) go to `MultiPHP Manager` on c-panel and confirm you have php 7.1 or higher.

(4) Zip the finished project (we will call this **myproject** from now on) and upload to root directory of hosting storage through File Manager (c-panel) or FileZilla; it should be something like `/home/domain-name`.

(5) Extract myproject in root directory(`/home/domain-name`) of hosting storage.

(6) Enable "Show Hidden Files". This is so you could see files like .htaccess, .env, etc.

(7) In `myproject/public`, move all it's contents to `/public_html`.

(8) Now edit  `/public_html/index.php` to point to your laravel project, like so:


```php
//previous state
require __DIR__.'/../vendor/autoload.php';

// changed to
require __DIR__.'/../myproject/vendor/autoload.php';

//previous state
$app = require_once __DIR__.'/../bootstrap/app.php';

//changed to
$app = require_once __DIR__.'/../myproject/bootstrap/app.php';

```


(9) Edit .env in `/myproject` with your config keys

(10) Import and run the `db.sql` script you exported earlier on your created database through phpmyadmin on c-panel.

The end!
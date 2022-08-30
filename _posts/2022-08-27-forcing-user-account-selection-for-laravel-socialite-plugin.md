---
layout: post
title: Forcing user account selection for Laravel Socialite plugin
date: 2022-08-27 00:32 +0530
categories: [Tech, Laravel, Socialite, PHP, Social Login]
tags: [tech, laravel, socialite, php, social login]
---

![Laravel Socialite](/assets/img/posts/laravel-socialite.png){: w="700" h="300" }

I remember when I implemented Laravel Socialite in one of my website for users to login using their Google+ accounts. 
Everything was working fine but there was one issue after login for the first time user were unable to switch to a different Google account if they log out and then log back in. 

Once the following code is executed in the `LoginController.php`{: .filepath} :
```php
public function redirectToProvider()
{
    return Socialite::driver('google')->redirect();
}
```
It automatically logs them in with Google account and redirects back to site without giving them the option to choose a different Google account.

The problem was same for other social login like Facebook and Linkedin.

The solution was to add a parameter `"select_account"` like below:

```php
public function redirectToProvider()
{
   return Socialite::driver('google')->with(["prompt" => "select_account"])->redirect();
}
```
This worked for me.

Fairly simple solution but was not mentioned in @laravel documentation anywhere.

I hope it help someone.

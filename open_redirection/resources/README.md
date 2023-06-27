# How

At the bottom of the page, there are three links to social medias.

1. Facebook
2. Twitter
3. Instagram

Clicking any of the Facebook icon redirects us to `http://localhost:4242/index.php?page=redirect&site=facebook` for example.

In a scenario where the `site` parameter isn't validated, we could redirect to an unexpected location, which could allow an attacker to perform a phishing attack.

# Why

The `site` parameter isn't verified by the server before performing the redirection, thus allowing an attacker to redirect to any location from a trusted origin.

# Fix

The server in this scenario does a good job at checking where the user should be redirected.

This could be the PHP code of the server :

```php

<?php

$social_medias = [
    'facebook' => 'https://www.facebook.com/42born2code',
    'twitter' => 'https://twitter.com/42born2code',
    'instagram' => 'https://www.instagram.com/42born2code/',
];

// Check if the site parameter is set
if (isset($_GET['site'])) {
    // Check if the site parameter is in the map of social medias
    if (array_key_exists($_GET['site'], $social_medias)) {
        // Redirect to the social media
        header('Location: ' . $social_medias[$_GET['site']]);
    } else {
        // Redirect to the index page
        header('Location: index.php');
    }
} else {
    // Redirect to the index page
    header('Location: index.php');
}
```

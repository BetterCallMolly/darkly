# How

The page at `index.php?page=feedback` is vulnerable to XSS.

You can post a feedback with any name, and this message :

```html
<img src=x onerror='alert("XSS")'>
```

Pops the alert, but doesn't gives the flag.

After noticing that the server strips the `<script>` string, we just tried to post a message with the string `script` and it worked.

After completing the project, we went to watch the project's code and we noticed that the challenge is flawed.

Here's the condition that checks if the message contains the keywords `script` or `alert` :

```php
<?php
    if (
        (strstr("<script>", strtolower($rows['name'])) != FALSE) ||
        (strstr("<script>", strtolower($rows['comment'])) != FALSE) ||
        (strstr("alert", strtolower($rows['name']))!= FALSE) ||
        (strstr("alert", strtolower($rows['comment']))!= False)
    )
?>
```

# Why

Let's assume that there are no problem on the server's end, this XSS payload will try to load the image located at `x`, which doesn't exist, and so it'll execute the code in the `onerror` attribute.

Since the server doesn't sanitize the user's input, we can basically inject any HTML / JS we want in here, as long as it fits in the character limit.

# Fix

Sanitize the user's output to not render raw HTML but text instead.
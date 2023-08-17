# How

Directory traversal vulnerabilities occur when the application uses untrusted user input
to display/execute the content of a file.

`http://192.168.122.233/?page=../../../../../../../../../etc/passwd`

Here the application is most likely using `include` in order to execute the PHP files.
Example:

```
$page = $_GET["page"];
include("/var/www/html/" . $page . ".php");
```

Which becomes `include("/var/www/html/../../../../../etc/passwd.php");` ->
`include("/etc/passwd.php");`

In this case `/etc/passwd.php` does not exist but we can include (execute)
any PHP file on the filesystem (this can become a bigger problem if we are able to
upload arbitrary files).

If the PHP version is old enough we may be able to put `%00` (URL encoded null byte)
at the end to make PHP stop parsing after it encounters it:

`http://192.168.122.233/?page=../../../../../../../../../etc/passwd%00` ->
`include("/etc/passwd");`

# Fix

- Rewrite application to use a proper architecture (don't use a `page` parameter)
- Validate user input. Example in python:

```
from pathlib import Path

WEB_ROOT = Path("/var/www/html")

if WEB_ROOT.resolve() not in requested_path.resolve().parents:
    raise ForbiddenPathError("Forbidden path")
```

`path.resolve()` will make the path absolute, turning `./index.php` into `/var/www/html/index.php`.

`path.parents` returns a list of all ancestors of the path (`/var/www/html` -> `/var/www`, `/var`).

Also note that `path.resolve()` will follow symlinks.

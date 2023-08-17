# How

The `/robots.txt` file is a well known mean of telling web crawlers (like Google's spiders)
to not index the pages listed in the file. In this case, it leaks 2 new directories:
`/whatever` and `/.hidden`.

Directory listing in enabled in the `/whatever` directory, revealing the presence
of a `htpasswd` file which contains credentials.

The password hash can easily be cracked with a tool like `hashcat`, allowing us to retrieve
the clear text password.

This password can then be used to login at `/admin` to get the flag (the `/admin`
directory was found via directory bruteforcing with a tool like `gobuster` or `feroxbuster`).

# Fix

- Don't use MD5 to hash passwords!

- Disable directory listing:
    - Apache: `Options -Indexes` in virtual host config or `.htaccess`
    - Nginx: `autoindex off` in virtual host config
    - put an empty `index.html` or `index.php` in the folder

- Do not expose sensitive files on a webserver (without authentication at the very least)!
Example config for Apache (put it in `/var/www/html/whatever/.htaccess` or virtual host config):

```
AuthType Basic
AuthName "Restricted Files"
AuthUserFile "/etc/apache2/.htpasswd"
Require user admin
```

Create the password file:

```
# htpasswd -c /etc/apache2/.htpasswd admin
New password: V3Ry5tr0nGP@ssW0rd!
Re-type new password: V3Ry5tr0nGP@ssW0rd!
Adding password for user admin
```

Example config for Nginx (in the virtual host config):

```
location /whatever/ {
    auth_basic "Restricted Files"
    auth_basic_user_file /etc/apache2/.htpasswd
}
```

Can use the same file/command as above for the password file.

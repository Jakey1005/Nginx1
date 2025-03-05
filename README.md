# Nginx1

## PostgreSQL
```
sudo apt update
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo -iu postgres psql
CREATE DATABASE flask_db;
CREATE USER sammy WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE flask_db TO sammy;
ALTER USER sammy WITH ENCRYPTED PASSWORD 'password';
ALTER SYSTEM SET password_encryption = 'scram-sha-256';
ALTER DATABASE flask_db OWNER TO sammy;
SELECT pg_reload_conf();
SELECT usename, passwd FROM pg_shadow WHERE usename = 'sammy';
\l
\q

sudo nano /etc/postgresql/17/main/postgresql.conf
listen_addresses = '*'

nano /etc/postgresql/17/main/pg_hba.conf
host flask_db sammy 172.16.10.10/24 scram-sha-256
sudo systemctl restart postgresql
```
## web
```
apt update
apt install nginx
apt install python3-pip
pip install Flask psycopg2-binary
```
If get externally managed environment error, do the folllowing
```
sudo apt install python3-venv -y
python3 -m venv my_flask_env
source my_flask_env/bin/activate
pip install Flask
pip install Flask psycopg2-binary
```
```
mkdir flask_app
nano flask_app/init_db.py
```
```
import os
import psycopg2

conn = psycopg2.connect(
        host="localhost",
        database="flask_db",
        user=os.environ['DB_USERNAME'],
        password=os.environ['DB_PASSWORD'])

# Open a cursor to perform database operations
cur = conn.cursor()

# Execute a command: this creates a new table
cur.execute('DROP TABLE IF EXISTS books;')
cur.execute('CREATE TABLE books (id serial PRIMARY KEY,'
                                 'title varchar (150) NOT NULL,'
                                 'author varchar (50) NOT NULL,'
                                 'pages_num integer NOT NULL,'
                                 'review text,'
                                 'date_added date DEFAULT CURRENT_TIMESTAMP);'
                                 )

# Insert data into the table

cur.execute('INSERT INTO books (title, author, pages_num, review)'
            'VALUES (%s, %s, %s, %s)',
            ('A Tale of Two Cities',
             'Charles Dickens',
             489,
             'A great classic!')
            )


cur.execute('INSERT INTO books (title, author, pages_num, review)'
            'VALUES (%s, %s, %s, %s)',
            ('Anna Karenina',
             'Leo Tolstoy',
             864,
             'Another great classic!')
            )

conn.commit()

cur.close()
conn.close()
```
creates database
```
export DB_USERNAME="sammy"
export DB_PASSWORD="password"
python3 flask_app/init_db.py
```
```
nano flask_app/app.py
import os
import psycopg2
from flask import Flask, render_template

app = Flask(__name__)

def get_db_connection():
    conn = psycopg2.connect(host='localhost',
                            database='flask_db',
                            user=os.environ['DB_USERNAME'],
                            password=os.environ['DB_PASSWORD'])
    return conn


@app.route('/')
def index():
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute('SELECT * FROM books;')
    books = cur.fetchall()
    cur.close()
    conn.close()
    return render_template('index.html', books=books)
```
```
mkdir flask_app/templates
nano flask_app/templates/base.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %} {% endblock %}- FlaskApp</title>
    <style>
        nav a {
            color: #d64161;
            font-size: 3em;
            margin-left: 50px;
            text-decoration: none;
        }

        .book {
            padding: 20px;
            margin: 10px;
            background-color: #f7f4f4;
        }

        .review {
                margin-left: 50px;
                font-size: 20px;
        }

    </style>
</head>
<body>
    <nav>
        <a href="{{ url_for('index') }}">FlaskApp</a>
        <a href="#">About</a>
    </nav>
    <hr>
    <div class="content">
        {% block content %} {% endblock %}
    </div>
</body>
</html>
```
```
nano flask_app/templates/index.html
{% extends 'base.html' %}

{% block content %}
    <h1>{% block title %} Books {% endblock %}</h1>
    {% for book in books %}
        <div class='book'>
            <h3>#{{ book[0] }} - {{ book[1] }} BY {{ book[2] }}</h3>
            <i><p>({{ book[3] }} pages)</p></i>
            <p class='review'>{{ book[4] }}</p>
            <i><p>Added {{ book[5] }}</p></i>
        </div>
    {% endfor %}
{% endblock %}
```
```
nano /etc/nginx/sites-available/default
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  server_name _;
  location / {
    proxy_pass http://127.0.0.1:5000;
  }
}
systemctl restart nginx
```
```
export FLASK_APP=app
export FLASK_ENV=development
cd flask_app
flask run
```

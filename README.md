# Nginx1

## PostgreSQL
```
sudo apt update
sudo apt install postgresql postgresql-contrib -y
sudo apt start postgresql
sudo apt enable postgresql
sudo -iu postgres psql
CREATE DATABASE flask_db;
CREATE USER sammy WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE flask_db TO sammy;
\l
\q
```
## web
```
pip install Flask psycopg2-binary

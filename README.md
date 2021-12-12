## Пример проектирование БД

Cобрать и запустить контейнер с postgresql

    `docker-compose up -d`

Cодединиться с БД

    `psql postgresql://userdb:password@localhost:5432/moviesdb`
    
Создать базу данных и пользователя:

`postgres=# CREATE DATABASE moviesdb;`
`postgres=# CREATE USER userdb WITH PASSWORD 'password';`
`postgres=# ALTER ROLE userdb SET client_encoding TO 'utf8';`
`postgres=# ALTER ROLE userdb SET default_transaction_isolation TO 'read committed';`
`postgres=# ALTER ROLE userdb SET timezone TO 'UTC';`
`postgres=# GRANT ALL PRIVILEGES ON DATABASE moviesdb TO userdb;`
`postgres=# \q`

Выполнить все шаги описанные в файле [schema_design.txt](schema_design.txt)

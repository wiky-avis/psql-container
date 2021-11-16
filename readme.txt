docker-compose up -d

psql postgresql://userdb:password@localhost:5432/moviesdb


# создать схему content
CREATE SCHEMA content;

# создаем таблицу film_work и поместим ее в схему content
CREATE TABLE IF NOT EXISTS content.film_work (id uuid PRIMARY KEY,title TEXT NOT NULL,description TEXT,creation_date DATE,certificate TEXT,file_path TEXT,rating
FLOAT,type TEXT not null,created_at timestamp with time zone,updated_at timestamp with time zone );

# Для выборки данных из таблицы необходимо точно указывать название схемы в каждом запросе.
SELECT * FROM content.film_work;

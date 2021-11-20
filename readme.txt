docker-compose up -d

psql postgresql://userdb:password@localhost:5432/moviesdb


# создать схему content
CREATE SCHEMA content;

# создаем таблицу film_work и поместим ее в схему content
CREATE TABLE IF NOT EXISTS content.film_work (id uuid PRIMARY KEY,title TEXT NOT NULL,description TEXT,creation_date DATE,certificate TEXT,file_path TEXT,rating FLOAT,type TEXT not null,created_at timestamp with time zone,updated_at timestamp with time zone );

# Для выборки данных из таблицы необходимо точно указывать название схемы в каждом запросе.
SELECT * FROM content.film_work;

# получить текущий search_path, по которому PostgreSQL ищет таблицы
SHOW search_path;

# Добавить вашу схему в search_path
SET search_path TO content,public;

# Но у такого подхода есть недостаток. Команда SET изменяет параметр search_path на время сессии с базой данных. Поэтому при подключении к СУБД с использованием драйвера psycopg2, необходимо передавать search_path в качестве опции.
conn = psycopg2.connect(
    dbname='movies',
    user='postgres',
    port=5432,
    options=f'-c search_path=content',
 )

-- Установка расширения для генерации UUID
CREATE EXTENSION "uuid-ossp";


-- Сгенерируем данные в интервале с 1900 по 2021 год с шагом в 1 час. В итоге сгенерируется 1060681 записей
INSERT INTO content.film_work (id, title, type,
creation_date, rating) SELECT uuid_generate_v4(), 'some
name', 'movie', date::DATE, floor(random() * 100)
FROM generate_series(
  '1900-01-01'::DATE,
  '2021-01-01'::DATE,
'1 hour'::interval
) date;

# Проанализируем запрос, выполнив команду EXPLAIN ANALYZE. Она покажет план запроса, по которому СУБД будет искать данные.
EXPLAIN ANALYZE SELECT * FROM content.film_work WHERE creation_date = '2020-04-01';

# Создадим индекс. По умолчанию будем использовать тип B-tree, который позволит искать данные за O(log n).
CREATE INDEX film_work_creation_date_idx ON content.film_work(creation_date);


EXPLAIN ANALYZE SELECT * FROM content.film_work WHERE creation_date BETWEEN '2020-04-01' AND '2020-09-01';

# Проиндексируем таблицу film_work и индекс film_work_creation_date_idx и посмотрим сколько памяти они потребляют
\dt+ content.film_work
\di+ content.film_work_creation_date_idx

# Включим в консоли psql вывод времени исполнения команд.
\timing on

# Скопируем текущую таблицу в csv-файл
\copy (select * from content.film_work) to '/output.csv' with csv

# Удалим данные из таблицы.
TRUNCATE content.film_work;

# Удалим индекс.
DROP INDEX content.film_work_creation_date_idx;

# Скопируем данные из файла в таблицу /output.csv замените на путь из операции сохранения данных.
COPY content.film_work FROM '/output.csv' WITH DELIMITER ',' NULL '';

# Вы можете заставить СУБД использовать индекс, добавив в запрос сортировку.
EXPLAIN ANALYZE SELECT * FROM content.film_work WHERE creation_date BETWEEN '1930-01-01' AND '2020-09-01' order by creation_date;

# Создадим таблицу, в которой будут содержаться жанры фильмов.
CREATE TABLE IF NOT EXISTS content.genre_film_work (
 id uuid PRIMARY KEY,
 film_work_id uuid NOT NULL,
 genre TEXT
);


# Создадим индекс
CREATE INDEX genres_idx ON content.genre_film_work(genre);

# Для создания уникального индекса достаточно добавить ключевое слово UNIQUE. По умолчанию создаётся и уникальный индекс на первичный ключ, но он не может быть NULL.
CREATE UNIQUE INDEX some_name_idx ON some_table(some_unique_field)

# Создадим таблицу , в которой будут содержаться участники фильмов, и таблицу   , которая связывает участников с кинофильмами.
CREATE TABLE content.person (
 id uuid PRIMARY KEY
);
CREATE TABLE IF NOT EXISTS content.person_film_work (
 id uuid PRIMARY KEY,
 film_work_id uuid NOT NULL,
 person_id uuid NOT NULL
);

# Создадим уникальный композитный индекс для таблицы film_work_person так, чтобы нельзя было добавить одного актёра несколько раз к одному фильму.
CREATE UNIQUE INDEX film_work_person ON content.person_film_work (film_work_id, person_id);

# Ниже — примеры запросов, которые будут использовать индекс. Учтите, id фильмов и актёров у вас будут отличаться, так как сгенерировались UUID-идентификаторы, которые создают уникальные значения без возможности повторения.
SELECT * FROM content.person_film_work WHERE film_work_id='24804716-0b7d-4c8f-8afe-fea8b0a890c7';

SELECT * FROM content.person_film_work WHERE film_work_id ='24804716-0b7d-4c8f-8afe-fea8b0a890c7' AND person_id='0745df17-c2f4-440a-9514-e4cabc1327b4';

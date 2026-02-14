# Filmorate-ER
![](https://github.com/dmitry-udovin/Filmorate-ER/blob/main/Filmorate%20ER.jpg)

## 🗄 Database Schema (PostgreSQL)

<details>
<summary>Click to expand SQL schema</summary>

```sql
-- =========================
-- Справочные таблицы
-- =========================

CREATE TABLE IF NOT EXISTS ratings (
    rating_id  integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       varchar(50) NOT NULL UNIQUE
);

CREATE TABLE IF NOT EXISTS genres (
    genre_id   integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       varchar(100) NOT NULL UNIQUE
);

-- =========================
-- Основные сущности
-- =========================

CREATE TABLE IF NOT EXISTS users (
    user_id   integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email     varchar(255) NOT NULL UNIQUE,
    login     varchar(50)  NOT NULL UNIQUE,
    name      varchar(255) NOT NULL,
    birthday  date
);

CREATE TABLE IF NOT EXISTS films (
    film_id      integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name         varchar(255) NOT NULL,
    description  text,
    releaseDate  date,
    duration     integer NOT NULL CHECK (duration > 0),
    rating_id    integer REFERENCES ratings(rating_id) ON DELETE SET NULL
);

-- =========================
-- Связующие таблицы
-- =========================

-- Лайки фильмов (многие-ко-многим)
CREATE TABLE IF NOT EXISTS film_likes (
    film_id   integer NOT NULL,
    user_id   integer NOT NULL,
    liked_at  timestamptz NOT NULL DEFAULT now(),

    CONSTRAINT film_likes_pk PRIMARY KEY (film_id, user_id),

    CONSTRAINT film_likes_film_fk FOREIGN KEY (film_id)
        REFERENCES films(film_id) ON DELETE CASCADE,

    CONSTRAINT film_likes_user_fk FOREIGN KEY (user_id)
        REFERENCES users(user_id) ON DELETE CASCADE
);

-- Взаимная дружба пользователей
-- Храним одну запись на пару (user_id < friend_id)
CREATE TABLE IF NOT EXISTS user_friends (
    user_id    BIGINT NOT NULL,
    friend_id  BIGINT NOT NULL,
    status varchar(20) NOT NULL DEFAULT 'UNCONFIRMED',
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT now(),
    confirmed_at TIMESTAMP WITH TIME ZONE,

    CONSTRAINT user_friends_pk PRIMARY KEY (user_id, friend_id),

    CONSTRAINT user_friends_user_fk FOREIGN KEY (user_id)
        REFERENCES users(user_id) ON DELETE CASCADE,

    CONSTRAINT user_friends_friend_fk FOREIGN KEY (friend_id)
        REFERENCES users(user_id) ON DELETE CASCADE,

    CONSTRAINT user_friends_no_self CHECK (user_id <> friend_id),
    CONSTRAINT user_friends_status_check CHECK (status IN ('UNCONFIRMED', 'CONFIRMED'))
);

-- Жанры фильмов (многие-ко-многим)
CREATE TABLE IF NOT EXISTS film_genres (
    film_id   integer NOT NULL,
    genre_id  integer NOT NULL,

    CONSTRAINT film_genres_pk PRIMARY KEY (film_id, genre_id),

    CONSTRAINT film_genres_film_fk FOREIGN KEY (film_id)
        REFERENCES films(film_id) ON DELETE CASCADE,

    CONSTRAINT film_genres_genre_fk FOREIGN KEY (genre_id)
        REFERENCES genres(genre_id) ON DELETE CASCADE
);
```

</details>

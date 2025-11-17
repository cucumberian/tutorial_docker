# Курс карпова

## 4.1 Volume, bind mounts

[](https://docs.docker.com/storage/volumes)

- bind mount
- volume
- tmpfs (не рассматриваем) (оперативка)

- Volume не зависят от хоста
- Volume более безопасный вариант (нельзя монтировать любые файлы)
- Volume находятся в специальном месте (удобно)
- Volumes привязывают только директории

Volume больше про сохранение данных, а bind mount больше про присоединение файлов.

Путь до волюма в системе: `/var/lib/docker/volumes/<volume name>`.

Иногда волюмы могут создаваться автоматически неявно с уникальным идетификатором.
Например при поднятии `yandex/clickhouse-server` создаются наявно волюмы. Послу удаления контейнеров эти аолюмы останутся в системе.

Почему создаются анонимные волюмы?

Анонимные волюмы создаются когда в `Dockerfile` встречается команда `VOLUME`. В случае с clickhouse это строчка `VOLUME [var/lib/clickhouse...]`.

```Dockerfile
FROM node:17

COPY script.js /app/script.js

VOLUME [/app/]

CMD ["node", "/app/script.js"]
```

```sh
docker build -t node-volume .
docker compose up -d --rm node-volume
```

## 4.2 Копирование файлов в рантайм контейнер

Можно скоприровать файлы с хоста внутрь контейнера. И обратною

```sh
docker cp <host_path> <container>:<path>
docker cp ./todo_result/ tg-bot:/app/todo_result/

docker cp <container>:<path> <host_path>
docker cp tg-bot:/app/todo_result/todolist.csv todo_result/todolist.csv
```

### docker volume

```sh
docker volume ls    # вывести список волюмов
docker voluem create <volume name>  # создание волюма
docker volum rm <volume name>   # удаление волюма

docker volume prune --force     # удаление неиспользуемых анонимных волюмов
docker volume prune -a --force  # удаление всех неиспользуеых волюмов
```

### Опасность при работе с bind mount

По умолчанию в конейнере суперпользовтель.
Поэтому из конетйнера можно получить доступ к файлам, которые доступны руту.

Поэтому лучше монтировать отдельные файлы, а не каталоги, даже если несколько раз приходится писать -v.

И не надо монтировать корень `/`.

Ещё можно потерять взаимодуйствие с логами, если их создвет контейнер, анпример `clickhouse`, а у текущего пользовтеля нету прав.

Поэтому можно монтировать в режиме чтения

```sh
docker run -v <host_path>:<cont_path>:ro image
```

или использовать директиву `USER` в Dockerfile, или указывать пользователя при запуске контейнера:

```sh
id -U   # узнаём ид пользователя
docker run --user=1000 image
```

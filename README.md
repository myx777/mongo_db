# Шпаргалка

### 1. установка mongo и mongo-express через docker

<details>

```
docker pull mongo

```

```
docker pull mongo
```
</details>

если mongo и приложение в разных контейнерах 
то у них будет разная сеть и работать не будет для этого нужно создать сеть для
общения между контейнерами:
```
 docker network create test-network
```

`test-network` - это пример можно придумать свою сеть
в результате в терминале выведется что-то наподобие
`58ff75a95d4b2a8b42cba6d5b58259635c48a90229a5e30fb56ac17bc2107e48`

### 2. создаем docker-compose.yml

<details>

файл взят [тут](https://hub.docker.com/_/mongo)

```
services:

  mongo:
    image: mongo
    restart: always
    volumes: // Добавляем привязку тома для хранения данных MongoDB локально на сервере
      - ./mongodb_data:/data/db 
    environment:
      MONGO_INITDB_ROOT_USERNAME: root 
      MONGO_INITDB_ROOT_PASSWORD: example 
    networks: // добавил для подключения к созданой сети
      - test-network

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/
      ME_CONFIG_BASICAUTH: false
    networks: // добавил для подключения к созданой сети
      - test-network

networks: // добавил для подключения к созданой сети
  test-network:
    external: true

```

</details>

### 3. поднял контейнеры

```
docker compose up -d
```
### 4. проверил mongo-express по адресу: _localhost:8081_ и создал новую БД _library_

### 5. создал нового пользователя с правами для будущего отдельного подключения им к БД library
<details>
посмотрел id контейнера mongo

```
docker ps
```
получил такое:

```
CONTAINER ID   IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
d76efb11b666   node:20.10-alpine   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:80->3025/tcp     library-main-1
17807aee003c   node:20.10-alpine   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:3001->3001/tcp   library-counter-1
5d3b96a1ad55   mongo-express       "/sbin/tini -- /dock…"   4 minutes ago        Up 4 minutes        0.0.0.0:8081->8081/tcp   mongo-mongo-express-1
52ac7265326e   mongo               "docker-entrypoint.s…"   4 minutes ago        Up 4 minutes        27017/tcp                mongo-mongo-1

```

вошел внутрь контейнера по его id (шаг раньше)

```
docker exec -it 52ac7265326e sh 

```

shell`а нет как я понял (но это не точно), для управлением mongo набрал 

```
mongosh

```

получил такое:

```
Current Mongosh Log ID: 664db92c1921b920ac99ea71
Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.2.5
Using MongoDB:          7.0.9
Using Mongosh:          2.2.5

For mongosh info see: https://docs.mongodb.com/mongodb-shell/


To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
You can opt-out by running the disableTelemetry() command.

test> 
```

перешел в админку и теперь стали доступны команды shell

```
use admin
```

пршел аутентиф

```
admin> db.auth("root", "example")
```

посмотрел список контейнеров
```
admin> show databases
```
получил

```
admin    100.00 KiB
config    60.00 KiB
library   56.00 KiB
local     72.00 KiB

```

перешел в library

```
use library
```

и создал нового пользователя

```
 db.createUser({user:"library", password:"library", roles:[{role: "readWrite", db:"library"}]})
```
</details>

###  в приложении в .env для подключения к базе c помощью заранее установленного  _mongoose_ прописал 
```
DB_USER=library
DB_PASSWORD=library
DB_NAME=library
DB_HOST=mongo
```

index.js, помимо другого кода

```javascript
const mongoose = require('mongoose');
const PORT = process.env.PORT || 3000;
const URL_DB = `mongodb://${process.env.DB_USER}:${process.env.DB_PASSWORD}@${process.env.DB_HOST}/${process.env.DB_NAME}`;


async function start (URL_DB, PORT) {
    try {
        await mongoose.connect(URL_DB);
        console.log("Подключен к базе данных")
        app.listen(PORT)
        console.log(`Сервер на порту: ${PORT}`)
    } catch (e) {
        console.error("Ошибка:", e)
    }
}
```
###  в docker-compose.yml добавил сеть как в файле для mongo

<details>

посмотреть [тут](https://github.com/myx777/library/blob/mongo/docker-compose.dev.yml)

</details>

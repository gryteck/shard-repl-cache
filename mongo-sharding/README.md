Инструкция по поднятию:

Поднимаем docker compose:
```bash
docker compose up --build -d
```
Инициализируем конфигурационный кластер
```bash
docker exec -it configsvr mongosh --port 27019 --eval 'rs.initiate({_id: "configReplSet", configsvr: true, members: [{ _id: 0, host: "configsvr:27019" }]})'
```

Инициализируем 1 шард
```bash
docker exec -it shard1 mongosh --port 27018 --eval 'rs.initiate({_id: "shard1ReplSet", members: [{ _id: 0, host: "shard1:27018" }]})'
```

Инициализируем 2 шард
```bash
docker exec -it shard2 mongosh --port 27016 --eval 'rs.initiate({_id: "shard2ReplSet", members: [{ _id: 0, host: "shard2:27016" }]})'
```

Добавляем шарды в роутер
```bash
docker exec -it mongos mongosh --port 27020 --eval 'sh.addShard("shard1ReplSet/shard1:27018")'
docker exec -it mongos mongosh --port 27020 --eval 'sh.addShard("shard2ReplSet/shard2:27016")'
```

Добавляем шардирование в роутере на основе хэширования
```bash
docker exec -it mongos mongosh --port 27020 --eval 'sh.enableSharding("somedb")'
docker exec -it mongos mongosh --port 27020 --eval 'sh.shardCollection("somedb.helloDoc", { name: "hashed" })'
```

Генерируем 1000 документов в роутере
```bash
docker exec -it mongos mongosh --port 27020 --eval 'use("somedb"); (function() { for (let i = 0; i < 1000; i++) db.helloDoc.insertOne({age: i, name: "ly" + i}); })()'
```
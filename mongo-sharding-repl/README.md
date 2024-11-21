Инструкция по поднятию:

Поднимаем docker compose:
```bash
docker compose up --build -d
```
Инициализируем конфигурационный кластер
```bash
docker compose exec -T configsvr1 mongosh <<EOF
rs.initiate({
  _id: "configReplSet",
  configsvr: true,
  members: [
    { _id: 0, host: "configsvr1:27017" },
    { _id: 1, host: "configsvr2:27017" },
    { _id: 2, host: "configsvr3:27017" }
  ]
});
EOF
```

Инициализируем 1 шард
```bash
docker compose exec -T shard1_primary mongosh <<EOF
rs.initiate({
  _id: "shard1ReplSet",
  members: [
    { _id: 0, host: "shard1_primary:27017", priority: 2 },
    { _id: 1, host: "shard1_secondary1:27017", priority: 1 },
    { _id: 2, host: "shard1_secondary2:27017", priority: 1 }
  ]
});
EOF
```

Инициализируем 2 шард
```bash
docker compose exec -T shard2_primary mongosh <<EOF
rs.initiate({
  _id: "shard2ReplSet",
  members: [
    { _id: 0, host: "shard2_primary:27017", priority: 2 },
    { _id: 1, host: "shard2_secondary1:27017", priority: 1 },
    { _id: 2, host: "shard2_secondary2:27017", priority: 1 }
  ]
});
EOF
```

Добавляем шарды в роутер
```bash
docker compose exec -T mongos mongosh <<EOF
sh.addShard("shard1ReplSet/shard1_primary:27017,shard1_secondary1:27017,shard1_secondary2:27017");
sh.addShard("shard2ReplSet/shard2_primary:27017,shard2_secondary1:27017,shard2_secondary2:27017");
EOF
```

Добавляем шардирование в роутере на основе хэширования
```bash
docker exec -it mongos mongosh --port 27017 --eval 'sh.enableSharding("somedb")'
docker exec -it mongos mongosh --port 27017 --eval 'sh.shardCollection("somedb.helloDoc", { name: "hashed" })'
```

Генерируем 1000 документов в роутере
```bash
docker exec -it mongos mongosh --port 27017 --eval 'use("somedb"); (function() { for (let i = 0; i < 1000; i++) db.helloDoc.insertOne({age: i, name: "ly" + i}); })()'
```
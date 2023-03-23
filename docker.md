![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/wbhiRD1.webp)

```
"registry-mirrors": [ "https://dockerproxy.com" ]
```

Desenvolvimento local e depuração

```
PG_HOST=pg

REDIS_HOST=redis
```

mudar para

```
PG_HOST=127.0.0.1

REDIS_HOST=127.0.0.1

```

E comente `NODE_ENV=production`
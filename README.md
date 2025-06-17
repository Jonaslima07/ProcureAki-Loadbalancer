# Tutorial de como colocar balanceador de carga do nginx em uma aplicação web.

## Etapa 1:
- primeiro você tem que ter o nginx instalado em sua máquina,caso não tenha, execute o seguinte comando:

```
sudo apt update
sudo apt install nginx
``` 

## Etapa 2:
- abra seu projeto:
```
git clone https://github.com/fulanodetal/ProjetoInovador.git
cd ProjetoInovador
```
## Etapa 3:
- Instale as dependências:

```
npm install
```

## Etapa 4: 
- Execute o comando a seguir para deixar sua aplicação pronta para produção:
```
npm run build
```
- Depois que executar, seu projeto irá aparecer a pasta dist.

## Etapa 5:
- crie o arquivo Dockerfile de configuração: 
```
FROM nginx:alpine
COPY dist/ /usr/share/nginx/html
```

## Etapa 6:
- criar o arquivo nginx.conf:
```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }
}

```
## Etapa 7:
- criar o arquivo e aqui configuramos para o loadbalancer default.conf:

```
upstream nodes_loadbalancer {
    server node1:80;
    server node2:80;
    server node3:80;
    server node4:80;
    server node5:80;
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://nodes_loadbalancer;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

}
```

## Etapa 8:
- criar o arquivo docker-compose.yml:
```
version: "3.8"

services:
  loadbalancer:
    image: nginx:latest
    container_name: loadbalancer
    ports:
      - "80:80"
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - node1
      - node2
      - node3
      - node4
      - node5
    networks:
      - web2-net

  node1:
    build:
      context: .
      dockerfile: dockerfile.app
    container_name: node1
    networks:
      - web2-net

  node2:
    build:
      context: ./build-node2
      dockerfile: ../dockerfile.node
    container_name: node2
    networks:
      - web2-net

  node3:
    build:
      context: ./build-node3
      dockerfile: ../dockerfile.node
    container_name: node3
    networks:
      - web2-net

  node4:
    build:
      context: ./build-node4
      dockerfile: ../dockerfile.node
    container_name: node4
    networks:
      - web2-net

  node5:
    build:
      context: ./build-node5
      dockerfile: ../dockerfile.node
    container_name: node5
    networks:
      - web2-net

networks:
  web2-net:
    driver: bridge

```
## Etapa 9:
- criamos as pastas dos nos cada um com seus arquivos estaticos:
```
mkdir build-node2 build-node3 build-node4 build-node5

echo "<h1>Node 2</h1>" > build-node2/index.html
echo "<h1>Node3</h1>" > build-node3/index.html
echo "<h1>Node4</h1>" > build-node4/index.html
echo "<h1>Node5</h1>" > build-node5/index.html


```
## Etapa 10:
- criamos o arquivo dockerfile.node onde tera as configurações dos nodes2 a node5 com as mensagens diferentes:

```
nano dockerfile.node

```

## Etapa 11:
- executamos esses comandos para você garantir que o conteúdo está correto nas pastas, rode: 
```
docker compose down
docker compose build --no-cache
docker compose up -d

``` 

# depois disso rode com localhost:
```
http://localhost
```

## Teste de carga:

```
Jonas@Tsi:~/Documentos/GCSI/ProcureAki$ ab -n 100 -c 10 http://localhost/
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:        nginx/1.27.5
Server Hostname:        localhost
Server Port:            80

Document Path:          /
Document Length:        732 bytes

Concurrency Level:      10
Time taken for tests:   0.046 seconds
Complete requests:      100
Failed requests:        20
   (Connect: 0, Receive: 0, Length: 20, Exceptions: 0)
Total transferred:      96400 bytes
HTML transferred:       73100 bytes
Requests per second:    2171.22 [#/sec] (mean)
Time per request:       4.606 [ms] (mean)
Time per request:       0.461 [ms] (mean, across all concurrent requests)
Transfer rate:          2044.00 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.4      0       2
Processing:     1    4   2.4      3      15
Waiting:        1    4   2.4      3      15
Total:          1    4   2.5      4      16

Percentage of the requests served within a certain time (ms)
  50%      4
  66%      4
  75%      5
  80%      6
  90%      8
  95%      8
  98%     14
  99%     16
 100%     16 (longest request)
Jonas@Tsi:~/Documentos/GCSI/ProcureAki$ 

``` 

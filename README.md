# DOCUMENTO DE PROCEDIMENTO DE INSTALAÇÃO


## 1. Visão Geral do Documento


Este documento descreve a instalação do docker registry. Software usado no armazenamento de imagem do docker, as quais são usado nos projetos de desenvolvimento da SEAS.


## 2. Pré Requisitos

Esta instalação parte do pressuposto  que o docker esteja já instalado no servidor.
A versão utilizada no momento desta instalação foi a versão 18.05.0-ce, build f150324

Intalar o docker

```
curl -sSL https://get.docker.com/ | sh
```

## 3. Baixando as imagens usadas no Docker Registry

```
docker pull registry:2
docker pull konradkleine/docker-registry-frontend:v2
```

## 4. Criação do diretório de volume
Para que a imagens seja armazenadas e não sejam perdidas com a deleção do container do docker registry, foi criado o diretório de armazenamento das imagem no seguinte path: “/home/backup/docker/registry”.

```
mkdir /home/backup/docker/registry
```

Criação do container do Docker Registry (no servidor)
O container foi criado com os seguintes parâmetros de comando:

```
# docker run -d -p 5000:5000 --restart=always --name registry 
-v /backup/home/docker/registry:/var/lib/registry registry:2
```

## 5. Criação do container da interface grafica do Docker Registry (no servidor)
Esse container tem como objetivo ter uma melhor visualização das imagens armazenadas no docker registry. Onde o acessor será realizado usando a seguinte url: http://192.168.0.1:8080

```
docker run -d -p 8080:80 -e ENV_DOCKER_REGISTRY_HOST=192.168.0.1 
-e ENV_DOCKER_REGISTRY_PORT=5000 --name registry-v2-gui 
--restart=always konradkleine/docker-registry-frontend:v2
```

## 6. Configurações do lado cliente 
Será necessario altera o seguinte arquivo: “/lib/systemd/system/docker.service” e adicionar a seguinte entrada:”ExecStart=/usr/bin/dockerd --insecure-registry 192.168.0.1:5000"

```
vim  lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd --insecure-registry 192.168.0.1:5000
```


Reiniciando o docker
```
systemctl enable docker.service
systemctl restart docker.service
systemctl status docker.service
```


## 7. Modo de uso 

Baixando uma imagem
```
docker pull mysql:5.7
```

Visualizado a imagem
```
docker images
```

mysql		5.7	0d16d0a97dd1	2 weeks	ago	372MB

Tageando imagem

As imagens são tageadas usando o seguinte comando:
docker tar <ID_DA_IMAGEM> <IP>:PORTA/<IMAGEM>:<VERSÃO>

```
docker tag  0d16d0a97dd1 192.168.0.1:5000/mysql:5.7
```

Enviando uma imagem para o servidor
As images são enviadas usando o seguinte comando: 
docker push <IP>:PORTA/<IMAGEM>:<VERSÃO>

```
docker push 192.168.0.1:5000/mysql:5.7
```

Baixando uma imagem do servidor
As images são baixadas usando o seguinte comando: 
docker pull <IP>:PORTA/<IMAGEM>:<VERSÃO>

```
docker pull 192.168.0.1:5000/mysql:5.7
```


## 8. Reiniciar o Docker

```
/etc/init.d/docker restart
```

## 9. OBS
Foi criado um redirecionamento no firewall (pfsense), com o objetivo das Vms de produção usarem o docker registry atraves do seguinte endereço: http://172.24.178.2:5000

docker run -p 5432:5432 --name postgresql --restart always -v /SERVICOS/postgresql:/var/lib/postgresql/data -e POSTGRES_PASSWORD=pr89e5p1onjiU -d postgres:9.6

# Levantando as imagens

## Postegres
```
docker run -p 5432:5432 --name postgresql --restart always 
-v /SERVICOS/postgresql:/var/lib/postgresql/data 
-e POSTGRES_PASSWORD=pr89e5p1onjiU -d postgres:9.6
```

## Mancache
```
docker run -p 11211:11211 --name my-memcache -d memcached
```

## Aplicação
```
docker run --p 7000:3000 \
-v $(pwd)/SERVICOS/database/database-principal.yml:/usr/src/app/config/database.yml \
-v $(pwd)/SERVICOS/old_srv_data/srv2/srv/docker/production.rb:/usr/src/config/environments/production.rb \
-v $(pwd)/SERVICOS/old_srv_data/srv2/srv/docker/system/:/usr/src/public/system/ \
--env RAILS_RELATIVE_URL_ROOT="/erp" --link my-memcache:memcached --link postgresql:psql \
--name imagem_producao -d 192.168.0.1:5000/imagem_producao
```

# Cron para automatizar periodocamente a aplicação

```
#!/bin/bash
echo -n "$(date -R): Checking if we need an update..."

APPNAME=seas_principal7000

docker pull registry.appbridge.space/principal | grep "Status: Downloaded newer image" 
&& sudo docker stop $APPNAME && sudo docker rm $APPNAME && sudo docker run --name $APPNAME 
--publish 7000:3000 -v /SERVICOS/old_srv_data/srv2/srv/docker/database-principal.yml:/usr/src/app/config/database.yml 
-d -v /SERVICOS/old_srv_data/srv2/srv/docker/production.rb:/usr/src/config/environments/production.rb 
-v /SERVICOS/old_srv_data/srv2/srv/docker/system/:/usr/src/public/system/ 
--env RAILS_RELATIVE_URL_ROOT="/sigi" --link my-memcache:memcached --link postgresql:psql 
registry.appbridge.space/principal && 
docker exec $APPNAME rake db:migrate && docker exec $APPNAME rake assets:precompile

```
# Comando basicos 

* Instalar Docker
```
wget -qO- https://get.docker.com/ | sh
```

* Entrar no modo iterativo
```
docker  exec -it  00be3855d14a /bin/bash
```

* Sair e derrubar a o container
```
exit
```

* Sair sem derrubar
```
Control PQ
```

* Monitorar a memória
```
docker stats
```

* Salvar e carregar imagens
```
docker save memcached > memcached.tar
docker load --input memcached.tar
```

* Contanier para imagem
```
docker commit 3b64e4b39127 imagemnginx:versao1
```

* renomear imagem
```
docker tag old new:1.0.0
```

* Renomear imagem
```
docker rename my_container my_new_container
```

* nginx 
```
docker run -p 80:80 --name servidornginx -v $(pwd)/angular_app:/usr/share/nginx/html -d  nginx
```

* levanta um docker-compose
```
docker-compose up -d
```

* derruba um docker-compose
```
docker-compose down
```

* Levanta a aplicação rails
```
docker-cedockompose run app bundle exec rails new . -d postgresql
docker-compose build
```

* Constroi imagem a partir de um Dockerfile
```
docker build . -t api-pessoas
```

* docker da imagem feita pelo arquivo Docker
```
docker run -p 8086:8086 --name api-pessoas --link postgres94:postegres -d api-pessoas
```

* Comandos
```
docker system df
docker container logs -f cont
docker rmi $(docker images -q -a)
```

* Apaga todas as imagens
```
docker rmi -f $(docker images -q -a)
```

# MXDATA

Olá!
Aqui estão as especificações do projeto e outras informações.

## Recursos usados
* Reactjs
* Bootstrap
* Jquery
* Sequelize
* Express
* Jest
* Docker

## Servers

* Nodejs
* Postgres

## Repositórios dos arquivos

* [API](https://github.com/HeiterDeveloper/mxdt-api)
* [SITE](https://github.com/HeiterDeveloper/mxdt-web)

## Como executar

>Baixe as seguintes imagens do docker:
```console
docker pull debian
docker pull node
docker pull nginx
```

>Crie o arquivo Dockerfile, para gerar a imagem do postgres
```console
FROM debian

RUN apt-get update

RUN apt-get -y install postgresql

# Run the rest of the commands as the ``postgres`` user created by the ``postgres-11`` package when it was ``apt-get installed``
USER postgres

# Create user and database
RUN    /etc/init.d/postgresql start &&\
    psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'docker';" &&\
    createdb -O docker docker &&\
    psql --command "CREATE USER dockertest WITH SUPERUSER PASSWORD 'dockertest';" &&\
    createdb -O dockertest dockertest

# Adjust PostgreSQL configuration so that remote connections to the
# database are possible.
RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/11/main/pg_hba.conf

# And add ``listen_addresses`` to ``/etc/postgresql/11/main/postgresql.conf``
RUN echo "listen_addresses='*'" >> /etc/postgresql/11/main/postgresql.conf

# Expose the PostgreSQL port
EXPOSE 5432

# Add VOLUMEs to allow backup of config, logs and databases
VOLUME  ["/etc/postgresql", "/var/log/postgresql", "/var/lib/postgresql"]

# Set the default command to run when starting the container
CMD ["/usr/lib/postgresql/11/bin/postgres", "-D", "/var/lib/postgresql/11/main", "-c", "config_file=/etc/postgresql/11/main/postgresql.conf"]

```
>Crie a imagem com o comando:
```console
docker build -t db-api .
```

>Entre na pasta da api e crie o arquivo Dockerfile

```console
FROM node

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
COPY package.json ./

RUN npm install

# Bundle app source
COPY . .

EXPOSE 8090

CMD [ "node", "/usr/src/app/src/server.js" ]
```
>Crie a imagem com o comando:
```console
docker build -t node-api .
```

>Entre na pasta do site e crie o arquivo Dockerfile:

```console
FROM nginx
COPY build /usr/share/nginx/html
COPY default.conf /etc/nginx/conf.d/default.conf
```

>Crie a imagem com o comando:
```console
docker build -t site-react .
```

>Com o docker compose instalado, crie o arquivo docker-compose.yml:

```console
version: "3.3"

services:
    db:
        container_name: db-api
        image: db-api
        ports:
            - "5432:5432"
        volumes: 
            - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
            - "dbdata:/var/lib/postgresql/data"

    api:
        container_name: node-api
        image: node-api
        ports:
            - "8090:8090"
        depends_on: 
            - db
        command:  node src/server.js
        environment: 
            - NODE_ENV=production

    web:
        container_name: site-react
        image: site-react
        ports:
            - "8080:80"
        depends_on: 
            - api 
            - db
volumes:
    dbdata:
```
    
>Inicie os containers

```console
   docker-compose up
```
    
>Rode o comando para popular o bando de dados (migrations e seeds)
    
```console
   docker exec -it node-api  npx sequelize db:migrate
   docker exec -it node-api npx sequelize db:seed:all
```

## Acesse a aplicação em:
```console
http://localhost:8080
```

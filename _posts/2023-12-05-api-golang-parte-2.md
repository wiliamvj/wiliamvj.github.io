---
title: API completa em Golang - Parte 2
author: wiliamvj
date: 2023-12-01 20:53:00 +0800
categories: [Golang, API]
tags: [Golang, SQL, SQLC, API, Migrations, Swagger, Docs]
pin: false
math: false
mermaid: true
image:
  path: /commons/thumbs/tydfjkGH768hHkh.png
  lqip: /commons/thumbs/tydfjkGH768hHkh.png
  alt: CRUD Golang.
---

## O que vamos fazer?

Na parte 2 do nosso crud, vamos configurar nosso banco de dados com docker, sqlc, migrations, go chi, vamos criar nossas primeiras tabelas no banco e também vamos configurar as assinaturas das nossas interfaces.

Se ainda não viu a [parte 1](https://wiliamvj.com/posts/api-golang-parte-1/){:target="\_blank"}, leia esse post primeiro.

## Configurando o banco de dados

Vamos criar na raiz do projeto um arquivo `docker-compose.yml` para criar uma imagem do postgreSQL:

```yml
version: "3.9"

services:
  postgres:
    container_name: postgres_apigo
    image: postgres:14.5
    environment:
      POSTGRES_HOST: ${POSTGRES_HOST}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_DB: ${POSTGRES_DB}
      PG_DATA: /var/lib/postgresql/data
    ports:
      - 5432:5432
    volumes:
      - apigo:/var/lib/postgresql/data
volumes:
  apigo:
```

Isso vai criar uma imagem do postgreSQL ee criar um volume, para garantir que nossos dados não sejam perdidos sempre que o docker parar. Vamos precisar também de um arquivo `.env` para guardadas as credenciais de acesso ao banco de dados, também na raiz do projeto:

```go
  POSTGRES_DB="golang_api_users"
  POSTGRES_USER="golang_api_users"
  POSTGRES_PASSWORD="golang_api_users"
  POSTGRES_HOST="localhost"
  POSTGRES_PORT="5432"
  DATABASE_URL="postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}?sslmode=disable"
```

Agora ao rodar o comando `docker compose up -d` nosso banco vai ser criado e estará pronto para uso.

## Configurando o Golang Migrate

Agora vamos configurar o golang migrate para lidar com nossas migrations, já fiz um [post](https://wiliamvj.com/posts/migrations-golang/){:target="\_blank"} sobre o assunto, recomendo ler antes.

Os comandos do golang migrate são um pouco extenso e cnsativo de digitar, por isso vamos criar o arquivo `makefile` para facilitar o uso dos comandos:

Crie um arquivo `makefile` na raiz do projeto:

```makefile
  include .env

  create_migration:
    migrate create -ext=sql -dir=internal/database/migrations -seq init

  migrate_up:
    migrate -path=internal/database/migrations -database "postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}?sslmode=disable" -verbose up

  migrate_down:
    migrate -path=internal/database/migrations -database "postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}?sslmode=disable" -verbose down

  .PHONY: create_migration migrate_up migrate_down
```

No arquivo incluimos o nosso `.env` para o makefile poder ter acesso as credenciais do banco de dados, depois criamos 3 comandos, para gerar novas migrations sequenciais, uma para aplicar as migrations e outra para reverter.

Vamos rodar o comando para criar as migrations:

```bash
  make create_migration
```

Você vai perceber que foi gerado uma pasta chamada migrations dentro de **internal/migrations\*** e dois arquivos `.sql`, vai ser nesses arquivos que vamos escrever nosso sql para manipular nosso banco de dados. Você pode alterar o caminho em que as migrations foram geradas, basta alterar no arquivo `makefile`.

## Criando nossa primeira tabela

Com nossa migrations pronta, podemos criar nossa primeira tabela, vamos criar a tabela user com os primeiros campos que vamos precisar, vamos criar no arquivo `up` gerado pelo golang-migrate, no meu caso chamado `000001_init.up.sql`:

```sql
  CREATE TABLE users (
    id VARCHAR(36) NOT NULL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP(3) NOT NULL
  );
```

Repare que o `id` da nossa tabela `users` é um `VARCHAR(36)`, vamos usar o [uuid](https://bit.ly/414qfto){:target="\_blank"}, poderíamos usar uma extensão do postgreSQL como o [uuid-ossp](https://www.postgresql.org/docs/current/uuid-ossp.html){:target="\_blank"} para gerar nossos ids, porém se nossa aplicação ficar responsável por gerar seu próprios ids, teremos a possibilidade de saber o `id` do nosso usuário antes mesmo de salvar no banco de dados, mas se preferir você pode instalar essa extensão e deixar o banco de dados responsável por gerar o `id` do usuário.

Nossa migration que desfaz isso, chamamos de down `000001_init.down.sql` ficaria assim:

```sql
  DROP TABLE IF EXISTS users;
```

Isso remove a tabela que criamos. Agora vamos rodar as migrations com nosso `make`:

```sql
  make migrate_up
```

Se tudo rodar sem erro, isso vai aparecer no seu terminal:

```bash
  2023/12/05 09:13:51 Start buffering 1/u init
  2023/12/05 09:13:51 Read and execute 1/u init
  2023/12/05 09:13:51 Finished 1/u init (read 4.167125ms, ran 6.650125ms)
  2023/12/05 09:13:51 Finished after 13.853791ms
  2023/12/05 09:13:51 Closing source and database
```

Agora se acessar a tabela, usando o [pgAdmin](https://www.pgadmin.org/){:target="\_blank"} ou [DBeaver](https://dbeaver.io/download/){:target="\_blank"} ou [BeeKeeper](https://www.beekeeperstudio.io/){:target="\_blank"}, vai ver ver que a tabela users foi gerada com sucesso.

![SQL struct](/commons/posts/2023-12-05-api-golang-parte-2/fgGHJ768ghghjghG.png){: .normal }

Rodando o comando, nossa tabela é deletada, mas cuidado! Esse comando apaga todos os dados!

```sql
  make migrate_down
```

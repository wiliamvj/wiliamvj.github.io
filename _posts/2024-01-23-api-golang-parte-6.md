---
title: API completa em Golang - Parte 6
author: wiliamvj
date: 2024-01-06T13:57:31.491Z
categories: [Golang, API]
tags: [Golang, SQL, SQLC, API, Migrations, Swagger, Docs]
pin: false
math: false
mermaid: true
image:
  path: /commons/thumbs/uidGhfFHhkn90ndftqcvkFfgFasSd.png
  lqip: /commons/thumbs/uidGhfFHhkn90ndftqcvkFfgFasSd.png
  alt: CRUD Golang.
---

## O que vamos fazer?

Na parte 6 do nosso crud vamos finalizar nosso repository, salvando os dados do usuários no banco de dados utilizando o [sqlc](https://sqlc.dev/).

Se ainda não viu os posts anteriores leia eles primeiro.

[parte 1](https://wiliamvj.com/posts/api-golang-parte-1/){:target="\_blank"} |
[parte 2](https://wiliamvj.com/posts/api-golang-parte-2/){:target="\_blank"} |
[parte 3](https://wiliamvj.com/posts/api-golang-parte-3/){:target="\_blank"} |
[parte 4](https://wiliamvj.com/posts/api-golang-parte-4/){:target="\_blank"} |
[parte 5](https://wiliamvj.com/posts/api-golang-parte-5/){:target="\_blank"} |

## Criandos nossas queries

Primeiro vamos criar nossas querias para salvar um usuário, buscar um usuário pelo id, buscar todos os usuários, deletar usuário, atualizar um usuário e atualizar a senha do usuário.

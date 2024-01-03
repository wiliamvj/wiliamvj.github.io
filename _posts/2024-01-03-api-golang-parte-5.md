---
title: API completa em Golang - Parte 5
author: wiliamvj
date: 2024-01-03 13:07:00 +0800
categories: [Golang, API]
tags: [Golang, SQL, SQLC, API, Migrations, Swagger, Docs]
pin: false
math: false
mermaid: true
image:
  path: /commons/thumbs/htyf78HJsvdfczaqfg6jiopl.png
  lqip: /commons/thumbs/htyf78HJsvdfczaqfg6jiopl.png
  alt: CRUD Golang.
---

## O que vamos fazer?

Na parte 5 do nosso crud, vamos fazer a autenticação do usuário, criando um endpoint de login que retorna um JWT, vamos proteger as rotas, impedindo o uso sem token, vamos criar uma função que recebe o token e devolve ums estrutura com os dados salvos no token e ainda vamos adicionar um middleware de logs que adiciona dados do usuário que fez a chamada a nossa api.

Se ainda não viu os posts anteriores leia eles primeiro.

[parte 1](https://wiliamvj.com/posts/api-golang-parte-1/){:target="\_blank"} |
[parte 2](https://wiliamvj.com/posts/api-golang-parte-2/){:target="\_blank"} |
[parte 3](https://wiliamvj.com/posts/api-golang-parte-3/){:target="\_blank"} |
[parte 4](https://wiliamvj.com/posts/api-golang-parte-4/){:target="\_blank"} |

---
title: API completa em Golang - Parte 7
author: wiliamvj
date: 2024-02-02T18:57:31.491Z
categories: [Golang, API]
tags: [Golang, SQL, SQLC, API, Migrations, Swagger, Docs]
pin: false
math: false
mermaid: true
image:
  path: /commons/thumbs/thumb_re435g5G5gGfgfOp.png
  lqip: /commons/thumbs/thumb_re435g5G5gGfgfOp.png
  alt: CRUD Golang.
---

## O que vamos fazer?

Na parte 7 do nosso crud vamos criar a funcionalidade de cadastro de produtos e categorias, vamos criar a funcionalidade de cadastrar produto e categoria, editar produto e categoria, listar todos os produtos, listar um produto pelo id e deletar produto e categoria. Vamos trabalhar com relacionamentos one to many e many to many, um produto pertence a um usuário e um usuário pode ter muitos produtos e uma categoria pode ter muitos produtos e um produto pode ter muitas categorias. Vamos fazer tudo nesse post, por isso deve ser um dos maiores da nossa série.

Se ainda não viu os posts anteriores leia eles primeiro.

[parte 1](https://wiliamvj.com/posts/api-golang-parte-1/){:target="\_blank"} |
[parte 2](https://wiliamvj.com/posts/api-golang-parte-2/){:target="\_blank"} |
[parte 3](https://wiliamvj.com/posts/api-golang-parte-3/){:target="\_blank"} |
[parte 4](https://wiliamvj.com/posts/api-golang-parte-4/){:target="\_blank"} |
[parte 5](https://wiliamvj.com/posts/api-golang-parte-5/){:target="\_blank"} |
[parte 6](https://wiliamvj.com/posts/api-golang-parte-6/){:target="\_blank"}

## Criando as entidades

Primeiramente vamos criar nossas entidades de **product** e **category**.

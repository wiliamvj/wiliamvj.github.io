---
title: API completa em Golang - Parte 3
author: wiliamvj
date: 2023-12-01 20:53:00 +0800
categories: [Golang, API]
tags: [Golang, SQL, SQLC, API, Migrations, Swagger, Docs]
pin: false
math: false
mermaid: true
image:
  path: /commons/thumbs/ODggcvG44fhfry546K.png
  lqip: /commons/thumbs/ODggcvG44fhfry546K.png
  alt: CRUD Golang.
---

## O que vamos fazer?

Na parte 3 do nosso crud, vamos configurar nosso dto para realizar as validações dos dados de entrada da nossa api, e vamos criar o handler do usuário.

Se ainda não viu a [parte 1](https://wiliamvj.com/posts/api-golang-parte-1/){:target="\_blank"} e a [parte 2](https://wiliamvj.com/posts/api-golang-parte-2/){:target="\_blank"} leia esse post primeiro.

## Padronizando erros http

Antes de iniciar com o dto, precisamos padronizar nossos erros http, é uma boa prática criar uma padronização dos nossos erros http que retornamos para nosso cliente, por isso vamos retornar sempre nesse padrão:

```json
{
  "message": "message error",
  "error": "bad_request",
  "code": 400,
  "fields": []
}
```

Vamos retornar sempre nesse formato, o campo `fields` vai ser preenchido pelo go playground com os dados dos campos que estão errados, ficando nesse formato:

```json
{
  "message": "message error",
  "error": "bad_request",
  "code": 400,
  "fields": [
    {
      "field": "email",
      "value": "email.com",
      "message": "invalid email"
    }
  ]
}
```

Dessa forma ajuda o consumidor da nossa api a identificar os campos errados.

Vamos criar dentro da pasta **handler** uma pasta chamada **httperr** e um arquivo dentro chamado `httperr`, ficando assim:

```go
  package httperr

  import "net/http"

  type RestErr struct {
    Message string   `json:"message"`
    Err     string   `json:"error,omitempty"`
    Code    int      `json:"code"`
    Fields  []Fields `json:"fields,omitempty"`
  }

  type Fields struct {
    Field   string      `json:"field"`
    Value   interface{} `json:"value,omitempty"`
    Message string      `json:"message"`
  }

  func (r *RestErr) Error() string {
    return r.Message
  }

  func NewRestErr(m, e string, c int, f []Fields) *RestErr {
    return &RestErr{
      Message: m,
      Err:     e,
      Code:    c,
      Fields:  f,
    }
  }

  func NewBadRequestError(message string) *RestErr {
    return &RestErr{
      Message: message,
      Err:     "bad_request",
      Code:    http.StatusBadRequest,
    }
  }

  func NewUnauthorizedRequestError(message string) *RestErr {
    return &RestErr{
      Message: message,
      Err:     "unauthorized",
      Code:    http.StatusUnauthorized,
    }
  }

  func NewBadRequestValidationError(m string, c []Fields) *RestErr {
    return &RestErr{
      Message: m,
      Err:     "bad_request",
      Code:    http.StatusBadRequest,
      Fields:  c,
    }
  }

  func NewInternalServerError(message string) *RestErr {
    return &RestErr{
      Message: message,
      Err:     "internal_server_error",
      Code:    http.StatusInternalServerError,
    }
  }

  func NewNotFoundError(message string) *RestErr {
    return &RestErr{
      Message: message,
      Err:     "not_found",
      Code:    http.StatusNotFound,
    }
  }

  func NewForbiddenError(message string) *RestErr {
    return &RestErr{
      Message: message,
      Err:     "forbidden",
      Code:    http.StatusForbidden,
    }
  }
```

Uma função simples, que vai apenas formatar e retornar uma mensagem padrão, para cada tipo de erro http, não criei todos os erros, apenas os mais comuns, você pode ver todos os erros http [aqui](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status){:target="\_blank"} e implementar.

## Configurando o Go Playground Validator

Vamos utilizar o [Go playground validator](https://github.com/go-playground/validator){:target="\_blank"} para nos ajudar a realizar as validações, para isso vamos separar em uma pasta dentro da nossa pasta **handler** chamada **validation** o arquivo vai se chamar `http_validation.go`.

Nossa função vai receber uma interface vazia e vai retornar um ponteiro do nosso `http_err`, veja como vai ficar nossa função:

```go
  func ValidateHttpData(d interface{}) *httperr.RestErr {
    val := validator.New(validator.WithRequiredStructEnabled())

    // extract json tag name
    val.RegisterTagNameFunc(func(fld reflect.StructField) string {
      name := strings.SplitN(fld.Tag.Get("json"), ",", 2)[0]
      if name == "-" {
        return ""
      }
      return name
    })

    if err := val.Struct(d); err != nil {
      var errorsCauses []httperr.Fields

      for _, e := range err.(validator.ValidationErrors) {
        cause := httperr.Fields{}
        fieldName := e.Field()

        switch e.Tag() {
        case "required":
          cause.Message = fmt.Sprintf("%s is required", fieldName)
          cause.Field = fieldName
          cause.Value = e.Value()
        case "uuid4":
          cause.Message = fmt.Sprintf("%s is not a valid uuid", fieldName)
          cause.Field = fieldName
          cause.Value = e.Value()
        case "boolean":
          cause.Message = fmt.Sprintf("%s is not a valid boolean", fieldName)
          cause.Field = fieldName
          cause.Value = e.Value()
        case "min":
          cause.Message = fmt.Sprintf("%s must be greater than %s", fieldName, e.Param())
          cause.Field = fieldName
          cause.Value = e.Value()
        case "max":
          cause.Message = fmt.Sprintf("%s must be less than %s", fieldName, e.Param())
          cause.Field = fieldName
          cause.Value = e.Value()
        case "email":
          cause.Message = fmt.Sprintf("%s is not a valid email", fieldName)
          cause.Field = fieldName
          cause.Value = e.Value()
        default:
          cause.Message = "invalid field"
          cause.Field = fieldName
          cause.Value = e.Value()
        }

        errorsCauses = append(errorsCauses, cause)
      }
      return httperr.NewBadRequestValidationError("some fields are invalid", errorsCauses)
    }
    return nil
  }
```

Primeiro iniciamos o validador com `validator.New()`, depois extraimos o nome da tag do json, isso vai servir para informar o campo que ocasionou o erro, depois nosso `val.Struct(d)` realiza a validação e se houver um erro cai no if, onde verificamos cada tipo de erro, como por exemplo se for `required`, adicionamos esse erro no `errorsCauses` e assim por diante. Por fim retornamos o `http_err.NewBadRequestValidationError` com nosso erro personalizado.

Note que é um pouco manual para configurar, mas não precisa colocar todos os `case` com todas as opções, conforme for utilizando você vai incrementando essas validações. Perceba que com isso você pode colocar a mensagem de erro que desejar no `cause.Message`, logo você vai entender onde vamos determinar as regras dos nossos campos como `max`, `min`, `required`, `uuidv4`. Você pode ver como configurar com mais detalhes o [Go playground validator](https://pkg.go.dev/github.com/go-playground/validator/v10){:target="\_blank"} na documentação oficial, essa foi a melhor forma que encontrei para utilizar em meus projetos, você pode encontrar outras implementações ou criar a sua.

## Iniciando os dtos

Calma, estamos quase chegando no nosso handler, mas antes precisamos criar nossos dtos (Data Transfer Object) que tem como responsabilidade validar os dados que transitam entre as camdas da nossa aplicação, no nosso caso vamos usar para trafegar entre handler e service.

Vamos criar na pasta **dto** um arquivo chamado `user_dto.go` responsável por salvar nossas structs e será nesse arquivo onde vamos definiar as regras de validações, aquelas regras que definimos no go playground na função `ValidateHttpData`.

```go
  type CreateUserDto struct {
    Name     string `json:"name" validate:"required,min=3,max=30"`
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8,max=30,containsany=!@#$%*"`
  }
```

Usamos as tags do Go para informar ao go playground as validações que desejamos para este campo, não vamos criar muitos campos para ser mais rápido, apenas isso já será suficiente para criar um usuário.

Como validamos o `containsany`, forçando o usuário a informar uma senha que tenha qualquer desses caracteres, precisamos atualizar nosso `RegisterTagNameFunc`, vamos adicionar:

```go
  case "containsany":
    cause.Message = fmt.Sprintf("%s must contain at least one of the following characters: !@#$%%*", fieldName)
    cause.Field = fieldName
    cause.Value = e.Value()
```

## Finalizando o handler

Finalmente vamos partir para o nosso handler!

### Criando o usuário

Agora dentro do nosso `user_handler.go` vamos iniciar a validação da request recebida:

```go
  var req dto.CreateUserDto

  if r.Body == http.NoBody {
    slog.Error("body is empty", slog.String("package", "userhandler"))
    w.WriteHeader(http.StatusBadRequest)
    msg := httperr.NewBadRequestError("body is required")
    json.NewEncoder(w).Encode(msg)
    return
  }
```

Primeiro criamos uma var do noss dto `CreateUserDto`, depois verificamos se o body da requisição não é vazio, se for já retornamos um erro, perceba que já estamos usando nossos logs.

```go
  err := json.NewDecoder(r.Body).Decode(&req)
  if err != nil {
    slog.Error("error to decode body", "err", err, slog.String("package", "handler_user"))
    w.WriteHeader(http.StatusBadRequest)
    msg := httperr.NewBadRequestError("error to decode body")
    json.NewEncoder(w).Encode(msg)
    return
  }
```

Agora estamos tranformando o body que é um `io` em uma struct Go, especificamente na struct `CreateUserDto` e se der um erro também retornamos retornamos o erro e finalizamos a requisição.

```go
  httpErr := validation.ValidateHttpData(req)
  if httpErr != nil {
    slog.Error(fmt.Sprintf("error to validate data: %v", httpErr), slog.String("package", "handler_user"))
    w.WriteHeader(httpErr.Code)
    json.NewEncoder(w).Encode(httpErr)
    return
  }
```

Acima chamamos o `ValidateHttpData` que vai colocar o go playground em ação e validar se os dados do nosso `CreateUserDto` estão de acordo com o que determinamos, caso não esteja retorna um erro.

Já podemos ver isso na prática, se estiver acompanhando seu projeto deve rodar sem problemas, rode com o comando `go run cmd/webserver/main.go`, vamos fazer uma chamada POST para `http://localhost:8080/user`, vou deixar no projeto o arquivo da extensão para vscode do [rest client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client){:target="\_blank"}

```json
  POST http://localhost:8080/user HTTP/1.1
  content-type: application/json

  {
    "name": "John Doe",
    "email": "john.doe@email.com",
    "password": "12345678"
  }
```

Vamos ter o retorno:

```json
  HTTP/1.1 400 Bad Request
  Date: Thu, 14 Dec 2023 23:33:21 GMT
  Content-Length: 205
  Content-Type: text/plain; charset=utf-8
  Connection: close

  {
    "message": "some fields are invalid",
    "error": "bad_request",
    "code": 400,
    "fields": [
      {
        "field": "password",
        "value": "12345678",
        "message": "password must contain at least one of the following characters: !@#$%*"
      }
    ]
  }
```

Perceba que nossas validações funcionaram, e fomos informados que nosso password está inválido, precisamos colocar qualquer caracter que solicitamos como obrigatório

```json
  POST http://localhost:8080/user HTTP/1.1
  content-type: application/json

  {
    "name": "John Doe",
    "email": "john.doe@email.com",
    "password": "12345678@"
  }
```

Resposta:

```json
  HTTP/1.1 200 OK
  Date: Thu, 14 Dec 2023 23:35:52 GMT
  Content-Length: 0
  Connection: close
```

Agora sim, tudo funcionando com validações e padronização, basta chamar o service, mas precisamos ajustar nosso service do usuário, para receber os parâmentros necessários.

`user_interface_service.go`:

```go
  type UserService interface {
    CreateUser(ctx context.Context, u dto.CreateUserDto) error
  }
```

Agora nossa interface receber um [context](https://aprendagolang.com.br/2022/11/02/onde-e-qual-context-utilizar/){:target="\_blank"} e o nosso dto, precisamos ajustar a implementação também:

`user_service.go`:

```go
  func (s *service) CreateUser(ctx context.Context, u dto.CreateUserDto) error {
    return nil
  }
```

Voltando ao nosso handler, vamos chamar nosso service:

```go
  err = h.service.CreateUser(r.Context(), req)
  if err != nil {
    slog.Error(fmt.Sprintf("error to create user: %v", err), slog.String("package", "handler_user"))
    w.WriteHeader(http.StatusInternalServerError)
    msg := httperr.NewInternalServerError("error to create user")
    json.NewEncoder(w).Encode(msg)
    return
  }
```

Pegamos o context da requisição e nosso req que é um `CreateUserDto` e passamos para o nosso `UserService`. E essa é toda a responsabilidade que nosso handler vai ter, apenas validar os dados antes de passar para o service, nosso service que vai conter a regra de negócio.

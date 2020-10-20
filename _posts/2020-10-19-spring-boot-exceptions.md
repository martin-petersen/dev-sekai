---
title:  "#3 - Spring Boot - Criando Exceptions"
date:   2020-10-19 13:38:53
categories: [java]
tags: [java,spring]
---
Continuando de onde nós paramos, vamos enviar uma requisição para salvar uma nova pessoa no nosso banco de dados, porém uma requisição com uma falha que nós consertamos no post passado
<br/>![exemplo de requisição](https://i.ibb.co/wYb5W4X/image.png)
<br/>observe o que nós temos como resposta a essa requisição
<br/>![exemplo de requisição](https://i.ibb.co/rHc8KgB/image.png)
nosso cliente recebendo isso como resposta vai entender pelo status code que algo deu errado e aquela requisição falhou, mas o que falhou? Como ele vai entender o que fez errado para poder consertar? Vamos fazer criar os erros da nossa API para sinalizar o cliente o que ele fez errado.

Primeiramente vamos criar um package <strong>error</strong> no projeto dois packages um chamado <strong>handler</strong> e outro chamado <strong>error</strong>
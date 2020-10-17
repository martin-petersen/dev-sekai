---
title:  "#1 - Spring Boot"
date:   2020-10-17 16:41:23
categories: [java]
tags: [java]
---
Nesse post nós vamos criar nossa primeira API Rest utilizando a linguagem Java no framework Spring Boot. Antes de mais nada para criar o projeto podemso acessar o site do [Spring Initializr](https://start.spring.io). Assim entrando no link marcamos as seguintes opções:
![](https://i.ibb.co/TLx4ZB6/spring-initializr.png)
Depois basta gerar e o seu navegador vai fazer o download do projeto.

A próxima parte é importante e se trata de configurar as depencências da sua api, vamos colocar para esse projeto:
<h4>
<p>
<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
</dependency>
</p>
<p>
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
</p>
<p>
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
</dependency>
</p>
</h4>
Para isso é necessário abrir o projeto na IDE de sua preferência e selecionar o arquivo <strong>pom.xml</strong> que fica na pasta raiz. Feito isso você precisa declarar as dependências dentro do bloco descrito por <strong>\<dependencies></dependencies></strong>, nesse trecho cole o seguinte código:
``` xml
<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
</dependency>
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
Feito isso, vamos entender como funciona uma API Rest:
O fluxo básico e usual funciona da seguinte forma: nossa API tem um controlador (Controller) que contém os endpoints, esses vão receber a requisição, porém o controler não processa o que ele recebe, isso fica a cargo de uma classe de serviço (Service), ela então vai realizar toda a lógica de negócio.

A classe de serviço por sua vez precisa de uma conexão com o banco de dados e para isso existe o repositório (Repository), é uma interface com a finalidade de gerenciar essa conexão para nós realizando o CRUD (Create, Read, Update e Delete) do nosso banco. A classe de serviço aciona o repositório para fazer uma determinada operação no banco e em seguida retorna ao controler o que deve ser servido ao cliente.

Em caso de não terem percebido nós precisamos de algo para manipular no banco utilizando esse fluxo, e isso fica a cargo dos modelos (Model), esses são os dados que a API consulta, salva, deleta ou altera no banco de dados dependendo do que for solicitado pelo cliente.

Entendido o fluxo <strong>Controller -> Service -> Repository -> Service -> Controller</strong> vamos à definição do propósito da nossa API. Nós criaremos uma aplicação com a única finalidade de receber dados de pessoa

Dentro do package model do projeto, nós vamos criar uma classe chamada Pessoa e anotar essa classe com @Entity, isso sinaliza para o Spring que essa classe é uma classe de domínio e gera uma entidade no banco de dados com cada atributo da classe sendo um campo na tabela. Imediatamente após a inserção dessa notação nosso programa vai dizer que algo está errado, isso porque obrigatóriamente nós precisamos definir que cada objeto gerado dessa classe de domónio tenha uma ID dentro do banco para isso o spring fornece ferramentas também, nós temos a notação @Id que indica a propriedade que vai se tornar o ID do nosso modelo e a notação @GeneratedValue que vai gerar nossos ids de forma automatica.

Para o nosso exemplo nossa entidade tem além do id mais quatro atributos:
+ nome(String),
+ dataNascimento(LocalDate),
+ naturalidade(String),
+ idade(int);

Ao final vamos ter algo assim:
``` java
package com.example.demo.model;


import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import java.time.LocalDate;

@Entity
public class Pessoa {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String nome;
    private LocalDate dataNascimento;
    private String naturalidade;
    private int idade;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }

    public LocalDate getDataNascimento() {
        return dataNascimento;
    }

    public void setDataNascimento(LocalDate dataNascimento) {
        this.dataNascimento = dataNascimento;
    }

    public String getNaturalidade() {
        return naturalidade;
    }

    public void setNaturalidade(String naturalidade) {
        this.naturalidade = naturalidade;
    }

    public int getIdade() {
        return idade;
    }

    public void setIdade(int idade) {
        this.idade = idade;
    }
}
```
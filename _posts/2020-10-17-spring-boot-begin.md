---
title:  "#1 - Spring Boot"
date:   2020-10-17 16:41:23
categories: [java]
tags: [java]
---
Nesse post nós vamos criar nossa primeira API Rest utilizando a linguagem Java no framework Spring Boot. Antes de mais nada para criar o projeto podemos acessar o site do [Spring Initializr](https://start.spring.io). Assim entrando no link marcamos as seguintes opções:
![](https://i.ibb.co/TLx4ZB6/spring-initializr.png)
Depois basta gerar e o seu navegador vai fazer o download do projeto. É importante salientar que o projeto já vem com um servidor Tomcat embutido então assim que você o abrir na IDE já pode executar, o servidor vai ser levantado e a aplicação vai estar no ar acessível no seu [localhost](http://localhost:8080/) por padrão na porta 8080.

A próxima parte é importante e se trata de configurar as dependências da sua API, vamos colocar para esse projeto:
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

Entendido o fluxo <strong>Controller -> Service -> Repository -> Service -> Controller</strong>. Vamos à definição do propósito da nossa API. Nós criaremos uma aplicação com a única finalidade de receber e persistir dados de pessoas

Dentro do package <strong>model</strong> do projeto, nós vamos criar uma classe chamada Pessoa e anotar essa classe com @Entity, isso sinaliza para o Spring que essa classe é uma classe de domínio e gera uma entidade no banco de dados com cada atributo da classe sendo um campo na tabela. Imediatamente após a inserção dessa notação nosso programa vai dizer que algo está errado, isso porque obrigatóriamente nós precisamos definir que cada objeto gerado dessa classe de domónio tenha uma ID dentro do banco para isso o spring fornece ferramentas também, nós temos a notação @Id que indica a propriedade que vai se tornar o ID do nosso modelo e a notação @GeneratedValue que vai gerar nossos ids de forma automatica.

Para o nosso exemplo a entidade tem, além do id, mais quatro atributos:
+ nome(String),
+ dataNascimento(LocalDate),
+ estado(String),
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
    private String estado;
    private int idade;

    public Pessoa() {
    }

    public Pessoa(Pessoa pessoa) {
        this.nome = pessoa.getNome().toUpperCase();
        this.dataNascimento = pessoa.dataNascimento;
        this.estado = pessoa.getEstado().toUpperCase();
        this.idade = pessoa.getIdade();
    }

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

    public String getEstado() {
        return estado;
    }

    public void setEstado(String estado) {
        this.estado = estado;
    }

    public int getIdade() {
        return idade;
    }

    public void setIdade(int idade) {
        this.idade = idade;
    }
}
```
Temos o modelo, agora precisamos criar nossa ferramenta para gerenciar o banco de dados, nosso Repository. Para isso criamos uma interface com o nome PessoaRepository dentro o package <strong>repository</strong> usamos a notação @Repository na interface e vamos usar o extends de JpaRepository passando como parâmetros a nossa classe Pessoa e o tipo do id da classe, Long nesse caso. Criar nosso repository dessa forma nos dá uma vantagem, alguns métodos já vem implementados o nosso trabalho aqui é implementar as buscas no banco baseadas nos atributos da classe Pessoa, para isso usamos o nome findBy e o nome do atributo na Entidade. No final teremos algo assim:
``` java
package com.example.demo.repository;

import com.example.demo.model.Pessoa;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.time.LocalDate;
import java.util.List;

@Repository
public interface PessoaRepository extends JpaRepository<Pessoa,Long> {
    /*
    * Aqui temos uma particularidade a notação @Query que é utilizada para
    * fazer modificações na função padrão criada pela JpaRepository, nesse caso
    * eu quero pegar buscar no banco todas as pessoas que tenham no nome o trecho
    * que foi passado no parâmetro do método.
    */
    @Query("from Pessoa i where upper(i.nome) like :nome")
    public List<Pessoa> findByNome(String nome);
    public List<Pessoa> findByEstado(String estado);
    public List<Pessoa> findByDataNascimentoAfter(LocalDate localDate);
    public List<Pessoa> findByDataNascimentoBefore(LocalDate localDate);
    public List<Pessoa> findByIdade(int idade);
}
```
Com o nosso repository pronto, vamos agora cuidar da nossa classe de serviço, criaremos a classe PessoaService no package <strong>service</strong> e anotaremos ela com @Service, aqui nós só temos um atributo uma injeção do nosso repository utilizando a notação @Autowired para utilizarmos os seus métodos e criaremos seis métodos de busca, um para insert, um para update e um para delete. Ao final teremos um código assim.
```java
package com.example.demo.service;

import com.example.demo.model.Pessoa;
import com.example.demo.repository.PessoaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.List;

@Service
public class PessoaService {
    @Autowired
    private PessoaRepository pessoaRepository;

    /*
    * A notação @Transactional cria uma transação com o banco de dados,
    * trocando em miúdos, em uma situação onde várias classes repository
    * estejam sendo utilizadas, se alguma transação com banco em um desses
    * repositorys falhar, então todos falham, mesmo que uma delas tenha sido
    * concluída com sucesso acontece um rollback. Ou todas as transações funcionam
    * ou nenhuma delas funciona. No nosso caso não é funcional pois só temos um
    * repository mas fica de aprendizado.
    */
    @Transactional
    public Pessoa save(Pessoa pessoa) {
        Pessoa novaPessoa = new Pessoa(pessoa);
        pessoaRepository.save(novaPessoa);
        return novaPessoa;
    }

    @Transactional
    public Pessoa update(Long id, Pessoa pessoa) {
        if(pessoaRepository.findById(id).isPresent()) {
            Pessoa attPessoa = pessoaRepository.findById(id).get();
            attPessoa.setNome(pessoa.getNome().toUpperCase());
            attPessoa.setEstado(pessoa.getEstado().toUpperCase());
            attPessoa.setDataNascimento(pessoa.getDataNascimento());
            attPessoa.setIdade(pessoa.getIdade());
            pessoaRepository.save(attPessoa);
            return attPessoa;
        } else {
            return null;
        }
    }

    @Transactional
    public void delete(Long id) {
        if(pessoaRepository.findById(id).isPresent()) {
            pessoaRepository.delete(pessoaRepository.findById(id).get());
        }
    }

    public List<Pessoa> findByNome(String nome) {
        return pessoaRepository.findByNome(nome.toUpperCase());
    }

    public List<Pessoa> findByUF(String naturalidade) {
        return pessoaRepository.findByEstado(naturalidade.toUpperCase());
    }

    public List<Pessoa> findByDataNascimentoAfter(String dataNascimento) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        LocalDate localDate = LocalDate.parse(dataNascimento,formatter);
        return pessoaRepository.findByDataNascimentoAfter(localDate);
    }

    public List<Pessoa> findByDataNascimentoBefore(String dataNascimento) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        LocalDate localDate = LocalDate.parse(dataNascimento,formatter);
        return pessoaRepository.findByDataNascimentoBefore(localDate);
    }

    public List<Pessoa> findByIdade(int idade) {
        return pessoaRepository.findByIdade(idade);
    }

    public List<Pessoa> findAll() {
        return pessoaRepository.findAll();
    }
}
```
Agora para a penúltima parte do nosso tutorial vamos construir nosso controlador, para isso dentro do package <strong>controller</strong> vamos criar uma classe chamada PessoaController e anotar ela com @RestController e @RequestMapping, essa segunda identifica a rota base do controller. Nessa classe assim como na classe de serviço, nós só temos uma injeção de classe com o @Autowired, mas diferente do service em que injetamos um repository aqui nós injetamos o próprio service. e criamos as funções que vão receber as requisições (GET, POST, PUT e DELETE). Ao fim teremos algo assim:
``` java
package com.example.demo.controller;

import com.example.demo.model.Pessoa;
import com.example.demo.service.PessoaService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.util.UriComponentsBuilder;

import java.net.URI;
import java.time.LocalDate;
import java.util.List;

@RestController
@RequestMapping(value = "/pessoa")
public class PessoaController {
    @Autowired
    private PessoaService pessoaService;

    /*
    * Essa notação indica que esse método vai ser chamado em requisições GET
    */
    @GetMapping
    public ResponseEntity<List<Pessoa>> listagemPessoas(@RequestParam(required = false)String nome,
                                                        @RequestParam(required = false)String uf,
                                                        @RequestParam(required = false)String depois,
                                                        @RequestParam(required = false)String antes,
                                                        @RequestParam(required = false)Integer idade) {
        if(nome!=null) {
            return ResponseEntity.ok(pessoaService.findByNome(nome.toUpperCase()));
        } else if(uf!=null) {
            return ResponseEntity.ok(pessoaService.findByUF(uf.toUpperCase()));
        } else if(idade!=null) {
            return ResponseEntity.ok(pessoaService.findByIdade(idade));
        } else if(depois!=null) {
            List<Pessoa> pessoas = pessoaService.findByDataNascimentoAfter(depois);
            return ResponseEntity.ok(pessoas);
        } else if(antes!=null) {
            List<Pessoa> pessoas = pessoaService.findByDataNascimentoBefore(antes);
            return ResponseEntity.ok(pessoas);
        } else {
            return ResponseEntity.ok(pessoaService.findAll());
        }
    }

    /*
     * Essa notação indica que esse método vai ser chamado em requisições POST
     */
    @PostMapping
    public ResponseEntity<Pessoa> salvar(@RequestBody Pessoa p,
                                         UriComponentsBuilder uriComponentsBuilder) {
        try {
            Pessoa pessoa = pessoaService.save(p);
            URI uri = uriComponentsBuilder.path("/pessoa/{id}").buildAndExpand(pessoa.getId()).toUri();
            return ResponseEntity.created(uri).body(pessoa);
        }catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }

    /*
     * Essa notação indica que esse método vai ser chamado em requisições PUT
     */

    /*
     * As notações @PathVariable e @RequestBody significam respectivamente que esse controller
     * recebe uma variável que vem do caminho da URL por isso Path Variable e ele também
     * recebe no corpo da requisição, por isso Request Body, um objeto json do tipo Pessoa
     */
    @PutMapping("{id}")
    public ResponseEntity<Pessoa> atualizar(@PathVariable Long id,
                                            @RequestBody Pessoa p) {
        try {
            Pessoa pessoa = pessoaService.update(id, p);
            return ResponseEntity.ok(pessoa);
        }catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }

    /*
     * Essa notação indica que esse método vai ser chamado em requisições DELETE
     */
    @DeleteMapping("{id}")
    public ResponseEntity<?> remover(@PathVariable Long id) {
        try {
            pessoaService.delete(id);
            return ResponseEntity.ok().build();
        }catch (Exception e) {
            return ResponseEntity.badRequest().build();
        }
    }
}
```
Agora para finalizar e podermos brincar com a nossa API só precisamos configurar o banco de dados. Para isso vamos ao nosso arquivo <strong>application.properties</strong> dentro do diretório resource do projeto. O arquivo deve ficar assim:
``` properties
# data source
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:cadastro-pessoa
spring.datasource.username=spring-tutorial
spring.datasource.password=123456

# jpa
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.show_sql = true
spring.jpa.properties.hibernate.format_sql = true

# h2
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```
Agora é só rodar o projeto e brincar com as requisições, o projeto está disponível no meu [Github](https://github.com/martin-petersen/spring-example.git), outra coisa que você pode fazer é ver as tabelas do seu banco acessando o console do [H2](http://localhost:8080/h2-console) quando o projeto estiver executando, dentro do projeto também vou deixar um package com requisições HTTP para facilitar os testes. Se divirtam e criem várias APIs para que o conhecimento fique fixado e até o próximo post!
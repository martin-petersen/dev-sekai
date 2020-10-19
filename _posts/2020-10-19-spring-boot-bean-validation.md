---
title:  "#2 - Spring Boot - Bean validation e conceito de DTO"
date:   2020-10-19 08:41:23
categories: [java]
tags: [java,spring]
---
Dando continuação ao nosso projeto verifique o seguinte, o projeto está sem qualquer tipo de validação para os campos que colocamos então se enviarmos uma requisição nesse formato:
<br/>![exemplo de requisição](https://i.ibb.co/wYb5W4X/image.png)
<br/>ela certamente irá funcionar e no nosso banco de dados um registro assim será criado:
<br/>![exemplo cadastrado no banco](https://i.ibb.co/PN4rqb9/image.png)
<br/>nós temos um registro de uma pessoa sem nome e sem estado, no nosso caso poderia ser feita uma verificação simples utilizando um bloco de código <strong>if/else</strong>, porém imagine o trabalho que seria fazer isso sempre em aplicações grandes com várias entidades e vários campos que não podem ser vazios ou nulos no banco. Para resolver isso o Spring tem uma ferramenta, o Bean Validation representado pela notação <strong>@Valid</strong>, mas antes para utilizar precisamos importar a dependência do Spring que nos fornece esse recurso, para isso precisamos ir nas dependências do nosso projeto no arquivo pom.xlm e adicionar

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
com a dependência importada nós podemos ir na nossa classe Pessoa e anotar os parâmetros obrigatórios com o @NotBlank que vai prevenir que o banco crie registros com aqueles campos em branco, também existe o @NotNull para prevenir registros nulos no caso da entidade receber um objeto de outra classe como parâmetro:

```java
@Entity
public class Pessoa {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @NotBlank
    private String nome;
    private LocalDate dataNascimento;
    @NotBlank
    private String estado;
    private int idade;
```
perfeito agora nós podemos ir na nossa classe PessoaController e anotar @Valid em todos os métodos que recebem essa classe como parâmetro

```java
@PostMapping
    public ResponseEntity<Pessoa> salvar(@Valid @RequestBody Pessoa p,
                                         UriComponentsBuilder uriComponentsBuilder)

@PutMapping("{id}")
public ResponseEntity<Pessoa> atualizar(@PathVariable Long id,
                                        @Valid @RequestBody Pessoa p)
```
isso resolve o nosso problema em receber campos em branco já está resolvido, agora vale chamar atenção para uma má prática que estamos fazendo que é utilizar nossa classe de domínio no parâmetro das funções, mas imagine que existe um parâmetro nessa classe entidade que não pode ser enviado pelo cliente mas também não pode ser nulo, esse atributo seria pego de outra forma dentro da lógica interna da aplicação. Para ilustrar vamos criar uma nova propriedade de Pessoa que é data de criação (dateCreated) que não pode ser nula e também não pode ser inserida pelo cliente.

```java
@Entity
public class Pessoa {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @NotBlank
    private String nome;
    private LocalDate dataNascimento;
    @NotBlank
    private String estado;
    private int idade;
    @NotNull
    private LocalDateTime dateCeated;

    public Pessoa() {
    }

    /*
     * Lembrando de ajustar o construtor para iniciar o nosso novo parâmetro,
     * já que ele nunca pode ser nulo
     */
    public Pessoa(PessoaFORM pessoa) {
        this.nome = pessoa.getNome().toUpperCase();
        this.dataNascimento = pessoa.getDataNascimento();
        this.estado = pessoa.getEstado().toUpperCase();
        this.idade = pessoa.getIdade();
        this.dateCeated = LocalDateTime.now();
    }
```
imagine o cliente ter a preocupação de colocar uma string especificando a data (ANO-MÊS-DIA) e hora (HORA:MINUTO:SEGUNDO:MILÉSIMOS) o trabalho que seria fazer isso com tamanha precisão, para isso nós vamos mudar o nosso controller para não mais receber Pessoa como parâmetro e sim um formulário com os campos que o cliete pode nos informar no nosso projeto então criamos um package <strong>form</strong> e uma classe <strong>PessoaFORM</strong>, para essa classe não precisamos de setters apenas os getters.

```java
package com.example.demo.form;

import javax.validation.constraints.NotBlank;
import java.time.LocalDate;

public class PessoaFORM {
    @NotBlank
    private String nome;
    private LocalDate dataNascimento;
    @NotBlank
    private String estado;
    private int idade;

    public String getNome() {
        return nome;
    }

    public LocalDate getDataNascimento() {
        return dataNascimento;
    }

    public String getEstado() {
        return estado;
    }

    public int getIdade() {
        return idade;
    }
}
```
Em seguida colocamos o controller para receber essa classe como parâmetro e ajustamos as demais classes para remover os avisos de erros:

```java
@PostMapping
    public ResponseEntity<Pessoa> salvar(@Valid @RequestBody PessoaFORM p,
                                         UriComponentsBuilder uriComponentsBuilder)

@PutMapping("{id}")
public ResponseEntity<Pessoa> atualizar(@PathVariable Long id,
                                        @Valid @RequestBody PessoaFORM p)
```
com tudo pronto nosso programa roda recebendo esse formulário, prevenindo atributos em branco, não precisamos pedir o <strong>dateCreated</strong>, porém os mais atentos perceberam que nossas requisições do tipo GET continuam enviando nossa entidade como resposta da requisição e para nosso exemplo não tem problema, mas imagine em um programa que lista usuários com informações sensíveis como senhas e documentos, como prevenir isso?

A resposta é simples, nós não retornamos classes de domínio e sim DTOs (Data Transfer Objects) uma classe que abstrai da nossa entidade apenas o que deve ser entregue ao cliente final, vamos criar uma também no nosso package <strong>dto</strong> nossa classe PessoaDTO com a finalidade de não entregar a data de nascimento e idade da pessoa, pensando no bem das moças que utilizarem nossa API.

```java
package com.example.demo.dto;

import java.time.LocalDateTime;

public class PessoaDTO {
    private Long id;
    private String nome;
    private String estado;
    private LocalDateTime dateCreated;

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

    public String getEstado() {
        return estado;
    }

    public void setEstado(String estado) {
        this.estado = estado;
    }

    public LocalDateTime getDateCreated() {
        return dateCreated;
    }

    public void setDateCreated(LocalDateTime dateCreated) {
        this.dateCreated = dateCreated;
    }
}
```
criado nosso dto vamos alterar nossos métodos do controller para retornar uma lista de PessoaDTO

```java
package com.example.demo.controller;

import com.example.demo.dto.PessoaDTO;
import com.example.demo.form.PessoaFORM;
import com.example.demo.model.Pessoa;
import com.example.demo.service.PessoaService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.util.UriComponentsBuilder;

import javax.validation.Valid;
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
    public ResponseEntity<List<PessoaDTO>> listagemPessoas(@RequestParam(required = false)String nome,
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
            return ResponseEntity.ok(pessoaService.findByDataNascimentoAfter(depois));
        } else if(antes!=null) {
            return ResponseEntity.ok(pessoaService.findByDataNascimentoBefore(antes));
        } else {
            return ResponseEntity.ok(pessoaService.findAll());
        }
    }

    /*
     * Essa notação indica que esse método vai ser chamado em requisições POST
     */
    @PostMapping
    public ResponseEntity<PessoaDTO> salvar(@Valid @RequestBody PessoaFORM p,
                                         UriComponentsBuilder uriComponentsBuilder) {
        try {
            PessoaDTO pessoa = pessoaService.save(p);
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
    public ResponseEntity<PessoaDTO> atualizar(@PathVariable Long id,
                                            @Valid @RequestBody PessoaFORM p) {
        try {
            PessoaDTO pessoa = pessoaService.update(id, p);
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
não podemos também trazer para o controller essa lista de Pessoa para o controller para aqui ser convertido, temos que trazer diretamente do service já na forma de lista de PessoaDTO, para isso alteramos o retorno dos nossos métodos na classe PessoaService e criamos um método privado para realizar essa conversão de uma lista para outra evitando repetição de código no nosso projeto


```java
package com.example.demo.service;

import com.example.demo.dto.PessoaDTO;
import com.example.demo.form.PessoaFORM;
import com.example.demo.model.Pessoa;
import com.example.demo.repository.PessoaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

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
    public PessoaDTO save(PessoaFORM pessoa) {
        Pessoa novaPessoa = new Pessoa(pessoa);
        pessoaRepository.save(novaPessoa);
        return new PessoaDTO(novaPessoa.getId(),novaPessoa.getNome(),novaPessoa.getEstado(),novaPessoa.getDateCeated());
    }

    @Transactional
    public PessoaDTO update(Long id, PessoaFORM pessoa) {
        if(pessoaRepository.findById(id).isPresent()) {
            Pessoa attPessoa = pessoaRepository.findById(id).get();
            attPessoa.setNome(pessoa.getNome().toUpperCase());
            attPessoa.setEstado(pessoa.getEstado().toUpperCase());
            attPessoa.setDataNascimento(pessoa.getDataNascimento());
            attPessoa.setIdade(pessoa.getIdade());
            pessoaRepository.save(attPessoa);
            return new PessoaDTO(attPessoa.getId(),attPessoa.getNome(),attPessoa.getEstado(),attPessoa.getDateCeated());
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

    public List<PessoaDTO> findByNome(String nome) {
        return conversorEntidadeParaDTO(pessoaRepository.findByNome(nome.toUpperCase()));
    }

    public List<PessoaDTO> findByUF(String naturalidade) {
        return conversorEntidadeParaDTO(pessoaRepository.findByEstado(naturalidade.toUpperCase()));
    }

    public List<PessoaDTO> findByDataNascimentoAfter(String dataNascimento) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        LocalDate localDate = LocalDate.parse(dataNascimento,formatter);
        return conversorEntidadeParaDTO(pessoaRepository.findByDataNascimentoAfter(localDate));
    }

    public List<PessoaDTO> findByDataNascimentoBefore(String dataNascimento) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        LocalDate localDate = LocalDate.parse(dataNascimento,formatter);
        return conversorEntidadeParaDTO(pessoaRepository.findByDataNascimentoBefore(localDate));
    }

    public List<PessoaDTO> findByIdade(int idade) {
        return conversorEntidadeParaDTO(pessoaRepository.findByIdade(idade));
    }

    public List<PessoaDTO> findAll() {
        return conversorEntidadeParaDTO(pessoaRepository.findAll());
    }

    /*
     * Nosso método com função de converter as listas de objetos, criado utilizando
     * streams presente a partir do Java 8, mas se não tiver domínio podem utilizar
     * for ou foreach sem qualquer prejuízo.
     */

    private List<PessoaDTO> conversorEntidadeParaDTO(List<Pessoa> pessoas) {
        List<PessoaDTO> pessoasDTO = new ArrayList<>();
        pessoasDTO = pessoas.stream().map(pessoa -> new PessoaDTO(pessoa.getId(),
                pessoa.getNome(),pessoa.getEstado(),pessoa.getDateCeated())).collect(Collectors.toList());
        return pessoasDTO;
    }
}
```
Agora com as alterações feitas, nosso projeto está mais consistente e empregando melhores práticas de programação. O link para download do projeto está [aqui](https://drive.google.com/file/d/1yxmlCuEcmO-isYIk7vNc95nDWQPMt8VX/view?usp=sharing) valendo ressaltar que a prática leva à perfeição então criem mais APIs para fixar o conhecimento e até o próximo post!
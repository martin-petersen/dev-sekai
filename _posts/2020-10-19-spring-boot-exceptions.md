---
title:  "#3 - Spring Boot - Criando Exceptions"
date:   2020-10-19 13:38:53
categories: [java]
tags: [java,spring]
---
Continuando de onde nós paramos, vamos enviar uma requisição para salvar uma nova pessoa no nosso banco de dados, porém uma requisição com uma falha que nós consertamos no post passado
<br/>![requisição](https://i.ibb.co/wYb5W4X/image.png)
<br/>observe o que nós temos como resposta a essa requisição
<br/>![resposta](https://i.ibb.co/ck3Dxqx/image.png)
<br/>nosso cliente recebendo isso como resposta vai entender pelo status code que algo deu errado e aquela requisição falhou, mas o que falhou? Como ele vai entender o que fez errado para poder consertar? Vamos criar os erros da nossa API para sinalizar o cliente o que ele fez não está correto.

Primeiramente vamos criar no projeto dois packages um chamado <strong>handler</strong> e outro chamado <strong>error</strong> vamos começar criando no package error nosso objeto de erro nosso ApiError. Para essa classe temos:

```java
package com.example.demo.error;

import org.springframework.http.HttpStatus;

import java.time.LocalDateTime;
import java.util.Map;

public class ApiError {
    private LocalDateTime timestamp;
    private Map<String,String> error;
    private int statusCode;
    private HttpStatus httpStatus;

    public ApiError(Map<String, String> error, HttpStatus httpStatus, int statusCode) {
        this.timestamp = LocalDateTime.now();
        this.error = error;
        this.httpStatus = httpStatus;
        this.statusCode = statusCode;
    }

    public LocalDateTime getTimestamp() {
        return timestamp;
    }

    public Map<String, String> getError() {
        return error;
    }

    public int getStatusCode() {
        return statusCode;
    }

    public HttpStatus getHttpStatus() {
        return httpStatus;
    }
}
```
depois nós criamos o handler para lidar com as exceptions que vierem a ser disparadas na execução do programa, a classe HandlerException:

```java
package com.example.demo.handler;

import com.example.demo.error.ApiError;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;

import javax.servlet.http.HttpServletRequest;
import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
@Order(Ordered.HIGHEST_PRECEDENCE)
public class HandlerException {

    @ExceptionHandler({MethodArgumentNotValidException.class})
    public ResponseEntity<ApiError> argumentNotValid(final HttpServletRequest req, final MethodArgumentNotValidException exception) {
        Map<String,String> error = new HashMap<>();
        exception.getBindingResult().getAllErrors().forEach(objectError -> {
            String fieldName = ((FieldError) objectError).getField();
            String errorMessage = ((FieldError) objectError).getDefaultMessage();
            error.put(fieldName,errorMessage);
        });
        return new ResponseEntity<>(new ApiError(error,HttpStatus.BAD_REQUEST,
        HttpStatus.BAD_REQUEST.value()),HttpStatus.BAD_REQUEST);
    }
}
```

feito isso só precisamos anotar qual a mensagem vamos receber para cada campo que for enviado errado pelo critério do Bean validation, e isso nós fazemos na nossa classe PessoaDTO:

```java
public class PessoaDTO {
    @NotBlank(message = "Nome não pode ser vazio")
    private String nome;
    private LocalDate dataNascimento;
    @NotBlank(message = "UF não pode ser vazio")
    private String estado;
    private int idade;
```
tentemos agora mandar uma requisição com o nome e estado em branco
<br/>![requisição](https://i.ibb.co/BTFTqCz/image.png)
<br/>temos como resposta dessa vez um objeto ApiError em JSON explicitando o que foi feito de errado
<br/>![resposta](https://i.ibb.co/M23fnQ6/image.png)
<br/>Esse post teve um exemplo bem genérico, com um projeto maior sempre surgem mais Exceptions types e cada uma tem seu próprio tratamento, então mãos a obra construam projetos e handlers para os diferentes tipos de exceptions que forem aparecendo para fixar o conhecimento. O link para donwload do código fonte desse projeto está [aqui](https://drive.google.com/file/d/1vNHhDVmFqRG91WmmKeBajPNk3lz12hnP/view?usp=sharing).
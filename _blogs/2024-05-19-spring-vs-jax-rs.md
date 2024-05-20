---
title: "Do not mess up Spring with Jax-RS"
author: yiqi
permalink: /blogs/2024-05-19-spring-vs-jax-rs
date: 2024-05-19
collection: blogs
tag:
- Programming
- Java
---

# Overview  

This might feels too simple to talk about. But I did see people (including myself), getting confused.  

For example, I remember one time I was adding one simple API call in a FeignClient interface where Spring annotation ```@GetMapping``` was used. I copy-pasted it without thinking too much.  

The third-party service I was calling provided 2 content types ```application/x-protobuf``` and ```application/json``` (default). I wated to use protobuf, so I added ```@Headers("Accept: application/x-protobuf")```, and did protobuf deserialization.  

Well, you probably already know what happened... I kept getting ```InvalidProtocolBufferException``` and spent like thounsands of years before realizing the consumed content type was JSON. ```@Headers("Accept: application/x-protobuf")``` was not recognized.  

If you don't get what was wrong, keep reading.  

# To be more specific  

When it comes to writing REST APIs in Java, you could use either Jax-RS or Spring. And they do NOT recognizes each other's annotations.  

JAX-RS is a set of specifications for building REST services. Its best-known reference implementations are RESTEasy and Jersey. As for Spring, I bet you already know what it is if you click into this post 😉. Just remember Spring does NOT implement JAX-RS spec.  

On implementing REST API or its client, the basic components we care about are:  

1. Method: GET, PUT, POST, etc.
2. Path
3. Path and query parameters
4. Headers (including content type negotiation)

Jax-RS and Spring defines different annotations and interfaces, as shown below  

|                                 | Jax-RS                              | Spring                                                      |
| REST Method                     | ```@GET```, ```@POST```, ```@PUT``` | ```@GetMapping```, ```@PostMapping```, ```@PutMapping```    |
| Path                            | ```@Path```                         | ```@RequestMapping```, ```@XXXMapping(value = "xxxx")```    |
| Path parameter                  | ```@PathParam```                    | ```@PathVariable```                                         |
| Query parameter                 | ```@QueryParam```                   | ```@RequestParam```                                         |
| Query parameter default value   | ```@DefaultValue```                 | ```@RequestParam(required = false, defaultValue = "xxx")``` |
| Produces header                 | ```@Produces```                     | ```@XXXMapping(produces = xxx)```                           |
| Consumes header                 | ```@Consumes```                     | ```@XXXMapping(consumes = xxx)```                           |
| (client) Generic request header | works with ```feign.Headers```      | ```@XXXMapping(headers = xxx)```                            |


# Case study  

Now let's play with a toy example: ```GET /book/{id}?edition={edition}```, content type is JSON  

The ```Book``` entity is defined as:  

```java
import lombok.Builder;
import lombok.Data;

@Data
@Builder
public class Book {
    private String id;
    private String title;
    private String author;
    private int edition;
}
```

## Server  

JAX-RS:  

```java
import javax.ws.rs.DefaultValue;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

import java.util.Optional;

@Path("book")
public class BookController {

    @GET
    @Path("/{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getBook(@PathParam("id") String bookId,
                            @QueryParam("edition") @DefaultValue("1") int edition) {
        // in real service, this should be some code to query database
        Optional<Book> book = Optional.of(Book.builder()
                .id(bookId)
                .edition(edition)
                .build());
        if (book.isPresent()) {
            return Response.ok(book.get()).build();
        }
        return Response.status(Response.Status.NOT_FOUND)
                .build();
    }

}
```

Spring:  

```java
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.Optional;

@RestController
@RequestMapping("book")
public class BookController {

    @GetMapping(value = "/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Book> getBook(@PathVariable("id") String bookId,
                                        @RequestParam(value = "edition", required = false, defaultValue = "1") int edition) {
        // in real service, this should be some code to query database
        Optional<Book> book = Optional.of(Book.builder()
                .id(bookId)
                .edition(edition)
                .build());
        if (book.isPresent()) {
            return ResponseEntity.ok(book.get());
        }
        return ResponseEntity.notFound().build();
    }
    
}
```

## Client  

JAX-RS:  

```java
import org.springframework.cloud.openfeign.FeignClient;

import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

@FeignClient(name = "BookApi", url = "https://www.xxx.com")
public interface BookApi {
    @GET
    @Path("{id}")
    @Consumes(MediaType.APPLICATION_JSON)
    Response getBook(@PathParam("id") String id,
                     @QueryParam("edition") int edition);
}
```

Spring:  

```java
import org.springframework.cloud.openfeign.FeignClient;

import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(name = "BookApi", url = "https://www.xxx.com")
public interface BookApi {
    @GetMapping(value = "/{id}", consumes = MediaType.APPLICATION_JSON_VALUE)
    ResponseEntity<Book> getBook(@PathVariable String id,
                                 @RequestParam(value = "edition") int edition);
}
```
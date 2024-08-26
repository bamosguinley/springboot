### Cas Pratique : Gestion d'une Bibliothèque

#### Objectif
Nous allons créer une application qui permet de gérer une bibliothèque. Les fonctionnalités de base incluent :
- Ajouter un nouveau livre.
- Consulter la liste des livres.
- Récupérer un livre par son identifiant.
- Mettre à jour les informations d'un livre.
- Supprimer un livre.

### 1. Configuration du projet

#### 1.1 Créer un projet Spring Boot avec Maven

Rendez-vous sur [Spring Initializr](https://start.spring.io/) pour configurer votre projet :
- **Project** : Maven Project
- **Language** : Java
- **Spring Boot** : Dernière version stable (3.x.x)
- **Group** : `com.example`
- **Artifact** : `library`
- **Name** : `Library`
- **Package name** : `com.example.library`
- **Packaging** : Jar
- **Java Version** : 8 ou supérieure

**Dépendances** :
- **Spring Web** : Pour créer l'API REST.
- **Spring Data JPA** : Pour interagir avec la base de données.
- **H2 Database** : Une base de données en mémoire pour le développement.

Cliquez sur **Generate** pour télécharger le projet. Décompressez-le et importez-le dans votre IDE (IntelliJ IDEA, Eclipse, etc.).

#### 1.2 Structure du projet

Votre projet devrait avoir la structure suivante :

```
src
└── main
    ├── java
    │   └── com
    │       └── example
    │           └── library
    │               ├── LibraryApplication.java
    │               ├── controller
    │               ├── model
    │               ├── repository
    │               └── service
    └── resources
        ├── application.properties
```

### 2. Configuration de la base de données

Nous allons utiliser la base de données H2, une base de données en mémoire, qui est parfaite pour les tests et le développement local.

#### 2.1 Configurer H2 dans `application.properties`

Dans le fichier `src/main/resources/application.properties`, ajoutez les propriétés suivantes :

```properties
# Configuration de la base de données H2
spring.datasource.url=jdbc:h2:mem:librarydb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.jpa.hibernate.ddl-auto=update
```

### 3. Créer l'Entité JPA pour `Book`

Une entité est une classe qui représente une table dans la base de données. Nous allons créer une entité `Book` qui représente un livre dans notre bibliothèque.

#### 3.1 Créer la classe `Book`

Dans le package `com.example.library.model`, créez une classe `Book` :

```java
package com.example.library.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String author;
    private String isbn;

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getIsbn() {
        return isbn;
    }

    public void setIsbn(String isbn) {
        this.isbn = isbn;
    }
}
```

### 4. Créer le Repository

Le repository est l'interface qui permet de faire des opérations CRUD sur l'entité `Book` en interagissant avec la base de données.

#### 4.1 Créer l'interface `BookRepository`

Dans le package `com.example.library.repository`, créez une interface `BookRepository` :

```java
package com.example.library.repository;

import com.example.library.model.Book;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface BookRepository extends JpaRepository<Book, Long> {
}
```

### 5. Créer le Service

Le service contient la logique métier de l'application. Il interagit avec le repository pour exécuter les opérations.

#### 5.1 Créer la classe `BookService`

Dans le package `com.example.library.service`, créez une classe `BookService` :

```java
package com.example.library.service;

import com.example.library.model.Book;
import com.example.library.repository.BookRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository;

    public List<Book> getAllBooks() {
        return bookRepository.findAll();
    }

    public Optional<Book> getBookById(Long id) {
        return bookRepository.findById(id);
    }

    public Book saveBook(Book book) {
        return bookRepository.save(book);
    }

    public void deleteBook(Long id) {
        bookRepository.deleteById(id);
    }
}
```

### 6. Créer le Controller

Le controller gère les requêtes HTTP et envoie les réponses appropriées aux clients.

#### 6.1 Créer la classe `BookController`

Dans le package `com.example.library.controller`, créez une classe `BookController` :

```java
package com.example.library.controller;

import com.example.library.model.Book;
import com.example.library.service.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/books")
public class BookController {

    @Autowired
    private BookService bookService;

    @GetMapping
    public List<Book> getAllBooks() {
        return bookService.getAllBooks();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Book> getBookById(@PathVariable Long id) {
        return bookService.getBookById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public Book createBook(@RequestBody Book book) {
        return bookService.saveBook(book);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Book> updateBook(@PathVariable Long id, @RequestBody Book bookDetails) {
        return bookService.getBookById(id)
                .map(book -> {
                    book.setTitle(bookDetails.getTitle());
                    book.setAuthor(bookDetails.getAuthor());
                    book.setIsbn(bookDetails.getIsbn());
                    return ResponseEntity.ok(bookService.saveBook(book));
                })
                .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
        return bookService.getBookById(id)
                .map(book -> {
                    bookService.deleteBook(id);
                    return ResponseEntity.ok().<Void>build();
                })
                .orElse(ResponseEntity.notFound().build());
    }
}
```

### 7. Tester l'API

#### 7.1 Lancer l'application

Lancez l'application Spring Boot en exécutant la classe principale `LibraryApplication`. Elle devrait démarrer sur `http://localhost:8080`.

#### 7.2 Tester avec Postman ou Curl

Utilisez Postman ou Curl pour interagir avec les endpoints créés :

1. **Ajouter un livre** :
   - **Endpoint** : `POST /api/books`
   - **Corps** (JSON) :
     ```json
     {
         "title": "Effective Java",
         "author": "Joshua Bloch",
         "isbn": "978-0134685991"
     }
     ```

2. **Récupérer tous les livres** :
   - **Endpoint** : `GET /api/books`

3. **Récupérer un livre par ID** :
   - **Endpoint** : `GET /api/books/{id}`

4. **Mettre à jour un livre** :
   - **Endpoint** : `PUT /api/books/{id}`
   - **Corps** (JSON) :
     ```json
     {
         "title": "Effective Java (3rd Edition)",
         "author": "Joshua Bloch",
         "isbn": "978-0134685991"
     }
     ```

5. **Supprimer un livre** :
   - **Endpoint** : `DELETE /api/books/{id}`

### 8. Ajouter la Documentation avec Swagger

#### 8.1 Ajouter Swagger au projet

Ajoutez les dépendances Swagger dans le `pom.xml` :

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

#### 8.2 Configurer Swagger

Créez une classe `SwaggerConfig` pour configurer Swagger

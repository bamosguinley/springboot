
### 1. Création du Projet Spring Boot avec Maven

#### 1.1 Utilisation de Spring Initializr
- Rendez-vous sur [Spring Initializr](https://start.spring.io/).
- Configurez les paramètres :
  - **Project** : Maven Project
  - **Language** : Java
  - **Spring Boot** : Dernière version stable (3.x.x)
  - **Group** : `com.example`
  - **Artifact** : `crudexample`
  - **Name** : `CrudExample`
  - **Package name** : `com.example.crudexample`
  - **Packaging** : Jar
  - **Java Version** : 8 ou supérieure
- **Dependencies** : Ajoutez les dépendances suivantes :
  - Spring Web
  - Spring Data JPA
  - H2 Database (base de données en mémoire pour les tests)
- Cliquez sur **Generate** pour générer et télécharger le projet.

#### 1.2 Importer dans l'IDE
- Décompressez le fichier ZIP téléchargé.
- Ouvrez votre IDE (IntelliJ IDEA, Eclipse, etc.) et importez le projet comme un projet Maven existant.

### 2. Configuration de la Base de Données

Nous allons utiliser H2, une base de données en mémoire pratique pour le développement et les tests.

#### 2.1 Modifier le fichier `application.properties`

Dans `src/main/resources/application.properties`, configurez la base de données H2 :

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.jpa.hibernate.ddl-auto=update
```

### 3. Création de l'Entité JPA

Créons une entité `Product` qui représentera un produit dans notre application.

#### 3.1 Créer la classe `Product`

Dans le package `com.example.crudexample.model`, créez une classe `Product` :

```java
package com.example.crudexample.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private Double price;

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }
}
```

### 4. Création du Repository

Le Repository permet d'interagir avec la base de données en utilisant JPA.

#### 4.1 Créer l'interface `ProductRepository`

Dans le package `com.example.crudexample.repository`, créez une interface `ProductRepository` :

```java
package com.example.crudexample.repository;

import com.example.crudexample.model.Product;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

### 5. Création du Service

Le Service contient la logique métier de l'application.

#### 5.1 Créer la classe `ProductService`

Dans le package `com.example.crudexample.service`, créez une classe `ProductService` :

```java
package com.example.crudexample.service;

import com.example.crudexample.model.Product;
import com.example.crudexample.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }

    public Optional<Product> getProductById(Long id) {
        return productRepository.findById(id);
    }

    public Product saveProduct(Product product) {
        return productRepository.save(product);
    }

    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }
}
```

### 6. Création du Controller

Le Controller gère les requêtes HTTP et renvoie les réponses appropriées.

#### 6.1 Créer la classe `ProductController`

Dans le package `com.example.crudexample.controller`, créez une classe `ProductController` :

```java
package com.example.crudexample.controller;

import com.example.crudexample.model.Product;
import com.example.crudexample.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.getAllProducts();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        return productService.getProductById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.saveProduct(product);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(@PathVariable Long id, @RequestBody Product productDetails) {
        return productService.getProductById(id)
                .map(product -> {
                    product.setName(productDetails.getName());
                    product.setPrice(productDetails.getPrice());
                    return ResponseEntity.ok(productService.saveProduct(product));
                })
                .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        return productService.getProductById(id)
                .map(product -> {
                    productService.deleteProduct(id);
                    return ResponseEntity.ok().<Void>build();
                })
                .orElse(ResponseEntity.notFound().build());
    }
}
```

### 7. Tester l'API

1. **Lancer l'application** :
   - Exécutez la classe principale `CrudExampleApplication`.
   - L'application Spring Boot démarrera sur `http://localhost:8080`.

2. **Tester les endpoints** :
   - Utilisez un outil comme **Postman** ou **Curl** pour tester les différentes routes.

   **Routes** :
   - `GET /api/products` : Récupérer tous les produits.
   - `GET /api/products/{id}` : Récupérer un produit par son ID.
   - `POST /api/products` : Créer un nouveau produit.
   - `PUT /api/products/{id}` : Mettre à jour un produit existant.
   - `DELETE /api/products/{id}` : Supprimer un produit existant.

### 8. Gestion des Erreurs et Validation

#### 8.1 Gestion des Exceptions

Créez une classe d'exception personnalisée et un gestionnaire global des exceptions.

**Créer `ResourceNotFoundException`:**

```java
package com.example.crudexample.exception;

public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

**Créer `GlobalExceptionHandler`:**

```java
package com.example.crudexample.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleResourceNotFoundException(ResourceNotFoundException ex) {
        return new ResponseEntity<>(ex.getMessage(), HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneralException(Exception ex) {
        return new ResponseEntity<>("An error occurred", HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

#### 8.2 Validation des Données

Utilisez les annotations de validation dans l'entité `Product`.

**Mise à jour de `Product` pour la validation:**

```java
package com.example.crudexample.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;

@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Le nom du produit est obligatoire")
    private String name;

    @NotNull(message = "Le prix du produit est obligatoire")
    @Min(value = 0, message = "Le prix doit être supérieur ou égal à 0")
    private Double price;

    // Getters and Setters
    // ...
}
```

**Validation dans le Controller:**

```java
@PostMapping
public Product createProduct(@Validated @RequestBody Product product) {
    return productService.saveProduct(product);
}

@PutMapping("/{id}")
public ResponseEntity<Product> updateProduct(@PathVariable Long id, @Validated @RequestBody Product productDetails) {
    return ResponseEntity.ok(productService.updateProduct(id, productDetails));
}
```

###### 1. Création du Projet Spring Boot avec Maven

#### 1.1 Utilisation de Spring Initializr
- Rendez-vous sur [Spring Initializr](https://start.spring.io/).
- Configurez les paramètres :
  - **Project** : Maven Project
  - **Language** : Java
  - **Spring Boot** : Dernière version stable (3.x.x)
  - **Group** : `com.example`
  - **Artifact** : `crudexample`
  - **Name** : `CrudExample`
  - **Package name** : `com.example.crudexample`
  - **Packaging** : Jar
  - **Java Version** : 8 ou supérieure
- **Dependencies** : Ajoutez les dépendances suivantes :
  - Spring Web
  - Spring Data JPA
  - H2 Database (base de données en mémoire pour les tests)
- Cliquez sur **Generate** pour générer et télécharger le projet.

#### 1.2 Importer dans l'IDE
- Décompressez le fichier ZIP téléchargé.
- Ouvrez votre IDE (IntelliJ IDEA, Eclipse, etc.) et importez le projet comme un projet Maven existant.

### 2. Configuration de la Base de Données

Nous allons utiliser H2, une base de données en mémoire pratique pour le développement et les tests.

#### 2.1 Modifier le fichier `application.properties`

Dans `src/main/resources/application.properties`, configurez la base de données H2 :

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.jpa.hibernate.ddl-auto=update
```

### 3. Création de l'Entité JPA

Créons une entité `Product` qui représentera un produit dans notre application.

#### 3.1 Créer la classe `Product`

Dans le package `com.example.crudexample.model`, créez une classe `Product` :

```java
package com.example.crudexample.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private Double price;

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }
}
```

### 4. Création du Repository

Le Repository permet d'interagir avec la base de données en utilisant JPA.

#### 4.1 Créer l'interface `ProductRepository`

Dans le package `com.example.crudexample.repository`, créez une interface `ProductRepository` :

```java
package com.example.crudexample.repository;

import com.example.crudexample.model.Product;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

### 5. Création du Service

Le Service contient la logique métier de l'application.

#### 5.1 Créer la classe `ProductService`

Dans le package `com.example.crudexample.service`, créez une classe `ProductService` :

```java
package com.example.crudexample.service;

import com.example.crudexample.model.Product;
import com.example.crudexample.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }

    public Optional<Product> getProductById(Long id) {
        return productRepository.findById(id);
    }

    public Product saveProduct(Product product) {
        return productRepository.save(product);
    }

    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }
}
```

### 6. Création du Controller

Le Controller gère les requêtes HTTP et renvoie les réponses appropriées.

#### 6.1 Créer la classe `ProductController`

Dans le package `com.example.crudexample.controller`, créez une classe `ProductController` :

```java
package com.example.crudexample.controller;

import com.example.crudexample.model.Product;
import com.example.crudexample.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.getAllProducts();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        return productService.getProductById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.saveProduct(product);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(@PathVariable Long id, @RequestBody Product productDetails) {
        return productService.getProductById(id)
                .map(product -> {
                    product.setName(productDetails.getName());
                    product.setPrice(productDetails.getPrice());
                    return ResponseEntity.ok(productService.saveProduct(product));
                })
                .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        return productService.getProductById(id)
                .map(product -> {
                    productService.deleteProduct(id);
                    return ResponseEntity.ok().<Void>build();
                })
                .orElse(ResponseEntity.notFound().build());
    }
}
```

### 7. Tester l'API

1. **Lancer l'application** :
   - Exécutez la classe principale `CrudExampleApplication`.
   - L'application Spring Boot démarrera sur `http://localhost:8080`.

2. **Tester les endpoints** :
   - Utilisez un outil comme **Postman** ou **Curl** pour tester les différentes routes.

   **Routes** :
   - `GET /api/products` : Récupérer tous les produits.
   - `GET /api/products/{id}` : Récupérer un produit par son ID.
   - `POST /api/products` : Créer un nouveau produit.
   - `PUT /api/products/{id}` : Mettre à jour un produit existant.
   - `DELETE /api/products/{id}` : Supprimer un produit existant.

### 8. Gestion des Erreurs et Validation

#### 8.1 Gestion des Exceptions

Créez une classe d'exception personnalisée et un gestionnaire global des exceptions.

**Créer `ResourceNotFoundException`:**

```java
package com.example.crudexample.exception;

public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

**Créer `GlobalExceptionHandler`:**

```java
package com.example.crudexample.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleResourceNotFoundException(ResourceNotFoundException ex) {
        return new ResponseEntity<>(ex.getMessage(), HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneralException(Exception ex) {
        return new ResponseEntity<>("An error occurred", HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

#### 8.2 Validation des Données

Utilisez les annotations de validation dans l'entité `Product`.

**Mise à jour de `Product` pour la validation:**

```java
package com.example.crudexample.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;

@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Le nom du produit est obligatoire")
    private String name;

    @NotNull(message = "Le prix du produit est obligatoire")
    @Min(value = 0, message = "Le prix doit être supérieur ou égal à 0")
    private Double price;

    // Getters and Setters
    // ...
}
```

**Validation dans le Controller:**

```java
@PostMapping
public Product createProduct(@Validated @RequestBody Product product) {
    return productService.saveProduct(product);
}

@PutMapping("/{id}")
public ResponseEntity<Product> updateProduct(@PathVariable Long id, @Validated @RequestBody Product productDetails) {
    return ResponseEntity.ok(productService.updateProduct(id, productDetails));
}
```

###

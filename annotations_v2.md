Bien sûr ! Voici une liste détaillée des annotations JPA (Java Persistence API) et Lombok, avec une explication pour chacune.

---

## **Annotations JPA**

Les annotations JPA sont utilisées pour définir la manière dont les entités Java sont mappées aux tables de base de données. Elles permettent de spécifier des relations entre entités, des contraintes, et des stratégies de gestion des données.

### **1. `@Entity`**

- **Description** : Indique que la classe est une entité JPA qui sera mappée à une table de base de données.
- **Exemple** :

  ```java
  @Entity
  public class Product {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
      private String name;
      private Double price;
      // Getters and setters
  }
  ```

### **2. `@Table`**

- **Description** : Permet de spécifier le nom de la table de la base de données qui correspond à l'entité. Si omis, le nom de la classe est utilisé.
- **Exemple** :

  ```java
  @Entity
  @Table(name = "products")
  public class Product {
      // Fields and methods
  }
  ```

### **3. `@Id`**

- **Description** : Indique que l'attribut est la clé primaire de l'entité.
- **Exemple** :

  ```java
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  ```

### **4. `@GeneratedValue`**

- **Description** : Spécifie la stratégie de génération des valeurs pour la clé primaire. Les stratégies communes sont `AUTO`, `IDENTITY`, `SEQUENCE`, et `TABLE`.
- **Exemple** :

  ```java
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  ```

### **5. `@Column`**

- **Description** : Décrit une colonne dans la table de base de données. Permet de spécifier des contraintes telles que `nullable`, `length`, `unique`, etc.
- **Exemple** :

  ```java
  @Column(name = "product_name", nullable = false, length = 100)
  private String name;
  ```

### **6. `@OneToOne`**

- **Description** : Définit une relation un-à-un entre deux entités.
- **Exemple** :

  ```java
  @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
  private Profile profile;
  ```

### **7. `@OneToMany`**

- **Description** : Définit une relation un-à-plusieurs entre deux entités. L'attribut `mappedBy` spécifie l'attribut de l'entité cible qui gère la relation.
- **Exemple** :

  ```java
  @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
  private List<Employee> employees;
  ```

### **8. `@ManyToOne`**

- **Description** : Définit une relation plusieurs-à-un entre deux entités. Utilisé généralement pour définir la partie "plusieurs" de la relation `OneToMany`.
- **Exemple** :

  ```java
  @ManyToOne
  @JoinColumn(name = "department_id")
  private Department department;
  ```

### **9. `@ManyToMany`**

- **Description** : Définit une relation plusieurs-à-plusieurs entre deux entités. L'annotation `@JoinTable` est utilisée pour spécifier la table de jointure.
- **Exemple** :

  ```java
  @ManyToMany
  @JoinTable(
      name = "student_course",
      joinColumns = @JoinColumn(name = "student_id"),
      inverseJoinColumns = @JoinColumn(name = "course_id")
  )
  private List<Course> courses;
  ```

### **10. `@JoinColumn`**

- **Description** : Spécifie la colonne de jointure pour les relations entre entités. Utilisé avec `@ManyToOne`, `@OneToOne`, et dans des relations bidirectionnelles.
- **Exemple** :

  ```java
  @JoinColumn(name = "customer_id")
  private Customer customer;
  ```

### **11. `@JoinTable`**

- **Description** : Spécifie la table de jointure pour les relations plusieurs-à-plusieurs.
- **Exemple** :

  ```java
  @JoinTable(
      name = "student_course",
      joinColumns = @JoinColumn(name = "student_id"),
      inverseJoinColumns = @JoinColumn(name = "course_id")
  )
  ```

### **12. `@Fetch`**

- **Description** : Spécifie la stratégie de chargement des associations. Les options sont `LAZY` (chargement à la demande) et `EAGER` (chargement immédiat).
- **Exemple** :

  ```java
  @OneToMany(fetch = FetchType.LAZY)
  private List<Employee> employees;
  ```

### **13. `@Cascade`** (Hibernate spécifique)

- **Description** : Définit les opérations qui doivent être propagées sur les entités associées. Les types de cascade incluent `PERSIST`, `MERGE`, `REMOVE`, etc.
- **Exemple** :

  ```java
  @OneToMany(cascade = CascadeType.ALL)
  private List<Employee> employees;
  ```

### **14. `@Transient`**

- **Description** : Indique que le champ ne doit pas être persisté dans la base de données.
- **Exemple** :

  ```java
  @Transient
  private String temporaryField;
  ```

### **15. `@Embeddable`**

- **Description** : Utilisé pour définir une classe qui peut être intégrée dans une autre entité.
- **Exemple** :

  ```java
  @Embeddable
  public class Address {
      private String street;
      private String city;
      // Getters and setters
  }
  ```

### **16. `@Embedded`**

- **Description** : Utilisé pour inclure une instance d'une classe `@Embeddable` dans une entité.
- **Exemple** :

  ```java
  @Embedded
  private Address address;
  ```

### **17. `@Enumerated`**

- **Description** : Spécifie comment les énumérations doivent être stockées dans la base de données. Les options sont `ORDINAL` (index) et `STRING` (nom).
- **Exemple** :

  ```java
  @Enumerated(EnumType.STRING)
  private Status status;
  ```

### **18. `@Lob`**

- **Description** : Spécifie qu'un champ doit être traité comme un large objet binaire (BLOB) ou texte (CLOB).
- **Exemple** :

  ```java
  @Lob
  private String description;
  ```

---

## **Annotations Lombok**

Lombok est une bibliothèque Java qui permet de réduire le code boilerplate en générant automatiquement des méthodes courantes (comme les getters, setters, etc.).

### **1. `@Getter`**

- **Description** : Génère des méthodes getter pour tous les champs de la classe.
- **Exemple** :

  ```java
  @Getter
  public class Person {
      private String name;
      private int age;
  }
  ```

### **2. `@Setter`**

- **Description** : Génère des méthodes setter pour tous les champs de la classe.
- **Exemple** :

  ```java
  @Setter
  public class Person {
      private String name;
      private int age;
  }
  ```

### **3. `@ToString`**

- **Description** : Génère une méthode `toString()` qui inclut tous les champs de la classe.
- **Exemple** :

  ```java
  @ToString
  public class Person {
      private String name;
      private int age;
  }
  ```

### **4. `@EqualsAndHashCode`**

- **Description** : Génère les méthodes `equals()` et `hashCode()` basées sur tous les champs ou un sous-ensemble.
- **Exemple** :

  ```java
  @EqualsAndHashCode
  public class Person {
      private String name;
      private int age;
  }
  ```

### **5. `@NoArgsConstructor`**

- **Description** : Génère un constructeur sans arguments.
- **Exemple** :

  ```java
  @NoArgsConstructor
  public class Person {
      private String name;
      private int age;
  }
  ```

### **6. `@AllArgsConstructor`**

- **Description** : Génère un constructeur avec un argument pour chaque champ de la classe.
- **Exemple** :

  ```java
  @AllArgsConstructor
  public class Person {
      private String name;
      private int age;
  }
  ```

### **7. `@RequiredArgsConstructor`**

- **Description** : Génère un constructeur avec les champs `final` ou ceux annotés avec `@NonNull`.
- **Exemple** :

  ```java
  @RequiredArgsConstructor
  public class Person {
      private final String name;
      private int age;
  }
  ```

### **8. `@Data`**

- **Description** : Combine les annotations `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, et `@RequiredArgsConstructor`.
- **

Exemple** :

  ```java
  @Data
  public class Person {
      private String name;
      private int age;
  }
  ```

### **9. `@Value`**

- **Description** : Génère une classe immuable (les champs sont `final`, il y a un constructeur pour tous les champs, et seuls les getters sont générés).
- **Exemple** :

  ```java
  @Value
  public class Person {
      String name;
      int age;
  }
  ```

### **10. `@Builder`**

- **Description** : Fournit un constructeur de type builder pour une classe. Utile pour construire des objets immuables de manière fluide.
- **Exemple** :

  ```java
  @Builder
  public class Person {
      private String name;
      private int age;
  }
  ```

### **11. `@Slf4j`**

- **Description** : Fournit un logger SLF4J pour la classe, permettant de l'utiliser sans avoir à définir explicitement un champ logger.
- **Exemple** :

  ```java
  @Slf4j
  public class Person {
      public void doSomething() {
          log.info("Doing something...");
      }
  }
  ```

---

Ces annotations sont largement utilisées dans les projets Java pour simplifier le code et améliorer la lisibilité et la maintenabilité. Les annotations JPA permettent de gérer la persistance des données, tandis que les annotations Lombok facilitent la création et la gestion des classes Java en réduisant le code boilerplate.

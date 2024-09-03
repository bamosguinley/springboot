Pour mettre en place un système d'authentification avec une API Spring Boot et des formulaires d'inscription et de connexion en Angular, vous pouvez suivre les étapes suivantes. Cette solution utilise JSON Web Tokens (JWT) pour l'authentification et le stockage du token sur le côté client.

### Étape 1 : Configurer l'API Spring Boot

#### 1.1. Ajouter des dépendances

Assurez-vous d'ajouter les dépendances nécessaires pour Spring Security et JWT dans votre `pom.xml` :

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

#### 1.2. Configurer Spring Security

Créez une classe de configuration de sécurité pour désactiver la sécurité CSRF et autoriser les requêtes à certaines URLs.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .antMatchers("/api/auth/**").permitAll() // Autorise toutes les requêtes pour le chemin /api/auth/
                .anyRequest().authenticated() // Toutes les autres requêtes nécessitent une authentification
                .and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}
```

#### 1.3. Créer le modèle `User`

Définissez une entité `User` pour gérer les utilisateurs.

```java
import javax.persistence.*;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;

    // Getters et Setters
}
```

#### 1.4. Créer un `UserRepository`

Créez un repository Spring Data JPA pour interagir avec la base de données.

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
}
```

#### 1.5. Créer un service d'authentification

Créez une classe de service qui gérera la logique de l'authentification.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class AuthService {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private BCryptPasswordEncoder bCryptPasswordEncoder;

    public User register(User user) {
        user.setPassword(bCryptPasswordEncoder.encode(user.getPassword()));
        return userRepository.save(user);
    }

    public User findByUsername(String username) {
        return userRepository.findByUsername(username);
    }
}
```

#### 1.6. Créer un contrôleur d'authentification

Créez un contrôleur pour gérer les requêtes d'inscription et de connexion.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private AuthService authService;

    @Autowired
    private AuthenticationManager authenticationManager;

    @PostMapping("/register")
    public ResponseEntity<?> registerUser(@RequestBody User user) {
        return ResponseEntity.ok(authService.register(user));
    }

    @PostMapping("/login")
    public ResponseEntity<?> authenticateUser(@RequestBody User user) {
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(user.getUsername(), user.getPassword())
        );
        SecurityContextHolder.getContext().setAuthentication(authentication);
        String token = "JWT_TOKEN"; // Générer un JWT ici
        return ResponseEntity.ok(new AuthResponse(token));
    }
}
```

### Étape 2 : Configurer Angular

#### 2.1. Créer un service d'authentification

Créez un service Angular pour gérer l'authentification.

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class AuthService {

  private baseUrl = 'http://localhost:8080/api/auth';

  constructor(private http: HttpClient) { }

  login(credentials: any): Observable<any> {
    return this.http.post(`${this.baseUrl}/login`, credentials);
  }

  register(user: any): Observable<any> {
    return this.http.post(`${this.baseUrl}/register`, user);
  }

  saveToken(token: string) {
    localStorage.setItem('token', token);
  }

  getToken() {
    return localStorage.getItem('token');
  }

  isAuthenticated(): boolean {
    return !!this.getToken();
  }
}
```

#### 2.2. Créer les composants de formulaire d'inscription et de connexion

Créez des composants Angular pour les formulaires d'inscription et de connexion.

**Inscription Component**:

```typescript
import { Component } from '@angular/core';
import { AuthService } from '../auth.service';

@Component({
  selector: 'app-register',
  templateUrl: './register.component.html',
})
export class RegisterComponent {
  user: any = {};

  constructor(private authService: AuthService) {}

  register() {
    this.authService.register(this.user).subscribe(response => {
      console.log('Utilisateur enregistré avec succès');
    });
  }
}
```

**Connexion Component**:

```typescript
import { Component } from '@angular/core';
import { AuthService } from '../auth.service';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
})
export class LoginComponent {
  credentials: any = {};

  constructor(private authService: AuthService) {}

  login() {
    this.authService.login(this.credentials).subscribe(response => {
      this.authService.saveToken(response.token);
      console.log('Connexion réussie');
    });
  }
}
```

### Étape 3 : Utiliser le token JWT pour les requêtes

Après la connexion réussie, le token JWT est stocké dans le `localStorage`. Utilisez-le dans vos requêtes HTTP suivantes pour accéder aux routes protégées en ajoutant un `HttpInterceptor` :

```typescript
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs';
import { AuthService } from './auth.service';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {

  constructor(private authService: AuthService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    if (token) {
      const cloned = req.clone({
        headers: req.headers.set('Authorization', 'Bearer ' + token)
      });
      return next.handle(cloned);
    } else {
      return next.handle(req);
    }
  }
}
```

Ajoutez l'intercepteur dans le module principal Angular :

```typescript
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { AuthInterceptor } from './auth.interceptor';

@NgModule({
  // ... autres configurations
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
  ]
})
export class AppModule { }
```

### Étape 4 : Test et validation

1. Lancez votre API Spring Boot.
2. Lancez votre application Angular.
3. Testez le processus d'inscription et de connexion.

En suivant ces étapes, vous aurez un système d'authentification fonctionnel avec Angular et Spring Boot utilisant JWT pour gérer la sécurité des requêtes.









Voici un schéma de fonctionnement pour un système d'authentification utilisant une API Spring Boot avec un front-end Angular. Ce système utilise JSON Web Tokens (JWT) pour sécuriser les interactions entre le client et le serveur.

### Schéma de Fonctionnement de l'Authentification JWT avec Spring Boot et Angular

1. **Inscription de l'utilisateur :**
   - **Étape 1** : L'utilisateur remplit un formulaire d'inscription sur le front-end Angular.
   - **Étape 2** : Angular envoie une requête HTTP POST avec les informations de l'utilisateur (nom d'utilisateur, mot de passe, etc.) à l'API Spring Boot.
   - **Étape 3** : L'API Spring Boot reçoit la requête, hache le mot de passe de l'utilisateur, puis enregistre l'utilisateur dans la base de données.
   - **Étape 4** : Une réponse est renvoyée à Angular pour confirmer l'inscription de l'utilisateur.

2. **Connexion de l'utilisateur :**
   - **Étape 1** : L'utilisateur remplit un formulaire de connexion sur le front-end Angular avec son nom d'utilisateur et son mot de passe.
   - **Étape 2** : Angular envoie une requête HTTP POST avec les informations de connexion à l'API Spring Boot.
   - **Étape 3** : Spring Boot vérifie les informations de l'utilisateur. S'elles sont correctes, un token JWT est généré.
   - **Étape 4** : L'API renvoie le token JWT à Angular.
   - **Étape 5** : Angular stocke le token JWT dans le `localStorage` ou `sessionStorage`.

3. **Accès aux Ressources Protégées :**
   - **Étape 1** : Chaque fois que l'utilisateur souhaite accéder à une ressource protégée, Angular envoie une requête HTTP avec le token JWT dans l'en-tête d'autorisation (Bearer Token).
   - **Étape 2** : L'API Spring Boot vérifie le token JWT. S'il est valide, l'accès à la ressource protégée est accordé.
   - **Étape 3** : La réponse de la ressource protégée est renvoyée à Angular.

4. **Déconnexion de l'utilisateur :**
   - **Étape 1** : Lors de la déconnexion, Angular supprime le token JWT du `localStorage` ou `sessionStorage`.
   - **Étape 2** : L'utilisateur est redirigé vers la page de connexion.

### Visualisation du Schéma de Fonctionnement

Le schéma de fonctionnement peut être visualisé comme suit :

```
+---------------------+          +----------------------+           +------------------+
|   Front-end Angular |          |   API Spring Boot    |           |   Base de données|
+---------------------+          +----------------------+           +------------------+
|                     |          |                      |           |                  |
| 1. Register/Login   | -------> | 2. Verify/Create User| ------->  | 3. Save/Retrieve |
|    Forms            |          |                      |           |    User Data     |
|                     | <------- | 4. Return JWT Token  | <-------- |                  |
|                     |          |                      |           |                  |
| 5. Store JWT Token  |          |                      |           |                  |
|    (localStorage)   |          |                      |           |                  |
|                     |          |                      |           |                  |
| 6. Access Resources | -------> | 7. Validate JWT Token|           |                  |
|    with JWT         |          |    & Return Data     | <-------- |                  |
|                     | <------- |                      |           |                  |
| 8. Log Out (Remove  |          |                      |           |                  |
|    JWT from Storage)|          |                      |           |                  |
+---------------------+          +----------------------+           +------------------+
```

### Explication du Schéma

- **Front-end Angular** : Gère l'interface

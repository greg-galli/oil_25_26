# Fiche Projet - Étape 3 : Orchestration et Communication Inter-Services

Dans cette étape, nous allons relier nos services. Nous allons créer le **Moteur de Jeu** (`game-engine-service`) qui agira comme un chef d'orchestre. Il ne gérera pas de base de données pour le moment, mais il interrogera les deux autres services pour construire une session de jeu.

## Objectifs de l'Étape 3

-   Créer le microservice **`game-engine-service`**.
    
-   Configurer les ports des 3 services pour qu'ils tournent simultanément sur la même machine.
    
-   Utiliser **`RestClient`** pour consommer les API REST des services `player` et `question`.
    
-   Implémenter un scénario métier : "Démarrer une partie" (récupérer un joueur et un lot de questions).

## Partie A : Configuration des Ports (Indispensable)

Comme nous n'utilisons pas encore de Gateway ni de Discovery Service, nous allons adresser les services via leurs URLs directes (`localhost`). Il faut donc attribuer un port unique à chaque service pour éviter les conflits.

Modifiez le fichier `application.properties` (ou `.yml`) de **chaque** projet :

1.  **`player-service`** :
    
    Properties
    
    ```
    server.port=8081
    ```
    
2.  **`question-catalog-service`** :
    
    Properties
    
    ```
    server.port=8082
    ```
    
3.  **`game-engine-service`** (Nouveau service à créer) :
    
    Properties
    
    ```
    server.port=8080
    ```
    

_Note : Redémarrez les services `player` et `question` pour appliquer les changements._

## Partie B : Création du `game-engine-service`

### 1. Génération du Projet

Créez un nouveau projet Spring Boot avec les dépendances suivantes :

-   `Spring Web`
    
-   `Lombok`
    
-   _(Pas de JPA ni de H2 pour l'instant, ce service ne stocke rien)._
    

### 2. Création des DTO (Data Transfer Objects)

Le moteur de jeu ne doit pas avoir de dépendance directe sur les classes des autres projets. Vous devez créer des classes simples (POJO/Record) pour mapper les réponses JSON des autres services.

**Dans un package `dto` :**



```java
// PlayerDTO.java
public record PlayerDTO(Long id, String pseudo, int score) {}

// QuestionDTO.java
public record QuestionDTO(Long id, String texte, String reponseCorrecte) {}
```

## Partie C : Mise en place de `RestClient`

Nous allons utiliser `RestClient` pour effectuer des appels synchrones (bloquants) vers les autres services.

### 1. Configuration du Bean

Créez une classe de configuration pour instancier le builder :



```java
@Configuration
public class ClientConfig {
    @Bean
    public RestClient.Builder restClientBuilder() {
        return RestClient.builder();
    }
}
```

### 2. Création des Connecteurs (Services Clients)

Créez des services dans le `game-engine-service` dont le seul rôle est d'appeler l'extérieur.

**Exemple pour le Joueur (`PlayerClient`) :**


```java
@Service
public class PlayerClient {
    private final RestClient restClient;

    public PlayerClient(RestClient.Builder builder) {
        // URL codée en dur pour l'instant (Port 8081)
        this.restClient = builder.baseUrl("http://localhost:8081/api/players").build();
    }

    public PlayerDTO getPlayerById(Long id) {
        return restClient.get()
                .uri("/{id}", id)
                .retrieve()
                .body(PlayerDTO.class);
    }
}
```

**À faire :** Réalisez la même chose pour `QuestionClient` (pointant vers `http://localhost:8082/api/questions`) pour récupérer une liste de questions.

## Partie D : Scénario "Démarrer une Partie"

C'est ici que l'orchestration prend forme. Vous allez créer un endpoint dans le `game-engine` qui appelle les deux autres services.

### 1. Le Scénario

L'objectif est de créer un endpoint : `POST /api/games/start/{playerId}?nbQuestions=3`

**Logique à implémenter :**

1.  Le moteur reçoit l'ID du joueur.
    
2.  Il appelle le `player-service` pour vérifier que le joueur existe et récupérer son pseudo.
    
3.  Il appelle le `question-catalog-service` pour récupérer une liste de X questions (ou toutes les questions pour simplifier dans un premier temps).
    
4.  Il agrège ces données dans un objet réponse `GameDTO` (ou `GameSession`).
    

### 2. Implémentation du GameService

```java
@Service
@RequiredArgsConstructor
public class GameService {
    private final PlayerClient playerClient;
    private final QuestionClient questionClient;

    public GameDTO startNewGame(Long playerId, int numberOfQuestions) {
        // 1. Récupération du joueur (Appel Synchrone)
        PlayerDTO player = playerClient.getPlayerById(playerId);
        
        // 2. Récupération des questions (Appel Synchrone)
        // Note: Idéalement, le service question devrait avoir un endpoint pour récupérer X questions aléatoires
        List<QuestionDTO> questions = questionClient.getAllQuestions(); 
        
        // 3. Construction de l'objet de jeu (Logique simple: on prend les N premières)
        List<QuestionDTO> selectedQuestions = questions.stream()
                .limit(numberOfQuestions)
                .toList();

        return new GameDTO(UUID.randomUUID().toString(), player, selectedQuestions);
    }
}
```

### 3. Le Contrôleur

Créez `GameController` pour exposer ce service.

```java
@RestController
@RequestMapping("/api/games")
@RequiredArgsConstructor
public class GameController {
    private final GameService gameService;

    @PostMapping("/start/{playerId}")
    public GameDTO startGame(@PathVariable Long playerId, @RequestParam(defaultValue = "3") int nb) {
        return gameService.startNewGame(playerId, nb);
    }
}
```

## Validation et Test

Pour valider cette étape, suivez la séquence suivante :

1.  Démarrez **`player-service`** (Port 8081).
    
2.  Démarrez **`question-catalog-service`** (Port 8082).
    
3.  Démarrez **`game-engine-service`** (Port 8080).
    
4.  Utilisez Postman ou curl :
    
    -   Envoyez une requête : `POST http://localhost:8080/api/games/start/1?nb=2`
        
5.  **Résultat attendu :** Vous devez recevoir un JSON unique contenant à la fois les informations du joueur (venant du service 1) et une liste de 2 questions (venant du service 2).
    

### 1. Gestion des Erreurs

Que se passe-t-il si le joueur n'existe pas ?

-   Le `RestClient` va lancer une exception `HttpClientErrorException`.
    
-   Observez comment l'erreur se propage. Essayez d'ajouter un `try/catch` dans le `GameService` pour renvoyer une erreur 404 propre si le microservice joueur renvoie une 404.

### 2. Rendu

Le rendu se fait exclusivement sur Github Classroom, voici les différents dépôts pour les différents groupes de TD :

-   Groupe de TD1 :  [https://classroom.github.com/a/OS70knsH](https://classroom.github.com/a/OS70knsH)
-   Groupe de TD2 :  [https://classroom.github.com/a/5azKubG6](https://classroom.github.com/a/5azKubG6)
-   Groupe de TD3 :  [https://classroom.github.com/a/U2uIBiIL](https://classroom.github.com/a/U2uIBiIL)

Ajoutez vos **nouvelles réalisations dans le readme** puis **rajoutez votre travail au dépôt** existant, enfin **finalisez votre livraison  en ajoutant un tag** ! 

```bash
git tag td3
# puis 
git push origin --tags
```

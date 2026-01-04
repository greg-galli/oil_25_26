# Fiche Projet - Étape 5 : Extension d'Architecture et Persistance Distribuée

_Pré-requis_ : Avoir terminé les TD 1, 2 et 3 (le TD4 est un plus pour la qualité mais non bloquant).

## Objectifs du TD 5

1.  Créer un 4ème microservice : **`score-service`** dédié à l'archivage des parties jouées.
    
2.  Implémenter la logique de **Fin de Partie** dans le `game-engine-service`.
    
3.  Orchestrer une _transaction distribuée_ simple (qui n'est pas réellement une transaction au sens ACID) : lorsqu'une partie se termine, le moteur doit **archiver la partie** (Service Score) et **mettre à jour le niveau du joueur** (Service Player).
    
4.  Manipuler les verbes HTTP modifiant l'état (POST, PATCH) via `RestClient`.


## Partie A : Création du `score-service`

Ce nouveau service a pour responsabilité de stocker l'historique de toutes les parties jouées (Date, Joueur, Score obtenu).

### 1. Configuration et Démarrage

-   Générez un projet Spring Boot (Web, JPA, H2, Lombok).
    
-   **Port :** Configurez le `server.port` sur **8083** dans `application.properties`.
    

### 2. Modèle de Données

Créez une entité `GameHistory` :


```java
@Entity
@Getter @Setter @Builder
@NoArgsConstructor @AllArgsConstructor
public class GameHistory {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private LocalDateTime playedAt; // Date de la partie
    private Long playerId;          // ID du joueur concerné
    private int score;              // Score obtenu durant cette partie
}

```

### 3. Couches Repository, Service et Controller

Implémentez le pattern classique pour exposer **un seul endpoint** permettant d'enregistrer une partie terminée.

-   **URI :** `POST /api/scores`
    
-   **Body attendu :** Un objet contenant `playerId` et `score`.
    
-   **Comportement :** Le service doit instancier un `GameHistory`, fixer la date actuelle (`LocalDateTime.now()`) et sauvegarder en base.

## Partie B : Mise à jour du `game-engine-service`

Le moteur de jeu sait démarrer une partie (TD3), mais ne sait pas encore la terminer. Nous allons ajouter cette logique.

### 1. Création du Client REST pour le Score

Dans le projet `game-engine-service`, ajoutez un `ScoreClient` (similaire à `PlayerClient` du TD3) pour communiquer avec le nouveau service sur le port 8083.

-   Il doit posséder une méthode `sendScore(Long playerId, int score)` qui effectue un `POST` vers le `score-service`.
    

### 2. Extension du Client REST pour le Joueur

Dans `PlayerClient`, ajoutez une méthode `updatePlayerScore(Long playerId, int scoreToAdd)`.

-   Cette méthode devra appeler le `player-service` (Port 8081).
    
-   **Attention :** Vous devez utiliser l'endpoint `PATCH` ou `PUT` (selon ce que vous avez fait au TD2) pour mettre à jour le score global du joueur.
    

### 3. Logique de "Fin de Partie"

Dans `GameService` (du moteur de jeu), créez une méthode `endGame(Long playerId, int score)`.

Cette méthode doit jouer le rôle d'orchestrateur final :

1.  **Archivage :** Appeler le `score-service` pour sauvegarder l'historique de cette session.
    
2.  **Mise à jour :** Appeler le `player-service` pour ajouter les points gagnés au score total du joueur.
    

### 4. Exposition dans le Contrôleur

Ajoutez un endpoint dans `GameController` :

-   `POST /api/games/end`
    
-   **Body :** Contiendra l'ID du joueur et le score réalisé.
    

## Partie C : Intégration et Tests Manuels

C'est l'heure de vérité. Lancez vos **4 microservices** simultanément (Ports 8080, 8081, 8082, 8083).

### Scénario de Test (Postman/cURL)

1.  **Vérification de l'état initial :**
    
    -   Récupérez le joueur 1 (GET `http://localhost:8081/api/players/1`). Notez son score (ex: 100).
        
2.  **Fin de partie :**
    
    -   Simulez une fin de partie via le moteur.
        
    -   `POST http://localhost:8080/api/games/end`
        
    -   Body : `{ "playerId": 1, "score": 50 }`
        
3.  **Vérifications finales :**
    
    -   Le moteur doit répondre 200 OK.
        
    -   Vérifiez le **Joueur** (GET port 8081) : Son score doit être passé à **150** (100 + 50).
        
    -   Vérifiez l'**Historique** (H2 Console ou GET port 8083 si implémenté) : Une ligne doit avoir été créée avec la date du jour.
        

## Partie D : Rendu

Le rendu se fait exclusivement sur Github Classroom, voici les différents dépôts pour les différents groupes de TD :

-   Groupe de TD1 :  [https://classroom.github.com/a/OS70knsH](https://classroom.github.com/a/OS70knsH)
-   Groupe de TD2 :  [https://classroom.github.com/a/5azKubG6](https://classroom.github.com/a/5azKubG6)
-   Groupe de TD3 :  [https://classroom.github.com/a/U2uIBiIL](https://classroom.github.com/a/U2uIBiIL)

Ajoutez vos **nouvelles réalisations dans le readme** (notamment l'explication de votre architecture à 4 services) puis **rajoutez votre travail au dépôt** existant, enfin **finalisez votre livraison en ajoutant un tag** !


```bash
git tag td5
git push origin --tags
```


# Fiche Projet - Étape 1 : Fondations et Service `player-service`

Ce document guide les étudiants dans la configuration de leur environnement de développement et la création du premier microservice : la gestion des joueurs.

## Objectifs de l'Étape 1

-   Configurer correctement l'environnement Java (JDK 21).
    
-   Créer le projet **`player-service`** (Spring Boot 4.0).
    
-   Implémenter l'Entité, le Repository, le Service et le Contrôleur du Joueur.
    
-   Exposer trois endpoints REST pour la consultation des joueurs (GET all / GET by ID).
    
-   Charger des données initiales via un `CommandLineRunner` **OU** un fichier SQL.
    

## Partie A : Configuration de l'Environnement

### 1. Installation du Java Development Kit (JDK)

-   **Version requise :** JDK **21** (recommandé).
    
-   **Vérification :** Exécutez `java -version` dans un terminal pour confirmer l'installation.
    

### 2. Configuration de la Variable d'Environnement `JAVA_HOME`

-   La variable `JAVA_HOME` doit pointer vers le répertoire racine de votre installation du JDK (excluant le répertoire `bin`).
    
-   **Vérification :** Assurez-vous que cette variable est correctement définie dans votre environnement système.
    

### 3. Installation et Configuration de l'IDE

-   **IDE Requis :** JetBrains **IntelliJ IDEA Ultimate**.
    
-   **Build Tool :** Les étudiants peuvent choisir d'utiliser **Maven** ou **Gradle**. Les deux outils sont acceptés pour la gestion des dépendances et du build du projet.
    

## Partie B : Création du Microservice `player-service`

### 1. Génération et Dépendances du Projet

Utilisez le **Spring Initializr** (`start.spring.io`) ou l'assistant d'IntelliJ pour créer le projet :

-   **Nom du Service :** `player-service`
    
-   **Version de Spring Boot :** 4.0.
    
-   **Dépendances Requises :**
    
    -   `Spring Web`
        
    -   `Spring Data JPA`
        
    -   `H2 Database`
        
    -   `Lombok`
        

### 2. Création de l'Entité `Player` (Joueur)

Créez une classe **`Player`** dans le package `model` ou `entity` avec les annotations JPA et Lombok.

```java
// Structure de la classe Player
@Entity
@Getter
@Setter
@Builder // Ajout du Builder pour l'alternative CommandLineRunner
@NoArgsConstructor
@AllArgsConstructor
public class Player {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String pseudo;

    private int score = 0; // Initialisé à 0
}
```

### 3. Création du `Repository`

Créez l'interface **`PlayerRepository`** qui étend `JpaRepository<Player, Long>` pour hériter des opérations CRUD standard.

### 4. Création du `Service`

Créez la classe **`PlayerService`** pour encapsuler la logique métier simple (injection du `PlayerRepository`).

-   Implémentez les méthodes : `findAllPlayers()`, `findPlayerById(Long id)`, `createPlayer(Player player)`, `updatePlayer(Long id, Player player)` et `deletePlayer(Long id)`.
    

### 5. Création du `Controller`

Créez la classe **`PlayerController`** (`@RestController`, `@RequestMapping("/api/players")`) pour exposer les Web Services REST.
| **Méthode** | **URI** | **Description** |
|--|--|--|
| `GET` | `/api/players` | Retourne la liste de tous les joueurs. |
| `GET` | `/api/players/{id}` | Retourne un seul joueur via son ID. |
| `POST` | `/api/players` | Crée un nouveau joueur. |

### 6. Chargement des Données Initiales

Les étudiants doivent choisir **UNE SEULE** des méthodes suivantes pour charger des joueurs de test dans la base H2 au démarrage :

#### Alternative 1 : Utilisation d'un `CommandLineRunner` (Programmation Java)

Ajoutez un `CommandLineRunner` dans votre classe principale (`PlayerServiceApplication`) pour injecter les données de manière programmatique en utilisant les _builders_ de Lombok :

```java
// Dans PlayerServiceApplication.java ou une classe @Configuration
@Bean
public CommandLineRunner loadData(PlayerRepository repository) {
    return (args) -> {
        // Chargement des données de test via Player.builder()
        repository.save(Player.builder().pseudo("Neo").score(500).build());
        repository.save(Player.builder().pseudo("Trinity").score(750).build());
        repository.save(Player.builder().pseudo("Morpheus").score(1000).build());

        System.out.println("-> " + repository.count() + " joueurs de test chargés.");
    };
}
```

#### Alternative 2 : Utilisation d'un Fichier SQL Dédié

Créez un fichier nommé **`data.sql`** dans le répertoire `src/main/resources`. Spring Boot le détectera automatiquement et exécutera les requêtes au démarrage :

```SQL
-- Fichier src/main/resources/data.sql
INSERT INTO player (pseudo, score) VALUES ('Alice', 100);
INSERT INTO player (pseudo, score) VALUES ('Bob', 50);
INSERT INTO player (pseudo, score) VALUES ('Charlie', 200);
```

### 7. Validation et Test

1.  Démarrez l'application Spring Boot.
    
2.  Vérifiez le fonctionnement des 3 points d'entrée d'API que vous avez réalisé.

3. Vérifiez que les méthodes que vous avez réalisé dans le service fonctionnent bien soit en créant des méthodes de contrôleur qui les appellent, soit en exécutant directement des appels vous même au lancement de l'application avec quelques logs.

### 8. Rendu

Le rendu se fait exclusivement sur Github Classroom, voici les différents dépôts pour les différents groupes de TD :

- Groupe de TD1 : https://classroom.github.com/a/OS70knsH
- Groupe de TD2 : https://classroom.github.com/a/5azKubG6
- Groupe de TD3 : https://classroom.github.com/a/U2uIBiIL

A chaque rendu, vous devrez penser à ajouter un readme et ce dernier devra préciser l'environnement que vous avez utilisé (versions de java / maven / gradle) ainsi que la liste de ce que vous avez réalisé avec des précisions afin que l'on puisse facilement tester votre production.

Sans readme, votre projet ne sera pas corrigé. 

Ecrivez votre readme à la main afin qu'il ne soit pas trop verbeux.

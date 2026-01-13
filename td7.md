# Fiche TD - Étape 7 : Service Discovery "Maison"


Objectif : Comprendre le fonctionnement interne d'un annuaire de services. Nous allons supprimer les URLs "en dur" (localhost:8081, etc.) et permettre à nos services de se trouver dynamiquement, même s'ils changent de port.

## Objectifs du TD

1.  Créer un microservice **`discovery-service`** (L'annuaire).
    
2.  Implémenter un mécanisme d'**Auto-Enregistrement** dans les autres services.
    
3.  Modifier le **`game-engine`** pour qu'il interroge l'annuaire avant d'appeler un service.
    
## Partie A : Création du `discovery-service` (L'Annuaire)

Ce service sera la "vérité centrale" de votre architecture. Il sait où se trouve tout le monde.

### 1. Configuration

-   Générez un projet Spring Boot (Web, Lombok).
    
-   **Port :** `8761` (C'est le port standard pour les services de discovery).
    

### 2. Le Registre (En mémoire)

Vous avez besoin d'une structure de données pour stocker les services.

-   Structure suggérée : `Map<String, List<String>>`.
    
    -   _Clé_ : Nom du service (ex: `player-service`).
        
    -   _Valeur_ : Liste des URLs disponibles (ex: `["http://localhost:8081", "http://localhost:8091"]`).
        

### 3. Les Endpoints

Exposez une API REST (`DiscoveryController`) :

1.  **Enregistrement (Register) :**
    
    -   `POST /api/registry`
        
    -   Body : `{ "serviceName": "...", "url": "..." }`
        
    -   Action : Ajoutez l'URL à la liste correspondante.
        
2.  **Découverte (Lookup)  :**
    
    -   `GET /api/discovery/{serviceName}`
        
    -   Action : Retournez **TOUTES** les URL pour le service correspondant.
            
    -   _Cas d'erreur :_ Renvoyer 404 si le service n'est pas connu.
        
## Partie B : Auto-Enregistrement des Services

Les services (`player`, `question`, `score`) doivent se signaler au démarrage.

### 1. Modification des Services

Dans **chacun** des 3 services de données :

1.  Ajoutez `RestClient` (si ce n'est pas déjà fait).
    
2.  Utilisez un `ApplicationListener<ApplicationReadyEvent>` ou un `CommandLineRunner` pour exécuter du code au démarrage.
    

### 2. Implémentation de l'appel

Au démarrage de l'application, le service doit envoyer une requête POST au `discovery-service` (http://localhost:8761/api/registry).

-   **Nom du service :** Utilisez `spring.application.name` (définissez-le dans `application.properties`).
    
-   **URL :** Construisez l'URL locale.
    
    -   _Astuce :_ Pour récupérer le port réel au runtime, vous pouvez injecter `Environment` de Spring.
        
```java
// Code partiel, à compléter
@Component
public class RegistrationRunner implements CommandLineRunner {
		// Récupérez les valeurs de configuration de cette manière
    @Value("${server.port}")
    private String port;
    @Value("${spring.application.name}")
    private String appName

    @Override
    public void run(String... args) {
        String myUrl = "http://localhost:" + port;
        // Appel POST vers http://localhost:8761/api/registry
        // Body: { "serviceName": appName, "url": myUrl }
        System.out.println("✅ Service enregistré : " + appName);
    }
}
```

## Partie C : Modification du `game-engine` (Le Client)

Le moteur de jeu ne doit plus appeler directement `http://localhost:8081`.

### 1. Adaptation des Clients

Modifiez vos classes qui interagissent avec les autres services

Au lieu d'avoir l'URL de base dans le constructeur ou la config :

1.  Contactez `discovery-service`
    
2.  Avant chaque appel (ou à l'initialisation), demandez à Discovery les URLs des services que vous voulez contacter et en choisir une, au hasard pour l'instant.
    

**Exemple de flux :**

1.  GameEngine veut un joueur.
    
2.  GameEngine appelle `GET http://localhost:8761/api/discovery/player-service`.
    
3.  Réponse : `http://localhost:8087`.
    
4.  GameEngine appelle `GET http://localhost:8087/api/players/1`.

Tout ce que vous avez fait jusqu'à maintenant doit continuer de fonctionner avec les changements que vous avez fait.
   
## Partie D : Démonstration et Load Balancing

C'est l'étape cruciale pour valider votre architecture dynamique. Nous allons lancer plusieurs instances d'un même service pour prouver que le `game-engine` est capable de piocher une adresse au hasard et de répartir la charge.

### 1. Lancer plusieurs instances

Pour cet exercice, nous allons dupliquer le `question-service`.

1.  Lancez le **`discovery-service`** en premier (c'est important, car les autres vont essayer de le contacter au démarrage).
    
2.  Lancez le **`player-service`** et le **`score-service`** normalement.
    
3.  Lancez une **première instance** du `question-service` (Port 8082 par défaut).
    
4.  Lancez une **seconde instance** du `question-service` sur un autre port :
    
    -   **Sous IntelliJ :** Allez dans "Edit Configurations" -> Sélectionnez `QuestionCatalogApplication` -> Cliquez sur l'icône "Duplicate" (copier).
        
    -   Dans la nouvelle configuration, ajoutez dans "Program arguments" ou "Environment variables" : `--server.port=8092`.
        
    -   Lancez cette nouvelle configuration.
        

### 2. Vérification de l'Enregistrement

Regardez les logs ou la console de votre **`discovery-service`**.

Vous devriez voir passer les requêtes d'enregistrement. Si vous interrogez l'API du discovery (`GET http://localhost:8761/api/discovery/question-service`), vous devez recevoir une réponse JSON contenant les deux URLs :

```json
[
  "http://localhost:8082",
  "http://localhost:8092"
]
```

### 3. Test de Répartition de Charge

Puisque vous avez codé une sélection aléatoire ("au hasard") dans le `game-engine`, nous allons vérifier que les deux instances de questions sont utilisées.

1.  Ouvrez Postman (ou votre collection de test).
    
2.  Lancez la requête de démarrage de partie (`POST /api/games/start/...`) environ **10 fois de suite**.
    
3.  Observez les logs (la console) des **deux instances** du `question-service`.
    
    -   **Attendu :** Vous devriez voir des traces de requêtes (SELECT, logs d'accès, etc.) apparaître sur l'instance du port 8082 et sur celle du port 8092.

_Note : Comme la sélection est aléatoire, la répartition ne sera pas parfaitement 50/50 sur un petit nombre d'essais, mais les deux instances doivent finir par travailler._

## Rendu

Le rendu se fait exclusivement sur Github Classroom, voici les différents dépôts pour les différents groupes de TD :

-   Groupe de TD1 : [https://classroom.github.com/a/OS70knsH](https://classroom.github.com/a/OS70knsH)
    
-   Groupe de TD2 : [https://classroom.github.com/a/5azKubG6](https://classroom.github.com/a/5azKubG6)
    
-   Groupe de TD3 : [https://classroom.github.com/a/U2uIBiIL](https://classroom.github.com/a/U2uIBiIL)
    

Ajoutez vos **nouvelles réalisations dans le readme** (expliquez brièvement comment vous avez testé le multi-instance) puis **rajoutez votre travail au dépôt** existant, enfin **finalisez votre livraison en ajoutant un tag** !

```bash
git tag td7
# puis 
git push origin --tags
```

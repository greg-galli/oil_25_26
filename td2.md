# Fiche Projet - Étape 2 : API REST Complètes et Service `question-catalog-service`

Cette étape se concentre sur l'achèvement des API REST pour les services **`player-service`** et la création du nouveau service **`question-catalog-service`**, en insistant sur le respect des standards RESTful.

## Objectifs de l'Étape 2

-   Compléter l'API REST du **`player-service`** (ajout des méthodes PUT, PATCH, DELETE).
    
-   Créer le nouveau microservice **`question-catalog-service`**.
    
-   Implémenter l'API REST complète (GET, POST, PUT, PATCH, DELETE) pour les questions.
    
-   Gérer les **codes d'état HTTP appropriés** (200, 201, 204, 400, 404, 500) et fournir des messages d'erreur détaillés.
    


## Partie A : Amélioration et Complément du `player-service`

Le service existant doit être étendu pour prendre en charge l'ensemble des opérations CRUD (Create, Read, Update, Delete) de manière RESTful.

### 1. Implémentation des Opérations d'Édition et de Suppression

Vous devez ajouter ou modifier les méthodes suivantes dans le `PlayerController` et le `PlayerService` :
| **Méthode HTTP** | **URI** | **Rôle** | **Description et Codes d'État** |
|--|--|--|--|
| `PUT` | `/api/players/{id}` | Mise à jour complète | Remplacer l'intégralité de la ressource. |
| `PATCH` | `/api/players/{id}` | Mise à jour partielle | Mettre à jour seulement certains champs (ex: score). |
| `DELETE` | `/api/players/{id}` | Suppression | Supprimer la ressource spécifiée. |


### 2. Gestion des Codes d'État et des Erreurs

La gestion des erreurs doit être mise en place de manière centralisée (via `@ControllerAdvice` ou des `ResponseStatusException`) pour s'assurer que l'API est robuste.
| **Code HTTP** | **Cas d'Usage** | **Message d'Erreur (Exemple)** |
|--|--|--|
| **200 OK** | Succès de `GET`, `PUT`, `PATCH`. | N/A |
| **201 Created** | Succès de `POST`. | N/A |
| **204 No Content** | Succès de `DELETE`. | N/A |
| **400 Bad Request** | Erreur dans la requête. | "Le pseudo est manquant dans le corps de la requête" ou "Le format du score est invalide." |
| **404 Not Found** | Ressource non trouvée. | "Joueur avec l'ID X non trouvé pour l'opération." |

**Attention:** à la distinction entre `PUT` (remplace tout) et `PATCH` (met à jour une partie).

----------

## Partie B : Création du Service `question-catalog-service`

Le deuxième microservice doit être créé avec la même rigueur, en suivant la décomposition classique de Spring.

### 1. Génération du Projet et Dépendances

Créer un nouveau projet Spring Boot (`question-catalog-service`) avec les mêmes dépendances que l'Étape 1 (`Spring Web`, `Spring Data JPA`, `H2 Database`, `Lombok`).

### 2. Structure de l'Entité `Question`

L'entité `Question` devra contenir au moins les champs suivants (utiliser les annotations JPA et Lombok) :
| **Champ** | **Type** | **Contrainte** |
|--|--|--|
| `id` | Long | Clé Primaire |
| `texte` | String | Le texte de la question. |
| `reponseCorrecte` | String | La réponse attendue. |
| `propositions` | List<String> | Liste des mauvaises et bonnes réponses (pour QCM). |
| `categorie` | String | Thème de la question (ex: "Java", "Architecture"). |

### 3. Déroulé Classique Spring (Entité / Repository / Service / Controller)

Vous devez suivre la structure établie dans l'Étape 1 :

1.  Créer l'Entité **`Question`**.
    
2.  Créer le **`QuestionRepository`** (`JpaRepository`).
    
3.  Créer le **`QuestionService`** (logique métier).
    
4.  Créer le **`QuestionController`** (exposition des API REST).
    

### 4. Implémentation des API REST Complètes

Le `QuestionController` doit exposer les endpoints suivants pour garantir une API complète :

| **Méthode HTTP** | **URI de Collection** | **URI de Singleton** | 
|--|--|--|
| **GET** | `/api/questions` | `/api/questions/{id}` |
| **POST** | `/api/questions` | N/A |
| **PUT** | N/A | `/api/questions/{id}` |
| **PATCH** | N/A | `/api/questions/{id}` |
| **DELETE** | N/A | `/api/questions/{id}` |



### 5. Chargement des Données Initiales

Utiliser au choix la méthode du `CommandLineRunner` (avec le `Builder` de Lombok) ou le fichier `data.sql` pour charger un jeu de 3 à 5 questions de test au démarrage de ce nouveau service.

## Vérification et Validation

### 1. Vérifications
-   Tester tous les endpoints (GET, POST, PUT, PATCH, DELETE) pour les deux services en vérifiant systématiquement :
    
    1.  La bonne exécution (codes 2xx).
        
    2.  La gestion des cas d'erreur (tentative de GET avec un ID inexistant doit retourner 404 avec un message précis).
        
    3.  La différence de comportement entre `PUT` et `PATCH`.

Pour vérifier le fonctionnement des endpoints, vous pouvez utiliser Postman ou tout autre outil de votre choix.
Bientôt, nous verrons le fonctionnement détaillé de ces outils et vous devrez revenir sur ce que vous avez créé, autant prendre un peu d'avance.

### 2. Rendu

Le rendu se fait exclusivement sur Github Classroom, voici les différents dépôts pour les différents groupes de TD :

-   Groupe de TD1 :  [https://classroom.github.com/a/OS70knsH](https://classroom.github.com/a/OS70knsH)
-   Groupe de TD2 :  [https://classroom.github.com/a/5azKubG6](https://classroom.github.com/a/5azKubG6)
-   Groupe de TD3 :  [https://classroom.github.com/a/U2uIBiIL](https://classroom.github.com/a/U2uIBiIL)

Ajoutez vos **nouvelles réalisations dans le readme** puis **rajoutez votre travail au dépôt** existant, enfin **finalisez votre livraison  en ajoutant un tag** ! 

```bash
git tag td2
# puis 
git push origin --tags
```

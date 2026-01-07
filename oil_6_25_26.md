# Fiche TD - Étape 6 : Industrialisation des Tests d'API

**Objectif :** Passer d'un test manuel "au cas par cas" à une suite de tests automatisés complète et pérenne. Vous allez constituer le filet de sécurité de votre application distribuée.

**Outils :** [Postman](https://www.postman.com/downloads/) ou [Bruno](https://www.usebruno.com/) (Alternative Open Source acceptée).

## Objectifs du TD

1.  Structurer une **Collection** de requêtes organisée par Microservice.
    
2.  Utiliser les **Variables d'Environnement** pour ne jamais hardcoder d'URLs.
    
3.  Couvrir **100% des cas d'usage** (Cas nominaux et Gestion d'erreurs).
    
4.  Écrire des **Scripts de Tests** (Assertions) pour valider automatiquement les réponses.
    
5.  Mettre en place des **Scénarios Dynamiques** (Chaînage de requêtes).
    
## Étape 1 : Configuration de l'Environnement

Comme vu en cours, une URL `localhost` ne doit jamais être écrite en dur dans une requête.

1.  Créez un nouvel Environnement nommé **"Local OIL"**.
    
2.  Définissez les variables suivantes correspondant à votre architecture actuelle :
    
    -   `url_player`: `http://localhost:8081`
        
    -   `url_question`: `http://localhost:8082`
        
    -   `url_game`: `http://localhost:8080`
        
    -   `url_score`: `http://localhost:8083`
        

> **Règle d'or :** Dans vos requêtes, vous devrez toujours utiliser `{{url_player}}/api/players`, etc.

## Étape 2 : Structure de la Collection

Créez une Collection nommée "Microservices_OIL".

Organisez-la avec des dossiers pour séparer les responsabilités :

-   📂 **01. Player Service**
    
-   📂 **02. Question Service**
    
-   📂 **03. Game Engine**
    
-   📂 **04. Score Service** 
    
## Étape 3 : Implémentation des Requêtes et Assertions

Pour **chaque endpoint** que vous avez développé dans les TD 1 à 5, vous devez créer plusieurs requêtes pour couvrir tous les cas.

### A. Les Cas de Succès (Happy Path)

Pour chaque requête "qui marche", ajoutez des tests dans l'onglet **"Tests"** (Postman) ou **"Assertions"** (Bruno).

**Exemple : `GET {{url_player}}/api/players/1`**

-   Vérifier que le Status est `200 OK`.
    
-   Vérifier que le temps de réponse est `< 500ms`.
    
-   Vérifier que le body contient bien un champ `id` égal à 1.
    
-   Vérifier que le Content-Type est bien `application/json`.
    
```javaScript
// Exemple de script Postman
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
pm.test("Response is fast", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});
```

### B. Les Cas d'Erreurs (Edge Cases)

Une API robuste se juge à sa gestion des erreurs. Vous devez créer des requêtes spécifiques pour provoquer des erreurs.

**Exemple : `GET {{url_player}}/api/players/99999` (ID Inconnu)**

-   **Attendu :** Status `404 Not Found`.
    
-   **Test :** Vérifier que le body contient un message d'erreur explicite.
    

**Exemple : `POST {{url_player}}/api/players` (Données Invalides)**

-   Envoyez un body vide ou avec un score négatif (si votre logique l'interdit).
    
-   **Attendu :** Status `400 Bad Request`.
    

**Checklist des codes à tester impérativement :**

-   🟢 **200 OK** (Lecture réussie)
    
-   🟢 **201 Created** (Création réussie)
    
-   🟢 **204 No Content** (Suppression réussie - body vide)
    
-   🟠 **400 Bad Request** (Validation des données échouée)
    
-   🟠 **404 Not Found** (Ressource inexistante)
    

## Étape 4 : Scénarios Dynamiques (Chaînage)

Dans le dossier **Game Engine**, vous devez démontrer que vous savez passer des données d'une requête à l'autre.

**Scénario à implémenter : "Cycle de vie complet d'une partie"**

1.  **Requête 1 :** Créer un joueur (`POST`).
    
    -   _Script :_ Récupérer l'ID du nouveau joueur et le stocker dans une variable de collection `current_player_id`.
        
2.  **Requête 2 :** Démarrer une partie (`POST /start`) en utilisant `{{current_player_id}}`.
    
    -   _Script :_ Vérifier que la partie contient bien des questions.
        
3.  **Requête 3 :** Terminer la partie (`POST /end`) en utilisant `{{current_player_id}}`.
    
4.  **Requête 4 :** Vérifier le score (`GET /players/{{current_player_id}}`).
    
    -   _Script :_ Vérifier que le score a bien augmenté.
        
## Étape 5 : Automatisation

1.  Utilisez le **"Collection Runner"** de Postman (ou l'équivalent dans Bruno) pour lancer l'intégralité de votre suite de tests d'un seul clic.
    
2.  Vérifiez que toutes les barres sont vertes (All Tests Passed).
    
3.  Si un test échoue, corrigez votre code (Java) ou votre test (JS).
    

##  Rendu et Maintenance

Ce travail n'est pas "one-shot". Cette collection devient une partie intégrante de votre projet.

1.  **Export :** Exportez votre Collection et votre Environnement au format JSON.
    
2.  **Git :** Placez ces fichiers dans un dossier `/tests/postman` (ou `/tests/bruno`) à la racine de votre dépôt Git.
    
3.  **Readme :** Ajoutez une section dans votre `README.md` expliquant comment importer et lancer ces tests.
    
Le rendu se fait exclusivement sur Github Classroom, voici les différents dépôts pour les différents groupes de TD :

-   Groupe de TD1 :  [https://classroom.github.com/a/OS70knsH](https://classroom.github.com/a/OS70knsH)
-   Groupe de TD2 :  [https://classroom.github.com/a/5azKubG6](https://classroom.github.com/a/5azKubG6)
-   Groupe de TD3 :  [https://classroom.github.com/a/U2uIBiIL](https://classroom.github.com/a/U2uIBiIL)

Poussez les modifications sur votre dépôt GitHub Classroom habituel avec le tag td6.

```bash
git tag td6
git push origin --tags
```

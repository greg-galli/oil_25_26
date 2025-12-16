# Fiche TD - Séance 4 : Introduction aux Tests Automatisés dans Spring Boot

**Objectif :** Sécuriser le code de nos microservices en implémentant les trois niveaux de la pyramide de tests: Unitaires, Intégration et End-to-End (E2E).

## 1. Tests Unitaires (Unit Tests)

**Concept :** Tester une unité de code (classe/méthode) de manière isolée sans charger le contexte Spring complet. Nous allons utiliser **Mockito** pour simuler les dépendances.

**Cible :** Le `GameService` du projet `game-engine-service`.
*Pourquoi ?* Ce service contient de la logique métier (orchestration) et dépend d'autres composants (`PlayerClient`, `QuestionClient`).

**Exercice :**
Dans `src/test/java/.../GameServiceTest.java` :

```java
@ExtendWith(MockitoExtension.class) // Extension JUnit 5 pour Mockito
class GameServiceTest {

    @Mock // Simule le client Player
    private PlayerClient playerClient;

    @Mock // Simule le client Question
    private QuestionClient questionClient;

    @InjectMocks // Injecte les mocks dans le service à tester
    private GameService gameService;

    @Test
    void shouldStartNewGameSuccessfully() {
        // 1. GIVEN (Préparation du comportement des mocks)
        Long playerId = 1L;
        PlayerDTO mockPlayer = new PlayerDTO(playerId, "Neo", 0);
        List<QuestionDTO> mockQuestions = List.of(new QuestionDTO(1L, "Q1", "R1"));

        // On définit ce que les dépendances doivent renvoyer (Stubbing)
        Mockito.when(playerClient.getPlayerById(playerId)).thenReturn(mockPlayer);
        Mockito.when(questionClient.getAllQuestions()).thenReturn(mockQuestions);

        // 2. WHEN (Exécution de la méthode à tester)
        GameDTO result = gameService.startNewGame(playerId, 1);

        // 3. THEN (Vérifications / Assertions)
        // On vérifie que le résultat est cohérent
        assertEquals("Neo", result.player().pseudo());
        assertEquals(1, result.questions().size());
        
        // On vérifie que les mocks ont bien été appelés
        Mockito.verify(playerClient).getPlayerById(playerId);
    }
}
```

## 2. Tests d'Intégration (Integration Tests)

**Concept :** Tester les interactions avec des composants externes comme la base de données.

**Outil :** `@DataJpaTest` qui charge uniquement la couche JPA et configure une base de données en mémoire (H2).

**Cible :** Le `PlayerRepository` du projet `player-service`.

**Exercice :**
Dans `src/test/java/.../PlayerRepositoryTest.java` :

```java
@DataJpaTest // Charge la config JPA et H2
class PlayerRepositoryTest {

    @Autowired
    private PlayerRepository playerRepository;

    @Test
    void shouldFindPlayerByPseudo() {
        // 1. Préparation (Sauvegarder un vrai objet en base H2)
        Player player = Player.builder().pseudo("Trinity").score(10).build();
        playerRepository.save(player);

        // 2. Exécution (Appel de la méthode du repository)
        Optional<Player> found = playerRepository.findByPseudo("Trinity");

        // 3. Vérification
        assertTrue(found.isPresent());
        assertEquals(10, found.get().getScore());
    }
    
    @Test
    void shouldSavePlayer() {
        // Test simple de persistance
        Player newPlayer = Player.builder().pseudo("Morpheus").build();
        Player saved = playerRepository.save(newPlayer);
        
        assertNotNull(saved.getId()); // L'ID doit avoir été généré
    }
}

```

---

## 3. Tests de Bout en Bout (E2E / Functional Tests)

**Concept :** Tester le service comme une "boîte noire" en envoyant de vraies requêtes HTTP.

**Outil :** `@SpringBootTest` avec un port aléatoire pour lancer tout le contexte applicatif.

**Cible :** Le `PlayerController` du projet `player-service`.

**Exercice :**
Dans `src/test/java/.../PlayerE2ETest.java` :

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) // Lance le serveur complet
class PlayerE2ETest {

    @Autowired
    private TestRestTemplate restTemplate; // Outil pour faire des appels REST vers nous-mêmes

    @Test
    void shouldCreateAndRetrievePlayer() {
        // 1. Création (POST)
        Player newPlayer = Player.builder().pseudo("Alice").score(999).build();
        ResponseEntity<Player> createResponse = restTemplate.postForEntity("/api/players", newPlayer, Player.class);

        assertEquals(HttpStatus.CREATED, createResponse.getStatusCode());
        assertNotNull(createResponse.getBody().getId());
        Long newId = createResponse.getBody().getId();

        // 2. Récupération (GET) - Vérification que la donnée est bien accessible via l'API
        ResponseEntity<Player> getResponse = restTemplate.getForEntity("/api/players/" + newId, Player.class);
        
        assertEquals(HttpStatus.OK, getResponse.getStatusCode());
        assertEquals("Alice", getResponse.getBody().getPseudo());
    }
}

```

---

## 4. Exécution des Tests

Vous devez savoir lancer des tests via votre IDE (clic droit sur le dossier `test` -> "Run 'All Tests'"), mais aussi via les lignes de commande, ce qui est essentiel pour l'automatisation (CI/CD) mentionnée dans le cours.

### Avec Maven
```bash
# Lancer tous les tests
mvn test

# Lancer un test spécifique
mvn -Dtest=GameServiceTest test
```

### Avec Gradle

```bash
# Lancer tous les tests (Linux/Mac)
./gradlew test

# Lancer tous les tests (Windows)
gradlew.bat test

# Lancer un test spécifique
./gradlew test --tests "fr.miage.game.GameServiceTest"
```

### Analyse des résultats

Si un test échoue, allez lire la StackTrace. Vous y trouverez surement :

* `AssertionError` : Le code fonctionne mais ne donne pas le résultat attendu.
* `MockitoException` : Les mocks sont mal configurés (ex: on teste une méthode qui appelle une dépendance non mockée).

## 5. Validation et Test

Pour cette étape vous devez implémenter ce qui est indiqué dans cette fiche et rajouter : 
- 4 tests unitaires supplémentaires
- 2 tests d'intégration supplémentaires

### Rendu

Le rendu se fait exclusivement sur Github Classroom, voici les différents dépôts pour les différents groupes de TD :

-   Groupe de TD1 :  [https://classroom.github.com/a/OS70knsH](https://classroom.github.com/a/OS70knsH)
-   Groupe de TD2 :  [https://classroom.github.com/a/5azKubG6](https://classroom.github.com/a/5azKubG6)
-   Groupe de TD3 :  [https://classroom.github.com/a/U2uIBiIL](https://classroom.github.com/a/U2uIBiIL)

Ajoutez vos **nouvelles réalisations dans le readme** puis **rajoutez votre travail au dépôt** existant, enfin **finalisez votre livraison  en ajoutant un tag** ! 

```bash
git tag td4
# puis 
git push origin --tags
```


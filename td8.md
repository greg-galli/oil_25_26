# Fiche TD - Séance 8 : WebSockets Avancés - Salles et Messages Privés

**Pré-requis :** Avoir réalisé le moteur de jeu (TD3/5) et le Discovery (TD7).

**Contexte:** Au lieu d'avoir un seul flux global (tout le monde voit tout), nous allons créer :

1.  **Des Salles de Jeu (Rooms) :** Les événements d'une partie ne sont reçus _que_ par les joueurs de _cette_ partie.
2.  **Des Messages Privés (Direct Messaging) :** Le moteur de jeu envoie une information secrète à un seul joueur (ex: "Tu es l'imposteur" ou "Voici ta main de cartes").
3.  **Chat In-Game :** Permettre aux joueurs d'envoyer des messages aux autres joueurs de la même partie.

Notez que c'est exactement comme cela que fonctionnent les jeux multijoueurs (Among Us, Kahoot, etc.).

## Objectifs du TD

1.  Comprendre la notion de **Topics Dynamiques** (Multicast).
2.  Implémenter l'envoi de messages **ciblés** vers un utilisateur spécifique (Unicast).
3.  Gérer la communication **Client vers Serveur** (et non plus seulement Serveur vers Client).
    

## Partie A : Isolation des Parties (Les "Rooms")

Actuellement, votre Dashboard écoute `/topic/events`. Si 1000 parties se jouent en parallèle, le client reçoit tout. C'est inefficace et non sécurisé.

Nous allons faire en sorte que les messages soient publiés sur `/topic/game/{gameId}`.

### 1. Modification du `GameEvent`

Assurez-vous que votre DTO `GameEvent` contient bien l'ID de la partie (`gameId`).

### 2. Modification du `GameService` (Backend)

Dans le `GameEngine`, modifiez la méthode d'envoi pour qu'elle soit dynamique.

```java
public void notifyGameUpdate(String gameId, GameEvent event) {
    // Au lieu d'envoyer sur un canal fixe, on inclut l'ID de la partie dans l'URL
    String destination = "/topic/game/" + gameId;
    this.messagingTemplate.convertAndSend(destination, event);
}
```

Mettez à jour vos méthodes `startGame` et `endGame` pour utiliser cette nouvelle méthode avec l'ID de la partie (vous allez devoir remplacer l'identifiant de la partie qui était un UUID random (cf. TD3) par une valeur que vous transmettrez au serveur vous même ex: gameId=partie-1 | gameId=partie-2 etc.)

### 3. Modification du Client (Frontend)

Dans votre `index.html` (ou un nouveau `game.html`), ajoutez un champ de saisie pour que l'utilisateur puisse dire quelle partie il veut observer.

```javascript
function connectToGame() {
    var gameId = document.getElementById("gameIdInput").value;
    // ... connexion SockJS standard ...
    
    stompClient.connect({}, function (frame) {
        // Souscription DYNAMIQUE
        stompClient.subscribe('/topic/game/' + gameId, function (message) {
            showEvent(JSON.parse(message.body));
        });
    });
}
```

## Partie B : Messages Privés (Unicast)

Imaginez que le moteur de jeu veuille envoyer un "Bonus Secret" à un seul joueur. Les autres ne doivent pas le savoir.

Sans implémenter une couche de sécurité complexe (Spring Security), nous allons utiliser une convention de nommage simple : chaque joueur écoute son propre canal.

### 1. Le Canal Privé

Le client (JS) devra désormais souscrire à **deux** canaux :

1.  La salle de jeu : `/topic/game/{gameId}`
2.  Son canal personnel : `/topic/player/{playerId}`

### 2. Backend : Envoi Ciblé

Ajoutez une méthode dans `GameService` pour envoyer un message à un joueur spécifique.

```java
public void sendPrivateMessage(Long playerId, String secretInfo) {
    String destination = "/topic/player/" + playerId;
    GameEvent secretEvent = new GameEvent("SECRET", playerId, secretInfo, LocalDateTime.now().toString());
    
    this.messagingTemplate.convertAndSend(destination, secretEvent);
}
```

### 3. Test Rapide

Créez un endpoint REST temporaire `POST /api/games/bonus/{playerId}` qui appelle cette méthode, pour pouvoir tester facilement avec Postman que seul le bon client reçoit le message.

----------

## Partie C : Interaction Client -> Serveur (Chat/Emotes)

Jusqu'ici, le client était passif (il écoutait). Nous allons permettre au joueur d'envoyer un message aux autres joueurs de la partie via WebSocket.

### 1. Création du Controller WebSocket

Dans Spring, on utilise `@MessageMapping` pour gérer les messages entrants (comme `@PostMapping` pour REST).

Créez `GameSocketController` :

```java
@Controller // Attention: @Controller, pas @RestController
public class GameSocketController {

    @MessageMapping("/chat/{gameId}") // L'URL d'arrivée (préfixée par /app dans la config)
    @SendTo("/topic/game/{gameId}")   // Rediffusion automatique vers la salle
    public GameEvent handleChatMessage(@DestinationVariable String gameId, ChatMessage incomingMessage) {
        // incomingMessage est un petit DTO { "sender": "Neo", "content": "Hello!" }
        // vous pouvez utiliser un record
        
        return new GameEvent(
            "CHAT", 
            null, 
            incomingMessage.sender() + " dit : " + incomingMessage.content(), 
            LocalDateTime.now().toString()
        );
    }
}
```

_Note : Assurez-vous que votre `WebSocketConfig` a bien `config.setApplicationDestinationPrefixes("/app");`._

### 2. Envoi depuis le Client JS

Ajoutez un bouton "Envoyer" dans votre page HTML.

```javascript
function sendChat() {
    var gameId = document.getElementById("gameIdInput").value;
    var message = {
        sender: "Moi",
        content: "Salut les amis !"
    };
    
    // Envoi vers /app/chat/{gameId}
    stompClient.send("/app/chat/" + gameId, {}, JSON.stringify(message));
}
```

## Partie D : Le Scénario de Validation

Pour valider ce TD, vous devez réaliser la démonstration suivante :

1.  **Préparation :**
    -   Ouvrez **Deux Navigateurs** différents (Client A et Client B).
    -   Client A : Saisit Game ID "partie-1" et Player ID "10". Se connecte.
    -   Client B : Saisit Game ID "partie-1" et Player ID "20". Se connecte.
    	- _Note importante : jusqu'à maintenant, un seul joueur pouvait se connecter à une partie, pour gérer ce cas vous avez 2 solutions :_
       
       		_1) Rendre la partie accessible à plusieurs joueurs ou..._
       
       		_2) Faire en sorte que l'on puisse rejoindre la partie mais uniquement pour l'aspect communication (websocket), sans toucher à la structure d'une partie qui conserve un seul "joueur", c'est sans doutes la solution la plus simple car elle n'implique pas de modification de votre projet en profondeur_
    -   Ouvrez un **Troisième Navigateur** (Client C).
    -   Client C : Saisit Game ID "partie-2". Se connecte.
   
2.  **Test Isolation (Rooms) :**
	-   Via Postman, démarrez une partie avec l'ID "partie-1".
    -   **Résultat :** Client A et B voient le message. Client C ne voit rien (car il est dans "partie-2").
        
3.  **Test Privé (Unicast) :**
    
    -   Via Postman, déclenchez le bonus pour le Player 10.
    -   **Résultat :** Seul le Client A voit le message "SECRET". Le Client B (pourtant dans la même partie) ne voit rien.
        
4.  **Test Chat :**
    
    -   Le Client A clique sur "Envoyer Message".
    -   **Résultat :** Le message apparaît instantanément chez Client A et Client B.
        
## Rendu

Le rendu se fait sur GitHub Classroom sur le dépôt habituel !

1.  **Readme :** Précisez l'URL pour accéder au tableau de bord.
    
2.  **Tag :**
  
```bash
git tag td8
git push origin --tags
```

Voici comment vous pouvez établir une communication TCP avec des messages XML en utilisant uniquement les sockets Java. Je vais vous montrer comment configurer un serveur et un client TCP qui échangent des messages XML.

### 1. Créer le Serveur TCP

Le serveur va écouter sur un port spécifique et répondre à chaque message reçu.

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
import org.w3c.dom.Element;

public class TcpXmlServer {

    public static void main(String[] args) {
        int port = 1234; // Port d'écoute du serveur

        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("Serveur en écoute sur le port " + port);

            while (true) {
                try (Socket clientSocket = serverSocket.accept()) {
                    System.out.println("Connexion acceptée de " + clientSocket.getInetAddress());

                    // Lire le message XML du client
                    BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                    StringBuilder xmlBuilder = new StringBuilder();
                    String line;
                    while ((line = in.readLine()) != null) {
                        xmlBuilder.append(line);
                    }

                    String requestXml = xmlBuilder.toString();
                    System.out.println("Reçu du client : " + requestXml);

                    // Traiter le message XML
                    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
                    DocumentBuilder builder = factory.newDocumentBuilder();
                    Document requestDocument = builder.parse(new InputSource(new StringReader(requestXml)));
                    Element requestElement = requestDocument.getDocumentElement();

                    // Construire la réponse XML
                    Document responseDocument = builder.newDocument();
                    Element responseElement = responseDocument.createElement("Response");
                    responseElement.appendChild(responseDocument.createTextNode("Bonjour, Client !"));
                    responseDocument.appendChild(responseElement);

                    // Envoyer la réponse au client
                    String responseXml = convertElementToString(responseElement);
                    PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
                    out.println(responseXml);
                    System.out.println("Envoyé au client : " + responseXml);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static String convertElementToString(Element element) {
        // Implémentation de la conversion de l'élément XML en chaîne de caractères
        // Vous pouvez utiliser un Transformer pour cela
        return "<Response>example</Response>"; // Modifier cette ligne pour la conversion appropriée
    }
}
```

### 2. Créer le Client TCP

Le client va se connecter au serveur et envoyer une requête XML, puis recevoir et afficher la réponse.

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.Socket;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
import org.w3c.dom.Element;

public class TcpXmlClient {

    public static void main(String[] args) {
        String hostname = "localhost";
        int port = 1234;

        try (Socket socket = new Socket(hostname, port)) {
            // Construire la requête XML
            DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
            DocumentBuilder builder = factory.newDocumentBuilder();
            Document document = builder.newDocument();
            Element requestElement = document.createElement("Request");
            requestElement.appendChild(document.createTextNode("Bonjour, Serveur !"));
            document.appendChild(requestElement);

            // Envoyer la requête au serveur
            String requestXml = convertElementToString(requestElement);
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            out.println(requestXml);
            System.out.println("Envoyé au serveur : " + requestXml);

            // Lire la réponse du serveur
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            StringBuilder xmlBuilder = new StringBuilder();
            String line;
            while ((line = in.readLine()) != null) {
                xmlBuilder.append(line);
            }

            String responseXml = xmlBuilder.toString();
            System.out.println("Réponse reçue du serveur : " + responseXml);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static String convertElementToString(Element element) {
        // Implémentation de la conversion de l'élément XML en chaîne de caractères
        // Vous pouvez utiliser un Transformer pour cela
        return "<Request>example</Request>"; // Modifier cette ligne pour la conversion appropriée
    }
}
```

### 3. Résumé des Étapes

- **Serveur TCP :** Écoute sur un port, reçoit les messages XML, les traite, et renvoie une réponse XML.
- **Client TCP :** Envoie un message XML au serveur et affiche la réponse reçue.

### 4. Lancer le Serveur et le Client

1. **Lancer le Serveur :** Exécutez `TcpXmlServer` en premier pour mettre le serveur en écoute.
2. **Lancer le Client :** Exécutez `TcpXmlClient` pour envoyer une requête au serveur et recevoir la réponse.

Avec cette configuration, vous avez une communication TCP en utilisant seulement les sockets Java, sans aucune dépendance externe.

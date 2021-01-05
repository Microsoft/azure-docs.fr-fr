---
title: En-têtes de réponse Gremlin d’Azure Cosmos DB
description: Documentation de référence sur les métadonnées de réponse de serveur permettant de résoudre des problèmes supplémentaires
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: reference
ms.date: 09/03/2019
author: christopheranderson
ms.author: chrande
ms.openlocfilehash: 3f5996b281c1985747f754e3796e9fb84f90fdd3
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93356958"
---
# <a name="azure-cosmos-db-gremlin-server-response-headers"></a>En-têtes de réponse du serveur Gremlin d’Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Cet article traite des en-têtes que le serveur Gremlin de Cosmos DB retourne à l’appelant lors de l’exécution d’une requête. Ces en-têtes sont utiles pour la résolution des problèmes de performance des requêtes et la création d’applications s’intégrant de manière native au service Cosmos DB. Ils contribuent également à simplifier les opérations du service clientèle.

Gardez à l’esprit que la dépendance à ces en-têtes limite la portabilité de votre application vers d’autres implémentations Gremlin. En revanche, vous bénéficiez d’une intégration plus étroite à l’API Gremlin de Cosmos DB. Ces en-têtes ne représentent pas une norme TinkerPop.

## <a name="headers"></a>headers

| En-tête | Type | Exemple de valeur | S’il est inclus | Explication |
| --- | --- | --- | --- | --- |
| **x-ms-request-charge** | double | 11.3243 | Success and Failure (« Succès et échec ») | Débit de collection ou de base de données consommé en [unités de requête (RU/s ou RU)](request-units.md) pour un message de réponse partielle. Cet en-tête est toujours présent dans les continuations de requêtes comportant plusieurs blocs. Il reflète les frais associés à un bloc de réponse particulier. Cet en-tête correspond au coût total du parcours uniquement pour les requêtes comportant un unique bloc de réponse. Toutefois, pour la majorité des parcours complexes, cette valeur représente un coût partiel. |
| **x-ms-total-request-charge** | double | 423.987 | Success and Failure (« Succès et échec ») | Débit de collection ou de base de données consommé en [unités de requête (RU/s ou RU)](request-units.md) pour une requête complète. Cet en-tête est toujours présent dans les continuations de requêtes comportant plusieurs blocs. Il indique les frais cumulés depuis le début de la requête. La valeur de cet en-tête dans le dernier bloc indique l’ensemble des frais de requête. |
| **x-ms-server-time-ms** | double | 13,75 | Success and Failure (« Succès et échec ») | Cet en-tête aide à résoudre les problèmes de latence. Il indique, en millisecondes, le temps passé par le serveur Gremlin de Cosmos DB pour exécuter et produire un message de réponse partielle. En utilisant la valeur de cet en-tête et en la comparant à la latence de requête globale, les applications peuvent calculer la surcharge de latence du réseau. |
| **x-ms-total-server-time-ms** | double | 130.512 | Success and Failure (« Succès et échec ») | Cet en-tête indique, en millisecondes, le temps total passé par le serveur Gremlin de Cosmos DB pour exécuter le parcours complet. Il est inclus dans chaque réponse partielle. Il représente le temps d’exécution cumulé depuis le début de la requête. La dernière réponse indique le temps d’exécution total. Cet en-tête est utile pour déterminer si la latence provient du client ou du serveur. Vous pouvez comparer le temps d’exécution du parcours sur le client avec la valeur de cet en-tête. |
| **x-ms-status-code** | long | 200 | Success and Failure (« Succès et échec ») | Cet en-tête indique le motif interne d’exécution ou d’achèvement d’une requête. L’application est invitée à examiner la valeur de cet en-tête et à appliquer une action corrective. |
| **x-ms-substatus-code** | long | 1003 | Échec uniquement | Cosmos DB est une base de données multimodèle basée sur une couche de stockage unifiée. Cet en-tête contient des insights supplémentaires concernant le motif de l’échec quand l’échec se produit dans les couches inférieures de la pile de haute disponibilité. L’application est invitée à stocker cet en-tête et à l’utiliser lors d’une communication avec le service clientèle Cosmos DB. La valeur de cet en-tête permet aux ingénieurs Cosmos DB de résoudre les problèmes plus rapidement. |
| **x-ms-retry-after-ms** | Chaîne (TimeSpan) | "00:00:03.9500000" | Échec uniquement | Cet en-tête est une représentation d’un type .NET [TimeSpan](/dotnet/api/system.timespan) sous forme de chaîne. Cette valeur est incluse uniquement dans les requêtes ayant échoué en raison de la saturation du débit provisionné. L’application doit soumettre de nouveau le parcours après le délai indiqué. |
| **x-ms-activity-id** | Chaîne (Guid) | "A9218E01-3A3A-4716-9636-5BD86B056613" | Success and Failure (« Succès et échec ») | Cet en-tête contient l’identificateur côté serveur unique d’une requête. Le serveur attribue un identificateur unique à chaque requête à des fins de suivi. Les applications doivent consigner les identificateurs d’activité retournés par le serveur en vue des situations où les clients souhaitent contacter le service clientèle au sujet d’une requête spécifique. Le personnel du service clientèle de Cosmos DB peut rechercher des requêtes spécifiques à l’aide de ces identificateurs dans les données de télémétrie du service Cosmos DB. |

## <a name="status-codes"></a>Codes d’état

Les codes d’état les plus courants retournés par le serveur sont listés ci-dessous.

| Statut | Explication |
| --- | --- |
| **401** | Le message d’erreur `"Unauthorized: Invalid credentials provided"` est retourné quand le mot de passe d’authentification ne correspond pas à la clé de compte Cosmos DB. Accédez à votre compte Gremlin Cosmos DB dans le portail Azure et vérifiez que la clé est correcte.|
| **404** | Des opérations simultanées tentent de supprimer et de mettre à jour la même arête ou le même sommet simultanément. Le message d’erreur `"Owner resource does not exist"` indique que la base de données ou collection spécifiée est incorrecte dans les paramètres de connexion au format `/dbs/<database name>/colls/<collection or graph name>`.|
| **408** | `"Server timeout"` indique que le parcours a pris plus de **30 secondes** et qu’il a été annulé par le serveur. Optimisez vos parcours pour qu’ils s’exécutent rapidement en filtrant les arêtes ou les sommets sur chaque tronçon afin d’affiner l’étendue de recherche.|
| **409** | `"Conflicting request to resource has been attempted. Retry to avoid conflicts."` Cela se produit généralement lorsque le sommet ou une arête avec identificateur existe déjà dans le graphique.| 
| **412** | Le code d’état s’accompagne du message d’erreur `"PreconditionFailedException": One of the specified pre-condition is not met`. Il indique une violation du contrôle d’accès concurrentiel optimiste entre la lecture d’une arête ou d’un sommet et sa réécriture dans le magasin après modification. Dans la plupart des cas, cette erreur se produit lors d’une modification de propriété, par exemple `g.V('identifier').property('name','value')`. Le moteur Gremlin lit le sommet, le modifie, puis procède à sa réécriture. Si un autre parcours exécuté en parallèle tente d’écrire le même sommet ou une arête, l’un d’eux recevra cette erreur. L’application doit soumettre de nouveau le parcours au serveur.| 
| **429** | La demande a été limitée et doit être retentée après la valeur définie dans **x-ms-retry-after-ms**.| 
| **500** | Le message d'erreur qui contient `"NotFoundException: Entity with the specified id does not exist in the system."` indique qu'une base de données et/ou une collection ont été recréées avec le même nom. Cette erreur disparaîtra dans les 5 minutes à mesure que la modification se propagera et invalidera les caches de différents composants Cosmos DB. Pour éviter ce problème, utilisez des noms de base de données et de collection uniques à chaque fois.| 
| **1 000** | Ce code d’état est retourné quand le serveur a correctement analysé un message, mais que l’exécution a échoué. Il indique généralement un problème avec la requête.| 
| **1001** | Ce code est retourné quand le serveur termine l’exécution du parcours, mais ne parvient pas à sérialiser la réponse en retour pour le client. Cette erreur peut se produire quand le parcours génère un résultat complexe, trop grand ou non conforme à la spécification du protocole TinkerPop. Quand elle rencontre cette erreur, l’application doit simplifier le parcours. | 
| **1003** | `"Query exceeded memory limit. Bytes Consumed: XXX, Max: YYY"` est retourné quand le parcours dépasse la limite de mémoire autorisée. La limite de mémoire est de **2 Go** par parcours.| 
| **1004** | Ce code d’état indique une requête de graphe incorrectement formée. Une requête peut être incorrectement formée quand la désérialisation échoue, quand un type qui n’est pas une valeur est désérialisé en tant que type valeur ou quand une opération Gremlin non prise en charge est demandée. L’application ne doit pas renouveler la requête, car elle échouera. | 
| **1007** | Ce code d’état est généralement retourné avec le message d’erreur `"Could not process request. Underlying connection has been closed."`. Cette situation peut se produire si le pilote client tente d’utiliser une connexion fermée par le serveur. L’application doit réessayer le parcours sur une autre connexion.
| **1008** | Le serveur Gremlin de Cosmos DB peut mettre fin aux connexions pour rééquilibrer le trafic dans le cluster. Les pilotes clients doivent gérer cette situation et utiliser des connexions actives uniquement pour envoyer des requêtes au serveur. Parfois, les pilotes clients ne détectent pas que la connexion était fermée. Quand l’application rencontre l’erreur `"Connection is too busy. Please retry after sometime or open more connections."`, elle doit réessayer le parcours sur une autre connexion.

## <a name="samples"></a>Exemples

Exemple d’application cliente basée sur Gremlin.Net qui lit un attribut d’état :

```csharp
// Following example reads a status code and total request charge from server response attributes.
// Variable "server" is assumed to be assigned to an instance of a GremlinServer that is connected to Cosmos DB account.
using (GremlinClient client = new GremlinClient(server, new GraphSON2Reader(), new GraphSON2Writer(), GremlinClient.GraphSON2MimeType))
{
  ResultSet<dynamic> responseResultSet = await GremlinClientExtensions.SubmitAsync<dynamic>(client, requestScript: "g.V().count()");
  long statusCode = (long)responseResultSet.StatusAttributes["x-ms-status-code"];
  double totalRequestCharge = (double)responseResultSet.StatusAttributes["x-ms-total-request-charge"];

  // Status code and request charge are logged into application telemetry.
}
```

Exemple montrant comment lire l’attribut d’état à partir d’un client Java Gremlin :

```java
try {
  ResultSet resultSet = this.client.submit("g.addV().property('id', '13')");
  List<Result> results = resultSet.all().get();

  // Process and consume results

} catch (ResponseException re) {
  // Check for known errors that need to be retried or skipped
  if (re.getStatusAttributes().isPresent()) {
    Map<String, Object> attributes = re.getStatusAttributes().get();
    int statusCode = (int) attributes.getOrDefault("x-ms-status-code", -1);

    // Now we can check for specific conditions
    if (statusCode == 409) {
        // Handle conflicting writes
      }
    }

    // Check if we need to delay retry
    if (attributes.containsKey("x-ms-retry-after-ms")) {
      // Read the value of the attribute as is
      String retryAfterTimeSpan = (String) attributes.get("x-ms-retry-after-ms"));

      // Convert the value into actionable duration
            LocalTime locaTime = LocalTime.parse(retryAfterTimeSpan);
            Duration duration = Duration.between(LocalTime.MIN, locaTime);

      // Perform a retry after "duration" interval of time has elapsed
    }
  }
}

```

## <a name="next-steps"></a>Étapes suivantes
* [Codes d’état HTTP pour Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) 
* [En-têtes de réponse REST courants d’Azure Cosmos DB](/rest/api/cosmos-db/common-cosmosdb-rest-response-headers)
* [TinkerPop Graph Driver Provider Requirements]( http://tinkerpop.apache.org/docs/current/dev/provider/#_graph_driver_provider_requirements) (Exigences liées aux fournisseurs de pilote de graphe TinkerPop)
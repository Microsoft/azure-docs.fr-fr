---
title: Instructions pour les requêtes limitées
description: Apprenez à regrouper, échelonner, paginer et interroger en parallèle pour éviter que les demandes soient limitées par Azure Resource Graph.
ms.date: 10/14/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 4a8ba991d13b9be221e67f2ff1e393fb01f8a2d4
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92056172"
---
# <a name="guidance-for-throttled-requests-in-azure-resource-graph"></a>Conseils pour les requêtes limitées dans Azure Resource Graph

Lors de la programmation et de l’utilisation fréquente des données Azure Resource Graph, vous devez prendre en compte la façon dont la limitation affecte les résultats des requêtes. Changer la manière dont les données sont demandées peut vous aider, vous et votre organisation, à éviter toute limitation et à maintenir le flux de données en temps voulu concernant vos ressources Azure.

Cet article traite de quatre domaines et modèles en rapport avec la création de requêtes dans Azure Resource Graph :

- Présentation des en-têtes de limitation
- Regroupement des requêtes
- Échelonnement des requêtes
- Impact de la pagination

## <a name="understand-throttling-headers"></a>Présentation des en-têtes de limitation

Azure Resource Graph alloue un quota à chaque utilisateur en fonction d’une fenêtre de temps. Par exemple, un utilisateur peut envoyer au maximum 15 requêtes durant chaque fenêtre de cinq secondes sans être limité. La valeur de quota est déterminée par de nombreux facteurs et est susceptible de changer.

Dans chaque réponse de requête, Azure Resource Graph ajoute deux en-têtes de limitation :

- `x-ms-user-quota-remaining` (int) : quota de ressources restant pour l'utilisateur. Cette valeur correspond au nombre de requêtes.
- `x-ms-user-quota-resets-after` (hh:mm:ss) : délai à attendre avant la réinitialisation de la consommation du quota d’un utilisateur.

Quand un principal de sécurité a accès à plus de 5 000 abonnements au sein du client ou du groupe d’administration [étendue de la requête](./query-language.md#query-scope), la réponse est limitée aux 5 000 premiers abonnements et l’en-tête `x-ms-tenant-subscription-limit-hit` est retourné en tant que `true`.

Pour illustrer le fonctionnement des en-têtes, jetons un œil à une réponse de requête qui a les en-têtes et valeurs `x-ms-user-quota-remaining: 10` et `x-ms-user-quota-resets-after: 00:00:03`.

- Au cours des trois secondes suivantes, 10 requêtes au plus peuvent être envoyées sans limitation.
- Dans trois secondes, les valeurs de `x-ms-user-quota-remaining` et `x-ms-user-quota-resets-after` seront réinitialisées respectivement à `15` et `00:00:05`.

Pour voir un exemple d’utilisation des en-têtes pour effectuer une _interruption_ lors des demandes de requête, consultez l’exemple fourni dans [Interroger en parallèle](#query-in-parallel).

## <a name="grouping-queries"></a>Regroupement des requêtes

Regrouper les requêtes d’après l’abonnement, le groupe de ressources ou la ressource est plus efficace que paralléliser les requêtes. Le coût du quota d’une requête plus grande est souvent inférieur à celui de nombreuses requêtes petites et ciblées. Nous vous recommandons de choisir une taille de groupe inférieure à _300_.

- Exemple d’approche mal optimisée

  ```csharp
  // NOT RECOMMENDED
  var header = /* your request header */
  var subscriptionIds = /* A big list of subscriptionIds */

  foreach (var subscriptionId in subscriptionIds)
  {
      var userQueryRequest = new QueryRequest(
          subscriptions: new[] { subscriptionId },
          query: "Resoures | project name, type");

      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
          .ConfigureAwait(false);

  // ...
  }
  ```

- Exemple n°1 d’approche de regroupement optimisée

  ```csharp
  // RECOMMENDED
  var header = /* your request header */
  var subscriptionIds = /* A big list of subscriptionIds */

  const int groupSize = 100;
  for (var i = 0; i <= subscriptionIds.Count / groupSize; ++i)
  {
      var currSubscriptionGroup = subscriptionIds.Skip(i * groupSize).Take(groupSize).ToList();
      var userQueryRequest = new QueryRequest(
          subscriptions: currSubscriptionGroup,
          query: "Resources | project name, type");

      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
          .ConfigureAwait(false);

    // ...
  }
  ```

- Exemple n°2 d'approche de regroupement optimisée afin d'obtenir plusieurs ressources dans une même requête

  ```kusto
  Resources | where id in~ ({resourceIdGroup}) | project name, type
  ```

  ```csharp
  // RECOMMENDED
  var header = /* your request header */
  var resourceIds = /* A big list of resourceIds */

  const int groupSize = 100;
  for (var i = 0; i <= resourceIds.Count / groupSize; ++i)
  {
      var resourceIdGroup = string.Join(",",
          resourceIds.Skip(i * groupSize).Take(groupSize).Select(id => string.Format("'{0}'", id)));
      var userQueryRequest = new QueryRequest(
          subscriptions: subscriptionList,
          query: $"Resources | where id in~ ({resourceIdGroup}) | project name, type");

      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
          .ConfigureAwait(false);

    // ...
  }
  ```

## <a name="staggering-queries"></a>Échelonnement des requêtes

En raison de la manière dont la limitation est appliquée, nous vous recommandons d’échelonner les requêtes. Autrement dit, au lieu d’envoyer 60 requêtes en même temps, échelonnez-les dans quatre fenêtres de cinq secondes :

- Planification de requêtes sans échelonnement

  | Nombre de requêtes         | 60  | 0    | 0     | 0     |
  |---------------------|-----|------|-------|-------|
  | Intervalle de temps (sec) | 0-5 | 5-10 | 10-15 | 15-20 |

- Planification de requêtes avec échelonnement

  | Nombre de requêtes         | 15  | 15   | 15    | 15    |
  |---------------------|-----|------|-------|-------|
  | Intervalle de temps (sec) | 0-5 | 5-10 | 10-15 | 15-20 |

Voici un exemple montrant comment respecter les en-têtes de limitation lors de l’interrogation d’Azure Resource Graph :

```csharp
while (/* Need to query more? */)
{
    var userQueryRequest = /* ... */
    // Send post request to Azure Resource Graph
    var azureOperationResponse = await this.resourceGraphClient
        .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
        .ConfigureAwait(false);

    var responseHeaders = azureOperationResponse.response.Headers;
    int remainingQuota = /* read and parse x-ms-user-quota-remaining from responseHeaders */
    TimeSpan resetAfter = /* read and parse x-ms-user-quota-resets-after from responseHeaders */
    if (remainingQuota == 0)
    {
        // Need to wait until new quota is allocated
        await Task.Delay(resetAfter).ConfigureAwait(false);
    }
}
```

### <a name="query-in-parallel"></a>Interroger en parallèle

Bien que le regroupement soit préférable à la parallélisation, il arrive parfois que les requêtes soient difficiles à regrouper. Dans ces cas-là, vous souhaiterez peut-être interroger Azure Resource Graph en envoyant plusieurs requêtes de manière parallèle. Voici un exemple montrant comment effectuer une _interruption_ en fonction des en-têtes de limitation dans ce genre de scénario :

```csharp
IEnumerable<IEnumerable<string>> queryGroup = /* Groups of queries  */
// Run groups in parallel.
await Task.WhenAll(queryGroup.Select(ExecuteQueries)).ConfigureAwait(false);

async Task ExecuteQueries(IEnumerable<string> queries)
{
    foreach (var query in queries)
    {
        var userQueryRequest = new QueryRequest(
            subscriptions: subscriptionList,
            query: query);
        // Send post request to Azure Resource Graph.
        var azureOperationResponse = await this.resourceGraphClient
            .ResourcesWithHttpMessagesAsync(userQueryRequest, header)
            .ConfigureAwait(false);
        
        var responseHeaders = azureOperationResponse.response.Headers;
        int remainingQuota = /* read and parse x-ms-user-quota-remaining from responseHeaders */
        TimeSpan resetAfter = /* read and parse x-ms-user-quota-resets-after from responseHeaders */
        if (remainingQuota == 0)
        {
            // Delay by a random period to avoid bursting when the quota is reset.
            var delay = (new Random()).Next(1, 5) * resetAfter;
            await Task.Delay(delay).ConfigureAwait(false);
        }
    }
}
```

## <a name="pagination"></a>Pagination

Étant donné qu’Azure Resource Graph retourne au maximum 1000 entrées dans une même réponse de requête, vous devrez peut-être [paginer](./work-with-data.md#paging-results) vos requêtes afin d’obtenir le jeu de données complet que vous recherchez. Toutefois, certains clients Azure Resource Graph gèrent la pagination différemment des autres.

- Kit de développement logiciel (SDK) C#

  Quand vous utilisez le SDK ResourceGraph, vous devez gérer la pagination en passant le jeton d’évitement ($skiptoken) retourné à partir de la réponse de requête précédente à la requête paginée suivante. Cette conception signifie que vous devez recueillir les résultats de tous les appels paginés et les combiner à la fin. Dans ce cas, chaque requête paginé que vous envoyez prend un quota de requête :

  ```csharp
  var results = new List<object>();
  var queryRequest = new QueryRequest(
      subscriptions: new[] { mySubscriptionId },
      query: "Resources | project id, name, type | top 5000");
  var azureOperationResponse = await this.resourceGraphClient
      .ResourcesWithHttpMessagesAsync(queryRequest, header)
      .ConfigureAwait(false);
  while (!string.Empty(azureOperationResponse.Body.SkipToken))
  {
      queryRequest.SkipToken = azureOperationResponse.Body.SkipToken;
      // Each post call to ResourceGraph consumes one query quota
      var azureOperationResponse = await this.resourceGraphClient
          .ResourcesWithHttpMessagesAsync(queryRequest, header)
          .ConfigureAwait(false);
      results.Add(azureOperationResponse.Body.Data.Rows);

      // Inspect throttling headers in query response and delay the next call if needed.
  }
  ```

- Azure CLI / Azure PowerShell

  Quand vous utilisez Azure CLI ou Azure PowerShell, les requêtes envoyées à Azure Resource Graph sont automatiquement paginées afin d’extraire au maximum 5000 entrées. Les résultats de requête retournent une liste combinée d’entrées à partir de tous les appels paginés. Dans ce cas, en fonction du nombre d’entrées dans le résultat de requête, une requête paginée peut consommer plus d’un quota de requête. Dans les exemples suivants, une seule exécution de la requête peut consommer jusqu’à cinq quotas de requête :

  ```azurecli-interactive
  az graph query -q 'Resources | project id, name, type' --first 5000
  ```

  ```azurepowershell-interactive
  Search-AzGraph -Query 'Resources | project id, name, type' -First 5000
  ```

## <a name="still-get-throttled"></a>Vous êtes toujours limité ?

Si vous êtes encore limité après avoir appliqué les recommandations ci-dessus, contactez l’équipe à l’adresse [resourcegraphsupport@microsoft.com](mailto:resourcegraphsupport@microsoft.com).

Fournissez les détails suivants :

- Votre cas d’usage spécifique et les axes stratégiques qui imposent de disposer d’une limite plus élevée.
- À combien de ressources avez-vous accès ? Combien d’entre elles sont retournées par une requête unique ?
- Quels sont les types de ressources qui vous intéressent ?
- Quel est votre modèle de requête ? X requêtes toutes les Y secondes, etc.

## <a name="next-steps"></a>Étapes suivantes

- Examinez le langage utilisé dans les [requêtes de démarrage](../samples/starter.md).
- Examinez les utilisations avancées dans les [Requêtes avancées](../samples/advanced.md).
- Découvrez plus en détails comment [explorer des ressources](explore-resources.md).
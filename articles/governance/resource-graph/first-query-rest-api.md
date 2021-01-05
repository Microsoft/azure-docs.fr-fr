---
title: 'Démarrage rapide : Votre première requête d’API REST'
description: Dans ce guide de démarrage rapide, vous suivez les étapes permettant d’appeler le point de terminaison Resource Graph pour l’API REST et exécutez votre première requête.
ms.date: 10/14/2020
ms.topic: quickstart
ms.openlocfilehash: eafbd0b408ab50f901d52e59efcfbd5f89b46fe5
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566076"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-rest-api"></a>Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de l’API REST

La première étape de l’utilisation d’Azure Resource Graph avec l’API REST consiste à vérifier que vous disposez d’un outil permettant d’appeler les API REST disponibles. Ce guide de démarrage rapide vous accompagne ensuite tout au long du processus d’exécution d’une requête et de récupération des résultats en appelant le point de terminaison d’API REST Azure Resource Graph.

À la fin de ce processus, vous disposerez des outils permettant d’appeler les points de terminaison d’API REST et d’exécuter votre première requête Resource Graph.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="getting-started-with-rest-api"></a>Bien démarrer avec l’API REST

Si vous ne connaissez pas bien l’API REST, commencez par passer en revue [Référence de l’API REST Azure](/rest/api/azure/) pour obtenir des informations générales sur l’API REST, en particulier sur l’URI de requête et le corps de la requête. Cet article utilise ces concepts pour fournir des instructions sur l’utilisation d’Azure Resource Graph et suppose une connaissance pratique de ce dernier. Des outils comme [ARMClient](https://github.com/projectkudu/ARMClient) qui peuvent gérer l’autorisation automatiquement sont recommandés pour les débutants.

Pour connaître les spécifications d’Azure Resource Graph, consultez [API REST Azure Resource Graph](/rest/api/azure-resourcegraph/).

### <a name="rest-api-and-powershell"></a>API REST et PowerShell

Si vous n’avez pas encore d’outil pour effectuer des appels d’API REST, songez à utiliser PowerShell pour ces instructions. L’exemple de code suivant obtient un en-tête pour l’authentification auprès d’Azure. Générez un en-tête d’authentification, parfois appelé **jeton du porteur**, puis indiquez l’URI d’API REST auquel se connecter avec un paramètre ou un **corps de demande** :

```azurepowershell-interactive
# Log in first with Connect-AzAccount if not using Cloud Shell

$azContext = Get-AzContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Invoke the REST API
$restUri = 'https://management.azure.com/subscriptions/{subscriptionId}?api-version=2020-01-01'
$response = Invoke-RestMethod -Uri $restUri -Method Get -Headers $authHeader
```

Remplacez `{subscriptionId}` dans la variable `$restUri` pour obtenir des informations sur votre abonnement.
La variable `$response` contient le résultat de l’applet de commande `Invoke-RestMethod` que vous pouvez analyser avec des applets de commande telles que [ConvertFrom-Json](/powershell/module/microsoft.powershell.utility/convertfrom-json). Si le point de terminaison de service de l’API REST attend un **corps de demande**, fournissez une variable au format JSON au paramètre `-Body` de `Invoke-RestMethod`.

## <a name="run-your-first-resource-graph-query"></a>Exécuter votre première requête Resource Graph

Une fois les outils d’API REST ajoutés à l’environnement de votre choix, vous pouvez exécuter une requête Resource Graph simple. La requête retourne les cinq premières ressources Azure avec le nom (**Name**) et le **Type** de chaque ressource.

Le corps de la demande de chaque appel d’API REST contient une variable que vous devez remplacer par votre propre valeur :

- Remplacer `{subscriptionID}` par votre ID d’abonnement

1. Exécutez votre première requête Azure Resource Graph à l’aide de l’API REST et du point de terminaison `resources` :

   - URI de l’API REST

     ```http
     POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2019-04-01
     ```

   - Corps de la requête

     ```json
     {
         "subscriptions": [
             "{subscriptionID}"
         ],
         "query": "Resources | project name, type | limit 5"
     }
     ```

   > [!NOTE]
   > Comme cet exemple de requête ne fournit pas de modificateur de tri tel que `order by`, l’exécution répétée de cette requête peut produire un ensemble distinct de ressources par requête.

1. Mettez à jour l’appel au point de terminaison `resouces` et changez la **requête** afin qu’elle effectue un tri (`order by`) en fonction de la propriété **name** :

   - URI de l’API REST

     ```http
     POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2019-04-01
     ```

   - Corps de la requête

     ```json
     {
         "subscriptions": [
             "{subscriptionID}"
         ],
         "query": "Resources | project name, type | limit 5 | order by name asc"
     }
     ```

   > [!NOTE]
   > Comme précédemment, l’exécution répétée de cette requête peut produire un ensemble différent de ressources. L’ordre des commandes de requête est important. Dans cet exemple, `order by` vient après `limit`. Cet ordre de commande limite d’abord les résultats de la requête, puis les classe.

1. Mettez à jour l’appel au point de terminaison `resources` et changez la **requête** pour qu’elle effectue un tri (`order by`) en fonction de la propriété **name**, puis qu’elle limite (`limit`) les données récupérées aux cinq premiers résultats :

   - URI de l’API REST

     ```http
     POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2019-04-01
     ```

   - Corps de la requête

     ```json
     {
         "subscriptions": [
             "{subscriptionID}"
         ],
         "query": "Resources | project name, type | order by name asc | limit 5"
     }
     ```

Si votre environnement ne change pas et si vous exécutez plusieurs fois la requête finale, les résultats retournés sont cohérents et classés en fonction de la propriété **Name**, mais toujours limités aux cinq premiers.

Pour obtenir plus d’exemples d’appels d’API REST pour Azure Resource Graph, consultez les [exemples REST Azure Resource Graph](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources#examples).

## <a name="clean-up-resources"></a>Nettoyer les ressources

L’API REST n’a pas de bibliothèques ou de modules à désinstaller. Si vous avez installé un outil tel qu’_ARMClient_ ou _Postman_ pour effectuer les appels et que vous n’en avez plus besoin, vous pouvez le désinstaller maintenant.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appelé le point de terminaison de l’API REST Resource Graph et exécuté votre première requête. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](./concepts/query-language.md)

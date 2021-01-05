---
title: Résolution des erreurs courantes
description: Découvrez comment résoudre les problèmes liés aux différents Kits de développement logiciel (SDK) lors de l’interrogation des ressources Azure avec Azure Resource Graph.
ms.date: 10/14/2020
ms.topic: troubleshooting
ms.openlocfilehash: 13c5d5ffde8b0b82fcafa5e8149400555a0b18a6
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92056955"
---
# <a name="troubleshoot-errors-using-azure-resource-graph"></a>Résoudre les erreurs à l’aide d’Azure Resource Graph

Vous pouvez rencontrer des erreurs lors de l’interrogation des ressources Azure avec Azure Resource Graph. Cet article décrit différentes erreurs qui peuvent se produire et explique comment les résoudre.

## <a name="finding-error-details"></a>Recherche des détails de l’erreur

La plupart des erreurs sont le résultat d’un problème lors de l’exécution d’une requête avec Azure Resource Graph. En cas d’échec d’une requête, le kit de développement logiciel (SDK) fournit des détails sur la requête qui a échoué. Ces informations précisent le problème afin que vous puissiez le résoudre et que la requête suivante réussisse.

## <a name="general-errors"></a>Erreurs générales.

### <a name="scenario-throttled-requests"></a><a name="throttled"></a>Scénario : Requêtes limitées

#### <a name="issue"></a>Problème

Lorsque les clients envoient des requêtes de ressources fréquentes ou volumineuses, ces dernières sont limitées.

#### <a name="cause"></a>Cause

Azure Resource Graph alloue un quota à chaque utilisateur en fonction d’une fenêtre de temps. Par exemple, un utilisateur peut envoyer au maximum 15 requêtes durant chaque fenêtre de cinq secondes sans être limité. La valeur de quota est déterminée par de nombreux facteurs et est susceptible de changer. Pour plus d’informations, voir [Throttle in Azure Resource Graph](../overview.md#throttling) (Limitation dans Azure Resource Graph).

#### <a name="resolution"></a>Résolution

Il existe plusieurs méthodes pour traiter les requêtes limitées :

- [Regroupement des requêtes](../concepts/guidance-for-throttled-requests.md#grouping-queries)
- [Échelonnement des requêtes](../concepts/guidance-for-throttled-requests.md#staggering-queries)
- [Interrogation en parallèle](../concepts/guidance-for-throttled-requests.md#query-in-parallel)
- [Pagination](../concepts/guidance-for-throttled-requests.md#pagination)

### <a name="scenario-too-many-subscriptions"></a><a name="toomanysubscription"></a>Scénario : Trop d’abonnements

#### <a name="issue"></a>Problème

Les clients ayant accès à plus de 1000 abonnements, y compris les abonnements inter-clients avec [Azure Lighthouse](../../../lighthouse/overview.md), ne peuvent pas extraire les données de tous les abonnements dans un seul appel à Azure Resource Graph.

#### <a name="cause"></a>Cause

Azure CLI et PowerShell transfèrent uniquement les 1000 premiers abonnements à Azure Resource Graph. L’API REST pour Azure Resource Graph accepte un nombre maximal d’abonnements pour exécuter la requête.

#### <a name="resolution"></a>Résolution

Requêtes par lots pour la requête avec un sous-ensemble d’abonnements à conserver sous la limite d’abonnement de 1000. La solution utilise le paramètre d’**Abonnement** dans PowerShell.

```azurepowershell-interactive
# Replace this query with your own
$query = 'Resources | project type'

# Fetch the full array of subscription IDs
$subscriptions = Get-AzSubscription
$subscriptionIds = $subscriptions.Id

# Create a counter, set the batch size, and prepare a variable for the results
$counter = [PSCustomObject] @{ Value = 0 }
$batchSize = 1000
$response = @()

# Group the subscriptions into batches
$subscriptionsBatch = $subscriptionIds | Group -Property { [math]::Floor($counter.Value++ / $batchSize) }

# Run the query for each batch
foreach ($batch in $subscriptionsBatch){ $response += Search-AzGraph -Query $query -Subscription $batch.Group }

# View the completed results of the query on all subscriptions
$response
```

### <a name="scenario-unsupported-content-type-rest-header"></a><a name="rest-contenttype"></a>Scénario : En-tête REST Content-Type non pris en charge

#### <a name="issue"></a>Problème

Les clients qui interrogent l’API REST Azure Resource Graph reçoivent une réponse _500_ (erreur de serveur interne).

#### <a name="cause"></a>Cause

L’API REST Azure Resource Graph ne prend en charge qu’un `Content-Type` **application/json**. Certains outils ou agents REST sont par défaut de type **text/plain** que l’API REST ne prend pas en charge.

#### <a name="resolution"></a>Résolution

Vérifiez que l’outil ou l’agent que vous utilisez pour interroger Azure Resource Graph a l’en-tête API REST `Content-Type` configuré pour **application/json**.

### <a name="scenario-no-read-permission-to-all-subscriptions-in-list"></a><a name="rest-403"></a>Scénario : Aucune autorisation de lecture sur tous les abonnements répertoriés

#### <a name="issue"></a>Problème

Les clients qui transmettent explicitement une liste d’abonnements à l’aide d’une requête Azure Resource Graph obtiennent une réponse _403_ (Interdit).

#### <a name="cause"></a>Cause

Si le client ne dispose pas d’autorisations de lecture sur tous les abonnements fournis, la requête est refusée en raison de l’absence de droits de sécurité appropriés.

#### <a name="resolution"></a>Résolution

Incluez dans la liste d’abonnements au moins un abonnement auquel le client qui exécute la requête a accès en lecture. Pour plus d’informations, voir [Autorisations dans Azure Resource Graph](../overview.md#permissions-in-azure-resource-graph).

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

- Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
- Connectez-vous avec [@AzureSupport](https://twitter.com/azuresupport), qui est le compte Microsoft Azure officiel pour améliorer l’expérience client en connectant la communauté Azure aux ressources appropriées : réponses, support technique et experts.
- Si vous avez besoin de plus d’aide, vous pouvez signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.
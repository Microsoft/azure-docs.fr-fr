---
title: Estimation des coûts d’un plan Consommation dans Azure Functions
description: Apprenez à mieux estimer les coûts induits par l’exécution de votre application de fonction dans un plan Consommation dans Azure.
ms.date: 9/20/2019
ms.topic: conceptual
ms.openlocfilehash: 58082e03c1416848e9aa1e97308bed1ceaa67295
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92168107"
---
# <a name="estimating-consumption-plan-costs"></a>Estimation des coûts d’un plan Consommation

Il existe actuellement trois types de plans d’hébergement pour une application qui s’exécute dans Azure Functions, chaque plan ayant son propre modèle de tarifs : 

| Plan | Description |
| ---- | ----------- |
| [**Consommation**](functions-scale.md#consumption-plan) | Vous êtes facturé uniquement pour la durée d’exécution de votre application de fonction. Ce plan comprend un [forfait gratuit][page des tarifs] par abonnement.|
| [**Premium**](functions-scale.md#premium-plan) | Fournit les mêmes fonctionnalités et le même mécanisme de mise à l’échelle que le plan Consommation, mais avec des performances améliorées et un accès au réseau virtuel. Le coût se base sur le niveau tarifaire choisi. Pour plus d’informations, consultez [Plan Premium Azure Functions](functions-premium-plan.md). |
| [**Dédié (App Service)**](functions-scale.md#app-service-plan) <br/>(niveau de base ou supérieur) | Si vous avez besoin d’exécuter sur des machines virtuelles dédiées ou en isolation, d’utiliser des images personnalisées ou si vous voulez utiliser votre capacité de plan App Service en excès. Utilise la [facturation du plan App Service standard](https://azure.microsoft.com/pricing/details/app-service/). Le coût se base sur le niveau tarifaire choisi.|

Vous avez choisi le plan qui répond le mieux à vos besoins de performance et à votre budget. Pour en savoir plus, voir [Mise à l’échelle et hébergement d’Azure Functions](functions-scale.md).

Cet article a trait uniquement au plan Consommation, car celui-ci entraîne des coûts variables. Cet article remplace l’article [Consumption plan cost billing FAQ](https://github.com/Azure/Azure-Functions/wiki/Consumption-Plan-Cost-Billing-FAQ).

Durable Functions peut également s’exécuter dans un plan Consommation. Pour en savoir plus sur les considérations financières lors de l’utilisation de Durable Functions, consultez [Facturation Durable Functions](./durable/durable-functions-billing.md).

## <a name="consumption-plan-costs"></a>Coûts d’un plan de consommation

Le *coût* de l’exécution d’une fonction unique se mesure en *Go-secondes*. Le coût d’exécution est calculé en combinant l’utilisation de sa mémoire et sa durée d’exécution. Une fonction qui s’exécute plus longtemps coûte plus cher, tout comme une fonction qui consomme plus de mémoire. 

Imaginez le cas où la quantité de mémoire utilisée par la fonction reste constante. Dans ce cas, le calcul du coût est une simple multiplication. Par exemple, supposons que votre fonction a consommé 0,5 Go pendant 3 secondes. Ainsi, le coût d’exécution s’élève à `0.5GB * 3s = 1.5 GB-seconds`. 

Étant donné que l’utilisation de la mémoire change au fil du temps, le calcul correspond au fond à l’intégrale de l’utilisation de la mémoire au fil du temps.  Le système effectue ce calcul en échantillonnant l’utilisation de la mémoire du processus (avec les processus enfants) à intervalles réguliers. Comme mentionné dans la [page des tarifs], l’utilisation de la mémoire est arrondie au compartiment de 128 Mo le plus proche. Quand votre processus utilise 160 Mo, vous êtes facturé pour 256 Mo. Le calcul prend en compte la concurrence, à savoir l’exécution de plusieurs fonctions simultanées dans le même processus.

> [!NOTE]
> Alors que l’utilisation du processeur n’est pas directement prise en compte dans le coût d’exécution, elle peut exercer un impact sur le coût quand elle affecte la durée d’exécution de la fonction.

Pour une fonction déclenchée par HTTP, quand une erreur se produit avant le début de l’exécution du code de votre fonction, vous n’êtes pas facturé pour une exécution. Cela signifie que les réponses 401 de la plateforme en raison de la validation de la clé API ou de la fonctionnalité d’authentification/d’autorisation d’App Service ne sont pas comptabilisées par rapport au coût d’exécution. De même, les réponses de code d’état 5xx ne sont pas comptabilisées lorsqu’elles se produisent dans la plateforme avant une fonction qui traite la requête. Une réponse 5xx générée par la plateforme après le début de l’exécution du code de votre fonction est toujours comptabilisée comme une exécution, même si l’erreur n’est pas déclenchée par le code de votre fonction.

## <a name="other-related-costs"></a>Autres coûts connexes

Quand vous estimez le coût global de l’exécution de vos fonctions dans un plan, quel qu’il soit, n’oubliez pas que le runtime Functions utilise plusieurs autres services Azure, facturés chacun séparément. Lors du calcul des tarifs des applications de fonction, tous les déclencheurs et toutes les liaisons que vous avez intégrés à d’autres services Azure vous obligent à créer et payer ces services supplémentaires. 

Pour les fonctions qui s’exécutent dans un plan Consommation, le coût total est le coût d’exécution de vos fonctions, auquel s’ajoute le coût de la bande passante et des services supplémentaires. 

Lorsque vous estimez les coûts globaux de votre application de fonction et des services associés, utilisez la [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/?service=functions). 

| Coût connexe | Description |
| ------------ | ----------- |
| **Compte de stockage** | Chaque application de fonction vous oblige à avoir un [compte de stockage Azure](../storage/common/storage-introduction.md#types-of-storage-accounts) universel associé, [facturé séparément](https://azure.microsoft.com/pricing/details/storage/). Ce compte est utilisé en interne par le runtime Functions, mais vous pouvez également l’utiliser pour les déclencheurs et les liaisons de stockage. Si vous n’avez pas de compte de stockage, un compte est créé pour vous lors de la création de l’application de fonction. Pour plus d’informations, consultez [Exigences pour le compte de stockage](storage-considerations.md#storage-account-requirements).|
| **Application Insights** | Functions s’appuie sur [Application Insights](../azure-monitor/app/app-insights-overview.md) pour fournir une expérience de supervision hautes performances à vos applications de fonction. Même si ce n’est pas obligatoire, il est préférable d’[activer l’intégration à Application Insights](configure-monitoring.md#enable-application-insights-integration). Un forfait gratuit de données de télémétrie est inclus chaque mois. Pour en savoir plus, consultez la [page des tarifs Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). |
| **Bande passante réseau** | Vous ne payez pas le transfert de données entre les services Azure dans la même région. En revanche, des coûts peuvent être induits pour les transferts de données sortants vers une autre région ou en dehors d’Azure. Pour plus d’informations, consultez les [détails de la tarification de la bande passante](https://azure.microsoft.com/pricing/details/bandwidth/). |

## <a name="behaviors-affecting-execution-time"></a>Comportements affectant la durée d’exécution

Les comportements suivants de vos fonctions peuvent exercer un impact sur la durée d’exécution :

+ **Déclencheurs et liaisons** : Le temps nécessaire pour lire l’entrée à partir de vos [liaisons de fonction](functions-triggers-bindings.md) et y écrire la sortie est compté dans la durée d’exécution. Par exemple, quand votre fonction utilise une liaison de sortie pour écrire un message dans une file d’attente de stockage Azure, votre durée d’exécution inclut le temps nécessaire à l’écriture du message dans la file d’attente, lequel est inclus dans le calcul du coût de la fonction. 

+ **Exécution asynchrone** : La durée pendant laquelle votre fonction attend les résultats d’une requête asynchrone (`await` en C#) est comptée dans la durée d’exécution. Le calcul des Go-secondes se base sur l’heure de début et de fin de la fonction, ainsi que sur l’utilisation de la mémoire pendant cette période. Ce qui se produit au cours de cette période en termes d’activité du processeur n’est pas pris en compte dans le calcul. Vous pouvez éventuellement réduire les coûts pendant les opérations asynchrones à l’aide de [Durable Functions](durable/durable-functions-overview.md). Vous n’êtes pas facturé pour le temps d’attente passé dans les fonctions d’orchestrateur.

## <a name="viewing-cost-related-data"></a>Affichage des données relatives aux coûts

Dans [votre facture](../cost-management-billing/understand/download-azure-invoice.md), vous pouvez voir les données relatives aux coûts **Nombre total d’exécutions - Fonctions** et **Durée d’exécution - Fonctions**, ainsi que les coûts réels facturés. En revanche, ces données de facture sont un agrégat mensuel correspondant à une période de facturation passée. 

### <a name="function-app-level-metrics"></a>Métriques au niveau de l’application de fonction

Pour mieux comprendre l’impact sur les coûts de vos fonctions, vous pouvez utiliser Azure Monitor pour voir les métriques relatives aux coûts en cours de génération par vos applications de fonction. Vous pouvez utiliser [Azure Monitor Metrics Explorer](../azure-monitor/platform/metrics-getting-started.md) dans le [Azure portal] ou les API REST pour obtenir ces données.

#### <a name="monitor-metrics-explorer"></a>Monitor Metrics Explorer

Utilisez [Azure Monitor Metrics Explorer](../azure-monitor/platform/metrics-getting-started.md) pour voir dans un format graphique les données relatives aux coûts de vos applications de fonction relevant du plan Consommation. 

1. En haut du [Azure portal], dans la zone de **recherche dans les services, ressources et documents**, recherchez `monitor` et sélectionnez **Monitor** sous **Services**.

1. Sur la gauche, sélectionnez **Metrics** > **Sélectionner une ressource**, puis utilisez les paramètres situés sous l’image pour choisir votre application de fonction.

    ![Sélectionner votre ressource d’application de fonction](media/functions-consumption-costing/select-a-resource.png)

      
    |Paramètre  |Valeur suggérée  |Description  |
    |---------|---------|---------|
    | Abonnement    |  Votre abonnement  | Abonnement contenant votre application de fonction.  |
    | Resource group     | Votre groupe de ressources  | Groupe de ressources qui contient votre application de fonction.   |
    | Type de ressource     |  App Services | Les applications de fonction s’affichent sous forme d’instances App Services dans Monitor. |
    | Ressource     |  Votre application de fonction  | Application de fonction à superviser.        |

1. Sélectionnez **Appliquer** pour choisir votre application de fonction comme ressource à superviser.

1. À partir de **Métrique**, choisissez **Nombre d’exécutions de la fonction** et **Somme** pour **Agrégation**. Ainsi, la somme du nombre d’exécutions pendant la période choisie est ajoutée au graphique.

    ![Définir une métrique d’application de fonction à ajouter au graphique](media/functions-consumption-costing/monitor-metrics-add-metric.png)

1. Sélectionnez **Ajouter une métrique** et répétez les étapes 2 à 4 pour ajouter **Unités d’exécution de la fonction** au graphique. 

Le graphique obtenu contient les totaux des deux métriques d’exécution dans l’intervalle de temps choisi, en l’occurrence deux heures.

![Graphe du nombre d’exécutions et des unités de la fonction](media/functions-consumption-costing/monitor-metrics-execution-sum.png)

Étant donné que le nombre d’unités d’exécution est bien supérieur au nombre d’exécutions, le graphique montre uniquement les unités d’exécution.

Ce graphique présente un total de 1,11 milliard `Function Execution Units` consommé sur une période de deux heures, exprimé en Mo-millisecondes. Pour convertir ce total en Go-secondes, divisez-le par 1 024 000. Dans cet exemple, l’application de fonction a consommé `1110000000 / 1024000 = 1083.98` Go-secondes. Vous pouvez utiliser cette valeur et la multiplier par le prix en vigueur de la durée d’exécution figurant dans la [page des tarifs Functions][page des tarifs], ce qui vous permet d’obtenir le coût de ces deux heures, en supposant que vous avez déjà utilisé tous les forfaits gratuits de durée d’exécution. 

#### <a name="azure-cli"></a>Azure CLI

[Azure CLI](/cli/azure/) contient des commandes pour récupérer des métriques. Vous pouvez utiliser CLI à partir d’un environnement de commande local ou directement à partir du portail en utilisant [Azure Cloud Shell](../cloud-shell/overview.md). Par exemple, la commande [az monitor metrics list](/cli/azure/monitor/metrics#az-monitor-metrics-list) suivante retourne les données horaires sur la même période que celle utilisée précédemment.

Veillez à remplacer `<AZURE_SUBSCRIPTON_ID>` par votre ID d’abonnement Azure en exécutant la commande.

```azurecli-interactive
az monitor metrics list --resource /subscriptions/<AZURE_SUBSCRIPTION_ID>/resourceGroups/metrics-testing-consumption/providers/Microsoft.Web/sites/metrics-testing-consumption --metric FunctionExecutionUnits,FunctionExecutionCount --aggregation Total --interval PT1H --start-time 2019-09-11T21:46:00Z --end-time 2019-09-11T23:18:00Z
```

Cette commande retourne une charge utile JSON semblable à l’exemple suivant :

```json
{
  "cost": 0.0,
  "interval": "1:00:00",
  "namespace": "Microsoft.Web/sites",
  "resourceregion": "centralus",
  "timespan": "2019-09-11T21:46:00Z/2019-09-11T23:18:00Z",
  "value": [
    {
      "id": "/subscriptions/XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXX/resourceGroups/metrics-testing-consumption/providers/Microsoft.Web/sites/metrics-testing-consumption/providers/Microsoft.Insights/metrics/FunctionExecutionUnits",
      "name": {
        "localizedValue": "Function Execution Units",
        "value": "FunctionExecutionUnits"
      },
      "resourceGroup": "metrics-testing-consumption",
      "timeseries": [
        {
          "data": [
            {
              "average": null,
              "count": null,
              "maximum": null,
              "minimum": null,
              "timeStamp": "2019-09-11T21:46:00+00:00",
              "total": 793294592.0
            },
            {
              "average": null,
              "count": null,
              "maximum": null,
              "minimum": null,
              "timeStamp": "2019-09-11T22:46:00+00:00",
              "total": 316576256.0
            }
          ],
          "metadatavalues": []
        }
      ],
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    },
    {
      "id": "/subscriptions/XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXX/resourceGroups/metrics-testing-consumption/providers/Microsoft.Web/sites/metrics-testing-consumption/providers/Microsoft.Insights/metrics/FunctionExecutionCount",
      "name": {
        "localizedValue": "Function Execution Count",
        "value": "FunctionExecutionCount"
      },
      "resourceGroup": "metrics-testing-consumption",
      "timeseries": [
        {
          "data": [
            {
              "average": null,
              "count": null,
              "maximum": null,
              "minimum": null,
              "timeStamp": "2019-09-11T21:46:00+00:00",
              "total": 33538.0
            },
            {
              "average": null,
              "count": null,
              "maximum": null,
              "minimum": null,
              "timeStamp": "2019-09-11T22:46:00+00:00",
              "total": 13040.0
            }
          ],
          "metadatavalues": []
        }
      ],
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    }
  ]
}
```
Cette réponse particulière indique qu’entre `2019-09-11T21:46` et `2019-09-11T23:18`, l’application a consommé 1110000000 Mo-millisecondes (1083,98 Go-secondes).

### <a name="function-level-metrics"></a>Métriques au niveau de la fonction

Les unités d’exécution de la fonction sont une combinaison de la durée d’exécution et de votre utilisation de la mémoire, ce qui en fait une mesure difficile pour comprendre l’utilisation de la mémoire. La métrique des données de mémoire n’est actuellement pas disponible par le biais d’Azure Monitor. En revanche, si vous voulez optimiser l’utilisation de la mémoire de votre application, vous pouvez utiliser les données du compteur de performances collectées par Application Insights.  

Si vous ne l’avez pas déjà fait, [activez Application Insights dans votre application de fonction](configure-monitoring.md#enable-application-insights-integration). Une fois cette intégration activée, vous pouvez [interroger ces données de télémétrie dans le portail](analyze-telemetry-data.md#query-telemetry-data). 

[!INCLUDE [functions-consumption-metrics-queries](../../includes/functions-consumption-metrics-queries.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur la supervision des applications de fonction](functions-monitoring.md)

[Page des tarifs]:https://azure.microsoft.com/pricing/details/functions/
[Azure portal]: https://portal.azure.com

---
title: Journaux d’exécution de flux dans Azure Functions
description: 115-145 caractères y compris les espaces. Ce résumé s’affiche dans les résultats de recherche.
ms.date: 9/1/2020
ms.topic: how-to
ms.custom: contperf-fy21q2, devx-track-azurecli
ms.openlocfilehash: 085849386fce929ceaec4536844cf31b94fe7539
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033049"
---
# <a name="enable-streaming-execution-logs-in-azure-functions"></a>Activer les journaux d’exécution de streaming dans Azure Functions

Quand vous développez une application, vous voulez souvent voir ce qui est écrit dans les journaux en quasi temps réel lors de l’exécution dans Azure.

Il existe deux façons d’afficher un flux des fichiers journaux générés par vos exécutions de fonction.

* **Streaming des journaux intégré** : la plateforme App Service vous permet d’afficher un flux de vos fichiers journaux d’application. Cela équivaut à la sortie observée quand vous déboguez vos fonctions au cours du [développement local](functions-develop-local.md) et quand vous utilisez l’onglet **Test** dans le portail. Toutes les informations basées sur les journaux sont affichées. Pour plus d’informations, consultez [Diffuser les journaux d’activité en continu](../app-service/troubleshoot-diagnostic-logs.md#stream-logs). Cette méthode de streaming ne prend en charge qu’une seule instance et ne peut pas être utilisée avec une application exécutée sur Linux dans un plan Consommation.

* **Flux de métriques temps réel** : quand votre application de fonction est [connectée à Application Insights](configure-monitoring.md#enable-application-insights-integration), vous pouvez voir les données des journaux et d’autres métriques en quasi temps réel dans le portail Azure avec [Flux de métriques temps réel](../azure-monitor/app/live-stream.md). Appliquez cette méthode lors de la supervision de fonctions s’exécutant sur plusieurs instances ou sur Linux dans un plan Consommation. Cette méthode utilise des [données échantillonnées](configure-monitoring.md#configure-sampling).

Les flux de journaux peuvent être affichés à la fois dans le portail et dans la plupart des environnements de développement locaux. 

## <a name="portal"></a>Portail

Vous pouvez afficher les deux types de flux de journaux dans le portail.

### <a name="built-in-log-streaming"></a>Streaming des journaux intégré

Pour afficher les journaux de diffusion en continu dans le portail, sélectionnez l'onglet **Fonctionnalités de la plateforme** dans votre application de fonction. Ensuite, sous **Surveillance**, sélectionnez **Diffusion en continu des journaux**.

![Activer les journaux de diffusion en continu dans le portail](./media/functions-monitoring/enable-streaming-logs-portal.png)

Cela permet de connecter votre application au service de diffusion en continu des journaux, et les journaux d'application s'affichent dans la fenêtre. Vous pouvez basculer entre **Journaux d’activité d’application** et **Journaux d’activité des serveurs Web**.  

![Afficher la diffusion de journaux d’activité en continu dans le portail](./media/functions-monitoring/streaming-logs-window.png)

### <a name="live-metrics-stream"></a>Live Metrics Stream (Flux continu de mesures)

Pour voir le flux de métriques temps réel de votre application, sélectionnez l’onglet **Vue d’ensemble** de votre application de fonction. Quand Application Insights est activé, un lien **Application Insights** est visible sous **Fonctionnalités configurées**. Ce lien vous mène à la page Application Insights de votre application.

Dans Application Insights, sélectionnez **Flux de métriques temps réel**. Les [entrées de journaux échantillonnées](configure-monitoring.md#configure-sampling) sont affichées sous **Exemple de télémétrie**.

![Voir les flux de métriques temps réel dans le portail](./media/functions-monitoring/live-metrics-stream.png) 

## <a name="visual-studio-code"></a>Visual Studio Code

[!INCLUDE [functions-enable-log-stream-vs-code](../../includes/functions-enable-log-stream-vs-code.md)]

## <a name="core-tools"></a>Core Tools

[!INCLUDE [functions-streaming-logs-core-tools](../../includes/functions-streaming-logs-core-tools.md)]

## <a name="azure-cli"></a>Azure CLI

Vous pouvez activer les journaux de diffusion en continu en utilisant [Azure CLI](/cli/azure/install-azure-cli). Utilisez les commandes suivantes pour vous connecter, choisir votre abonnement et diffuser en continu les fichiers journaux :

```azurecli
az login
az account list
az account set --subscription <subscriptionNameOrId>
az webapp log tail --resource-group <RESOURCE_GROUP_NAME> --name <FUNCTION_APP_NAME>
```

## <a name="azure-powershell"></a>Azure PowerShell

Vous pouvez activer les journaux de diffusion en continu en utilisant [Azure PowerShell](/powershell/azure/). Pour PowerShell, utilisez la commande [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) pour activer la journalisation sur l’application de fonction, comme illustré dans l’extrait de code suivant : 

:::code language="powershell" source="~/powershell_scripts/app-service/monitor-with-logs/monitor-with-logs.ps1" range="19-20":::

Pour plus d’informations, consultez [l’exemple de code complet](../app-service/scripts/powershell-monitor.md#sample-script). 

## <a name="next-steps"></a>Étapes suivantes

+ [Superviser Azure Functions](functions-monitoring.md)
+ [Analyser la télémétrie d’Azure Functions dans Application Insights](analyze-telemetry-data.md)

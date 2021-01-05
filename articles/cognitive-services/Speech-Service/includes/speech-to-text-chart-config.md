---
title: Installer des conteneurs Speech
titleSuffix: Azure Cognitive Services
description: Détaille les options de configuration de graphique Helm pour la reconnaissance vocale.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 05/05/2020
ms.author: trbye
ms.openlocfilehash: 85c4e0641e1989ddea6c8aa8b8a8895a966a5ddb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002269"
---
### <a name="speech-to-text-sub-chart-chartsspeechtotext"></a>Reconnaissance vocale (sous-graphique : charts/speechToText)

Pour remplacer le graphique en parapluie, ajoutez le préfixe `speechToText.` à n’importe quel paramètre pour le rendre plus spécifique. Il remplacera le paramètre correspondant. Par exemple, `speechToText.numberOfConcurrentRequest` remplace `numberOfConcurrentRequest`.

|Paramètre|Description|Default|
| -- | -- | -- |
| `enabled` | Indique si la **reconnaissance vocale** est activée ou non. | `false` |
| `numberOfConcurrentRequest` | Le nombre de requêtes simultanées pour le service de **reconnaissance vocale**. Ce graphique calcule automatiquement les ressources processeur et mémoire en fonction de cette valeur. | `2` |
| `optimizeForAudioFile`| Indique si le service doit optimiser pour l’entrée audio via des fichiers audio. Si `true`, ce graphique alloue plus de ressources processeur au service. | `false` |
| `image.registry`| Le registre d’image docker pour la **reconnaissance vocale**. | `containerpreview.azurecr.io` |
| `image.repository` | Le référentiel d’image docker pour la **reconnaissance vocale**. | `microsoft/cognitive-services-speech-to-text` |
| `image.tag` | La balise d’image docker pour la **reconnaissance vocale**. | `latest` |
| `image.pullSecrets` | Les secrets de l’image pour l’extraction de l’image docker de **reconnaissance vocale**. | |
| `image.pullByHash`| Indique si l’image docker est extraite par hachage. Si `true`, `image.hash` est requis. | `false` |
| `image.hash`| Le hachage d’image docker pour la **reconnaissance vocale**. Utilisé uniquement si `image.pullByHash: true`.  | |
| `image.args.eula` (requis) | Indique que vous avez accepté la licence. La seule valeur valide est `accept` | |
| `image.args.billing` (requis) | La valeur de l’URI de point de terminaison de facturation est disponible dans la page Vue d’ensemble de Speech du portail Azure. | |
| `image.args.apikey` (requis) | Utilisé pour le suivi des informations de facturation. ||
| `service.type` | Le type de service Kubernetes du service de **reconnaissance vocale**. Consultez les [Instructions relatives aux types de service Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/) pour plus d’informations et pour vérifier la prise en charge des fournisseurs cloud. | `LoadBalancer` |
| `service.port`|  Le port du service de **reconnaissance vocale**. | `80` |
| `service.annotations` | Les annotations de **reconnaissance vocale** pour les métadonnées du service. Les annotations sont des paires clé-valeur. <br>`annotations:`<br>&nbsp;&nbsp;`some/annotation1: value1`<br>&nbsp;&nbsp;`some/annotation2: value2` | |
| `service.autoScaler.enabled` | Indique si [l’Autoscaler de pods élastique](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) est activé. Si `true`, le `speech-to-text-autoscaler` sera déployé dans le cluster Kubernetes. | `true` |
| `service.podDisruption.enabled` | Indique si le [Budget de perturbation de pod](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) est activé. Si `true`, le `speech-to-text-poddisruptionbudget` sera déployé dans le cluster Kubernetes. | `true` |

#### <a name="sentiment-analysis-sub-chart-chartsspeechtotext"></a>Analyse des sentiments (sous-graphique : charts/speechToText)

À compter de la version 2.2.0 du conteneur de reconnaissance vocale en texte et de la version 0.2.0 du graphique Helm, les paramètres suivants sont utilisés pour l’analyse des sentiments à l’aide de l’API Analyse de texte.

|Paramètre|Description|Valeurs|Default|
| --- | --- | --- | --- |
|`textanalytics.enabled`| Indique si le service d’**analyse de texte** est activé ou non| true/false| `false`|
|`textanalytics.image.registry`| Le registre d’images Docker pour l’**analyse de texte**| registre d’images Docker valide| |
|`textanalytics.image.repository`| Le dépôt d’images Docker pour l’**analyse de texte**| dépôt d’images Docker valide| |
|`textanalytics.image.tag`| La balise d’image Docker pour l’**analyse de texte**| balise d’image Docker valide| |
|`textanalytics.image.pullSecrets`| Les secrets de l’image pour l’extraction de l’image Docker de l’**analyse de texte**| noms de secrets valides| |
|`textanalytics.image.pullByHash`| Spécifie si vous extrayez l’image Docker par hachage.  Si la valeur est `yes`, `image.hash` doit également avoir cette valeur. Si la valeur est `no`, affectez-lui la valeur « false ». La valeur par défaut est `false`.| true/false| `false`|
|`textanalytics.image.hash`| Le hachage d’image Docker pour l’**analyse de texte**. Utilisez-le uniquement avec `image.pullByHash:true`.| hachage d’image Docker valide | |
|`textanalytics.image.args.eula`| Un des arguments exigés par le conteneur d’**analyse de texte**, qui indique que vous avez accepté la licence. La valeur de cette option doit être : `accept`.| `accept` si vous voulez utiliser le conteneur | |
|`textanalytics.image.args.billing`| Un des arguments exigés par le conteneur d’**analyse de texte**, qui spécifie l’URI du point de terminaison de facturation. La valeur de l’URI de point de terminaison de facturation est disponible dans la page Vue d’ensemble de Speech du portail Azure.|URI de point de terminaison de facturation valide||
|`textanalytics.image.args.apikey`| Un des arguments exigés par le conteneur d’**analyse de texte**, est utilisé pour effectuer le suivi des informations de facturation.| valeur apikey valide||
|`textanalytics.cpuRequest`| UC demandée pour le conteneur d’**analyse de texte**| int| `3000m`|
|`textanalytics.cpuLimit`| UC limitée pour le conteneur d’**analyse de texte**| | `8000m`|
|`textanalytics.memoryRequest`| Mémoire demandée pour le conteneur d’**analyse de texte**| | `3Gi`|
|`textanalytics.memoryLimit`| Mémoire limitée pour le conteneur d’**analyse de texte**| | `8Gi`|
|`textanalytics.service.sentimentURISuffix`| Suffixe d’URI d’analyse des sentiments. Le format de l’URI complet est « http://`<service>`:`<port>`/`<sentimentURISuffix>` ». | | `text/analytics/v3.0-preview/sentiment`|
|`textanalytics.service.type`| Type de service d’**analyse de texte** dans Kubernetes. Consultez [Types de service Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/) | type de service Kubernetes valide | `LoadBalancer` |
|`textanalytics.service.port`| Port du service d’**analyse de texte**| int| `50085`|
|`textanalytics.service.annotations`| Annotations que les utilisateurs peuvent ajouter aux métadonnées du service d’**analyse de texte**. Exemple :<br/> **Annotations :**<br/>`   ` **some/annotation1: value1**<br/>`  ` **some/annotation2: value2** | annotations, une par ligne| |
|`textanalytics.serivce.autoScaler.enabled`| Indique si l’option [Autoscaler de pods élastique](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) est activée. Si c’est le cas, `text-analytics-autoscaler`, est déployé dans le cluster Kubernetes | true/false| `true`|
|`textanalytics.service.podDisruption.enabled`| Indique si l’option [Budget de perturbation de pod](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) est activée. Si c’est le cas, `text-analytics-poddisruptionbudget`, est déployé dans le cluster Kubernetes| true/false| `true`|

---
title: Installer des conteneurs Speech
titleSuffix: Azure Cognitive Services
description: Détaille les options de configuration de graphique Helm pour la synthèse vocale.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 22168974ab8b285413b4fa6e947c05f65a73ae12
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002270"
---
### <a name="text-to-speech-sub-chart-chartstexttospeech"></a>Synthèse vocale (sous-graphique : charts/textToSpeech)

Pour remplacer le graphique en parapluie, ajoutez le préfixe `textToSpeech.` à n’importe quel paramètre pour le rendre plus spécifique. Il remplacera le paramètre correspondant. Par exemple, `textToSpeech.numberOfConcurrentRequest` remplace `numberOfConcurrentRequest`.

|Paramètre|Description|Default|
| -- | -- | -- |
| `enabled` | Indique si le service de **synthèse vocale** est activé ou non. | `false` |
| `numberOfConcurrentRequest` | Le nombre de requêtes simultanées pour le service de **synthèse vocale**. Ce graphique calcule automatiquement les ressources processeur et mémoire en fonction de cette valeur. | `2` |
| `optimizeForTurboMode`| Indique si le service doit optimiser pour l’entrée texte via des fichiers texte. Si `true`, ce graphique alloue plus de ressources processeur au service. | `false` |
| `image.registry`| Le registre d’image docker pour la **synthèse vocale**. | `containerpreview.azurecr.io` |
| `image.repository` | Le référentiel d’image docker pour la **synthèse vocale**. | `microsoft/cognitive-services-text-to-speech` |
| `image.tag` | La balise d’image docker pour la **synthèse vocale**. | `latest` |
| `image.pullSecrets` | Les secrets de l’image pour l’extraction de l’image docker de **synthèse vocale**. | |
| `image.pullByHash`| Indique si l’image docker est extraite par hachage. Si `true`, `image.hash` est requis. | `false` |
| `image.hash`| Le hachage d’image docker pour la **synthèse vocale**. Utilisé uniquement si `image.pullByHash: true`.  | |
| `image.args.eula` (requis) | Indique que vous avez accepté la licence. La seule valeur valide est `accept` | |
| `image.args.billing` (requis) | La valeur de l’URI de point de terminaison de facturation est disponible dans la page Vue d’ensemble de Speech du portail Azure. | |
| `image.args.apikey` (requis) | Utilisé pour le suivi des informations de facturation. ||
| `service.type` | Le type de service Kubernetes du service de **synthèse vocale**. Consultez les [Instructions relatives aux types de service Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/) pour plus d’informations et pour vérifier la prise en charge des fournisseurs cloud. | `LoadBalancer` |
| `service.port`|  Le port du service de **synthèse vocale**. | `80` |
| `service.annotations` | Les annotations de **synthèse vocale** pour les métadonnées du service. Les annotations sont des paires clé-valeur. <br>`annotations:`<br>&nbsp;&nbsp;`some/annotation1: value1`<br>&nbsp;&nbsp;`some/annotation2: value2` | |
| `service.autoScaler.enabled` | Indique si [l’Autoscaler de pods élastique](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) est activé. Si `true`, le `text-to-speech-autoscaler` sera déployé dans le cluster Kubernetes. | `true` |
| `service.podDisruption.enabled` | Indique si le [Budget de perturbation de pod](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/) est activé. Si `true`, le `text-to-speech-poddisruptionbudget` sera déployé dans le cluster Kubernetes. | `true` |
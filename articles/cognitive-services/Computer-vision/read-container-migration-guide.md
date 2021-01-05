---
title: Migration vers les conteneurs OCR Read v3.x
titleSuffix: Azure Cognitive Services
description: Découvrir comment migrer vers les conteneurs OCR Read v3
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: overview
ms.date: 10/23/2020
ms.author: aahi
ms.openlocfilehash: 0ae7bb9acde3cc8552a23db12d208a82ba0fb2f3
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95800874"
---
# <a name="migrate-to-the-read-v3x-ocr-containers"></a>Migrer vers des conteneurs OCR Read v3.x

Si vous utilisez la version 2 du conteneur OCR Read de Vision par ordinateur, utilisez cet article pour apprendre à mettre à niveau votre application afin d’utiliser la version 3.x du conteneur. 


## <a name="configuration-changes"></a>Modifications de configuration

* `ReadEngineConfig:ResultExpirationPeriod` n’est plus pris en charge. Le conteneur Read a une tâche Cron intégrée qui supprime les résultats et les métadonnées associés à une requête à l’issue de 48 heures.
* `Cache:Redis:Configuration` n’est plus pris en charge. Le cache n’est pas utilisé dans les conteneurs v3.x, donc vous n’avez pas besoin de le définir.

## <a name="api-changes"></a>Modifications d'API

Le conteneur Read v3.2 utilise la version 3 de l’API Vision par ordinateur et a les points de terminaison suivants :

* `/vision/v3.2-preview.1/read/analyzeResults/{operationId}`
* `/vision/v3.2-preview.1/read/analyze`
* `/vision/v3.2-preview.1/read/syncAnalyze`

Pour plus d’informations sur la mise à jour de vos applications pour utiliser la version 3 de l’API Read cloud, consultez le [Guide de migration de l’API REST Vision par ordinateur v3](./upgrade-api-versions.md). Ces informations s’appliquent également au conteneur. Notez que les opérations de synchronisation sont uniquement prises en charge dans les conteneurs.

## <a name="memory-requirements"></a>Besoins en mémoire

Les exigences et les suggestions sont basées sur des tests d’évaluation effectués avec une seule requête par seconde, à partir d’une image de 8 Mo d’une lettre d’entreprise analysée qui contient 29 lignes et 803 caractères au total. Le tableau suivant indique l’allocation de ressources minimale et recommandée pour chaque conteneur Lire.

|Conteneur  |Minimum | Recommandé  |
|---------|---------|------|
|Read 3.2-preview | 8 cœurs, 16 Go de mémoire         | 8 cœurs, 24 Go de mémoire |

Chaque cœur doit être cadencé à au moins 2,6 gigahertz (GHz).

Le cœur et la quantité de mémoire correspondent aux paramètres `--cpus` et `--memory` qui sont utilisés dans le cadre de la commande docker run.

## <a name="storage-implementations"></a>Implémentations de stockage

>[!NOTE]
> MongoDB n’est plus pris en charge dans les versions 3.x du conteneur. En effet, les conteneurs prennent plutôt en charge le stockage Azure et les systèmes de fichiers hors connexion.

| Implémentation |    Argument(s) de runtime nécessaire(s) |
|---------|---------|
|Niveau de fichier (par défaut)    | Aucun argument de runtime n’est nécessaire. Le répertoire `/share` sera utilisé. |
|Objets blob Azure    | `Storage:ObjectStore:AzureBlob:ConnectionString={AzureStorageConnectionString}` |

## <a name="queue-implementations"></a>Implémentations de file d’attente

Dans un conteneur v3.x, RabbitMQ n’est pas pris en charge pour l’instant. Les implémentations prises en charge sont les suivantes :

| Implémentation | Argument(s) de runtime | Usage prévu |
|---------|---------|-------|
| En mémoire (par défaut) | Aucun argument de runtime n’est nécessaire. | Développement et test |
| Files d'attente Azure | `Queue:Azure:ConnectionString={AzureStorageConnectionString}` | Production |
| RabbitMQ    | Indisponible | Production |

Pour une redondance accrue, le conteneur Read v3.x utilise un minuteur de visibilité pour vérifier que les demandes peuvent être traitées correctement en cas d’incident, dans le cadre de l’exécution dans une configuration à plusieurs conteneurs. 

Définissez le minuteur avec `Queue:Azure:QueueVisibilityTimeoutInMilliseconds`, qui définit le délai à l’issue duquel un message devient invisible car un rôle de travail est en train de le traiter. Pour éviter le traitement redondant des pages, nous vous recommandons de définir le délai d’attente sur 120 secondes. La valeur par défaut est de 30 secondes.

| Valeur par défaut | Valeur recommandée |
|---------|---------|
| 30000 |    120 000 |


## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir les paramètres de configuration, passez en revue [Configurer des conteneurs](computer-vision-resource-container-config.md).
* Pour en savoir plus sur la reconnaissance du texte imprimé et manuscrit, passez en revue [Vue d’ensemble de Vision par ordinateur](overview.md).
* Pour plus d’informations sur les méthodes prises en charge par le conteneur, reportez-vous à l’[API Vision par ordinateur](//westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa).
* Pour résoudre les problèmes liés à la fonctionnalité Vision par ordinateur, reportez-vous au [Forum aux questions (FAQ)](FAQ.md).
* Utiliser davantage de [conteneurs Cognitive Services](../cognitive-services-container-support.md)
---
title: Streaming de données dans Azure SQL Edge
description: Découvrez le streaming de données dans Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: f63ab040e750c0c642c9656a5482529b926e9295
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392110"
---
# <a name="data-streaming-in-azure-sql-edge"></a>Streaming de données dans Azure SQL Edge

Azure SQL Edge fournit une implémentation native des fonctionnalités de diffusion en continu de données appelées diffusion en continu T-SQL. Ce dernier permet le streaming de données en temps réel, l’analytique et le traitement des événements pour analyser et traiter rapidement et simultanément d’importants volumes de données de streaming à partir de plusieurs sources. Le streaming T-SQL repose sur l’utilisation du moteur de streaming haute performance sur lequel s’appuie [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md) dans Microsoft Azure. La fonctionnalité prend en charge un ensemble similaire de fonctionnalités offertes par Azure Stream Analytics s’exécutant sur la périphérie.

Comme avec Stream Analytics, le streaming T-SQL reconnaît les modèles et les relations des informations extraites d’un certain nombre de sources d’entrée IoT, parmi lesquelles les appareils, les capteurs et les applications. Vous pouvez utiliser ces modèles pour déclencher des actions et lancer des workflows. Par exemple, vous pouvez créer des alertes, transmettre des informations à une solution de création de rapports ou de visualisation ou stocker les données pour les utiliser ultérieurement. 

Le streaming T-SQL peut vous aider à effectuer les tâches suivantes :

* Analyse des flux de données de télémétrie en temps réel à partir d’appareils IoT.
* Utilisation de l’analytique en temps réel des données générées à partir de véhicules autonomes et sans pilote.
* Utilisation de la supervision à distance et de la maintenance prédictive de ressources stratégiques dans les domaines de l’industrie et de la fabrication.
* Utilisation de la détection des anomalies et de la reconnaissance des modèles des relevés de capteurs IoT dans une ferme agricole ou énergétique.

## <a name="how-does-t-sql-streaming-work"></a>Comment fonctionne le streaming T-SQL ?

Le streaming T-SQL fonctionne exactement de la même façon qu’[Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md#how-does-stream-analytics-work). Par exemple, il utilise le concept de streaming de *travaux* pour le traitement de streaming de données en temps réel. 

Un travail Stream Analytics se compose de ce qui suit :

- **Entrée de flux**  : définit les connexions à une source de données à partir de laquelle lire le flux de données. Actuellement, Azure SQL Edge prend en charge les types d’entrée de flux suivants :
    - Hub Edge
    - Kafka (la prise en charge des entrées Kafka est uniquement disponible sur les versions Intel/AMD64 d’Azure SQL Edge.)

- **Sortie de flux**  : définit les connexions à une source de données dans laquelle écrire le flux de données. Actuellement, Azure SQL Edge prend en charge les types de sortie de flux suivants :
    - Hub Edge
    - SQL (La sortie SQL peut être une base de données locale au sein de l’instance d’Azure SQL Edge ou une instance SQL Server ou Azure SQL Database distante.) 

- **Requête de flux**  : définit la transformation, les agrégations, le filtrage, le tri et les jointures à appliquer au flux d’entrée avant qu’il ne soit écrit dans la sortie de flux. La requête de flux est basée sur le même langage de requête que celui utilisé par Stream Analytics. Pour plus d’informations, consultez [Langage de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference).

> [!IMPORTANT]
> Contrairement à Stream Analytics, le streaming T-SQL ne prend pas en charge l’[utilisation de données de référence pour les recherches](../stream-analytics/stream-analytics-use-reference-data.md) ou l’[utilisation d’UDF et d’UDA dans un travail de flux](../stream-analytics/streaming-technologies.md#you-want-to-write-udfs-udas-and-custom-deserializers-in-a-language-other-than-javascript-or-c).

> [!NOTE]
> Le streaming T-SQL ne prend en charge qu’un sous-ensemble de la surface d’exposition au langage prise en charge par Stream Analytics. Pour plus d’informations, consultez [Langage de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference).

## <a name="limitations-and-restrictions"></a>Limitations et restrictions

Les limitations et restrictions suivantes s’appliquent au streaming T-SQL. 

- Seule un travail de streaming peut être actif à un moment donné. Les travaux déjà en cours d’exécution doivent être arrêtés avant le démarrage d’un autre travail.
- Chaque exécution de travail de streaming est monothread. Si le travail de streaming contient plusieurs requêtes, chaque requête est évaluée dans l’ordre séquentiel.
- Lorsque vous avez arrêté un travail de diffusion en continu dans Azure SQL Edge, un certain temps peut s’écouler avant le démarrage du travail de diffusion en continu suivant. Ce délai est introduit, car le processus de diffusion en continu sous-jacent doit être arrêté en réponse à la demande d’arrêt du travail, puis redémarré en réponse à la demande de démarrage du travail. 
- Diffuser en continu T-SQL jusqu’à 32 partitions pour un flux kafka. Toute tentative de configuration d’un nombre de partitions supérieur génère une erreur. 

## <a name="next-steps"></a>Étapes suivantes

- [Créer une tâche Stream Analytics dans Azure SQL Edge](create-stream-analytics-job.md)
- [Afficher les métadonnées associées aux tâches de flux dans Azure SQL Edge ](streaming-catalog-views.md)
- [Créer un flux externe](create-external-stream-transact-sql.md)
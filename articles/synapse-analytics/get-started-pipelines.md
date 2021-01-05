---
title: 'Tutoriel : Commencer l’intégration avec les pipelines'
description: Dans ce tutoriel, vous allez découvrir comment intégrer des pipelines et des activités en utilisant Synapse Studio.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: pipeline
ms.topic: tutorial
ms.date: 10/27/2020
ms.openlocfilehash: af01d5b5e424dd5ea229115f7aa3570d0b7cd511
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92744921"
---
# <a name="integrate-with-pipelines"></a>Intégrer avec des pipelines

Dans ce tutoriel, vous allez découvrir comment intégrer des pipelines et des activités en utilisant Synapse Studio. 

## <a name="overview"></a>Vue d’ensemble

Vous pouvez intégrer un large éventail de tâches dans Azure Synapse.

1. Dans Synapse Studio, accédez au hub **Intégrer**.
1. Sélectionnez **+**  > **Pipeline** pour créer un pipeline.
1. Accédez au hub **Développer** , puis sélectionnez l’un des notebooks que vous avez créés précédemment.
1. Faites glisser ce notebook dans le pipeline. ( **Remarque**  : Ajoutez l’étape d’importation des modules dans le notebook (comme spécifié dans [ce document](https://docs.microsoft.com/azure/synapse-analytics/spark/synapse-spark-sql-pool-import-export#transfer-data-to-or-from-a-sql-pool-attached-with-the-workspace)), car ces modules sont nécessaires pendant l’exécution à partir du pipeline).
1. Dans le pipeline, sélectionnez **Ajouter un déclencheur** > **Nouveau/modifier**.
1. Dans **Choisir un déclencheur** , sélectionnez **Nouveau** , puis définissez la **Récurrence** sur « Toutes les heures ».
1. Sélectionnez **OK**. 
1. Sélectionnez **Publier tout**.
1. Si vous souhaitez que le pipeline s’exécute immédiatement, sans attendre l’heure suivante, sélectionnez **Ajouter un déclencheur** > **Déclencher maintenant**.



## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Visualiser des données avec Power BI](get-started-visualize-power-bi.md)
                                 

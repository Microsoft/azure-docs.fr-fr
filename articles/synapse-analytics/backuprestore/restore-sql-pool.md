---
title: Restaurer un pool SQL dédié existant
description: Guide pratique de restauration d’un pool SQL dédié existant.
services: synapse-analytics
author: joannapea
manager: igorstan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 10/29/2020
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 86150107273b247bef2c74e5b8c5272d7148587e
ms.sourcegitcommit: 46c5ffd69fa7bc71102737d1fab4338ca782b6f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94331985"
---
# <a name="restore-an-existing-dedicated-sql-pool"></a>Restaurer un pool SQL dédié existant

Dans cet article, vous allez apprendre à restaurer un pool SQL dédié existant dans Azure Synapse Analytics à l’aide du portail Azure et de Synapse Studio. Cet article s’applique à la fois aux restaurations et aux restaurations géographiques. 

## <a name="restore-an-existing-dedicated-sql-pool-through-the-synapse-studio"></a>Restaurer un pool SQL dédié existant via Synapse Studio

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Accédez à votre espace de travail Synapse. 
3. Sous Prise en main-> Ouvrir Synapse Studio, sélectionnez **Ouvrir**.

    ![ Synapse Studio](../media/sql-pools/open-synapse-studio.png)
4. Dans le volet de navigation de gauche, cliquez sur **Données**.
5. Sélectionnez **Gérer les pools**. 
6. Sélectionnez **+ Nouveau** pour créer un pool SQL dédié. 
7. Sous l’onglet Paramètres supplémentaires, sélectionnez un point de restauration à partir duquel effectuer la restauration. 

    Si vous souhaitez effectuer une restauration géographique, sélectionnez l’espace de travail et le pool SQL dédié que vous souhaitez récupérer. 

8. Sélectionnez **Points de restauration automatiques** ou **Points de restauration définis par l’utilisateur**. 

    ![Points de restauration](../media/sql-pools/restore-point.PNG)

    Si le pool SQL dédié n’a pas de points de restauration automatiques, patientez quelques heures ou créez un point de restauration défini par l’utilisateur avant la restauration. Pour les points de restauration définis par l’utilisateur, sélectionnez un point existant ou créez-en un nouveau.

    Si vous restaurez une sauvegarde géographique, il vous suffit de sélectionner l’espace de travail situé dans la région source et le pool SQL dédié que vous souhaitez restaurer. 

9. Sélectionnez **Vérifier + créer**.

## <a name="restore-an-existing-dedicated-sql-pool-through-the-azure-portal"></a>Restaurer un pool SQL dédié existant via le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Accédez au pool SQL dédié à partir duquel effectuer la restauration.
3. En haut du panneau Vue d’ensemble, sélectionnez **Restaurer**.

    ![ Présentation de la restauration](../media/sql-pools/restore-sqlpool-01.png)

4. Sélectionnez **Points de restauration automatiques** ou **Points de restauration définis par l’utilisateur**. 

    Si le pool SQL dédié n’a pas de points de restauration automatiques, patientez quelques heures ou créez un point de restauration défini par l’utilisateur avant la restauration. 

    Si vous souhaitez effectuer une restauration géographique, sélectionnez l’espace de travail et le pool SQL dédié que vous souhaitez récupérer. 

5. Sélectionnez **Vérifier + créer**.

## <a name="next-steps"></a>Étapes suivantes

- [Créer un point de restauration](sqlpool-create-restore-point.md)

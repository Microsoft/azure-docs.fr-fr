---
author: hrasheed-msft
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: include
ms.date: 11/20/2020
ms.author: hrasheed
ms.openlocfilehash: 6207e5720f2b92a260a541ab3b8134ba73e1b720
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96549720"
---
## <a name="creating-and-running-a-scan"></a>Création et exécution d’une analyse

> [!Note] 
> Les étapes et les captures d’écran ci-dessous illustrent le processus général de gestion des analyses sur différents types de sources de données. Vos options peuvent différer légèrement selon les types de sources de données que vous utilisez.

Pour créer une analyse et l’exécuter, procédez comme suit :

1. Accédez aux **Sources**.

1. Sélectionnez la source de données que vous avez inscrite.

1. Sélectionnez **+ Nouvelle analyse**.

1. Sélectionnez les informations d’identification pour vous connecter à votre source de données. 

   :::image type="content" source="media/manage-scans/set-up-scan.png" alt-text="Configurer l’analyse":::

1. Vous pouvez limiter votre analyse à des parties spécifiques de la source de données, telles que des dossiers, des collections ou des schémas, en cochant les éléments appropriés dans la liste.

   :::image type="content" source="media/manage-scans/scope-your-scan.png" alt-text="Définir la portée de votre analyse":::

1. Sélectionnez ensuite un ensemble de règles d’analyse pour votre analyse. Vous pouvez choisir entre l’ensemble système par défaut, les ensembles personnalisés existants ou créer un ensemble inline.

   :::image type="content" source="media/manage-scans/scan-rule-set.png" alt-text="Ensemble de règles d’analyse":::

1. Choisissez votre déclencheur d’analyse. Vous pouvez configurer une planification ou exécuter l’analyse une seule fois.

   :::image type="content" source="media/manage-scans/trigger-scan.png" alt-text="trigger":::

1. Passez en revue votre analyse et sélectionnez **Enregistrer et exécuter**.

## <a name="viewing-your-scans-and-scan-runs"></a>Affichage des analyses et des exécutions d’analyse

Pour afficher les analyses existantes, procédez comme suit :

1. Accédez au centre d’administration. Sélectionnez **Sources de données** sous la section **Sources et analyse**. 

2. Sélectionnez la source de données souhaitée. La liste des analyses existantes sur cette source de données apparaît.

3. Sélectionnez l’analyse dont vous souhaitez afficher les résultats.

4. Cette page affiche toutes les exécutions précédentes de l’analyse ainsi que les métriques et l’état de chaque exécution de l’analyse. Elle indique également si votre analyse a été planifiée ou était manuelle, le nombre de ressources pour lesquelles ont été appliquées des classifications, le nombre total de ressources découvertes, l’heure de début et de fin de l’analyse et la durée totale de l’analyse.

## <a name="manage-your-scans---edit-delete-or-cancel"></a>Gérer vos analyses : modifier, supprimer ou annuler

Pour gérer ou supprimer une analyse, procédez comme suit :

1. Accédez au centre d’administration. Sélectionnez **Sources de données** sous la section **Sources et analyse**, puis sélectionnez la source de données souhaitée.

2. Sélectionnez l’analyse que vous souhaitez gérer. Vous pouvez modifier l’analyse en sélectionnant **Modifier**.

3. Vous pouvez supprimer votre analyse en sélectionnant **Supprimer**. 

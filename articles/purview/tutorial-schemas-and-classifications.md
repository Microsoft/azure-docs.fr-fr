---
title: 'Tutoriel : Explorer les ensembles de ressources, les détails, les schémas et les classifications dans Azure Purview (préversion)'
description: Ce tutoriel explique comment utiliser les ensembles de ressources, les détails des ressources, les schémas et les classifications.
author: animukherjee
ms.author: anmuk
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: c74324ebeeefeed361c0557c45a280a411effa22
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97693331"
---
# <a name="tutorial-explore-resource-sets-details-schemas-and-classifications-in-azure-purview-preview"></a>Tutoriel : Explorer les ensembles de ressources, les détails, les schémas et les classifications dans Azure Purview (préversion)

> [!IMPORTANT]
> Azure Purview est actuellement disponible en PRÉVERSION. L’[Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) contient des conditions légales supplémentaires qui s’appliquent aux fonctionnalités Azure en version bêta, en préversion ou pas encore en disponibilité générale.

Dans ce tutoriel, vous explorez les composants clés de votre catalogue : les ensembles de ressources, les détails des ressources, les schémas et les classifications.

Ce tutoriel est la *partie 4 d’une série de cinq tutoriels* où vous apprenez les principes fondamentaux de la gestion de la gouvernance des données dans un patrimoine de données à l’aide d’Azure Purview. Ce tutoriel s’appuie sur les exercices que vous avez effectués dans [Partie 3 : Parcourir les ressources dans Azure Purview (préversion) et afficher leur traçabilité](tutorial-browse-and-view-lineage.md).

Dans ce tutoriel, vous apprenez à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Afficher les ensembles de ressources.
> * Afficher les détails des ressources.
> * Modifier le schéma et ajouter des classifications.

## <a name="prerequisites"></a>Prérequis

* Mener à bien le [didacticiel :  Parcourir les ressources dans Azure Purview (préversion) et afficher leur traçabilité](tutorial-browse-and-view-lineage.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="view-resource-sets"></a>Afficher les ensembles de ressources

Un ensemble de ressources est un objet unique dans le catalogue qui représente un grand nombre d’objets physiques dans le stockage. Les objets partagent généralement un schéma commun et, dans la plupart des cas, une convention de nommage ou une structure de dossiers. Par exemple, le format de date est *aaaa/mm/jj*. Pour plus d’informations sur les ensembles de ressources, consultez [Présentation des ensembles de ressources](concept-resource-sets.md).

1. Accédez à votre ressource Azure Purview dans le portail Azure, puis sélectionnez **Ouvrir Purview Studio**. Vous êtes automatiquement dirigé vers la page d’accueil de Purview Studio.

2. Entrez **contoso_staging_positivecashflow_ n** dans la zone de **recherche de ressources**, puis sélectionnez **Contoso_Staging_PositiveCashFlow_{N}.csv** dans les résultats de la recherche.

## <a name="view-asset-details"></a>Afficher les détails des ressources

1. L’onglet **Vue d’ensemble** affiche la **description**, les **termes du glossaire** et les **propriétés**, comme par exemple **qualifiedName**.

   :::image type="content" source="./media/tutorial-schemas-and-classifications/overview-tab.png" alt-text="Capture d’écran montrant l’onglet Vue d’ensemble d’une page de ressources d’un ensemble de ressources":::

1. Sous **Propriétés**, notez les deux champs suivants :

   * **partitionCount** : indique le nombre de fichiers physiques associés à cet ensemble de ressources.
   * **schemaCount** : indique le nombre de variations de schéma qui ont été trouvées dans cet ensemble de ressources.

   Azure Purview remplit ces propriétés dans un délai de 24 heures après que vous avez effectué l’analyse dans la [partie 1 de cette série de tutoriels](tutorial-scan-data.md).

1. Sélectionnez l’onglet **Contacts** pour passer en revue les valeurs **Experts** et **Propriétaires**.

## <a name="edit-the-schema-and-add-classifications"></a>Modifier le schéma et ajouter des classifications

1. Sélectionnez l’onglet **Schéma**. Observez les noms de colonnes et les classifications qui leur sont associées. Notez que l’analyse a rempli les propriétés automatiquement.

   :::image type="content" source="./media/tutorial-schemas-and-classifications/schema-tab.png" alt-text="Capture d’écran montrant l’onglet Schéma":::

1. Pour modifier la ressource, sélectionnez le bouton **Modifier** en haut à gauche. Ensuite, sélectionnez l’onglet **Schéma**.

1. Ajoutez une **description** pour une colonne ou renommez la colonne avec un nom plus convivial.

1. Ajoutez une classification au niveau de la ressource en sélectionnant la liste déroulante **Classification au niveau des colonnes** pour un nom de colonne qui n’a pas de classification définie.

   :::image type="content" source="./media/tutorial-schemas-and-classifications/edit-schema.png" alt-text="Page de modification du schéma":::

1. Une fois les modifications terminées, sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> * Afficher les ensembles de ressources.
> * Afficher les détails des ressources.
> * Modifier le schéma et ajouter des classifications.

Passez au tutoriel suivant pour en savoir plus sur le glossaire et la façon de définir et d’importer de nouveaux termes pour les ressources.

> [!div class="nextstepaction"]
> [Créer et importer des termes de glossaire](tutorial-import-create-glossary-terms.md)
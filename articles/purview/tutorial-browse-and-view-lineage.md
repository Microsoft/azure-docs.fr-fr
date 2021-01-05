---
title: 'Tutoriel : Parcourir les ressources dans Azure Purview et afficher leur traçabilité'
description: Ce tutoriel explique comment parcourir les ressources dans le catalogue et afficher la traçabilité des données.
author: djpmsft
ms.author: daperlov
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: 7ffbe2ded44ded4f580655f6ae9e98391490f94a
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97696098"
---
# <a name="tutorial-browse-assets-in-azure-purview-preview-and-view-their-lineage"></a>Tutoriel : Parcourir les ressources dans Azure Purview (préversion) et afficher leur traçabilité

> [!IMPORTANT]
> Azure Purview est actuellement disponible en PRÉVERSION. L’[Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) contient des conditions légales supplémentaires qui s’appliquent aux fonctionnalités Azure en version bêta, en préversion ou pas encore en disponibilité générale.

Ce tutoriel montre comment parcourir les ressources dans Azure Purview et afficher des informations importantes à leur sujet, telles que la traçabilité.

Ce tutoriel est la *partie 3 d’une série de cinq tutoriels* où vous apprenez les principes fondamentaux de la gestion de la gouvernance des données dans un patrimoine de données à l’aide d’Azure Purview. Ce tutoriel s’appuie sur les exercices que vous avez effectués dans [Partie 2 : Parcourir la page d’accueil d’Azure Purview (préversion) et rechercher une ressource](tutorial-asset-search.md).

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
>
> * Parcourir les ressources dans le catalogue.
> * Afficher la traçabilité des ressources.

## <a name="prerequisites"></a>Prérequis

* Mener à bien le [didacticiel : Parcourir la page d’accueil d’Azure Purview (préversion) et rechercher une ressource](tutorial-asset-search.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="browse-for-assets-in-the-catalog"></a>Parcourir les ressources dans le catalogue

Dans le tutoriel précédent, vous avez appris comment la recherche peut vous aider à découvrir des ressources dans le catalogue Azure Purview. Une autre façon de découvrir des ressources dans le catalogue consiste à utiliser l’expérience de navigation du catalogue.

Cette section montre comment parcourir le catalogue Azure Purview.

1. Accédez à votre ressource Azure Purview dans le portail Azure, puis sélectionnez **Ouvrir Purview Studio**. Vous êtes automatiquement dirigé vers la page d’accueil de Purview Studio.

1. Dans la **page d’accueil**, sélectionnez **Parcourir les ressources**.

   La page **Parcourir les ressources** apparaît et affiche un résumé de tous les types de ressources figurant dans votre catalogue.

1. Pour explorer les différentes ressources de type Azure Data Lake Gen2 disponibles dans votre catalogue, sélectionnez la vignette **Azure Data Lake Gen2**.

   :::image type="content" source="./media/tutorial-browse-and-view-lineage/browse-by-asset-type.png" alt-text="Capture d’écran montrant la page de navigation par type de ressources avec le type Azure Data Lake Gen2 sélectionné":::

1. S’il existe plusieurs ressources, vous pouvez sélectionner l’en-tête de colonne **Nom** pour trier les ressources par ordre alphabétique. Dans ce tutoriel, il existe une seule ressource Azure Data Lake Storage Gen2.

1. Sélectionnez le nom de la ressource.

1. Sélectionnez l’ensemble de ressources **Contoso_GrossProfit_{N}.ssv**. Si cette ressource n’existe pas dans votre catalogue, choisissez-en une autre.

   :::image type="content" source="media/tutorial-browse-and-view-lineage/view-adls-assets.png" alt-text="Liste des ressources Azure Data Lake Storage Gen2":::

## <a name="view-the-lineage-of-assets"></a>Afficher la traçabilité des ressources

Dans la page de détails des ressources, explorez la source des données.

1. Sélectionnez l’onglet **Traçabilité** de l’ensemble de ressources **Contoso_GrossProfit_{N}.ssv**.

   La ressource que vous avez sélectionnée s’affiche. Les informations de traçabilité que vous voyez indiquent que cet ensemble de ressources a été copié à partir de votre compte de stockage Blob Azure vers Azure Data Lake Storage Gen2.

   :::image type="content" source="./media/tutorial-browse-and-view-lineage/view-lineage.png" alt-text="Capture d’écran montrant la vue Traçabilité pour la ressource.":::

1. Sélectionnez la ressource d’objet blob dans cette page, puis sélectionnez le lien **Basculer vers la ressource**.

   Notez que la fenêtre a basculé vers l’ensemble de ressources Azure Blob, qui était la source des données Azure Data Lake Storage Gen2 d’origine.

1. Sélectionnez l’onglet **Vue d’ensemble** pour examiner la ressource et confirmer ses détails.

Pour obtenir des informations sur la façon de créer une activité de flux de données Azure Data Factory entre un objet blob Azure et un ensemble de ressources Azure Data Lake Storage Gen2 et d’observer la traçabilité, consultez [Activité de flux de données Azure Data Factory](../data-factory/concepts-data-flow-overview.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> * Parcourir les ressources dans le catalogue.
> * Afficher la traçabilité des ressources.

Passez au tutoriel suivant pour en savoir plus sur les ensembles de ressources, les détails des ressources, les schémas et les classifications.

> [!div class="nextstepaction"]
> [Ensembles de ressources, détails des ressources, schémas et classifications](tutorial-schemas-and-classifications.md)

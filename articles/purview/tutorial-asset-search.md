---
title: 'Tutoriel : Parcourir la page d’accueil d’Azure Purview et rechercher une ressource'
description: Ce tutoriel explique comment utiliser les fonctionnalités de la page d’accueil d’Azure Purview et effectuer des recherches dans le catalogue.
author: djpmsft
ms.author: daperlov
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: f778a55c206e012884b44c1fea0a63e2de6b481a
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97693339"
---
# <a name="tutorial-navigate-the-azure-purview-preview-home-page-and-search-for-an-asset"></a>Tutoriel : Parcourir la page d’accueil d’Azure Purview (préversion) et rechercher une ressource

> [!IMPORTANT]
> Azure Purview est actuellement disponible en PRÉVERSION. L’[Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) contient des conditions légales supplémentaires qui s’appliquent aux fonctionnalités Azure en version bêta, en préversion ou pas encore en disponibilité générale.

Dans ce tutoriel, vous allez vous familiariser avec Azure Purview en parcourant les fonctionnalités de la page d’accueil et en recherchant une ressource dans le catalogue.

Cet article est la *partie 2 d’une série de cinq tutoriels* où vous apprenez les principes fondamentaux de la gestion de la gouvernance des données dans un patrimoine de données à l’aide d’Azure Purview. Ce tutoriel s’appuie sur les exercices que vous avez effectués dans [Partie 1 : Analyser des données avec Azure Purview](tutorial-scan-data.md)

Ce didacticiel présente les procédures suivantes :

> [!div class="checklist"]
>
> * Parcourir la page d’accueil d’Azure Purview.
> * Rechercher une ressource.
> * Ajouter un propriétaire de ressource.

## <a name="prerequisites"></a>Prérequis

* Mener à bien le [didacticiel : Analyser des données avec Azure Purview](tutorial-scan-data.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="navigate-the-azure-purview-home-page"></a>Parcourir la page d’accueil d’Azure Purview

Les étapes suivantes vous guideront dans la page d’accueil d’Azure Purview.

1. Accédez à votre ressource Azure Purview dans le portail Azure, puis sélectionnez **Ouvrir Purview Studio**. Vous êtes automatiquement dirigé vers la page d’accueil de Purview Studio.

   La partie supérieure de la page d’accueil affiche le nom de votre catalogue et un ensemble d’analytiques du catalogue. Elles comprennent le nombre d’utilisateurs, de ressources de données et de termes de glossaire. Le résumé indique qu’il existe plus de 200 ressources au total et 113 termes de glossaire. Ce nombre est actualisé à mesure que votre organisation analyse et ajoute d’autres termes à Azure Purview.

   :::image type="content" source="./media/tutorial-asset-search/purview-home-page.png" alt-text="Capture d’écran montrant la page d’accueil d’Azure Purview":::

1. Sélectionnez **Parcourir les ressources** pour voir toutes vos ressources.

## <a name="search-for-an-asset"></a>Rechercher une ressource

Avant de commencer, voici un bref rappel de quelques termes élémentaires :

* **Ressource** : objet unique et concis dans le catalogue, qui contient la définition d’une entité dans le patrimoine de données. Une table SQL, un rapport Power BI et votre activité Data Factory sont des exemples d’entités.
  
* **Schéma** : également appelé colonne ou attribut, un schéma représente la structure d’une ressource ou d’un ensemble de ressources.

* **Ensemble de ressources** : objet unique dans le catalogue, qui représente un grand nombre d’objets physiques dans le stockage. Ces objets partagent généralement un schéma commun et, dans la plupart des cas, une convention de nommage ou une structure de dossiers. Par exemple, le format de date est *aaaa/mm/jj*. Pour plus d’informations, consultez [Présentation des ensembles de ressources](concept-resource-sets.md).

* **Type de ressource** : regroupement de ressources et de groupes de ressources sous un nom logique, qui correspond généralement au nom de la plateforme de données.

Suivez la procédure ci-dessous pour rechercher une ressource et indiquez-vous comme propriétaire :

1. Dans la zone **Rechercher dans le catalogue** de votre page d’accueil, entrez le nom du groupe de ressources qui contient votre patrimoine de données (le groupe de ressources que vous avez créé dans le tutoriel précédent). Une liste de suggestions s’affiche.

1. Sélectionnez **Afficher les résultats de la recherche**. Comme toutes vos ressources commencent par le nom de votre groupe de ressources, elles apparaissent toutes dans les résultats de la recherche. En dehors de ce tutoriel, vous rechercheriez des noms de ressources spécifiques, et non des groupes de ressources.

    :::image type="content" source="./media/tutorial-asset-search/search-suggestions.png" alt-text="Capture d’écran montrant la liste des suggestions de recherche dans le catalogue":::

1. Vous pouvez utiliser les filtres du volet de navigation gauche pour filtrer par type de ressource, classification, contact, étiquette et terme de glossaire.

1. Pour rechercher un ensemble de ressources, commencez à taper le nom de l’ensemble. Une liste de suggestions de recherche avec les mots clés corrects s’affiche. Vous pouvez également rechercher des noms absolus en les mettant entre guillemets.

   :::image type="content" source="media/tutorial-asset-search/keyword-search.png" alt-text="Recherche de ressources par mots clés Azure Purview":::

## <a name="edit-an-asset"></a>Modifier une ressource

1. Sélectionnez l’une des ressources dans les résultats de la recherche. Ensuite, sélectionnez **Modifier**.

1. Sous l’onglet **Vue d’ensemble**, vous pouvez ajouter une description.

    :::image type="content" source="./media/tutorial-asset-search/overview-tab.png" alt-text="Capture d’écran montrant le champ Description sous l’onglet Vue d’ensemble":::

1. Sélectionnez l’onglet **Contacts**. En regard de **Propriétaires**, dans **Sélectionner un utilisateur ou un groupe**, commencez à taper votre adresse e-mail d’entreprise.

    :::image type="content" source="./media/tutorial-asset-search/contacts-tab.png" alt-text="Capture d’écran montrant les champs remplis":::

    Une suggestion de nom s’affiche automatiquement.

1. En regard de **Experts**, dans **Sélectionner un utilisateur ou un groupe**, entrez un nom (par exemple, le nom de votre responsable), puis sélectionnez **Enregistrer**.

    Les champs Description, Nom du propriétaire et Nom de l’expert sont maintenant remplis.

## <a name="next-steps"></a>Étapes suivantes

Avant de passer au tutoriel suivant de cette série, passez un peu plus de temps à explorer la page d’accueil d’Azure Purview. Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> * Parcourir la page d’accueil d’Azure Purview.
> * Rechercher une ressource.
> * Ajouter un propriétaire de ressource.

Passez au tutoriel suivant pour découvrir comment parcourir les ressources dans Azure Purview et découvrir la traçabilité des ressources.

> [!div class="nextstepaction"]
> [Parcourir les ressources et afficher leur traçabilité](tutorial-browse-and-view-lineage.md)

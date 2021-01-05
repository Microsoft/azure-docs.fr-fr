---
title: Se connecter à Azure Data Share
description: Cet article explique comment connecter un compte Azure Data Share à Azure Purview pour rechercher des ressources et suivre la traçabilité des données.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/25/2020
ms.openlocfilehash: edea881d67d5a677339c6ff357c1ac45f5dfd420
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550562"
---
# <a name="how-to-connect-azure-data-share-and-azure-purview"></a>Connexion d’Azure Data Share et d’Azure Purview

Cet article explique comment connecter votre compte [Azure Data Share](../data-share/overview.md) à Azure Purview et régir les jeux de données partagés (sortants et entrants) dans votre patrimoine de données. Plusieurs rôles de gouvernance des données peuvent découvrir et suivre la traçabilité des données entre les organisations, les services et même les centres de données.

## <a name="common-scenarios"></a>Scénarios courants

La traçabilité Azure Data Share est destinée à fournir des informations détaillées pour l’analyse de la cause racine et l’analyse de l’impact.

### <a name="scenario-1-360-view-of-datasets-shared-inout-for-a-partner-organization-or-internal-department"></a>Scénario 1 : Vue à 360° des jeux de données partagés en entrée/sortie pour une organisation partenaire ou un service interne

Les responsables des données peuvent voir une liste de tous les jeux de données qui sont partagés de manière bidirectionnelle avec leurs organisations partenaires. Ils peuvent rechercher et découvrir les jeux de données par nom d’organisation et voir une vue complète de tous les partages sortants et entrants.

### <a name="scenario-2-root-cause-analysis---upstream-dependency-on-datasets-coming-into-organization-consumer-view-of-incoming-shares"></a>Scénario 2 : Analyse de la cause racine : dépendance en amont sur les jeux de données arrivant dans l’organisation (affichage consommateur des partages entrants)

Un rapport contient des informations incorrectes en raison de problèmes de données en amont provenant d’un compte Data Share externe. Les ingénieurs de données peuvent comprendre les défaillances en amont, être informés des raisons et contacter le propriétaire du partage pour résoudre les problèmes à l’origine de l’incohérence des données.

### <a name="scenario-3-impact-analysis-on-datasets-going-outside-organization-provider-view-of-outgoing-shares"></a>Scénario 3 : Analyse de l’impact sur les jeux de données en dehors de l’organisation (vue fournisseur des partages sortants)

Les producteurs de données veulent savoir qui sera impacté en cas de modification de leur jeu de données. En utilisant la traçabilité, un producteur de données peut facilement comprendre l’impact sur les partenaires internes ou externes en aval qui consomment des données à l’aide d’Azure Data Share.

## <a name="azure-data-share-and-purview-connected-experience"></a>Expérience connectée - Azure Data Share et Purview

Pour connecter Azure Data Share et votre compte Azure Purview, procédez comme suit :

1. Créez un compte Purview. Toutes les informations de la traçabilité Data Share seront collectées par un compte Purview. Vous pouvez utiliser un compte Purview existant ou en créer un nouveau.

1. Connectez votre Azure Data Share à votre compte Purview.

    1. Dans le portail Purview, vous pouvez accéder à **Centre de gestion** et connecter votre Azure Data Share sous la section **Connexions externes**.
    1. Sélectionnez **+ Nouveau** dans la barre supérieure, recherchez votre Azure Data Share dans la barre contextuelle et ajoutez le compte Data Share. Exécutez un travail d’instantané une fois que vous avez connecté votre Data Share au compte Purview, afin que les ressources Data Share et les informations de traçabilité soient visibles dans Purview.

       :::image type="content" source="media/how-to-link-azure-data-share/connect-to-data-share.png" alt-text="Centre de gestion pour lier Azure Data Share":::

1. Exécutez votre instantané dans Azure Data Share.

    - Une fois la connexion à Azure Data Share établie avec Azure Purview, vous pouvez exécuter un instantané pour vos partages existants. 
    - Si vous n’avez pas de partages existants, accédez au portail Azure Data Share pour [partager vos données](../data-share/share-your-data.md) [et vous abonner à un partage de données](../data-share/subscribe-to-data-share.md).
    - Une fois l’instantané du partage terminé, vous pouvez afficher les ressources Data Share associées et la traçabilité dans Purview.

1. Découvrez les comptes Data Share et partagez des informations dans votre compte Purview.

    - Dans la page d’accueil du compte Purview, sélectionnez **Parcourir par type de ressource** et sélectionnez la vignette **Azure Data Share**. Vous pouvez rechercher un nom de compte, un nom de partage, un instantané de partage ou une organisation partenaire. Dans le cas contraire, appliquez des filtres sur la page des résultats de la recherche pour le nom du compte, le type de partage (partages envoyés ou reçus).

       :::image type="content" source="media/how-to-link-azure-data-share/azure-data-share-search-result-page.png" alt-text="Azure Data Share dans la page de résultats de la recherche":::

    >[!Important]
    >Pour que les ressources Data Share s’affichent dans Purview, un travail d’instantané doit être exécuté une fois que vous avez connecté votre Data Share à Purview.

1. Suivez la traçabilité des jeux de données partagés avec Azure Data Share.

    - À partir de la page de résultats de la recherche Purview, choisissez l’instantané Data Share (reçu/envoyé) et sélectionnez l’onglet **Traçabilité** pour afficher un graphe de traçabilité avec les dépendances en amont et en aval.

    :::image type="content" source="media/how-to-link-azure-data-share/azure-data-share-lineage.png" alt-text="Traçabilité des jeux de données partagés avec Azure Data Share":::

## <a name="next-steps"></a>Étapes suivantes

- [Guide de l’utilisateur sur la traçabilité du catalogue](catalog-lineage-user-guide.md)
- [Lien vers Azure Data Factory pour la traçabilité](how-to-link-azure-data-factory.md)

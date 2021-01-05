---
title: Créer et partager des tableaux de bord dans le portail Azure
description: Cet article décrit comment créer, personnaliser, publier et partager des tableaux de bord dans le portail Azure.
ms.assetid: ff422f36-47d2-409b-8a19-02e24b03ffe7
ms.topic: how-to
ms.date: 03/23/2020
ms.openlocfilehash: 10c322c9d9d18f7b79a4419bd9efa2e366a5b7c3
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745823"
---
# <a name="create-and-share-dashboards-in-the-azure-portal"></a>Créer et partager des tableaux de bord dans le portail Azure

Les tableaux de bord sont une vue ciblée et organisée de vos ressources cloud dans le portail Azure. Servez-vous des tableaux de bord comme d’un espace de travail où vous pouvez rapidement lancer des tâches pour vos opérations quotidiennes ainsi que superviser vos ressources. Créez des tableaux de bord personnalisés en fonction des projets, des tâches ou des rôles utilisateur, par exemple.

Le portail Azure propose un tableau de bord par défaut comme point de départ. Vous pouvez modifier le tableau de bord par défaut. Créez et personnalisez des tableaux de bord supplémentaires, et publiez et partagez des tableaux de bord afin de les mettre à la disposition d’autres utilisateurs. Cet article montre comment créer un tableau de bord et personnaliser son interface, puis comment publier et partager ce tableau de bord.

## <a name="create-a-new-dashboard"></a>Créer un tableau de bord

Dans cet exemple, nous créons un tableau de bord privé auquel nous donnons un nom. Effectuez d’abord ces étapes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le menu du portail Azure, sélectionnez **Tableau de bord**. Votre vue par défaut peut déjà être définie sur le tableau de bord.

    ![Ouvrir le tableau de bord](./media/azure-portal-dashboards/portal-menu-dashboard.png)

1. Sélectionnez **Nouveau tableau de bord**.

    ![Capture d’écran du nouveau tableau de bord](./media/azure-portal-dashboards/create-new-dashboard.png)

    Cette action affiche la **Galerie de vignettes** dans laquelle vous sélectionnez les vignettes, ainsi qu’une grille vide sur laquelle vous disposez les vignettes.

    ![Capture d’écran de la Galerie de vignettes et de la grille vide](./media/azure-portal-dashboards/dashboard-name.png)

1. Sélectionnez le texte **Mon tableau de bord** dans l’étiquette du tableau de bord et entrez un nom permettant d’identifier facilement le tableau de bord personnalisé.

1. Sélectionnez **Personnalisation terminée**  dans l’en-tête de page pour quitter le mode d’édition.

La vue du tableau de bord affiche maintenant votre nouveau tableau de bord. Sélectionnez la flèche en regard du nom du tableau de bord pour afficher les tableaux de bord disponibles. La liste peut inclure des tableaux de bord que d’autres utilisateurs ont créés et partagés.

## <a name="edit-a-dashboard"></a>Modifier un tableau de bord

Maintenant, nous allons modifier le tableau de bord pour ajouter, redimensionner et réorganiser les vignettes qui représentent vos ressources Azure.

### <a name="add-tiles-from-the-dashboard"></a>Ajouter des vignettes à partir du tableau de bord

Pour ajouter des vignettes à un tableau de bord, effectuez les étapes suivantes :

1. Sélectionnez ![icône d’édition](./media/azure-portal-dashboards/dashboard-edit-icon.png) **Modifier** dans l’en-tête de page.

    ![Capture d’écran du tableau de bord montrant l’icône d’édition sélectionnée](./media/azure-portal-dashboards/dashboard-edit.png)

1. Parcourez la **Galerie de vignettes** ou utilisez le champ de recherche pour trouver la vignette souhaitée.

1. Sélectionnez **Ajouter** pour ajouter au tableau de bord la vignette de la taille et à l’emplacement par défaut. Vous pouvez aussi faire glisser la vignette dans la grille jusqu’à l’emplacement voulu.

> [!TIP]
> Si vous travaillez avec plusieurs organisations, ajoutez la vignette **Identité d’organisation** au tableau de bord pour montrer clairement à quelle organisation les ressources appartiennent.

### <a name="add-tiles-from-a-resource-page"></a>Ajouter des vignettes à partir d’une page de ressources

Il existe une autre façon d’ajouter des vignettes à votre tableau de bord. Bon nombre de pages de ressources contiennent une icône de punaise dans la barre de commandes. Si vous sélectionnez cette icône, une vignette représentant la page source est épinglée au tableau de bord actif. 

![Capture d’écran de la barre de commandes d’une page avec l’icône de punaise](./media/azure-portal-dashboards/dashboard-pin-blade.png)

### <a name="resize-or-rearrange-tiles"></a>Redimensionner ou réorganiser des vignettes

Pour changer la taille d’une vignette ou réorganiser les vignettes dans un tableau de bord, effectuez les étapes suivantes :

1. Sélectionnez ![icône d’édition](./media/azure-portal-dashboards/dashboard-edit-icon.png) **Modifier** dans l’en-tête de page.

1. Sélectionnez le menu contextuel dans le coin supérieur droit d’une vignette. Ensuite, choisissez une taille de vignette. Les vignettes qui prennent en charge toutes les tailles présentent aussi une « poignée » dans le coin inférieur droit, que vous pouvez faire glisser pour changer la taille de la vignette.

    ![Capture d’écran du tableau de bord avec le menu des tailles de vignette ouvert](./media/azure-portal-dashboards/dashboard-tile-resize.png)

1. Sélectionnez une vignette et faites-la glisser vers un nouvel emplacement sur la grille pour réorganiser votre tableau de bord.

### <a name="additional-tile-configuration"></a>Configuration supplémentaire des vignettes

Certaines vignettes peuvent nécessiter une configuration supplémentaire pour afficher les informations dont vous avez besoin. Par exemple, la vignette **Graphique des métriques** doit être configurée pour afficher une métrique d’**Azure Monitor**. Vous pouvez également personnaliser les données des vignettes pour remplacer les paramètres de date et d’heure par défaut du tableau de bord.

Une bannière **Configurer la vignette** reste affichée sur chaque vignette qui n’a pas encore été personnalisée. Pour personnaliser la vignette :

1. Sélectionnez **Personnalisation terminée**  dans l’en-tête de page pour quitter le mode d’édition.

1. Sélectionnez la bannière, puis effectuez la configuration requise.

    ![Capture d’écran d’une vignette à configurer](./media/azure-portal-dashboards/dashboard-configure-tile.png)

> [!NOTE]
> Utilisez une étiquette Markdown pour afficher un contenu statique personnalisé dans votre tableau de bord. Par exemple, vous pouvez afficher des instructions de base, une image, un ensemble de liens hypertexte ou même des informations de contact. Pour plus d’informations sur l’utilisation d’une vignette Markdown, voir [Utiliser une vignette Markdown sur les tableaux de bord Azure pour afficher un contenu personnalisé](azure-portal-markdown-tile.md).

### <a name="customize-tile-data"></a>Personnaliser les données de vignette

Le tableau de bord affiche automatiquement les données d’activité des dernières 24 heures. Pour changer l’intervalle de temps des données de cette vignette, effectuez les étapes suivantes :

1. Sélectionnez **Personnaliser les données de vignette** dans le menu contextuel ou le filtre ![icône de filtre](./media/azure-portal-dashboards/dashboard-filter.png) dans l’angle supérieur gauche de la vignette.

    ![Capture d’écran du menu contextuel de la vignette](./media/azure-portal-dashboards/dashboard-customize-tile-data.png)

1. Cochez la case **Remplacer les paramètres d’heure du tableau de bord au niveau de la vignette**.

    ![Capture d’écran de la boîte de dialogue où configurer les paramètres d’heure de la vignette](./media/azure-portal-dashboards/dashboard-override-time-settings.png)

1. Choisissez l’intervalle de temps des données à afficher sur cette vignette. Vous pouvez choisir un intervalle allant des 30 dernières minutes aux 30 derniers jours, ou définir une plage personnalisée.

1. Choisissez la granularité temporelle de l’affichage. Vous pouvez choisir d’afficher les données par incréments d’une minute à un mois.

1. Sélectionnez **Appliquer**.

## <a name="delete-a-tile"></a>Supprimer une vignette

Pour supprimer une vignette d’un tableau de bord, effectuez les étapes suivantes :

* Sélectionnez le menu contextuel dans le coin supérieur droit de la vignette, puis sélectionnez **Supprimer du tableau de bord**. Ou,

* Sélectionnez ![icône d’édition](./media/azure-portal-dashboards/dashboard-edit-icon.png) **Modifier** pour passer en mode de personnalisation. Pointez sur le coin supérieur droit de la vignette, puis sélectionnez l’icône de suppression ![icône de suppression](./media/azure-portal-dashboards/dashboard-delete-icon.png) pour supprimer la vignette du tableau de bord.

   ![Capture d’écran montrant comment supprimer une vignette du tableau de bord](./media/azure-portal-dashboards/dashboard-delete-tile.png)

## <a name="clone-a-dashboard"></a>Cloner un tableau de bord

Pour utiliser un tableau de bord existant comme modèle dans un nouveau tableau de bord, effectuez les étapes suivantes :

1. Assurez-vous que la vue de tableau de bord affiche le tableau de bord que vous souhaitez copier.

1. Dans l’en-tête de page, sélectionnez ![icône de clonage](./media/azure-portal-dashboards/dashboard-clone.png) **Cloner**.

1. Une copie du tableau de bord nommée **Clone de** *nom de votre tableau de bord* s’ouvre en mode d’édition. Suivez les étapes précédentes dans cet article pour renommer et personnaliser le tableau de bord.

## <a name="publish-and-share-a-dashboard"></a>Publier et partager un tableau de bord

Quand vous créez un tableau de bord, celui-ci est privé par défaut, ce qui signifie que vous êtes la seule personne à pouvoir le voir. Pour rendre des tableaux de bord visibles par d’autres, vous pouvez les publier et les partager. Pour plus d’informations, consultez [Partager des tableaux de bord Azure en utilisant le contrôle d’accès en fonction du rôle Azure](azure-portal-dashboard-share-access.md).

### <a name="open-a-shared-dashboard"></a>Ouvrir un tableau de bord partagé

Pour rechercher et ouvrir un tableau de bord partagé, effectuez les étapes suivantes :

1. Sélectionnez la flèche en regard du nom du tableau de bord.

1. Effectuez une sélection dans la liste des tableaux de bord affichée. Si le tableau de bord que vous souhaitez ouvrir n’est pas listé :

    1. Sélectionnez **Parcourir tous les tableaux de bord**.

        ![Capture d’écran du menu de sélection du tableau de bord](./media/azure-portal-dashboards/dashboard-browse.png)

    1. Dans le champ **Type**, sélectionnez **Tableaux de bord partagés**.

        ![Capture d’écran du menu de sélection de tous les tableaux de bord](./media/azure-portal-dashboards/dashboard-browse-all.png)

    1. Sélectionnez un ou plusieurs abonnements. Vous pouvez également entrer du texte pour filtrer les tableaux de bord par nom.

    1. Sélectionnez un tableau de bord dans la liste des tableaux de bord partagés.

## <a name="delete-a-dashboard"></a>Supprimer un tableau de bord

Pour supprimer définitivement un tableau de bord privé ou partagé, procédez comme suit :

1. Sélectionnez le tableau de bord à supprimer dans la liste en regard du nom de tableau de bord.

1. Sélectionnez ![icône de suppression](./media/azure-portal-dashboards/dashboard-delete-icon.png) **Supprimer** dans l’en-tête de page.

1. S’il s’agit d’un tableau de bord privé, sélectionnez **OK** dans la boîte de dialogue de confirmation pour le supprimer. Si c’est un tableau de bord partagé, dans la boîte de dialogue de confirmation, cochez la case pour confirmer que le tableau de bord publié ne sera plus visible par d’autres utilisateurs. Ensuite, sélectionnez **OK**.

    ![Capture d’écran de la confirmation de la suppression](./media/azure-portal-dashboards/dashboard-delete-dash.png)

## <a name="next-steps"></a>Étapes suivantes

* [Partager des tableaux de bord Azure à l’aide du contrôle d’accès en fonction du rôle Azure](azure-portal-dashboard-share-access.md)
* [Créer par programmation des tableaux de bord Azure](azure-portal-dashboards-create-programmatically.md)

---
title: Créer et gérer des projets Azure Migrate
description: Rechercher, créer, gérer et supprimer des projets dans Azure Migrate.
author: ms-psharma
ms.author: panshar
ms.manager: abhemraj
ms.topic: how-to
ms.date: 11/23/2020
ms.openlocfilehash: 445e08b255e5b4dd67dd1c6a47c8df6ce59df5bd
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753822"
---
# <a name="create-and-manage-azure-migrate-projects"></a>Créer et gérer des projets Azure Migrate

Cet article explique comment créer, gérer et supprimer des projets [Azure Migrate](migrate-services-overview.md).

Un projet de Azure Migrate est utilisé pour stocker les métadonnées de découverte, d’évaluation et de migration collectées à partir de l’environnement que vous évaluez ou migrez. Dans un projet, vous pouvez suivre les ressources découvertes, créer des évaluations et orchestrer des migrations vers Azure.  

## <a name="verify-permissions"></a>Vérification des autorisations

Vérifiez que vous disposez des autorisations correctes pour créer un projet Azure Migrate :

1. Dans le portail Azure, ouvrez l’abonnement approprié, puis sélectionnez  **Contrôle d’accès (IAM)** .
2. Dans **Vérifier l’accès**, recherchez le compte approprié, puis sélectionnez-le pour afficher les autorisations. Vous devez disposer des autorisations de *Contributeur* ou de *Propriétaire*. 


## <a name="create-a-project-for-the-first-time"></a>Créer un projet pour la première fois

Configurez un nouveau projet Azure Migrate dans un abonnement Azure.

1. Dans le portail Azure, recherchez *Azure Migrate*.
2. Dans **Services**, sélectionnez **Azure Migrate**.
3. Dans **Vue d’ensemble**, sélectionnez **Évaluer et migrer des serveurs**.

    ![Option dans Vue d’ensemble pour évaluer et migrer des serveurs](./media/create-manage-projects/assess-migrate-servers.png)

4. Dans **Serveurs**, sélectionnez **Créer un projet**.

    ![Bouton permettant de démarrer la création d’un projet](./media/create-manage-projects/create-project.png)

5. Dans **Créer un projet**, sélectionnez l’abonnement et le groupe de ressources Azure. Créez un groupe de ressources si vous n’en avez pas.
6. Dans **Détails du projet**, spécifiez le nom du projet ainsi que la zone géographique où vous souhaitez le créer.
    - La zone géographique est utilisée uniquement pour stocker les métadonnées collectées à partir de machines virtuelles locales. Vous pouvez sélectionner n’importe quelle région cible pour la migration. 
    - Passez en revue les zones géographiques prises en charge pour les clouds [publics](migrate-support-matrix.md#supported-geographies-public-cloud) et du [secteur public](migrate-support-matrix.md#supported-geographies-azure-government).

8. Sélectionnez **Create** (Créer).

   ![Page dans laquelle entrer les paramètres du projet](./media/create-manage-projects/project-details.png)


Attendez quelques minutes, le temps nécessaire au déploiement du projet Azure Migrate.

## <a name="create-a-project-in-a-specific-region"></a>Créer un projet dans une région spécifique

Dans le portail, vous pouvez sélectionner la zone géographique dans laquelle vous voulez créer le projet. Si vous voulez créer le projet dans une région Azure spécifique, utilisez la commande API suivante pour créer le projet.

```rest
PUT /subscriptions/<subid>/resourceGroups/<rg>/providers/Microsoft.Migrate/MigrateProjects/<mymigrateprojectname>?api-version=2018-09-01-preview "{location: 'centralus', properties: {}}"
```

## <a name="create-additional-projects"></a>Créer des projets supplémentaires

Si vous disposez déjà d’un projet Azure Migrate et que vous souhaitez créer un projet supplémentaire, procédez comme suit :  

1. Dans le [portail public Azure](https://portal.azure.com) ou [Azure Government](https://portal.azure.us), recherchez **Azure Migrate**.
2. Dans le tableau de bord Azure Migrate > **Serveurs**, sélectionnez **Modifier** dans le coin supérieur droit.

   ![Modifier un projet Azure Migrate](./media/create-manage-projects/switch-project.png)

3. Pour créer un nouveau projet, sélectionnez **cliquez ici**.


## <a name="find-a-project"></a>Rechercher un projet

Recherchez un projet comme suit :

1. Accédez au [Portail Azure](https://portal.azure.com) et recherchez *Azure Migrate*.
2. Dans le tableau de bord Azure Migrate > **Serveurs**, sélectionnez **Modifier** dans le coin supérieur droit.

    ![Passer à un projet Azure Migrate existant](./media/create-manage-projects/switch-project.png)

3. Sélectionnez l’abonnement et le projet Azure Migrate de votre choix.


### <a name="find-a-legacy-project"></a>Rechercher un projet hérité

Si vous avez créé le projet dans la [version précédente](migrate-services-overview.md#azure-migrate-versions) d’Azure Migrate, recherchez-le comme suit :

1. Accédez au [Portail Azure](https://portal.azure.com) et recherchez *Azure Migrate*.
2. Dans le tableau de bord Azure Migrate, si vous avez créé un projet dans la version précédente, une bannière faisant référence aux projets plus anciens s’affiche. Sélectionnez la bannière.

    ![Accéder aux projets existants](./media/create-manage-projects/access-existing-projects.png)

3. Passez en revue la liste des anciens projets.


## <a name="delete-a-project"></a>Supprimer un projet

Supprimez comme suit :

1. Ouvrez le groupe de ressources Azure dans lequel le projet a été créé.
2. Dans la page du groupe de ressources, sélectionnez **Afficher les types masqués**.
3. Sélectionnez le projet de migration à supprimer, ainsi que les ressources qui lui sont associées.
    - Le type de ressource est **Microsoft.Migrate/migrateprojects**.
    - Si le groupe de ressources est utilisé uniquement par le projet Azure Migrate, vous pouvez supprimer l’intégralité du groupe de ressources.

Notez les points suivants :

- Quand vous supprimez, le projet et les métadonnées concernant les machines découvertes sont supprimés.
- Si vous utilisez l’ancienne version d’Azure Migrate, ouvrez le groupe de ressources Azure dans lequel le projet a été créé. Sélectionnez le projet de migration à supprimer (le type de ressource est **Migration project**).
- Si vous utilisez l’analyse des dépendances avec un espace de travail Azure Log Analytics :
    - Cependant, si vous avez attaché un espace de travail Log Analytics à l’outil Server Assessment, l’espace de travail n’est pas supprimé automatiquement. Le même espace de travail Log Analytics peut être utilisé dans plusieurs scénarios.
    - Si vous souhaitez supprimer l’espace de travail Log Analytics, faites-le manuellement.
- La suppression du projet est irréversible. Les objets supprimés ne peuvent pas être récupérés.

### <a name="delete-a-workspace-manually"></a>Supprimer un espace de travail manuellement

1. Accédez à l’espace de travail Log Analytics associé au projet.

    - Si vous n’avez pas supprimé le projet Azure Migrate, vous trouverez le lien vers l’espace de travail dans **Essentials** > **Server Assessment**.
       ![Espace de travail LA](./media/create-manage-projects/loganalytics-workspace.png).
       
    - Si vous avez déjà supprimé le projet Azure Migrate, sélectionnez **Groupes de ressources** dans le volet gauche du Portail Azure et recherchez l’espace de travail.
       
2. [Suivez les instructions](../azure-monitor/platform/delete-workspace.md) pour supprimer l’espace de travail.

## <a name="next-steps"></a>Étapes suivantes

Ajoutez des outils d’[évaluation](how-to-assess.md) ou de [migration](how-to-migrate.md) à des projets Azure Migrate.

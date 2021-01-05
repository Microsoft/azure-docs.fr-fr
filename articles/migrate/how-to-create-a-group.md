---
title: Grouper des machines pour l’évaluation avec Azure Migrate | Microsoft Docs
description: Explique comment grouper des machines avant d’exécuter une évaluation avec le service Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: how-to
ms.date: 07/17/2019
ms.openlocfilehash: 364b5949e944a4317aa25f1f1b12545122881cec
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752105"
---
# <a name="create-a-group-for-assessment"></a>Créer un groupe pour l’évaluation

Cet article décrit comment créer des groupes de machines à évaluer avec Azure Migrate : Server Assessment.

[Azure Migrate](migrate-services-overview.md) vous aide à migrer vers Azure. Azure Migrate offre un hub centralisé pour suivre la découverte, l’évaluation et la migration d’une infrastructure, d’applications et de données locales vers Azure. Le hub fournit des outils Azure pour l’évaluation et la migration, ainsi que des offres d’éditeurs de logiciels indépendant (ISV) tiers. 

## <a name="grouping-machines"></a>Regroupement de machines

Vous rassemblez des machines dans des groupes pour déterminer si elles sont adaptées à la migration vers Azure et pour obtenir des estimations de coût et de dimensionnement Azure. Il existe deux façons de créer des groupes :

- Si vous connaissez les machines à migrer ensemble, vous pouvez créer manuellement le groupe dans Azure Migrate.
- Si vous n’êtes pas certain des machines à regrouper, vous pouvez utiliser la fonctionnalité de visualisation des dépendances dans Azure Migrate pour créer des groupes. 

> [!NOTE]
> La fonctionnalité de visualisation de dépendance n’est pas disponible dans Azure Government.

## <a name="create-a-group-manually"></a>Créer un groupe manuellement

Vous pouvez créer un groupe en même temps que vous [créez une évaluation](how-to-create-assessment.md).

Si vous souhaitez créer un groupe manuellement en dehors de la création d’une évaluation, procédez comme suit :

1. Dans le projet Azure Migrate > **Vue d’ensemble**, cliquez sur **Évaluer et migrer des serveurs**. Dans **Azure Migrate : Évaluation de serveur**, cliquez sur **Groupes**
    - Si vous n’avez pas encore ajouté l’outil Azure Migrate : Évaluation de serveur, cliquez pour l’ajouter. [Plus d’informations](how-to-assess.md)
    - Si vous n’avez pas encore créé de projet Azure Migrate, [cliquez ici pour en savoir plus](./create-manage-projects.md).

    ![Sélection de groupes](./media/how-to-create-a-group/select-groups.png)

2. Cliquez sur l’icône **Groupe**.
3. Dans **Créer un groupe**, spécifiez un nom de groupe et, dans **Nom de l’appliance**, sélectionnez l’appliance Azure Migrate que vous utilisez pour la détection de machines.
4. Dans la liste de machine, sélectionnez les machines que vous souhaitez ajouter au groupe > **Créer**.

    ![Créer un groupe](./media/how-to-create-a-group/create-group.png)

Vous pouvez maintenant utiliser ce groupe lorsque vous [Créez une évaluation de machine virtuelle Azure](how-to-create-assessment.md) ou [une Évaluation Azure VMware Solution (AVS)](how-to-create-azure-vmware-solution-assessment.md). Notez que vous pouvez créer une évaluation AVS sur des groupes avec uniquement des machines virtuelles VMware. 

## <a name="refine-a-group-with-dependency-mapping"></a>Affiner un groupe avec un mappage des dépendances

Le mappage des dépendances vous permet de visualiser les dépendances entre les machines. En général, vous utilisez le mappage des dépendances lorsque vous souhaitez évaluer les groupes de machines avec des niveaux de confiance élevés.
- Il vous aide à vérifier les dépendances des machines avant d’exécuter une évaluation. 
- Il vous aide également à planifier efficacement votre migration vers Azure, en vous assurant que rien n’est oublié et en évitant ainsi les interruptions surprises durant la migration.
- Vous pouvez découvrir les systèmes interdépendants qui doivent migrer en même temps et déterminer si un système en cours d’exécution continue de servir les utilisateurs ou si une mise hors service peut être envisagée au lieu de la migration.

Si vous avez déjà [configuré le mappage des dépendances](how-to-create-group-machine-dependencies.md) et que vous souhaitez affiner un groupe existant, procédez comme suit :

1. Sous l’onglet **Serveurs**, dans la vignette **Azure Migrate : Évaluation de serveur**, cliquez sur **Groupes**.
2. Cliquez sur le groupe que vous souhaitez affiner.
    - Si vous n’avez pas encore configuré le mappage des dépendances, la colonne **Dépendances** indique un état **Installation requise**. Pour chaque machine virtuelle pour laquelle vous souhaitez visualiser les dépendances, cliquez sur **Installation requise**. Installez quelques agents sur chaque machine virtuelle avant de pouvoir mapper les dépendances de la machine. [Plus d’informations](how-to-create-group-machine-dependencies.md)

        ![Ajouter un mappage des dépendances](./media/how-to-create-a-group/add-dependency-mapping.png)

    - Si vous avez déjà configuré le mappage des dépendances, dans la page groupe, cliquez sur **Afficher les dépendances** pour ouvrir le mappage des dépendances de groupe.

3. Après avoir cliqué sur **Afficher les dépendances**, la carte des dépendances de groupe affiche les éléments suivants :

    - Connexions TCP entrantes (clients) et sortantes (serveurs) vers toutes les machines du groupe sur lesquelles sont installées les agents de dépendance, et à partir de celles-ci.
    - Les machines dépendantes sur lesquels les agents de dépendance ne sont pas installés sont regroupées par numéro de port.
    - Les machines dépendantes avec des agents de dépendance installés sont affichées dans des zones distinctes.
    - Processus en cours d’exécution à l’intérieur de la machine. Développez chaque boîte de machine pour afficher les processus.
    - Propriétés de la machine ( y compris le nom de domaine complet, le système d’exploitation, l’adresse MAC). Cliquez sur chaque zone de la machine pour afficher les détails.

4. Pour afficher les dépendances dans un intervalle de temps de votre choix, modifiez l’intervalle de temps (par défaut, une heure) en spécifiant les dates de début et de fin, ou la durée.

    > [!NOTE]
    > L’intervalle de temps peut atteindre une heure. Si vous avez besoin d’un intervalle plus long, utilisez [Azure Monitor pour interroger les données dépendantes](how-to-create-group-machine-dependencies.md) pendant une période plus longue.

5. Une fois que vous avez identifié les dépendances que vous souhaitez ajouter ou supprimer dans le groupe, vous pouvez modifier le groupe. Utilisez Ctrl+clic pour ajouter ou supprimer des machines dans le groupe.

    - Seules les machines qui ont été détectées peuvent être ajoutées.
    - L’ajout et la suppression de machines au sein d’un groupe invalide ses évaluations passées.
    - Vous pouvez, si vous le souhaitez, créer une nouvelle évaluation lorsque vous modifiez le groupe.


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment configurer et utiliser le [mappage des dépendances](how-to-create-group-machine-dependencies.md) pour créer des groupes de confiance élevée.
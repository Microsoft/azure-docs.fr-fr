---
title: Gérer les stratégies de compte Azure Data Lake Analytics
description: Découvrez comment utiliser des stratégies de compte pour contrôler l’utilisation d’un compte Data Lake Analytics, par exemple, un nombre maximal d’unités Analytics ou de travaux.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 04/30/2018
ms.openlocfilehash: ba69098f32e131714a15923aef64c3f6ba17e18f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013307"
---
# <a name="manage-azure-data-lake-analytics-using-account-policies"></a>Gérer Azure Data Lake Analytics à l’aide de stratégies de compte

Les stratégies de compte vous aident à contrôler l’utilisation des ressources d’un compte Azure Data Lake Analytics. Ces stratégies vous permettent de contrôler le coût d’utilisation d’Azure Data Lake Analytics. Par exemple, grâce à ces stratégies, vous pouvez empêcher des pics de coût inattendus en limitant le nombre d’unités Analytics (AU) que le compte peut utiliser simultanément.## Stratégies au niveau du compte

Ces stratégies s’appliquent à tous les travaux dans un compte Data Lake Analytics.

## <a name="maximum-number-of-aus-in-a-data-lake-analytics-account"></a>Nombre maximal d’unités Analytics dans un compte Data Lake Analytics

Une stratégie contrôle le nombre total d’unités Analytics que votre compte Data Lake Analytics peut utiliser. Par défaut, la valeur est définie sur 250. Par exemple, si cette valeur est définie sur 250 unités Analytics, vous pouvez avoir un travail qui s’exécute avec 250 unités Analytics assignées ou 10 travaux exécutant chacun 25 unités Analytics. Les travaux supplémentaires qui sont envoyés sont placés en file d’attente jusqu'à ce que les travaux en cours d’exécution soient terminés. Lorsque les travaux en cours d’exécution sont terminés, des unités Analytics sont libérées pour l’exécution des travaux dans la file d’attente.

Pour modifier le nombre d’unités Analytics pour votre compte Data Lake Analytics :

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.
2. Cliquez sur **Limites et stratégies**.
3. Sous **Nombre maximale d’unités Analytics**, déplacez le curseur pour sélectionner une valeur ou entrez la valeur dans la zone de texte.
4. Cliquez sur **Enregistrer**.

   > [!NOTE]
   > Si vous avez besoin de plus d’unités Analytics que la valeur par défaut (250), cliquez sur **Aide + support** dans le portail pour envoyer une demande de support. Il est possible d’augmenter le nombre d’unités Analytics disponibles dans votre compte Data Lake Analytics.

## <a name="maximum-number-of-jobs-that-can-run-simultaneously"></a>Nombre maximal de travaux pouvant s’exécuter simultanément

Cette stratégie limite le nombre de travaux pouvant s’exécuter simultanément. Par défaut, cette valeur est définie sur 20. Si des unités Analytics sont disponibles dans votre Data Lake Analytics, de nouveaux travaux sont planifiés pour s’exécuter immédiatement jusqu’à ce que le nombre total de travaux en cours d’exécution atteigne la valeur de cette stratégie. Lorsque vous atteignez le nombre maximal de travaux pouvant s’exécuter simultanément, les travaux suivants sont placés en file d’attente par ordre de priorité jusqu’à ce qu’un ou plusieurs des travaux en cours d’exécution se terminent (selon la disponibilité des unités Analytics).

Pour modifier le nombre maximal de travaux pouvant s’exécuter simultanément :

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.
2. Cliquez sur **Limites et stratégies**.
3. Sous **Nombre maximal de travaux en cours d’exécution**, déplacez le curseur pour sélectionner une valeur ou entrez la valeur dans la zone de texte.
4. Cliquez sur **Enregistrer**.

   > [!NOTE]
   > Si vous avez besoin d’exécuter plus de travaux que la valeur par défaut (20), cliquez sur **Aide + support** dans le portail pour envoyer une demande de support. Il est possible d’augmenter le nombre de travaux pouvant s’exécuter simultanément dans votre compte Data Lake Analytics.

## <a name="how-long-to-keep-job-metadata-and-resources"></a>Durée de conservation des métadonnées et des ressources d’un travail

Lorsque vos utilisateurs exécutent des travaux U-SQL, le service Data Lake Analytics conserve tous les fichiers associés. Ces fichiers associés incluent le script U-SQL, les fichiers DLL référencés dans le script U-SQL, les ressources compilées et les statistiques. Les fichiers se trouvent dans le dossier /system/ du compte de stockage Azure Data Lake par défaut. Cette stratégie contrôle la durée pendant laquelle ces ressources sont stockées avant d’être automatiquement supprimées (la valeur par défaut est de 30 jours). Vous pouvez utiliser ces fichiers pour le débogage et pour régler les performances des travaux que vous exécuterez à nouveau à l’avenir.

Pour modifier la durée de conservation des métadonnées et des ressources d’un travail :

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.
2. Cliquez sur **Limites et stratégies**.
3. Sous **Jours de rétention des requêtes de tâche**, déplacez le curseur pour sélectionner une valeur ou entrez la valeur dans la zone de texte.  
4. Cliquez sur **Enregistrer**.

## <a name="job-level-policies"></a>Stratégies au niveau du travail

Les stratégies au niveau du travail vous permettent de contrôler les unités Analytics maximales et la priorité maximale que les utilisateurs individuels (ou les membres de groupes de sécurité spécifiques) peuvent définir sur les travaux qu’ils soumettent. Cette stratégie vous permet de contrôler les coûts générés par les utilisateurs. Cela vous permet également de contrôler l’effet que les travaux planifiés peuvent avoir sur des travaux de production à priorité élevée qui s’exécutent dans le même compte Data Lake Analytics.

Data Lake Analytics inclut deux stratégies que vous pouvez définir au niveau du travail :

- **Limite d’unités Analytics par travail** : les utilisateurs peuvent uniquement soumettre des travaux dont le nombre maximal d’unités Analytics est inférieur ou égal à cette valeur. Par défaut, cette limite est identique à la limite maximale d’unités Analytics du compte.

- **Priorité** : les utilisateurs peuvent uniquement soumettre les travaux dont la priorité est inférieure ou égale à cette valeur. Un nombre plus élevé indique une priorité plus faible. Par défaut, cette limite est définie sur 1, qui est la priorité la plus élevée.

Chaque compte contient une stratégie par défaut. La stratégie par défaut s’applique à tous les utilisateurs du compte. Vous pouvez créer des stratégies supplémentaires pour des utilisateurs et des groupes spécifiques.

> [!NOTE]
> Les stratégies au niveau du compte et les stratégies au niveau du travail s’appliquent simultanément.

## <a name="add-a-policy-for-a-specific-user-or-group"></a>Ajouter une stratégie pour un utilisateur ou un groupe spécifique

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.

2. Cliquez sur **Limites et stratégies**.

3. Sous **Limites d’envoi de tâches**, cliquez sur le bouton **Ajouter une stratégie**. Puis, sélectionnez ou saisissez les paramètres suivants :

   1. **Nom de la stratégie de calcul** : entrez un nom de stratégie, pour vous rappeler de l’objectif de la stratégie.

   2. **Sélectionner un utilisateur ou un groupe** : sélectionnez l’utilisateur ou le groupe auquel la stratégie s’applique.

   3. **Définir la limite d’unités Analytique du travail** : définissez la limite d’unités Analytique qui s’applique à l’utilisateur ou au groupe sélectionné.

   4. **Définir la limite de priorité** : définissez la limite de priorité qui s’applique à l’utilisateur ou au groupe sélectionné.

4. Cliquez sur **OK**.

5. La nouvelle stratégie est répertoriée dans la table des stratégies **Par défaut**, sous **Limites d’envoi de tâches**.

## <a name="delete-or-edit-an-existing-policy"></a>Supprimer ou modifier une stratégie existante

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.

2. Cliquez sur **Limites et stratégies**.

3. Sous **Limites d’envoi de tâches**, recherchez la stratégie que vous souhaitez modifier.

4. Pour voir les options **Supprimer** et **Modifier**, dans la colonne la plus à droite de la table, cliquez sur `...`.## Ressources supplémentaires relatives aux stratégies de travail

- [Post de blog Vue d’ensemble de la stratégie](/archive/blogs/azuredatalake/managing-your-azure-data-lake-analytics-compute-resources-overview)
- [Post de blog Stratégies au niveau du compte](/archive/blogs/azuredatalake/managing-your-azure-data-lake-analytics-compute-resources-account-level-policy)
- [Post de blog Stratégies au niveau du travail](/archive/blogs/azuredatalake/managing-your-azure-data-lake-analytics-compute-resources-job-level-policy)

## <a name="next-steps"></a>Étapes suivantes

- [Présentation d’Azure Data Lake Analytics](data-lake-analytics-overview.md)
- [Prise en main de Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-get-started-portal.md)
- [Gestion d’Azure Data Lake Analytics à l’aide d’Azure PowerShell](data-lake-analytics-manage-use-powershell.md)
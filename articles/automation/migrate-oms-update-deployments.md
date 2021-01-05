---
title: Effectuer la migration des déploiements de mise à jour des journaux Azure Monitor vers le portail Azure
description: Cet article explique comment effectuer la migration des déploiements de mises à jour des journaux Azure Monitor vers le portail Azure.
services: automation
ms.subservice: update-management
ms.date: 07/16/2018
ms.topic: conceptual
ms.openlocfilehash: a2226f55c829afa4316a92888d16f6dc68e1f931
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183598"
---
# <a name="migrate-azure-monitor-logs-update-deployments-to-azure-portal"></a>Effectuer la migration des déploiements de mise à jour des journaux Azure Monitor vers le portail Azure

Le portail Operations Management Suite (OMS) est en cours de [dépréciation](../azure-monitor/platform/oms-portal-transition.md). Toutes les fonctionnalités qui étaient disponibles dans le portail OMS pour Update Management sont disponibles dans le portail Azure, par le biais des journaux Azure Monitor. Cet article fournit les informations nécessaires pour migrer vers le portail Azure.

## <a name="key-information"></a>Informations essentielles

* Les déploiements existants continueront de fonctionner. Une fois que vous avez recréé le déploiement dans Azure, vous pouvez supprimer l’ancien déploiement.
* Toutes les fonctionnalités dont vous disposiez dans OMS sont disponibles dans Azure. Pour en savoir plus sur Update Management, consultez [Présentation d’Update Management](./update-management/overview.md).

## <a name="access-the-azure-portal"></a>Accéder au portail Azure

1. Dans votre espace de travail, cliquez sur **Ouvrir dans Azure**. 

    ![Ouvrir dans Azure - Log Analytics](media/migrate-oms-update-deployments/link-to-azure-portal.png)

2. Dans le portail Azure, cliquez sur **Compte Automation**.

    ![Journaux d’activité Azure Monitor](media/migrate-oms-update-deployments/log-analytics.png)

3. Dans votre compte Automation, cliquez sur **Update Management**.

    :::image type="content" source="media/migrate-oms-update-deployments/azure-automation.png" alt-text="Capture d'écran de la page Gestion des mises à jour.":::

4. Dans le portail Azure, sélectionnez **Comptes Automation** sous **Tous les services**. 

5. Sous **Outils de gestion**, sélectionnez le compte Automation approprié, puis cliquez sur **Update Management**.

## <a name="recreate-existing-deployments"></a>Recréer des déploiements existants

Tous les déploiements de mises à jour créés dans le portail OMS ont une [recherche enregistrée](../azure-monitor/platform/computer-groups.md), également appelée « groupe d’ordinateurs », avec le même nom que le déploiement de mises à jour existant. La recherche enregistrée contient la liste des ordinateurs pour lesquels le déploiement de mises à jour a été planifié.

:::image type="content" source="media/migrate-oms-update-deployments/oms-deployment.png" alt-text="Capture d'écran de la page Déploiements des mises à jour sur laquelle les champs Nom et Serveurs sont en surbrillance.":::

Pour utiliser cette recherche enregistrée existante, suivez ces étapes :

1. Pour créer un déploiement de mises à jour, accédez au portail Azure, sélectionnez le compte Automation qui est utilisé, puis cliquez sur **Update Management**. Cliquez sur **Planifier le déploiement de la mise à jour**.

    ![Planifier le déploiement de la mise à jour](media/migrate-oms-update-deployments/schedule-update-deployment.png)

2. Le volet Nouveau déploiement de mises à jour s’ouvre. Entrez les valeurs des propriétés décrites dans le tableau suivant, puis cliquez sur **Créer** :

3. Pour les **ordinateurs à mettre à jour**, sélectionnez la recherche enregistrée utilisée par le déploiement OMS.

    | Propriété | Description |
    | --- | --- |
    |Nom |Nom unique identifiant le déploiement de mises à jour. |
    |Système d’exploitation| Sélectionnez **Linux** ou **Windows**.|
    |Ordinateurs à mettre à jour |Sélectionnez une recherche enregistrée ou un groupe importé, ou choisissez un ordinateur dans la liste déroulante, puis sélectionnez des ordinateurs. Si vous choisissez **Machines**, l’état de préparation de la machine est indiqué dans la colonne **PRÉPARATION À LA MISE À JOUR DE L’AGENT**.</br> Pour en savoir plus sur les différentes méthodes de création de groupes d’ordinateurs dans les journaux Azure Monitor, consultez [Groupes d’ordinateurs dans les journaux Azure Monitor](../azure-monitor/platform/computer-groups.md). |
    |Classifications des mises à jour|Sélectionnez toutes les classifications des mises à jour dont vous avez besoin. CentOS ne prend pas directement en charge cette fonction.|
    |Mises à jour à exclure|Entrez les mises à jour à exclure. Pour Windows, entrez la version KB sans le préfixe **KB**. Pour Linux, entrez le nom du package ou utilisez un caractère générique.  |
    |Paramètres de planification|Sélectionnez l’heure de début, puis la périodicité (**Une fois** ou **Récurrent**). | 
    | Fenêtre de maintenance |Nombre de minutes défini pour les mises à jour. La valeur ne peut pas être inférieure à 30 minutes ni supérieure à 6 heures. |
    | Contrôle du redémarrage| Détermine la façon dont doivent être gérés les redémarrages.</br>Options disponibles :</br>Redémarrer si nécessaire (par défaut)</br>Toujours redémarrer</br>Ne jamais redémarrer</br>Redémarrer uniquement : les mises à jour ne sont pas installées|

4. Cliquez sur **Déploiements de mise à jour planifiés** pour voir l’état du déploiement de mises à jour nouvellement créé.

    ![Nouveau déploiement de mises à jour](media/migrate-oms-update-deployments/new-update-deployment.png)

5. Comme mentionné précédemment, une fois que vos nouveaux déploiements sont configurés via le portail Azure, vous pouvez supprimer les déploiements existants dans le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Update Management dans Azure Automation, consultez [Update Management](./update-management/overview.md).
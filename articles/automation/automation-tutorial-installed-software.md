---
title: Détecter les logiciels installés sur vos machines virtuelles avec Azure Automation | Microsoft Docs
description: Cet article décrit les logiciels installés sur les machines virtuelles de votre environnement.
services: automation
keywords: inventaire, inventorier, automation, automatisation, suivi des modifications, change tracking
ms.date: 04/11/2018
ms.topic: tutorial
ms.subservice: change-inventory-management
ms.custom: mvc
ms.openlocfilehash: 0c083f4576e123ea14d837ed3915c56d18d84623
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207561"
---
# <a name="discover-what-software-is-installed-on-your-vms"></a>Détecter les logiciels installés sur vos machines virtuelles

Dans ce tutoriel, vous apprenez à utiliser la fonctionnalité Change Tracking and Inventory d’Azure Automation pour déterminer quels logiciels sont installés dans votre environnement. Vous pouvez collecter et afficher l’inventaire des logiciels, fichiers, démons Linux, services Windows et clés de Registre Windows présents sur vos ordinateurs. Le suivi des configurations de vos machines peut vous aider à identifier les problèmes opérationnels au sein de votre environnement et à mieux comprendre l’état de vos machines.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Activer Change Tracking et Inventory
> * Activer une machine virtuelle Azure
> * Activer une machine virtuelle non-Azure
> * Afficher les logiciels installés
> * Rechercher les journaux d’activité d’inventaire des logiciels installés

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un [compte Automation](./index.yml) qui contiendra les runbooks Watcher et d’action, ainsi que la tâche Watcher.
* Une [machine virtuelle](../virtual-machines/windows/quick-create-portal.md) à activer pour la fonctionnalité.

## <a name="log-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="enable-change-tracking-and-inventory"></a>Activer Change Tracking et Inventory

Pour ce didacticiel, vous devez d’abord activer le suivi des modifications et l’inventaire. Si vous avez déjà activé la fonctionnalité, cette étape n’est pas nécessaire.

>[!NOTE]
>Si les champs sont grisés, une autre solution Automation est activée pour la machine virtuelle et vous devez utiliser les mêmes espace de travail et compte Automation.

1. Accédez à votre compte Automation et sélectionnez **Inventory** ou **Change Tracking** sous **Gestion de la configuration**.

2. Choisissez l’espace de travail [Log Analytics](../azure-monitor/log-query/log-query-overview.md). Cet espace de travail collecte les données qui sont générées par des fonctionnalités, telles que Change Tracking and Inventory. L’espace de travail fournit un emplacement unique permettant de consulter et d’analyser les données provenant de plusieurs sources.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

3. Sélectionnez le compte Automation à utiliser.

4. Configurez l’emplacement pour le déploiement.

5. Cliquez sur **Activer** pour déployer la fonctionnalité destinée à votre machine virtuelle. 

    ![Bannière de configuration de Inventory](./media/automation-tutorial-installed-software/enableinventory.png)

Pendant la configuration, la machine virtuelle est provisionnée avec l’agent Log Analytics pour Windows et un [Runbook Worker hybride](automation-hybrid-runbook-worker.md). L’activation de Change Tracking and Inventory peut prendre 15 minutes. Pendant ce temps, vous ne devez pas fermer la fenêtre du navigateur.

Dès la fonctionnalité activée, des informations sur les logiciels installés et les modifications apportées à la machine virtuelle sont envoyées aux journaux Azure Monitor. Entre 30 minutes et 6 heures peuvent être nécessaires pour que les données soient disponibles pour l’analyse.

## <a name="add-an-azure-vm-to-change-tracking-and-inventory"></a>Ajouter une machine virtuelle Azure à Change Tracking and Inventory

1. Dans votre compte Automation, accédez à **Inventory** ou à **Change Tracking** sous **Gestion de la configuration**.

2. Sélectionnez **+ Ajouter une machine virtuelle Azure**.

3. Sélectionnez votre machine virtuelle dans la liste des machines virtuelles. 

4. Cliquez sur **Activer** pour activer Change Tracking and Inventory sur la machine virtuelle. L’agent Log Analytics pour Windows est déployé sur la machine virtuelle et configure celle-ci pour communiquer avec l’espace de travail Log Analytics. L’opération de configuration peut prendre plusieurs minutes. 

5. À ce stade, si vous le souhaitez, vous pouvez sélectionner une nouvelle machine virtuelle dans la liste sur laquelle activer la fonctionnalité.

## <a name="add-a-non-azure-machine-to-change-tracking-and-inventory"></a>Ajouter une machine non-Azure à Change Tracking and Inventory

Si vous voulez activer des machines non-Azure pour la fonctionnalité :

1. Installez l’[agent Log Analytics pour Windows](../azure-monitor/platform/agent-windows.md) ou l’[agent Log Analytics pour Linux](automation-linux-hrw-install.md) selon votre système d’exploitation. 

2. Affichez votre compte Automation et accédez à **Inventory** ou à **Change Tracking** sous **Gestion de la configuration**. 

3. Cliquez sur **Gérer des machines**. Une liste s’affiche répertoriant les machines qui signalent à votre espace de travail Log Analytics qu’elles ne disposent pas de la fonctionnalité Change Tracking and Inventory activée. Sélectionnez l’option appropriée pour votre environnement :

    * **Activer sur toutes les machines disponibles** : cette option active la fonctionnalité sur toutes les machines communiquant avec votre espace de travail Log Analytics à cet instant.
    * **Activer sur toutes les machines disponibles et à venir** : cette option active la fonctionnalité sur toutes les machines communiquant avec votre espace de travail Log Analytics, ainsi que sur toutes les machines ajoutées ultérieurement à l’espace de travail.
    * **Activer sur les machines sélectionnées** : cette option active la fonctionnalité uniquement sur les machines que vous avez sélectionnées.

    ![Gérer les machines](./media/automation-tutorial-installed-software/manage-machines.png)

## <a name="view-installed-software"></a>Afficher les logiciels installés

Lorsque la fonctionnalité Change Tracking and Inventory est activée, vous pouvez afficher les résultats dans la page Inventory.

1. Dans votre compte Automation, sélectionnez **Inventaire** sous **Gestion de la configuration**.

2. Dans la page Inventaire, cliquez sur l’onglet **Logiciels**.

3. Remarquez le tableau qui dresse la liste des logiciels détectés. Les logiciels sont regroupés par nom et par version. Les informations détaillées sur chaque enregistrement de logiciel sont visibles dans le tableau. Ces détails comprennent le nom du logiciel, la version, l’éditeur, l’heure de la dernière actualisation (heure de l’actualisation la plus récente signalée par une machine du groupe) et les machines (nombre de machines possédant ce logiciel).

    ![Inventaires des logiciels](./media/automation-tutorial-installed-software/inventory-software.png)

4. Cliquez sur une ligne pour afficher les propriétés de l’enregistrement du logiciel et le nom des machines qui le possèdent.

5. Pour rechercher un logiciel ou un groupe de logiciels spécifique, vous pouvez faire une recherche dans la zone de texte située juste au-dessus de la liste des logiciels.
Le filtre vous permet de faire une recherche à partir du nom du logiciel, de la version ou de l’éditeur. Par exemple, la recherche de **Contoso** renvoie tous les logiciels dont le nom, l’éditeur ou la version contient **Contoso**.

## <a name="search-inventory-logs-for-installed-software"></a>Rechercher les journaux d’activité d’inventaire des logiciels installés

Change Tracking and Inventory génère des données de journal qui sont envoyées aux journaux Azure Monitor. Pour rechercher dans les journaux en exécutant des requêtes, sélectionnez **Log Analytics** en haut de la page Inventaire. Les données d’inventaire sont stockées sous le type `ConfigurationData`.

L’exemple de requête Log Analytics suivant retourne les résultats d’inventaire pour l’éditeur Microsoft Corporation.

```loganalytics
ConfigurationData
| where ConfigDataType == "Software"
| where Publisher == "Microsoft Corporation"
| summarize arg_max(TimeGenerated, *) by SoftwareName, Computer
```

Pour en savoir plus sur l’exécution et la recherche de fichiers journaux dans les journaux d’activité Azure Monitor, consultez [Journaux d’activité Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

## <a name="see-the-software-inventory-for-a-single-machine"></a>Afficher l’inventaire des logiciels d’une seule machine

Pour voir l’inventaire des logiciels d’une seule machine, vous pouvez accéder à l’inventaire à partir de la page de ressources de la machine virtuelle Azure ou utiliser les journaux d’activité Azure Monitor pour filtrer la machine correspondante. L’exemple de requête Log Analytics suivant retourne la liste des logiciels pour une machine nommée **ContosoVM**.

```loganalytics
ConfigurationData
| where ConfigDataType == "Software"
| summarize arg_max(TimeGenerated, *) by SoftwareName, CurrentVersion
| where Computer =="ContosoVM"
| render table
| summarize by Publisher, SoftwareName
```

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a appris à afficher l’inventaire des logiciels :

> [!div class="checklist"]
> * Activer Change Tracking et Inventory
> * Activer une machine virtuelle Azure
> * Activer une machine virtuelle non-Azure
> * Afficher les logiciels installés
> * Rechercher les journaux d’activité d’inventaire des logiciels installés

Accédez ensuite à la présentation de la fonctionnalité Change Tracking and Inventory pour en savoir plus.

> [!div class="nextstepaction"]
> [Vue d’ensemble de Change Tracking and Inventory](change-tracking/overview.md)

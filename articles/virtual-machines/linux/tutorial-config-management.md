---
title: 'Tutoriel : Gérer la configuration d’une machine virtuelle Linux dans Azure'
description: Dans ce tutoriel, vous allez apprendre à identifier les changements et à gérer les mises à jour de packages d’une machine virtuelle Linux
services: virtual-machines-linux
documentationcenter: virtual-machines
author: mgoedtel
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/27/2019
ms.author: magoedte
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 3210829b3281aa862cdf0dbdc9c915249a55e423
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94518003"
---
# <a name="tutorial-monitor-changes-and-update-a-linux-virtual-machine-in-azure"></a>Tutoriel : Superviser les changements et mettre à jour une machine virtuelle Linux dans Azure

Azure [Change Tracking](../../automation/change-tracking/overview.md) vous permet d’identifier facilement les changements, tandis que [Update Management](../../automation/update-management/overview.md) vous permet de gérer les mises à jour du système d’exploitation de vos machines virtuelles Linux Azure.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Gérer les mises à jour Linux
> * Surveiller les modifications et l’inventaire

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Ce tutoriel nécessite Azure CLI version 2.0.30 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-vm"></a>Créer une machine virtuelle

Pour voir les diagnostics et les métriques en action, vous avez besoin d’une machine virtuelle. Tout d’abord, créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). L’exemple suivant crée un groupe de ressources nommé *myResourceGroupMonitor* à l’emplacement *eastus*.

```azurecli-interactive
az group create --name myResourceGroupMonitor --location eastus
```

Créez maintenant une machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create). L’exemple suivant crée une machine virtuelle nommée *myVM* et génère des clés SSH si elles n’existent pas déjà dans *~/.ssh/* :

```azurecli-interactive
az vm create \
  --resource-group myResourceGroupMonitor \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="manage-software-updates"></a>Gérer les mises à jour logicielles

La gestion des mises à jour vous permet de gérer les mises à jour et les correctifs pour vos machines virtuelles Linux Azure.
Directement à partir de votre machine virtuelle, vous pouvez rapidement évaluer l’état des mises à jour disponibles, planifier l’installation des mises à jour nécessaires et passer en revue les résultats des déploiements pour vérifier que les mises à jour ont été appliquées correctement à la machine virtuelle.

Pour plus d’informations sur les prix, consultez [Tarification Automation pour la gestion des mises à jour](https://azure.microsoft.com/pricing/details/automation/)

### <a name="enable-update-management"></a>Activer la gestion des mises à jour

Activer la gestion des mises à jour pour votre machine virtuelle :

1. Sur le côté gauche de l’écran, sélectionnez **Machines virtuelles**.
2. Sélectionnez une machine virtuelle dans la liste.
3. Dans l’écran de la machine virtuelle, dans la section **Opérations**, sélectionnez **Gestion des mises à jour**. L’écran **Activer la gestion des mises à jour** s’ouvre.

Une validation est effectuée pour déterminer si la gestion des mises à jour est activée pour cette machine virtuelle.
La validation inclut la vérification de l’existence d’un espace de travail Log Analytics et d’un compte Automation lié, et si la solution est dans l’espace de travail.

Un espace de travail [Log Analytics](../../azure-monitor/log-query/log-query-overview.md) est utilisé pour collecter les données générées par les fonctionnalités et les services, comme la gestion des mises à jour.
L’espace de travail fournit un emplacement unique permettant de consulter et d’analyser les données provenant de plusieurs sources.
Pour effectuer une action supplémentaire sur les machines virtuelles qui nécessitent des mises à jour, Azure Automation vous permet d’exécuter des runbooks sur les machines virtuelles, par exemple télécharger et appliquer des mises à jour.

Le processus de validation vérifie également que la machine virtuelle est provisionnée avec l’agent Log Analytics et un Runbook Worker hybride Automation. Cet agent est utilisé pour communiquer avec la machine virtuelle et pour obtenir des informations sur l’état des mises à jour.

Choisissez l’espace de travail Log Analytics et le compte Automation, puis cliquez sur **Activer** pour activer la solution. L’activation de la solution prend jusqu’à 15 minutes.

Si l’intégration n’identifie pas l’un des prérequis suivants, il est automatiquement ajouté :

* Espace de travail [Log Analytics](../../azure-monitor/log-query/log-query-overview.md)
* [Compte Automation](../../automation/index.yml)
* Un [worker runbook hybride](../../automation/automation-hybrid-runbook-worker.md) est activé sur la machine virtuelle

L’écran **Gestion des mises à jour** s’ouvre. Configurez l’emplacement, l’espace de travail Log Analytics et le compte Automation à utiliser, puis cliquez sur **Activer**. Si les champs sont grisés, cela signifie qu’une autre solution d’automatisation est activée pour la machine virtuelle, et les mêmes espace de travail et compte Automation doivent être utilisés.

![Activer la solution de gestion des mises à jour](./media/tutorial-monitoring/manage-updates-update-enable.png)

L’activation de la solution peut prendre jusqu’à 15 minutes. Pendant ce temps, vous ne devez pas fermer la fenêtre du navigateur. Une fois la solution activée, des informations sur les mises à jour manquantes sur la machine virtuelle sont envoyées aux journaux Azure Monitor. Entre 30 minutes et 6 heures peuvent être nécessaires pour que les données soient disponibles pour l’analyse.

### <a name="view-update-assessment"></a>Afficher l’évaluation des mises à jour

Une fois la **gestion des mises à jour** activée, l’écran **Gestion des mises à jour** apparaît. Une fois l’évaluation des mises à jour terminée, vous pouvez afficher une liste des mises à jour manquantes sous l’onglet **Mises à jour manquantes**.

 ![Afficher l’état des mises à jour](./media/tutorial-monitoring/manage-updates-view-status-linux.png)

### <a name="schedule-an-update-deployment"></a>Planifier un déploiement de mises à jour

Pour installer les mises à jour, planifiez un déploiement qui suit votre fenêtre de planification et de maintenance des versions. Vous pouvez choisir les types de mises à jour à inclure dans le déploiement. Par exemple, vous pouvez inclure des mises à jour critiques ou de sécurité et exclure des correctifs cumulatifs.

Planifier un nouveau déploiement de mises à jour pour la machine virtuelle en cliquant sur **Planifier le déploiement de la mise à jour** en haut de l’écran **Gestion des mises à jour**. Dans l’écran **Nouveau déploiement de mises à jour**, spécifiez les informations suivantes :

Pour créer un déploiement de mises à jour, sélectionnez **Planifier le déploiement de la mise à jour**. La page **Nouveau déploiement de mises à jour** s’ouvre. Entrez les valeurs des propriétés décrites dans le tableau suivant, puis cliquez sur **Créer** :

| Propriété | Description |
| --- | --- |
| Nom |Nom unique identifiant le déploiement de mises à jour. |
|Système d’exploitation| Linux ou Windows|
| Groupes à mettre à jour |Pour des machines Azure, définissez une requête basée sur une combinaison de l’abonnement, des groupes de ressources, des emplacements et des étiquettes pour créer un groupe dynamique de machines virtuelles Azure à inclure dans votre déploiement. </br></br>Pour les machines non-Azure, sélectionnez une recherche existante enregistrée pour sélectionner un groupe de machines non-Azure à inclure dans le déploiement. </br></br>Pour plus d’informations, consultez [Groupes dynamiques](../../automation/update-management/configure-groups.md)|
| Ordinateurs à mettre à jour |Sélectionnez une recherche enregistrée, un groupe importé ou choisissez un ordinateur dans la liste déroulante, puis sélectionnez des ordinateurs individuels. Si vous choisissez **Machines**, l’état de préparation de la machine est indiqué dans la colonne **PRÉPARATION À LA MISE À JOUR DE L’AGENT**.</br> Pour en savoir plus sur les différentes méthodes de création de groupes d’ordinateurs dans les journaux Azure Monitor, consultez [Groupes d’ordinateurs dans les journaux Azure Monitor](../../azure-monitor/platform/computer-groups.md). |
|Classifications des mises à jour|Sélectionnez toutes les classifications des mises à jour dont vous avez besoin.|
|Inclure/exclure des mises à jour|La page **Inclure/Exclure** s’ouvre. Les mises à jour à inclure ou à exclure sont sous des onglets distincts. Pour plus d’informations sur la façon dont l’inclusion est gérée, consultez [Planifier un déploiement de mises à jour](../../automation/update-management/deploy-updates.md#schedule-an-update-deployment) |
|Paramètres de planification|Sélectionnez l’heure de début, puis la périodicité.|
| Préscripts + postscripts|Sélectionnez les scripts à exécuter avant et après votre déploiement|
| Fenêtre de maintenance |Nombre de minutes défini pour les mises à jour. La valeur ne peut pas être inférieure à 30 minutes ni supérieure à 6 heures |
| Contrôle du redémarrage| Détermine la façon dont doivent être gérés les redémarrages. Options disponibles :</br>Redémarrer si nécessaire (par défaut)</br>Toujours redémarrer</br>Ne jamais redémarrer</br>Redémarrer uniquement : les mises à jour ne sont pas installées|

Vous pouvez également créer des déploiements de mises à jour par programmation. Pour savoir comment créer un déploiement de mises à jour avec l’API REST, consultez [Configurations des mises à jour logicielles - Créer](/rest/api/automation/softwareupdateconfigurations/create). Vous pouvez également utiliser un exemple de runbook fourni pour créer un déploiement de mises à jour hebdomadaires. Pour en savoir plus sur ce runbook, consultez [Créer un déploiement de mises à jour hebdomadaires pour une ou plusieurs machines virtuelles dans un groupe de ressources](https://gallery.technet.microsoft.com/scriptcenter/Create-a-weekly-update-2ad359a1).

Une fois que vous avez terminé la configuration de la planification, cliquez sur le bouton **Créer**. Vous revenez ensuite au tableau de bord des états.
Notez que le tableau **Planifié** montre la planification de déploiement que vous avez créée.

### <a name="view-results-of-an-update-deployment"></a>Afficher les résultats d’un déploiement de mises à jour

Une fois que le déploiement planifié démarre, vous pouvez voir l’état de ce déploiement sous l’onglet **Déploiements des mises à jour** dans l’écran **Gestion des mises à jour**.
S’il est en cours d’exécution, son état est **En cours d’exécution**. Une fois le déploiement terminé, s’il est réussi, l’état passe à **Réussi**.
S’il existe un échec avec une ou plusieurs mises à jour dans le déploiement, l’état est **Échec partiel**.
Sélectionnez le déploiement des mises à jour terminé pour voir le tableau de bord dédié à ce déploiement de mises à jour.

![Tableau de bord des états de déploiement des mises à jour pour un déploiement spécifique](./media/tutorial-monitoring/manage-updates-view-results.png)

Dans la vignette **Résultats des mises à jour**, vous pouvez voir un récapitulatif du nombre total de mises à jour et les résultats du déploiement sur la machine virtuelle.
Dans le tableau de droite se trouve une répartition détaillée de chaque mise à jour et les résultats de l’installation, soit l’une des valeurs suivantes :

* **Aucune tentative effectuée** : la mise à jour n’a pas été installée, car le temps disponible était insuffisant d’après la durée définie pour la fenêtre de maintenance.
* **Réussi** : la mise à jour a réussi
* **Échec** : la mise à jour a échoué

Pour afficher toutes les entrées de journal d’activité créées par le déploiement, sélectionnez **Tous les journaux d’activité**.

Cliquez sur la vignette **Sortie** pour voir le flux des tâches du runbook chargé de gérer le déploiement des mises à jour sur la machine virtuelle cible.

Pour afficher les informations détaillées sur les erreurs du déploiement, sélectionnez **Erreurs**.

## <a name="monitor-changes-and-inventory"></a>Surveiller les modifications et l’inventaire

Vous pouvez collecter et afficher l’inventaire des logiciels, fichiers, démons Linux, services Windows et clés de Registre Windows présents sur vos ordinateurs. Le suivi des configurations de vos machines peut vous aider à identifier les problèmes opérationnels au sein de votre environnement et à mieux comprendre l’état de vos machines.

### <a name="enable-change-and-inventory-management"></a>Activer la gestion des modifications et de l’inventaire

Activez la gestion des modifications et de l’inventaire pour votre machine virtuelle :

1. Sur le côté gauche de l’écran, sélectionnez **Machines virtuelles**.
2. Sélectionnez une machine virtuelle dans la liste.
3. Sur l’écran de la machine virtuelle, dans la section **Opérations**, sélectionnez **Inventaire** ou **Suivi des modifications**. L’écran **Enable Change Tracking and Inventory** (Activer le suivi des modifications et inventaire) s’ouvre.

Configurez l’emplacement, l’espace de travail Log Analytics et le compte Automation à utiliser, puis cliquez sur **Activer**. Si les champs sont grisés, cela signifie qu’une autre solution d’automatisation est activée pour la machine virtuelle, et les mêmes espace de travail et compte Automation doivent être utilisés. Même si les solutions sont distinctes dans le menu, elles forment une même solution. Le fait d’en activer une active les deux pour votre machine virtuelle.

![Activer le suivi des modifications et de l’inventaire](./media/tutorial-monitoring/manage-inventory-enable.png)

Une fois la solution activée, un délai peut être nécessaire pour la collecte de l’inventaire sur la machine virtuelle avant l’affichage de données.

### <a name="track-changes"></a>Suivre les modifications

Sur votre machine virtuelle, sélectionnez **Suivi des modifications** sous **OPÉRATIONS**. Cliquez sur **Modifier les paramètres**, la page **Suivi des modifications** s’affiche. Sélectionnez le type de paramètre que vous souhaitez suivre, puis cliquez sur **+ Ajouter** pour configurer les paramètres. L’option disponible pour Linux est **Fichiers Linux**

Pour plus d’informations sur Change Tracking, consultez [Dépanner les modifications apportées à votre environnement](../../automation/automation-tutorial-troubleshoot-changes.md)

### <a name="view-inventory"></a>Afficher l’inventaire

Sur votre machine virtuelle, sélectionnez **Inventaire** sous **OPÉRATIONS**. Dans l’onglet **Logiciels**, un tableau dresse la liste des logiciels détectés. Les informations détaillées sur chaque enregistrement de logiciel sont visibles dans le tableau. Ces détails incluent le nom du logiciel, la version, l’éditeur, l’heure de la dernière actualisation.

![Afficher l’inventaire](./media/tutorial-monitoring/inventory-view-results.png)

### <a name="monitor-activity-logs-and-changes"></a>Surveiller les journaux d’activité et les modifications

À partir de la page **Suivi des modifications** sur votre machine virtuelle, sélectionnez **Manage Activity Log Connection** (Gérer la connexion du journal d’activité). Cette tâche ouvre la page **Journal des activités Azure**. Sélectionnez **Connexion** pour connecter le suivi des modifications au journal des activités Azure pour votre machine virtuelle.

Une fois ce paramètre activé, accédez à la page **Vue d’ensemble** de votre machine virtuelle, puis sélectionnez **Arrêter** pour l’arrêter. Lorsque vous y êtes invité, sélectionnez **Oui** pour arrêter la machine virtuelle. Celle-ci étant libérée, sélectionnez **Démarrer** pour redémarrer votre machine virtuelle.

Le fait d’arrêter et de démarrer une machine virtuelle enregistre un événement dans son journal d’activité. Revenez à la page **Suivi des modifications**. Sélectionnez l’onglet **Événements** au bas de la page. Après un certain temps, les événements apparaissent dans le graphique et le tableau. Chaque événement peut être sélectionné pour en afficher le détail.

![Afficher les modifications dans le journal d’activité](./media/tutorial-monitoring/manage-activitylog-view-results.png)

Le graphique affiche les modifications qui se sont produites au fil du temps. Après avoir ajouté une connexion au journal d’activité, le graphique linéaire dans la partie supérieure affiche les événements du journal des activités Azure. Chacune des lignes des graphiques à barres représente un autre type de modification dont il est possible d’effectuer le suivi. Ces types sont les démons Linux, les fichiers et les logiciels. L’onglet Modification présente le détail des modifications indiquées dans la visualisation, par ordre décroissant, c’est-à-dire le plus récent en premier.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré et passé en revue les solutions Change Tracking et Update Management pour votre machine virtuelle. Vous avez appris à :

> [!div class="checklist"]
> * Créer un groupe de ressources et une machine virtuelle
> * Gérer les mises à jour Linux
> * Surveiller les modifications et l’inventaire

Passez au tutoriel suivant pour en savoir plus sur la supervision de votre machine virtuelle.

> [!div class="nextstepaction"]
> [Surveiller les machines virtuelles](tutorial-monitor.md)

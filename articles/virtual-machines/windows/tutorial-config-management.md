---
title: Tutoriel - Gérer la configuration d’une machine virtuelle Windows dans Azure
description: Dans ce tutoriel, vous allez apprendre à identifier les changements et à gérer les mises à jour de packages d’une machine virtuelle Windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 12/05/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 8d6e10b025b9e9524982b1558beacfab1970eb59
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92216434"
---
# <a name="tutorial-monitor-changes-and-update-a-windows-virtual-machine-in-azure"></a>Tutoriel : Superviser les changements et mettre à jour une machine virtuelle Windows dans Azure

Avec Azure [Change Tracking](../../automation/change-tracking/overview.md) et [Update Management](../../automation/update-management/overview.md), vous pouvez facilement identifier les modifications apportées à vos machines virtuelles Windows dans Azure, et gérer les mises à jour du système d’exploitation pour ces machines virtuelles.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Gérer les mises à jour Windows
> * Superviser les modifications et l’inventaire

## <a name="open-azure-cloud-shell"></a>Ouvrir Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants qui sont préinstallés et configurés en vue d’être utilisés avec votre compte.

Pour ouvrir un bloc de code Cloud Shell, sélectionnez **Essayer** en haut à droite d’un bloc de code.

Vous pouvez également ouvrir Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les sous l’onglet Cloud Shell, puis sélectionnez la touche Entrée pour exécuter le code.

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Pour configurer la gestion de la surveillance et de la mise à jour Azure dans ce didacticiel, vous avez besoin d’une machine virtuelle Windows Azure.

Tout d’abord, définissez un nom d’utilisateur administrateur et un mot de passe pour la machine virtuelle avec [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) :

```azurepowershell-interactive
$cred = Get-Credential
```

Ensuite, créez la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant crée une machine virtuelle nommée `myVM` à l’emplacement `East US`. S’ils n’existent pas déjà, le groupe de ressources `myResourceGroupMonitor` et les ressources réseau qui lui sont associées sont créés :

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupMonitor" `
    -Name "myVM" `
    -Location "East US" `
    -Credential $cred
```

Quelques minutes sont nécessaires à la création des ressources et de la machine virtuelle.

## <a name="manage-windows-updates"></a>Gérer les mises à jour Windows

Update Management vous permet de gérer les mises à jour et les correctifs pour vos machines virtuelles Windows Azure. Sur votre machine virtuelle, vous pouvez directement :

- Évaluer l’état des mises à jour disponibles.
- Planifier l’installation des mises à jour requises.
- Passer en revue les résultats du déploiement pour vérifier que les mises à jour ont été correctement appliquées à la machine virtuelle

Pour plus d’informations sur les prix, consultez [Tarification Automation pour Update Management](https://azure.microsoft.com/pricing/details/automation/).

### <a name="enable-update-management"></a>Activer Update Management

Pour activer Update Management sur votre machine virtuelle :

1. Tout à gauche de la fenêtre, sélectionnez **Machines virtuelles**.
1. Choisissez une machine virtuelle dans la liste.
1. Dans le volet **Opérations** de la fenêtre de la machine virtuelle, sélectionnez **Update Management**.
1. La fenêtre **Activer la gestion des mises à jour** s’ouvre.

Une vérification est effectuée pour déterminer si Update Management est activé pour cette machine virtuelle. Cette vérification recherche la présence d’un espace de travail Log Analytics et d’un compte Automation lié, et détermine si la solution se trouve dans l’espace de travail.

Vous pouvez utiliser un espace de travail [Log Analytics](../../azure-monitor/log-query/log-query-overview.md) pour collecter les données générées par les fonctionnalités et les services, comme Update Management. L’espace de travail fournit un emplacement unique permettant de consulter et d’analyser les données provenant de plusieurs sources.

Pour effectuer des actions supplémentaires sur les machines virtuelles qui nécessitent des mises à jour, vous pouvez utiliser Azure Automation afin d’exécuter des runbooks sur les machines virtuelles. Ces actions impliquent le téléchargement ou l’application de mises à jour.

Le processus de validation vérifie également que la machine virtuelle est provisionnée avec le Microsoft Monitoring Agent (MMA) et un runbook Worker hybride Automation. Vous pouvez utiliser cet agent pour communiquer avec la machine virtuelle et pour obtenir des informations sur l’état des mises à jour.

Dans la fenêtre **Activer la gestion des mises à jour**, choisissez l’espace de travail Log Analytics et le compte Automation, puis cliquez sur **Activer**. L’activation de la solution prend jusqu’à 15 minutes.

Si l’un des prérequis suivants n’est pas détecté lors de l’intégration, il est automatiquement ajouté :

* Espace de travail [Log Analytics](../../azure-monitor/log-query/log-query-overview.md)
* [Automation](../../automation/index.yml)
* [Worker runbook hybride](../../automation/automation-hybrid-runbook-worker.md) activé sur la machine virtuelle

Une fois la solution activée, la fenêtre **Update Management** s’ouvre. Configurez l’emplacement, l’espace de travail Log Analytics et le compte Automation à utiliser, puis cliquez sur **Activer**. Si les champs sont grisés, cela signifie qu’une autre solution d’automatisation est activée pour la machine virtuelle, et donc, que vous devez utiliser l’espace de travail et le compte Automation de cette solution.

![Activer la solution Update Management](./media/tutorial-monitoring/manageupdates-update-enable.png)

L’activation de la solution Update Management peut prendre jusqu’à 15 minutes. Pendant ce temps, ne fermez pas la fenêtre du navigateur. Une fois la solution activée, des informations sur les mises à jour manquantes sur la machine virtuelle sont envoyées aux journaux Azure Monitor. Entre 30 minutes et 6 heures peuvent être nécessaires pour que les données soient disponibles en vue d’être analysées.

### <a name="view-an-update-assessment"></a>Afficher une évaluation des mises à jour

Une fois qu’Update Management est activé, la fenêtre **Update management** s’affiche. Une fois l’évaluation des mises à jour terminée, vous pouvez afficher la liste des mises à jour manquantes sous l’onglet **Mises à jour manquantes**.

 ![Afficher l’état des mises à jour](./media/tutorial-monitoring/manageupdates-view-status-win.png)

### <a name="schedule-an-update-deployment"></a>Planifier un déploiement de mises à jour

Pour installer les mises à jour, planifiez un déploiement qui suit votre fenêtre de planification et de maintenance des versions. Vous pouvez choisir les types de mises à jour à inclure dans le déploiement. Par exemple, vous pouvez inclure des mises à jour critiques ou de sécurité et exclure des correctifs cumulatifs.

Pour planifier un nouveau déploiement de mises à jour pour la machine virtuelle, sélectionnez **Planifier le déploiement de la mise à jour** en haut de la fenêtre **Update management**. Dans la fenêtre **Nouveau déploiement de mises à jour**, spécifiez les informations suivantes :

| Option | Description |
| --- | --- |
| **Nom** |entrez un nom unique pour identifier le déploiement de mises à jour. |
|**Système d’exploitation**| Sélectionnez **Linux** ou **Windows**.|
| **Groupes à mettre à jour** |Pour les machines virtuelles hébergées dans Azure, définissez une requête basée sur un ensemble constitué d’un abonnement, de groupes de ressources, d’emplacements et d’étiquettes. Cette requête crée un groupe dynamique comprenant les machines virtuelles hébergées dans Azure qui doivent être incluses dans votre déploiement. </br></br>Pour les machines virtuelles qui ne sont pas hébergées dans Azure, sélectionnez une recherche enregistrée. Avec cette recherche, vous pouvez sélectionner un groupe de machines virtuelles à inclure dans le déploiement. </br></br> Pour plus d’informations, consultez [Groupes dynamiques](../../automation/update-management/configure-groups.md).|
| **Machines à mettre à jour** |Sélectionnez **Recherche enregistrée**, **Groupe importé** ou **Machines**.<br/><br/>Si vous sélectionnez **Machines**, vous pouvez choisir des machines dans la liste déroulante. L’état de préparation de chaque machine est indiqué dans la colonne **Préparation de la mise à jour de l’agent** de la table.</br></br> Pour en savoir plus sur les différentes méthodes de création de groupes d’ordinateurs dans les journaux Azure Monitor, consultez [Groupes d’ordinateurs dans les journaux Azure Monitor](../../azure-monitor/platform/computer-groups.md). |
|**Classifications des mises à jour**|Choisissez toutes les classifications de mise à jour nécessaires.|
|**Inclure/Exclure des mises à jour**|Sélectionnez cette option pour ouvrir le volet **Inclure/Exclure**. Les mises à jour à inclure et à exclure se trouvent sous deux onglets différents. Pour plus d’informations sur la façon dont l’inclusion est gérée, consultez [Planifier un déploiement de mises à jour](../../automation/update-management/deploy-updates.md#schedule-an-update-deployment). |
|**Paramètres de planification**|Sélectionnez l’heure de début, puis sélectionnez **Une fois** ou **Récurrent**.|
| **Préscripts + postscripts**|Choisissez les scripts à exécuter avant et après votre déploiement.|
| **Fenêtre de maintenance** | Entrez le nombre de minutes défini pour les mises à jour. La plage de valeurs valide est comprise entre 30 et 360 minutes. |
| **Contrôle du redémarrage**| Sélectionnez le mode de gestion des redémarrages. Les sélections disponibles sont les suivantes :<ul><li>**Redémarrer si nécessaire**</li><li>**Toujours redémarrer**</li><li>**Ne jamais redémarrer**</li><li>**Redémarrer uniquement**</li></ul>**Redémarrer si nécessaire** est sélectionné par défaut. Si vous sélectionnez **Redémarrer uniquement**, les mises à jour ne sont pas installées.|

Une fois que vous avez terminé la configuration de la planification, cliquez sur **Créer** pour revenir au tableau de bord des états. Le tableau **Planifié** indique la planification de déploiement que vous avez créée.

Vous pouvez également créer des déploiements de mises à jour par programmation. Pour savoir comment créer un déploiement de mises à jour avec l’API REST, consultez [Configurations des mises à jour logicielles - Créer](/rest/api/automation/softwareupdateconfigurations/create). Vous pouvez également utiliser un exemple de runbook fourni pour créer un déploiement de mises à jour hebdomadaires. Pour en savoir plus sur ce runbook, consultez [Créer un déploiement de mises à jour hebdomadaires pour une ou plusieurs machines virtuelles dans un groupe de ressources](https://gallery.technet.microsoft.com/scriptcenter/Create-a-weekly-update-2ad359a1).

### <a name="view-results-of-an-update-deployment"></a>Afficher les résultats d’un déploiement de mises à jour

Une fois que le déploiement planifié démarre, vous pouvez voir l’état de ce déploiement sous l’onglet **Déploiements des mises à jour** de la fenêtre **Update management**.

Si le déploiement est en cours d’exécution, son état est « En cours ». Une fois l’opération terminée, l’état passe à « Réussi ». Toutefois, si une ou plusieurs mises à jour du déploiement échouent, l’état passe à « Échec partiel ».

Sélectionnez le déploiement de mises à jour terminé pour voir le tableau de bord qui lui est consacré.

![Tableau de bord des états de déploiement des mises à jour pour un déploiement spécifique](./media/tutorial-monitoring/manageupdates-view-results.png)

La vignette **Résultats des mises à jour** affiche un récapitulatif du nombre total de mises à jour et les résultats du déploiement sur la machine virtuelle. Le tableau de droite montre une répartition détaillée de chaque mise à jour, ainsi que les résultats de l’installation. Chaque résultat doit avoir l’une des valeurs suivantes :

* **Aucune tentative effectuée** : La mise à jour n’a pas été installée. Le temps disponible n’était pas suffisant pour la durée définie de la fenêtre de maintenance.
* **Opération réussie** : la mise à jour a réussi.
* **Échec** : la mise à jour a échoué.

Pour afficher toutes les entrées de journal d’activité créées par le déploiement, sélectionnez **Tous les journaux d’activité**.

Sélectionnez la vignette **Sortie** pour voir le flux des tâches du runbook chargé de gérer le déploiement des mises à jour sur la machine virtuelle cible.

Pour afficher les informations détaillées sur les erreurs du déploiement, sélectionnez **Erreurs**.

## <a name="monitor-changes-and-inventory"></a>Surveiller les modifications et l’inventaire

Vous pouvez collecter et afficher l’inventaire des logiciels, fichiers, démons Linux, services Windows et clés de Registre Windows présents sur vos ordinateurs. Le suivi des configurations de vos machines peut vous aider à identifier les problèmes opérationnels au sein de votre environnement et à mieux comprendre l’état de vos machines.

### <a name="enable-change-and-inventory-management"></a>Activer la gestion des modifications et de l’inventaire

Pour activer la gestion des modifications et de l’inventaire pour votre machine virtuelle :

1. Tout à gauche de la fenêtre, sélectionnez **Machines virtuelles**.
1. Choisissez une machine virtuelle dans la liste.
1. Dans la fenêtre de la machine virtuelle, sous **Opérations**, sélectionnez **Inventaire** ou **Suivi des modifications**.
1. Le volet **Activer le suivi des modifications et l’inventaire** s’ouvre.

Configurez l’emplacement, l’espace de travail Log Analytics et le compte Automation à utiliser, puis cliquez sur **Activer**. Si les options sont grisées, cela signifie qu’une solution d’automatisation est déjà activée pour la machine virtuelle. Dans ce cas, vous devez utiliser l’espace de travail et le compte Automation qui sont déjà activés.

Même si les solutions sont séparées dans le menu, elles forment une même solution. Le fait d’en activer une active les deux pour votre machine virtuelle.

![Activer le suivi des modifications et de l’inventaire](./media/tutorial-monitoring/manage-inventory-enable.png)

Une fois la solution activée, un délai peut être nécessaire avant que l’inventaire ne soit effectué sur la machine virtuelle et que les données ne s’affichent.

### <a name="track-changes"></a>Suivre les modifications

Sur votre machine virtuelle, sous **Opérations**, sélectionnez **Suivi des modifications**, puis sélectionnez **Modifier les paramètres**. Le volet **Suivi des modifications** s’ouvre. Sélectionnez le type de paramètre que vous souhaitez suivre, puis cliquez sur **+ Ajouter** pour configurer les paramètres.

Les paramètres disponibles pour Windows sont les suivants :

* Registre Windows
* Fichiers Windows

Pour plus d’informations sur le suivi des modifications, consultez [Résoudre les modifications sur une machine virtuelle](../../automation/automation-tutorial-troubleshoot-changes.md).

### <a name="view-inventory"></a>Afficher l’inventaire

Sur votre machine virtuelle, sélectionnez **Inventaire** sous **OPÉRATIONS**. Sous l’onglet **Logiciels**, un tableau dresse la liste des logiciels détectés. Les informations concernant chaque enregistrement de logiciel figurent dans le tableau. Ces informations incluent le nom du logiciel, la version, l’éditeur et l’heure de la dernière actualisation.

![Afficher l’inventaire](./media/tutorial-monitoring/inventory-view-results.png)

### <a name="monitor-activity-logs-and-changes"></a>Superviser les journaux d’activité et les modifications

Dans la fenêtre **Suivi des modifications** de votre machine virtuelle, sélectionnez **Gérer la connexion du journal d’activité** pour ouvrir le volet **Journal d’activité Azure**. Sélectionnez **Connexion** afin de connecter Change Tracking au journal des activités Azure pour votre machine virtuelle.

Une fois Change Tracking activé, accédez au volet **Vue d’ensemble** de votre machine virtuelle, puis sélectionnez **Arrêter** pour l’arrêter. Lorsque vous y êtes invité, sélectionnez **Oui** pour arrêter la machine virtuelle. Une fois la machine virtuelle libérée, sélectionnez **Démarrer** pour redémarrer votre machine virtuelle.

Le fait d’arrêter et de démarrer une machine virtuelle journalise un événement dans son journal d’activité. Revenez au volet **Suivi des modifications**, puis sélectionnez l’onglet **Événements** au bas du volet. Après un certain temps, les événements s’affichent dans le graphique et le tableau. Vous pouvez sélectionner chaque événement pour afficher des informations détaillées le concernant.

![Afficher les modifications dans le journal d’activité](./media/tutorial-monitoring/manage-activitylog-view-results.png)

Le graphique précédent affiche les modifications qui ont eu lieu. Après avoir ajouté une connexion au journal d’activité Azure, le graphique linéaire dans la partie supérieure affiche les événements du journal d’activité Azure.

Chacune des lignes des graphiques à barres représente un type de modification différent dont il est possible d’effectuer le suivi. Ces types sont les démons Linux, les fichiers, les clés de Registre Windows, les logiciels et les services Windows. L’onglet **Modifier** affiche le détail des modifications. Les modifications s’affichent dans l’ordre où elles ont été effectuées, de la plus récente à la plus ancienne.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré et passé en revue les solutions Change Tracking et Update Management pour votre machine virtuelle. Vous avez appris à :

> [!div class="checklist"]
> * Créez un groupe de ressources et une machine virtuelle.
> * Gérer les mises à jour Windows
> * Superviser les modifications et l’inventaire

Passez au tutoriel suivant pour en savoir plus sur la supervision de votre machine virtuelle.

> [!div class="nextstepaction"]
> [Surveiller les machines virtuelles](tutorial-monitor.md)

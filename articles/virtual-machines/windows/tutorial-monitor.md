---
title: Tutoriel - Superviser des machines virtuelles Windows dans Azure
description: Dans ce tutoriel, vous allez apprendre à superviser le niveau de performance ainsi que les composants d’application découverts qui s’exécutent sur vos machines virtuelles Windows.
author: mgoedtel
manager: carmonm
ms.service: virtual-machines-windows
ms.subservice: monitoring
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 09/27/2018
ms.author: magoedte
ms.custom: mvc
ms.openlocfilehash: 154e4c9421d1c0a54b3d9b5f53424e7640fe62d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87323568"
---
# <a name="tutorial-monitor-a-windows-virtual-machine-in-azure"></a>Tutoriel : Superviser une machine virtuelle Windows dans Azure

La supervision Azure utilise des agents pour collecter les données de démarrage et de performance des machines virtuelles Azure, stocker ces données dans le stockage Azure et les rendre accessibles via le portail, le module Azure PowerShell et Azure CLI. La supervision avancée est fournie avec Azure Monitor pour machines virtuelles à travers la collecte des métriques de performance, la découverte des composants d’applications installés sur la machine virtuelle ainsi que des graphiques de performance et une carte de dépendances.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Activer les diagnostics de démarrage sur une machine virtuelle
> * Afficher les diagnostics de démarrage
> * Afficher les métriques de l’hôte de machine virtuelle
> * Activer Azure Monitor pour machines virtuelles
> * Voir les métriques de performances d’une machine virtuelle
> * Créer une alerte

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="create-virtual-machine"></a>Créer une machine virtuelle

Pour configurer la gestion de la surveillance et de la mise à jour Azure dans ce didacticiel, vous avez besoin d’une machine virtuelle Windows Azure. Tout d’abord, définissez un nom d’utilisateur administrateur et un mot de passe pour la machine virtuelle avec [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) :

```azurepowershell-interactive
$cred = Get-Credential
```

Créez à présent la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant permet de créer une machine virtuelle nommée *myVM* dans l’emplacement *EastUS*. S’ils n’existent pas déjà, le groupe de ressources *myResourceGroupMonitorMonitor* et les ressources réseau prises en charge sont créés :

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupMonitor" `
    -Name "myVM" `
    -Location "East US" `
    -Credential $cred
```

Quelques minutes sont nécessaires à la création des ressources et de la machine virtuelle.

## <a name="view-boot-diagnostics"></a>Afficher les diagnostics de démarrage

Au démarrage des machines virtuelles Windows, l’agent de diagnostic de démarrage capture le résultat à l’écran. Celui-ci peut être utilisé pour résoudre certains problèmes. Cette fonctionnalité est activée par défaut. Les captures d’écran sont stockées dans un compte de stockage Azure, qui est également créé par défaut.

Vous pouvez obtenir les données de diagnostic de démarrage avec la commande [Get-AzureRmVMBootDiagnosticsData](/powershell/module/az.compute/get-azvmbootdiagnosticsdata). Dans l’exemple suivant, les diagnostics de démarrage sont téléchargés à la racine du lecteur *c:\*.

```powershell
Get-AzVMBootDiagnosticsData -ResourceGroupName "myResourceGroupMonitor" -Name "myVM" -Windows -LocalPath "c:\"
```

## <a name="view-host-metrics"></a>Afficher les métriques de l’hôte

Une machine virtuelle Windows possède une machine virtuelle hôte dédiée dans Azure, avec qui elle interagit. Les métriques sont automatiquement collectées pour l’hôte et peuvent être visualisées dans le portail Azure.

1. Dans le portail Azure, cliquez sur **Groupes de ressources**, sélectionnez **myResourceGroupMonitor** puis sélectionnez **myVM** dans la liste des ressources.
2. Pour voir comment la machine virtuelle hôte fonctionne, cliquez sur **Métriques** dans le panneau de la machine virtuelle, puis sélectionnez une des métriques de l’hôte sous **Métriques disponibles**.

    ![Afficher les métriques de l’hôte](./media/tutorial-monitoring/tutorial-monitor-host-metrics.png)

## <a name="enable-advanced-monitoring"></a>Activer la supervision avancée

Pour activer la supervision de votre machine virtuelle Azure avec Azure Monitor pour machines virtuelles :

1. Dans le portail Azure, cliquez sur **Groupes de ressources**, sélectionnez **myResourceGroupMonitor** puis sélectionnez **myVM** dans la liste des ressources.

2. Sur la page de la machine virtuelle, dans la section **Surveillance**, sélectionnez **Insights (préversion)** .

3. Sur la page **Insights (préversion)** , sélectionnez **Essayer maintenant**.

    ![Activer Azure Monitor pour machines virtuelles pour une machine virtuelle](../../azure-monitor/insights/media/vminsights-enable-single-vm/enable-vminsights-vm-portal.png)

4. Dans la page **Intégration des insights de Azure Monitor**, si vous disposez d’un espace de travail Log Analytics appartenant au même abonnement, sélectionnez-le dans la liste déroulante.  

    La liste présélectionne l’espace de travail par défaut et l’emplacement où la machine virtuelle est déployée dans l’abonnement. 

    >[!NOTE]
    >Pour créer un espace de travail Log Analytics pour stocker les données de surveillance de la machine virtuelle, consultez [Créer un espace de travail Log Analytics](../../azure-monitor/learn/quick-create-workspace.md). L’espace de travail doit appartenir à l’une des [régions prises en charge](../../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).

Après avoir activé la supervision, vous devrez peut-être attendre plusieurs minutes avant de pouvoir voir les métriques de performances de la machine virtuelle.

![Activer Azure Monitor pour machines virtuelles surveillant le traitement du déploiement](../../azure-monitor/insights/media/vminsights-enable-single-vm/onboard-vminsights-vm-portal-status.png)

## <a name="view-vm-performance-metrics"></a>Voir les métriques de performances d’une machine virtuelle

Azure Monitor pour les machines virtuelles comprend un ensemble de graphiques de performances qui ciblent divers indicateurs de performance clés (KPI) pour vous aider à déterminer l’intégrité du fonctionnement d’une machine virtuelle. Pour y accéder à partir de votre machine virtuelle, procédez comme suit.

1. Dans le portail Azure, cliquez sur **Groupes de ressources**, sélectionnez **myResourceGroupMonitor** puis sélectionnez **myVM** dans la liste des ressources.

2. Sur la page de la machine virtuelle, dans la section **Surveillance**, sélectionnez **Insights (préversion)** .

3. Sélectionnez l’onglet **Performances**.

En plus d’inclure des graphiques d’utilisation de performances, cette page fournit un tableau affichant pour chaque disque logique détecté, sa capacité, son utilisation et la moyenne totale pour chaque mesure.

## <a name="create-alerts"></a>Créez des alertes

Vous pouvez créer des alertes en fonction de métriques de performances spécifiques. Les alertes peuvent être utilisées par exemple pour notifier que l’utilisation moyenne de l’UC dépasse un certain seuil ou que l’espace disque disponible est inférieur à une certaine quantité. Les alertes sont affichées dans le portail Azure ou peuvent être envoyées par e-mail. Vous pouvez également déclencher des runbooks Azure Automation ou Azure Logic Apps en réponse aux alertes générées.

L’exemple suivant crée une alerte pour l’utilisation moyenne de l’UC.

1. Dans le portail Azure, cliquez sur **Groupes de ressources**, sélectionnez **myResourceGroupMonitor** puis sélectionnez **myVM** dans la liste des ressources.

2. Cliquez sur **Règles d’alerte** dans le panneau de la machine virtuelle puis cliquez sur **Ajouter une alerte Métrique** dans la partie supérieure du panneau des alertes.

3. Spécifiez un **Nom** pour votre alerte, comme *myAlertRule*

4. Pour déclencher une alerte quand le pourcentage d’UC dépasse 1,0 pendant cinq minutes, laissez toutes les autres valeurs par défaut sélectionnées.

5. Si vous le souhaitez, cochez la case *Envoyer un e-mail aux propriétaires, aux contributeurs et aux lecteurs* pour envoyer la notification par e-mail. L’action par défaut est de présenter une notification dans le portail.

6. Cliquez sur le bouton **OK** .

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré et visualisé le niveau de performance de votre machine virtuelle. Vous avez appris à :

> [!div class="checklist"]
> * Créer un groupe de ressources et une machine virtuelle
> * Activer les diagnostics de démarrage sur la machine virtuelle
> * Afficher les diagnostics de démarrage
> * Afficher les métriques de l’hôte
> * Activer Azure Monitor pour machines virtuelles
> * Afficher les métriques de la machine virtuelle
> * Créer une alerte

Passez au didacticiel suivant pour découvrir plus d’informations sur Azure Security Center.

> [!div class="nextstepaction"]
> [Gérer la sécurité des machines virtuelles](../../security/fundamentals/overview.md)

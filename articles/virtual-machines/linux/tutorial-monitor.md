---
title: 'Tutoriel : Superviser des machines virtuelles Linux dans Azure'
description: Dans ce tutoriel, vous allez apprendre à superviser les performances ainsi que les composants d’application découverts qui s’exécutent sur vos machines virtuelles Linux.
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
ms.date: 09/30/2019
ms.author: magoedte
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: a21df6e5f8d437415bb5376969d56d26153b5c5f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500459"
---
# <a name="tutorial-monitor-a-linux-virtual-machine-in-azure"></a>Tutoriel : Superviser une machine virtuelle Linux dans Azure

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

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, ce didacticiel nécessite que vous exécutiez Azure CLI version 2.0.30 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="create-vm"></a>Créer une machine virtuelle

Pour voir les diagnostics et les métriques en action, vous avez besoin d’une machine virtuelle. Tout d’abord, créez un groupe de ressources avec la commande [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create). L’exemple suivant crée un groupe de ressources nommé *myResourceGroupMonitor* à l’emplacement *eastus*.

```azurecli-interactive
az group create --name myResourceGroupMonitor --location eastus
```

Créez maintenant une machine virtuelle avec la commande [az vm create](/cli/azure/vm?view=azure-cli-latest#az-vm-create). L’exemple suivant crée une machine virtuelle nommée *myVM* et génère des clés SSH si elles n’existent pas déjà dans *~/.ssh/* :

```azurecli-interactive
az vm create \
  --resource-group myResourceGroupMonitor \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="enable-boot-diagnostics"></a>Activer les diagnostics de démarrage

Au démarrage des machines virtuelles Linux, l’extension Diagnostics de démarrage capture la sortie du démarrage et la stocke dans le stockage Azure. Ces données peuvent être utilisées pour résoudre les problèmes de démarrage des machines virtuelles. Les diagnostics de démarrage ne sont pas activés automatiquement quand vous créez une machine virtuelle Linux à l’aide d’Azure CLI.

Avant d’activer les diagnostics de démarrage, vous devez créer un compte de stockage pour stocker les journaux d’activité de démarrage. Les comptes de stockage doivent avoir un nom global unique, comprenant entre 3 et 24 caractères, et contenant seulement des chiffres et des lettres minuscules. Créez un compte de stockage avec la commande [az storage account create](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create). Dans cet exemple, une chaîne aléatoire est utilisée pour créer un nom de compte de stockage unique.

```azurecli-interactive
storageacct=mydiagdata$RANDOM

az storage account create \
  --resource-group myResourceGroupMonitor \
  --name $storageacct \
  --sku Standard_LRS \
  --location eastus
```

Lors de l’activation des diagnostics de démarrage, l’URI vers le conteneur de stockage d’objets blob est nécessaire. La commande suivante interroge le compte de stockage et retourne cet URI. La valeur de l’URI est stockée dans une variable nommée *bloburi*, qui est utilisé à l’étape suivante.

```azurecli-interactive
bloburi=$(az storage account show --resource-group myResourceGroupMonitor --name $storageacct --query 'primaryEndpoints.blob' -o tsv)
```

Activez maintenant les diagnostics de démarrage avec la commande [az vm boot-diagnostics enable](/cli/azure/vm/boot-diagnostics#az-vm-boot-diagnostics-enable). La valeur de `--storage` est l’URI de l’objet blob collecté à l’étape précédente.

```azurecli-interactive
az vm boot-diagnostics enable \
  --resource-group myResourceGroupMonitor \
  --name myVM \
  --storage $bloburi
```

## <a name="view-boot-diagnostics"></a>Afficher les diagnostics de démarrage

Quand les diagnostics de démarrage sont activés, chaque fois que vous arrêtez et que vous démarrez la machine virtuelle, des informations sur le processus de démarrage sont écrites dans un fichier journal. Pour cet exemple, désallouez d’abord la machine virtuelle avec la commande [az vm deallocate](/cli/azure/vm?view=azure-cli-latest#az-vm-deallocate) comme suit :

```azurecli-interactive
az vm deallocate --resource-group myResourceGroupMonitor --name myVM
```

Démarrez maintenant la machine virtuelle avec la commande [az vm start](/cli/azure/vm?view=azure-cli-latest#az-vm-start) comme suit :

```azurecli-interactive
az vm start --resource-group myResourceGroupMonitor --name myVM
```

Vous pouvez obtenir les données des diagnostics de démarrage pour *myVM* avec la commande [az vm boot-diagnostics get-boot-log](/cli/azure/vm/boot-diagnostics#az-vm-boot-diagnostics-get-boot-log) comme suit :

```azurecli-interactive
az vm boot-diagnostics get-boot-log --resource-group myResourceGroupMonitor --name myVM
```

## <a name="view-host-metrics"></a>Afficher les métriques de l’hôte

Une machine virtuelle Linux a un hôte dédié dans Azure qui interagit avec elle. Les métriques sont automatiquement collectées pour l’hôte et peuvent être visualisées dans le portail Azure comme suit :

1. Dans le portail Azure, sélectionnez **Groupes de ressources**, choisissez **myResourceGroupMonitor** puis sélectionnez **myVM** dans la liste des ressources.
1. Pour voir comment la machine virtuelle hôte fonctionne, sélectionnez **Métriques** dans la fenêtre de la machine virtuelle, puis sélectionnez une des métriques de l’ *[hôte]* sous **Métriques disponibles**.

    ![Afficher les métriques de l’hôte](./media/tutorial-monitoring/monitor-host-metrics.png)

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
> [Gérer la sécurité des machines virtuelles](../tutorial-azure-security.md)
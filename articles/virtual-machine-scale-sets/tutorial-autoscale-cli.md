---
title: Tutoriel - Effectuer une mise à l’échelle automatique d’un groupe identique avec Azure CLI
description: Découvrez comment utiliser Azure CLI pour mettre à l’échelle automatiquement un groupe de machines virtuelles identiques en fonction de l’augmentation et de la diminution des demandes de l’UC.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 05/18/2018
ms.reviewer: avverma
ms.custom: avverma, devx-track-azurecli
ms.openlocfilehash: 68f311a949d6c7663c5602c444d1b7b9af09dcad
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016724"
---
# <a name="tutorial-automatically-scale-a-virtual-machine-scale-set-with-the-azure-cli"></a>Tutoriel : Effectuer une mise à l’échelle automatique d’un groupe de machines virtuelles identiques avec Azure CLI

Lorsque vous créez un groupe identique, vous définissez le nombre d’instances de machine virtuelle que vous souhaitez exécuter. À mesure que la demande de votre application change, vous pouvez augmenter ou diminuer automatiquement le nombre d’instances de machine virtuelle. La capacité de mise à l’échelle automatique vous permet de suivre la demande du client ou de répondre aux changements de performances de votre application tout au long de son cycle de vie. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Utiliser la mise à l’échelle automatique avec un groupe identique
> * Créer et utiliser des règles de mise à l’échelle automatique
> * Effectuer un test de contrainte sur les instances de machine virtuelle et déclencher des règles de mise à l’échelle automatique
> * Remettre à l’échelle automatiquement en cas de baisse de la demande

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Ce tutoriel nécessite Azure CLI version 2.0.32 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-scale-set"></a>Créer un groupe identique

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group) comme suit :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Créez à présent un groupe de machines virtuelles identiques avec [az vmss create](/cli/azure/vmss). L’exemple suivant crée un groupe identique avec *2* instances, et génère des clés SSH si elles n’existent pas :

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="define-an-autoscale-profile"></a>Définir un profil de mise à l’échelle automatique

Pour activer la mise à l’échelle automatique sur un groupe identique, vous devez d’abord définir un profil de mise à l’échelle automatique. Ce profil définit les capacités par défaut, minimale et maximale du groupe identique. Ces limites vous permettent de contrôler le coût en évitant de créer continuellement des instances de machine virtuelle et d’équilibrer les performances acceptables sur un nombre minimal d’instances qui restent dans un événement de diminution du nombre d’instances. Créez un profil de mise à l’échelle automatique [az monitor autoscale create](/cli/azure/monitor/autoscale#az-monitor-autoscale-create). L’exemple suivant définit les capacités par défaut et minimale de *2* instances de machine virtuelle, et la capacité maximale de *10* :

```azurecli-interactive
az monitor autoscale create \
  --resource-group myResourceGroup \
  --resource myScaleSet \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2
```

## <a name="create-a-rule-to-autoscale-out"></a>Créer une règle pour le scale-out automatique

Si la demande de votre application augmente, la charge sur les instances de machine virtuelle dans votre groupe identique augmente. Si cette augmentation de la charge est cohérente, au lieu d’une brève demande, vous pouvez configurer des règles de mise à l’échelle automatique pour augmenter le nombre d’instances de machine virtuelle dans le groupe identique. Lorsque ces instances de machine virtuelle sont créées et que vos applications sont déployées, le groupe identique commence à distribuer le trafic vers les instances via l’équilibreur de charge. Vous contrôlez les métriques à surveiller, telles que l’usage du processeur ou du disque, la durée pendant laquelle la charge de l’application doit respecter un seuil donné, et le nombre d’instances de machine virtuelle à ajouter au groupe identique.

Créons avec [az monitor autoscale rule create](/cli/azure/monitor/autoscale/rule#az-monitor-autoscale-rule-create) une règle qui augmente le nombre d’instances de machine virtuelle dans un groupe identique lorsque la charge moyenne du processeur est supérieure à 70 % pendant 5 minutes. Lorsque la règle se déclenche, le nombre d’instances de machine virtuelle est majoré de trois unités.

```azurecli-interactive
az monitor autoscale rule create \
  --resource-group myResourceGroup \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 3
```

## <a name="create-a-rule-to-autoscale-in"></a>Créer une règle pour le scale-in automatique

Au cours d’une soirée ou d’un week-end, la demande de votre application peut diminuer. Si cette charge réduite est constante pendant un certain temps, vous pouvez configurer des règles de mise à l’échelle automatique pour réduire le nombre d’instances de machine virtuelle dans le groupe identique. Cette action de diminution du nombre d’instances a pour effet de réduire le coût d’exécution de votre groupe identique, car vous seul exécutez le nombre d’instances requis pour répondre à la demande en cours.

Créez avec [az monitor autoscale rule create](/cli/azure/monitor/autoscale/rule#az-monitor-autoscale-rule-create) une autre règle qui réduit le nombre d’instances de machine virtuelle dans un groupe identique lorsque la charge moyenne du processeur est inférieure à 30 % pendant 5 minutes. L’exemple suivant définit la règle pour effectuer un scale-in du nombre d’instances de machine virtuelle par un :

```azurecli-interactive
az monitor autoscale rule create \
  --resource-group myResourceGroup \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1
```

## <a name="generate-cpu-load-on-scale-set"></a>Générer une charge d’UC sur un groupe identique

Pour tester les règles de mise à l’échelle automatique, générez des charges d’UC sur les instances de machine virtuelle dans le groupe identique. Cette charge d’UC simulée provoque le scale-out de la mise à l’échelle automatique et l’augmentation du nombre d’instances de machine virtuelle. Comme la charge d’UC simulée diminue ensuite, les règles de mise à l’échelle automatique effectuent un scale-out du nombre d’instances de machine virtuelle.

Tout d’abord, affichez l’adresse et les ports à connecter aux instances de machine virtuelle d’un groupe identique avec la commande [az vmss list-instance-connection-info](/cli/azure/vmss) :

```azurecli-interactive
az vmss list-instance-connection-info \
  --resource-group myResourceGroup \
  --name myScaleSet
```

L’exemple de sortie suivant affiche le nom de l’instance, l’adresse IP publique de l’équilibreur de charge et le numéro de port où les règles de traduction d’adresse réseau (NAT) transfèrent le trafic :

```json
{
  "instance 1": "13.92.224.66:50001",
  "instance 3": "13.92.224.66:50003"
}
```

SSH vers votre première instance de machine virtuelle. Spécifiez votre propre adresse IP publique et le numéro de port avec le paramètre `-p`, comme indiqué dans la commande précédente :

```console
ssh azureuser@13.92.224.66 -p 50001
```

Une fois connecté, installez l’utilitaire **stress**. Démarrez *10* workers de **contrainte** qui génèrent une charge au niveau du processeur. Ces rôles de travail sont exécutés pendant *420* secondes, ce qui est suffisant pour que les règles de mise à l’échelle automatique implémentent l’action souhaitée.

```console
sudo apt-get update
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 420 &
```

Lorsque **stress** affiche une sortie du type *stress: info: [2688] dispatching hogs: 10 cpu, 0 io, 0 vm, 0 hdd*, appuyez sur la touche *Entrée* pour revenir à l’invite de commandes.

Pour vérifier que **stress** génère une charge d’UC, examinez la charge du système actif avec l’utilitaire **top** :

```console
top
```

Quittez **top**, puis fermez votre connexion à l’instance de machine virtuelle. **stress** continue de s’exécuter sur l’instance de machine virtuelle.

```console
Ctrl-c
exit
```

Connectez-vous à la deuxième instance de machine virtuelle avec le numéro de port indiqué à partir de la commande [az vmss list-instance-connection-info](/cli/azure/vmss) précédente :

```console
ssh azureuser@13.92.224.66 -p 50003
```

Installez et exécutez **stress**, puis démarrez dix rôles de travail dans cette deuxième instance de machine virtuelle.

```console
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 420 &
```

Là encore, si **stress** affiche une sortie semblable à *stress: info: [2713] dispatching hogs: 10 cpu, 0 io, 0 vm, 0 hdd*, appuyez sur la touche *Entrée* pour revenir à l’invite de commandes.

Fermez votre connexion à la deuxième instance de machine virtuelle. **stress** continue de s’exécuter sur l’instance de machine virtuelle.

```console
exit
```

## <a name="monitor-the-active-autoscale-rules"></a>Surveiller les règles actives de mise à l’échelle automatique

Pour surveiller le nombre d’instances de machine virtuelle dans votre groupe identique, utilisez **watch**. Les règles de mise à l’échelle automatique prennent 5 minutes pour commencer le processus de scale-out en réponse à la charge d’UC générée par **stress** sur chacune des instances de machine virtuelle :

```azurecli-interactive
watch az vmss list-instances \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --output table
```

Une fois que le seuil de l’UC est atteint, les règles de mise à l’échelle automatique augmentent le nombre d’instances de machine virtuelle au sein du groupe identique. La sortie suivante montre trois machines virtuelles créées au moment de la montée en puissance automatique du groupe identique :

```output
Every 2.0s: az vmss list-instances --resource-group myResourceGroup --name myScaleSet --output table

  InstanceId  LatestModelApplied    Location    Name          ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  ------------  -------------------  ---------------  ------------------------------------
           1  True                  eastus      myScaleSet_1  Succeeded            myResourceGroup  4f92f350-2b68-464f-8a01-e5e590557955
           2  True                  eastus      myScaleSet_2  Succeeded            myResourceGroup  d734cd3d-fb38-4302-817c-cfe35655d48e
           4  True                  eastus      myScaleSet_4  Creating             myResourceGroup  061b4c90-0d73-49fc-a066-19eab0b3d95c
           5  True                  eastus      myScaleSet_5  Creating             myResourceGroup  4beff8b9-4e65-40cb-9652-43899309da27
           6  True                  eastus      myScaleSet_6  Creating             myResourceGroup  9e4133dd-2c57-490e-ae45-90513ce3b336
```

Une fois que **stress** s’arrête sur les instances initiales de machine virtuelle, la charge d’UC moyenne retourne à la normale. Lorsque 5 autres minutes ont passé, les règles de mise à l’échelle automatique effectuent un scale-in du nombre d’instances de machine virtuelle. Les actions de scale-in suppriment des instances de machine virtuelle disposant des ID les plus élevés. Lorsqu’un groupe identique utilise des groupes à haute disponibilité ou des zones de disponibilité, les actions de scale-in sont réparties uniformément entre les instances de machine virtuelle. La sortie d’exemple suivante montre une instance de machine virtuelle supprimée lors de la diminution automatique :

```output
           6  True                  eastus      myScaleSet_6  Deleting             myResourceGroup  9e4133dd-2c57-490e-ae45-90513ce3b336
```

Fermez *watch* avec `Ctrl-c`. Le groupe identique continue d’effectuer un scale-in toutes les 5 minutes et supprime une instance de machine virtuelle jusqu’à ce que la quantité minimale d’instances (2) soit atteinte.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer votre groupe identique et les ressources supplémentaires, supprimez le groupe de ressources et toutes ses ressources avec [az group delete](/cli/azure/group). Le paramètre `--no-wait` retourne le contrôle à l’invite de commandes sans attendre que l’opération se termine. Le paramètre `--yes` confirme que vous souhaitez supprimer les ressources sans passer par une invite supplémentaire à cette fin.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer automatiquement un scale-in ou un scale-out d’un groupe identique avec Azure CLI :

> [!div class="checklist"]
> * Utiliser la mise à l’échelle automatique avec un groupe identique
> * Créer et utiliser des règles de mise à l’échelle automatique
> * Effectuer un test de contrainte sur les instances de machine virtuelle et déclencher des règles de mise à l’échelle automatique
> * Remettre à l’échelle automatiquement en cas de baisse de la demande

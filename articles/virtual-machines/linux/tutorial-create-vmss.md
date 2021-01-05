---
title: 'Tutoriel : Créer un groupe de machines virtuelles identiques Linux'
description: Découvrez comment utiliser Azure CLI pour créer et déployer une application hautement disponible sur des machines virtuelles Linux à l’aide d’un groupe de machines virtuelles identiques
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: linux
ms.date: 06/01/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-js, devx-track-azurecli
ms.openlocfilehash: c38fb976ca597647493f3dc3d32be79040ded6eb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91320181"
---
# <a name="tutorial-create-a-virtual-machine-scale-set-and-deploy-a-highly-available-app-on-linux-with-the-azure-cli"></a>Tutoriel : Créer un groupe de machines virtuelles identiques et déployer une application à haute disponibilité sur Linux avec Azure CLI

Un groupe de machines virtuelles identiques vous permet de déployer et de gérer un ensemble de machines virtuelles identiques prenant en charge la mise à l’échelle automatique. Vous pouvez mettre à l’échelle manuellement le nombre de machines virtuelles du groupe identique ou définir des règles pour mettre à l’échelle automatiquement en fonction de l’utilisation des ressources (processeur, demande de mémoire ou trafic réseau). Ce didacticiel explique comment déployer un groupe de machines virtuelles identiques dans Azure. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Utiliser cloud-init pour créer une application à l’échelle
> * Créer un groupe de machines virtuelles identiques
> * Augmenter ou réduire le nombre d’instances dans un groupe identique
> * Créer des règles de mise à l’échelle automatique
> * Afficher les informations de connexion pour les instances de groupe identique
> * Utiliser des disques de données dans un groupe identique

Ce tutoriel utilise l’interface CLI disponible dans [Azure Cloud Shell](../../cloud-shell/overview.md), qui est constamment mise à jour vers la dernière version. Pour ouvrir Cloud Shell, sélectionnez **Essayer** en haut d’un bloc de code.

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, ce didacticiel nécessite que vous exécutiez Azure CLI version 2.0.30 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="scale-set-overview"></a>Vue d’ensemble des groupes identiques
Un groupe de machines virtuelles identiques vous permet de déployer et de gérer un ensemble de machines virtuelles identiques prenant en charge la mise à l’échelle automatique. Les machines virtuelles d’un groupe identique sont réparties entre les domaines d’erreur logique et de mise à jour, dans un ou plusieurs *groupes de placement*. Ces groupes contiennent des machines virtuelles configurées de manière similaire, semblables à des [groupes à haute disponibilité](tutorial-availability-sets.md).

Les machines virtuelles sont créées en fonction des besoins dans un groupe identique. En définissant des règles de mise à l’échelle automatique, vous pouvez contrôler quand et comment les machines virtuelles sont ajoutées ou supprimées au niveau du groupe identique. Ces règles peuvent être déclenchées en fonction de métriques telles que la charge du processeur, l’utilisation de la mémoire ou le trafic réseau.

Les groupes identiques prennent en charge jusqu’à 1 000 machines virtuelles lorsque vous utilisez une image de plateforme Azure. Pour les charges de travail qui s’accompagnent de contraintes importantes en matière d’installation ou de personnalisation de machines virtuelles, vous pouvez [créer une image de machine virtuelle personnalisée](tutorial-custom-images.md). Vous pouvez créer un maximum de 300 machines virtuelles dans un groupe identique lorsque vous utilisez une image personnalisée.


## <a name="create-an-app-to-scale"></a>Créer une application à mettre à l’échelle
À des fins de production, vous pouvez [créer une image de machine virtuelle personnalisée](tutorial-custom-images.md) qui inclut votre application installée et configurée. Pour ce didacticiel, nous allons personnaliser les machines virtuelles au premier démarrage pour voir fonctionner un groupe identique rapidement.

Dans le didacticiel précédent, vous avez appris à [personnaliser une machine virtuelle Linux au premier démarrage](tutorial-automate-vm-deployment.md) avec cloud-init. Vous pouvez utiliser le même fichier de configuration cloud-init pour installer NGINX et exécuter une simple application Node.js « Hello World ».

Dans l’interpréteur de commandes actuel, créez un fichier nommé *cloud-init.txt* et collez la configuration suivante. Par exemple, créez le fichier dans l’interpréteur de commandes Cloud et non sur votre ordinateur local. Entrez `sensible-editor cloud-init.txt` pour créer le fichier et afficher la liste des éditeurs disponibles. Vérifiez que l’intégralité du fichier cloud-init est copiée, en particulier la première ligne :

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
  - path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```


## <a name="create-a-scale-set"></a>Créer un groupe identique
Pour pouvoir créer un groupe identique, vous devez créer un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). L’exemple suivant crée un groupe de ressources nommé *myResourceGroupScaleSet* à l’emplacement *eastus*:

```azurecli-interactive
az group create --name myResourceGroupScaleSet --location eastus
```

Créez à présent un groupe de machines virtuelles identiques avec [az vmss create](/cli/azure/vmss#az-vmss-create). L’exemple suivant crée un groupe identique nommé *myScaleSet*, utilise le fichier cloud-init pour personnaliser la machine virtuelle et génère des clés SSH si elles n’existent pas :

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroupScaleSet \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.txt \
  --admin-username azureuser \
  --generate-ssh-keys
```

La création et la configuration des l’ensemble des ressources et des machines virtuelles du groupe identique prennent quelques minutes. Certaines tâches en arrière-plan continuent à s’exécuter une fois que l’interface CLI Azure vous renvoie à l’invite de commandes. Un délai de quelques minutes peut être nécessaire avant que vous puissiez accéder à l’application.


## <a name="allow-web-traffic"></a>Autoriser le trafic web
Un équilibrage de charge a été créé automatiquement dans le cadre du groupe de machines virtuelles identiques. L’équilibrage de charge répartit le trafic sur un ensemble de machines virtuelles définies à l’aide de règles d’équilibrage de charge. Vous trouverez plus d’informations sur les concepts de l’équilibrage de charge et la configuration dans le didacticiel suivant, intitulé [Équilibrage de charge des machines virtuelles dans Azure](tutorial-load-balancer.md).

Pour autoriser le trafic à atteindre l’application web, créez une règle avec [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create). L’exemple suivant crée une règle nommée *myLoadBalancerRuleWeb* :

```azurecli-interactive
az network lb rule create \
  --resource-group myResourceGroupScaleSet \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```

## <a name="test-your-app"></a>Test de l'application
Pour voir votre application Node.js sur le web, obtenez l’adresse IP publique de votre équilibrage de charge avec [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show). L’exemple suivant obtient l’adresse IP pour *myScaleSetLBPublicIP* qui a été créée dans le cadre du groupe identique :

```azurecli-interactive
az network public-ip show \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSetLBPublicIP \
    --query [ipAddress] \
    --output tsv
```

Entrez l’adresse IP publique dans un navigateur web. L’application s’affiche, avec notamment le nom d’hôte de la machine virtuelle sur laquelle l’équilibrage de charge a distribué le trafic :

![Exécution de l’application Node.js](./media/tutorial-create-vmss/running-nodejs-app.png)

Pour voir le groupe identique en action, vous pouvez forcer l’actualisation de votre navigateur web pour visualiser la distribution de trafic par l’équilibrage de charge sur l’ensemble des machines virtuelles exécutant votre application.


## <a name="management-tasks"></a>Tâches de gestion
Tout au long du cycle de vie du groupe identique, vous devrez peut-être exécuter une ou plusieurs tâches de gestion. En outre, vous souhaiterez peut-être créer des scripts pour automatiser les diverses tâches liées au cycle de vie. Azure CLI offre un moyen rapide d’effectuer ces tâches. Voici quelques tâches courantes.

### <a name="view-vms-in-a-scale-set"></a>Afficher les machines virtuelles d’un groupe identique
Pour afficher une liste des machines virtuelles exécutées dans votre groupe identique, utilisez [az vmss list-instances](/cli/azure/vmss#az-vmss-list-instances) comme suit :

```azurecli-interactive
az vmss list-instances \
  --resource-group myResourceGroupScaleSet \
  --name myScaleSet \
  --output table
```

Le résultat ressemble à l’exemple suivant :

```bash
  InstanceId  LatestModelApplied    Location    Name          ProvisioningState    ResourceGroup            VmId
------------  --------------------  ----------  ------------  -------------------  -----------------------  ------------------------------------
           1  True                  eastus      myScaleSet_1  Succeeded            MYRESOURCEGROUPSCALESET  c72ddc34-6c41-4a53-b89e-dd24f27b30ab
           3  True                  eastus      myScaleSet_3  Succeeded            MYRESOURCEGROUPSCALESET  44266022-65c3-49c5-92dd-88ffa64f95da
```


### <a name="manually-increase-or-decrease-vm-instances"></a>Augmenter ou diminuer manuellement les instances de machines virtuelles
Pour afficher le nombre d’instances présentes dans un groupe identique, utilisez [az vmss show](/cli/azure/vmss#az-vmss-show) et interrogez *sku.capacity* :

```azurecli-interactive
az vmss show \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --query [sku.capacity] \
    --output table
```

Vous pouvez ensuite augmenter ou diminuer manuellement le nombre de machines virtuelles dans le groupe identique avec [az vmss scale](/cli/azure/vmss#az-vmss-scale). L’exemple suivant fixe le nombre de machines virtuelles présentes dans votre groupe identique à *3* :

```azurecli-interactive
az vmss scale \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --new-capacity 3
```

### <a name="get-connection-info"></a>Obtenir des informations de connexion
Pour obtenir des informations de connexion sur les machines virtuelles dans vos groupes identiques, utilisez [az vmss list-instance-connection-info](/cli/azure/vmss#az-vmss-list-instance-connection-info). Cette commande renvoie l’adresse IP publique et le port pour chaque machine virtuelle pour vous permettre de vous connecter avec SSH :

```azurecli-interactive
az vmss list-instance-connection-info \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet
```


## <a name="use-data-disks-with-scale-sets"></a>Utiliser des disques de données avec des groupes identiques
Vous pouvez créer et utiliser des disques de données avec des groupes identiques. Dans un tutoriel précédent, vous avez appris comment [gérer des disques Azure](tutorial-manage-disks.md). Ce tutoriel inclut également une description des meilleures pratiques et des améliorations des performances pour créer des applications sur des disques de données plutôt que sur le disque du système d’exploitation.

### <a name="create-scale-set-with-data-disks"></a>Créer un groupe identique avec des disques de données
Pour créer un groupe identique et y rattacher des disques de données, ajoutez le paramètre `--data-disk-sizes-gb` à la commande [az vmss create](/cli/azure/vmss#az-vmss-create). L’exemple suivant crée un groupe identique avec des disques de données de *50* Go associés à chaque instance :

```azurecli-interactive
az vmss create \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSetDisks \
    --image UbuntuLTS \
    --upgrade-policy-mode automatic \
    --custom-data cloud-init.txt \
    --admin-username azureuser \
    --generate-ssh-keys \
    --data-disk-sizes-gb 50
```

Lorsque les instances sont supprimées d’un groupe identique, les disques de données associés sont également supprimés.

### <a name="add-data-disks"></a>Ajouter des disques de données
Pour ajouter un disque de données à des instances de votre groupe identique, utilisez [az vmss disk attach](/cli/azure/vmss/disk#az-vmss-disk-attach). L’exemple suivant ajoute un disque de données de *50* Go chaque instance :

```azurecli-interactive
az vmss disk attach \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --size-gb 50 \
    --lun 2
```

### <a name="detach-data-disks"></a>Détacher des disques de données
Pour supprimer un disque de données dans des instances de votre groupe identique, utilisez [az vmss disk detach](/cli/azure/vmss/disk#az-vmss-disk-detach). L’exemple suivant supprime le disque de données au numéro d’unité logique *2* de chaque instance :

```azurecli-interactive
az vmss disk detach \
    --resource-group myResourceGroupScaleSet \
    --name myScaleSet \
    --lun 2
```


## <a name="next-steps"></a>Étapes suivantes
Ce didacticiel vous a montré comment créer un groupe de machines virtuelles identiques. Vous avez appris à :

> [!div class="checklist"]
> * Utiliser cloud-init pour créer une application à l’échelle
> * Créer un groupe de machines virtuelles identiques
> * Augmenter ou réduire le nombre d’instances dans un groupe identique
> * Créer des règles de mise à l’échelle automatique
> * Afficher les informations de connexion pour les instances de groupe identique
> * Utiliser des disques de données dans un groupe identique

Passez au didacticiel suivant pour en savoir plus sur les concepts de l’équilibrage de charge des machines virtuelles.

> [!div class="nextstepaction"]
> [Équilibrage de charge des machines virtuelles](tutorial-load-balancer.md)

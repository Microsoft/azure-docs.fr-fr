---
title: Tutoriel - Installer des applications dans un groupe identique avec Azure CLI
description: Découvrez comment utiliser Azure CLI pour installer des applications dans des groupes de machines virtuelles identiques à l’aide de l’extension de script personnalisé
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: cli
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: d9969cf0fa453f857de421dd10934f63f5773f6c
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94516728"
---
# <a name="tutorial-install-applications-in-virtual-machine-scale-sets-with-the-azure-cli"></a>Tutoriel : Installer des applications dans des groupes de machines virtuelles identiques avec Azure CLI
Pour exécuter des applications sur des instances de machine virtuelle d’un groupe identique, vous devez d’abord installer les composants d’application et les fichiers requis. Dans un didacticiel précédent, vous avez appris à créer et utiliser une image personnalisée de machine virtuelle pour déployer vos instances de machine virtuelle. Cette image personnalisée comprenait l’installation et la configuration manuelles d’applications. Vous pouvez également automatiser l’installation des applications pour un groupe identique après le déploiement de chaque instance de machine virtuelle, ou mettre à jour une application déjà exécutée dans un groupe identique. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Installer automatiquement des applications dans votre groupe identique
> * Utiliser l’extension de script personnalisé Azure
> * Mettre à jour une application en cours d’exécution dans un groupe identique

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.29 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée. 


## <a name="what-is-the-azure-custom-script-extension"></a>Qu’est-ce que l’extension de script personnalisé Azure ?
L’extension de script personnalisé télécharge et exécute des scripts sur des machines virtuelles Azure. Cette extension est utile pour la configuration post-déploiement, l’installation de logiciels ou toute autre tâche de configuration ou de gestion. Des scripts peuvent être téléchargés à partir de Stockage Azure ou de GitHub, ou fournis dans le portail Azure lors de l’exécution de l’extension.

L’extension de script personnalisé s’intègre aux modèles Azure Resource Manager et peut être utilisée avec Azure CLI, Azure PowerShell, le Portail Azure ou l’API REST. Pour plus d’informations, consultez [Vue d’ensemble de l’extension de script personnalisé](../virtual-machines/extensions/custom-script-linux.md).

Pour utiliser l’extension de script personnalisé avec l’interface CLI Azure, vous créez un fichier JSON qui définit les fichiers à obtenir et les commandes à exécuter. Ces définitions JSON peuvent être réutilisées sur plusieurs déploiements de groupe identique pour garantir des installations d’applications cohérentes.


## <a name="create-custom-script-extension-definition"></a>Créer une définition d’extension de script personnalisé
Pour voir l’extension de script personnalisé en action, il faut créer un groupe identique qui installe le serveur web NGINX et affiche le nom d’hôte de l’instance de machine virtuelle du groupe identique. La définition d’extension de script personnalisé suivante télécharge un exemple de script à partir de GitHub, installe les packages requis, puis écrit le nom d’hôte de l’instance de machine virtuelle sur une page HTML de base.

Dans l’interpréteur de commandes actuel, créez un fichier nommé *customConfig.json* et collez la configuration suivante. Par exemple, créez le fichier dans l’interpréteur de commandes Cloud et non sur votre ordinateur local. Vous pouvez utiliser l’éditeur de votre choix. Entrez `sensible-editor customConfig.json` dans Cloud Shell pour créer le fichier et afficher la liste des éditeurs disponibles.

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],
  "commandToExecute": "./automate_nginx.sh"
}
```

> [!CAUTION]
> Il peut être nécessaire d’inverser l’utilisation des guillemets simples (') et des guillemets doubles (") au sein du bloc JSON si vous décidez de référencer le JSON directement (au lieu de référencer le fichier *customConfig.json*) dans le paramètre *--settings* ci-dessous. 


## <a name="create-a-scale-set"></a>Créer un groupe identique
Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Créez à présent un groupe de machines virtuelles identiques avec [az vmss create](/cli/azure/vmss). L’exemple suivant crée un groupe identique nommé *myScaleSet*, et génère des clés SSH si elles n’existent pas :

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys
```

La création et la configuration des l’ensemble des ressources et des machines virtuelles du groupe identique prennent quelques minutes.


## <a name="apply-the-custom-script-extension"></a>Appliquer l’extension de script personnalisé
Appliquez la configuration de l’extension de script personnalisé aux instances de machine virtuelle dans votre groupe identique avec [az vmss extension set](/cli/azure/vmss/extension). L’exemple suivant applique la configuration *customConfig.json* aux instances de machine virtuelle *myScaleSet* dans le groupe de ressources nommé *myResourceGroup* :

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings @customConfig.json
```

Chaque instance de machine virtuelle dans le groupe identique télécharge et exécute le script à partir de GitHub. Dans un exemple plus complexe, les composants et fichiers de plusieurs applications peuvent être installés. Si le groupe identique monte en puissance, les nouvelles instances de machine virtuelle appliquent automatiquement la même définition d’extension de script personnalisé et installent l’application requise.


## <a name="test-your-scale-set"></a>Tester votre groupe identique
Pour autoriser le trafic à atteindre le serveur web, créez une règle d’équilibreur de charge avec [az network lb rule create](/cli/azure/network/lb/rule). L’exemple suivant crée une règle nommée *myLoadBalancerRuleWeb* :

```azurecli-interactive
az network lb rule create \
  --resource-group myResourceGroup \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```

Pour voir votre serveur web en action, obtenez l’adresse IP publique de votre équilibreur de charge avec [az network public-ip show](/cli/azure/network/public-ip). L’exemple suivant obtient l’adresse IP pour *myScaleSetLBPublicIP* qui a été créée dans le cadre du groupe identique :

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query [ipAddress] \
  --output tsv
```

Saisissez l’adresse IP publique de l’équilibreur de charge dans un navigateur web. L’équilibreur de charge répartit le trafic vers l’une de vos instances de machine virtuelle, comme illustré dans l’exemple suivant :

![Page web de base dans NGINX](media/tutorial-install-apps-cli/running-nginx.png)

Laissez le navigateur web ouvert afin que vous puissiez voir une version mise à jour à l’étape suivante.


## <a name="update-app-deployment"></a>Déployer une mise à jour d’application
Tout au long du cycle de vie d’un groupe identique, vous devrez peut-être déployer une version mise à jour de votre application. Avec l’extension de script personnalisé, vous pouvez faire référence à un script de déploiement de mises à jour puis réappliquer l’extension à votre groupe identique. Une fois le groupe identique créé à l’étape précédente, `--upgrade-policy-mode` a été défini sur *Automatique*. Ce paramètre permet aux instances de machine virtuelle dans le groupe identique de mettre automatiquement à jour et d’appliquer la dernière version de votre application.

Dans l’interpréteur de commandes actuel, créez un fichier nommé *customConfigv2.json* et collez la configuration suivante. Cette définition exécute une version *v2* mise à jour du script d’installation de l’application :

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx_v2.sh"],
  "commandToExecute": "./automate_nginx_v2.sh"
}
```

Appliquez une nouvelle fois la configuration de l’extension de script personnalisé aux instances de machine virtuelle dans votre groupe identique avec [az vmss extension set](/cli/azure/vmss/extension). Le fichier *customConfigv2.json* sert à appliquer la version mise à jour de l’application :

```azurecli-interactive
az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroup \
    --vmss-name myScaleSet \
    --settings @customConfigv2.json
```

Toutes les instances de machine virtuelle dans le groupe identique sont automatiquement mises à jour avec la dernière version de la page web exemple. Pour afficher la version mise à jour, actualisez le site web dans votre navigateur :

![Page web mise à jour dans NGINX](media/tutorial-install-apps-cli/running-nginx-updated.png)


## <a name="clean-up-resources"></a>Nettoyer les ressources
Pour supprimer votre groupe identique et les ressources supplémentaires, supprimez le groupe de ressources et toutes ses ressources avec [az group delete](/cli/azure/group). Le paramètre `--no-wait` retourne le contrôle à l’invite de commandes sans attendre que l’opération se termine. Le paramètre `--yes` confirme que vous souhaitez supprimer les ressources sans passer par une invite supplémentaire à cette fin.

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez appris à installer et mettre à jour automatiquement des applications dans votre groupe identique avec Azure CLI :

> [!div class="checklist"]
> * Installer automatiquement des applications dans votre groupe identique
> * Utiliser l’extension de script personnalisé Azure
> * Mettre à jour une application en cours d’exécution dans un groupe identique

Passez au didacticiel suivant pour apprendre à mettre automatiquement à l’échelle votre groupe identique.

> [!div class="nextstepaction"]
> [Mettre automatiquement à l’échelle vos groupes identiques](tutorial-autoscale-cli.md)

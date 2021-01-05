---
title: Déployer OKD dans Azure
description: Déployez OKD dans Azure.
author: haroldwongms
manager: joraio
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/15/2019
ms.author: haroldw
ms.openlocfilehash: ed74d0829a8372ea8391fcd331eef2ef3af3dae0
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97704909"
---
# <a name="deploy-okd-in-azure"></a>Déployer OKD dans Azure

Vous pouvez utiliser l’une des deux manières de déployer OKD (anciennement OpenShift Origin) dans Azure :

- Vous pouvez déployer manuellement tous les composants d’infrastructure Azure nécessaires, puis suivre la [documentation d’OKD](https://docs.okd.io).
- Vous pouvez également utiliser un [modèle Resource Manager](https://github.com/Microsoft/openshift-origin) existant qui simplifie le déploiement du cluster OKD.

## <a name="deploy-using-the-okd-template"></a>Déployer avec le modèle OKD

Pour effectuer le déploiement à l’aide du modèle Resource Manager, vous utilisez un fichier de paramètres qui fournit les paramètres d’entrée. Pour personnaliser davantage le déploiement, dupliquez (fork) le dépôt GitHub et changez les éléments appropriés.

Voici une liste non exhaustive d’options de personnalisation courantes :

- Taille de machine virtuelle bastion (variable dans azuredeploy.json)
- Conventions de nommage (variables dans azuredeploy.json)
- Caractéristiques de cluster OpenShift, modifiées dans le fichier hosts (deployOpenShift.sh)

Le [modèle OKD](https://github.com/Microsoft/openshift-origin) comporte plusieurs branches disponibles pour différentes versions d’OKD.  Selon vos besoins, vous pouvez effectuer le déploiement directement à partir du dépôt, ou dupliquer (fork) le dépôt et effectuer des modifications personnalisées avant le déploiement.

Utilisez la valeur `appId` du principal de service créé précédemment pour le paramètre `aadClientId`.

L’exemple suivant montre un fichier de paramètres nommé azuredeploy.parameters.json avec toutes les entrées obligatoires.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "masterVmSize": {
            "value": "Standard_E2s_v3"
        },
        "infraVmSize": {
            "value": "Standard_E2s_v3"
        },
        "nodeVmSize": {
            "value": "Standard_E2s_v3"
        },
        "storageKind": {
            "value": "managed"
        },
        "openshiftClusterPrefix": {
            "value": "mycluster"
        },
        "masterInstanceCount": {
            "value": 3
        },
        "infraInstanceCount": {
            "value": 2
        },
        "nodeInstanceCount": {
            "value": 2
        },
        "dataDiskSize": {
            "value": 128
        },
        "adminUsername": {
            "value": "clusteradmin"
        },
        "openshiftPassword": {
            "value": "{Strong Password}"
        },
        "sshPublicKey": {
            "value": "{SSH Public Key}"
        },
        "enableMetrics": {
            "value": "true"
        },
        "enableLogging": {
            "value": "false"
        },
        "keyVaultResourceGroup": {
            "value": "keyvaultrg"
        },
        "keyVaultName": {
            "value": "keyvault"
        },
        "keyVaultSecret": {
            "value": "keysecret"
        },
        "enableAzure": {
            "value": "true"
        },
        "aadClientId": {
            "value": "11111111-abcd-1234-efgh-111111111111"
        },
        "aadClientSecret": {
            "value": "{Strong Password}"
        },
        "defaultSubDomainType": {
            "value": "nipio"
        }
    }
}
```

Remplacez les paramètres par vos informations spécifiques.

Les paramètres peuvent différer selon les versions. Vérifiez donc quels sont les paramètres nécessaires pour la branche que vous utilisez.

### <a name="deploy-using-azure-cli"></a>Déployer à l’aide d’Azure CLI


> [!NOTE] 
> La commande suivante requiert Azure CLI 2.0.8 ou version ultérieure. Pour vérifier la version d’Azure CLI, exécutez la commande `az --version`. Pour mettre à jour la version de l’interface CLI, consultez [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

L’exemple suivant déploie le cluster OKD et toutes les ressources associées dans un groupe de ressources nommé openshiftrg, avec le nom de déploiement myOpenShiftCluster. Le modèle est référencé directement à partir du dépôt GitHub à l’aide d’un fichier de paramètres locaux nommé azuredeploy.parameters.json.

```azurecli 
az deployment group create -g openshiftrg --name myOpenShiftCluster \
      --template-uri https://raw.githubusercontent.com/Microsoft/openshift-origin/master/azuredeploy.json \
      --parameters @./azuredeploy.parameters.json
```

La durée du déploiement varie en fonction du nombre total de nœuds déployés, avec un minimum de 30 minutes. L’URL de la console OpenShift et le nom DNS de l’OpenShift master s’affichent sur le terminal à l’issue du déploiement. Vous pouvez également afficher la section des sorties du déploiement à partir du portail Azure.

```json
{
  "OpenShift Console Url": "http://openshiftlb.cloudapp.azure.com/console",
  "OpenShift Master SSH": "ssh -p 2200 clusteradmin@myopenshiftmaster.cloudapp.azure.com"
}
```

Si vous ne souhaitez pas lier la ligne de commande d’attente de la fin du déploiement, ajoutez `--no-wait` aux options pour le déploiement du groupe. La sortie du déploiement peut être récupérée à partir du portail Azure, dans la section du déploiement du groupe de ressources.

## <a name="connect-to-the-okd-cluster"></a>Se connecter au cluster OKD

Une fois le déploiement terminé, connectez-vous à la console OpenShift dans un navigateur à l’aide de la valeur `OpenShift Console Url`. Vous pouvez également établir une connexion SSH au maître OKD. Voici un exemple qui utilise la sortie du déploiement :

```bash
$ ssh -p 2200 clusteradmin@myopenshiftmaster.cloudapp.azure.com
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Utilisez la commande [az group delete](/cli/azure/group) pour supprimer le groupe de ressources, le cluster OpenShift et toutes les ressources associées quand vous n’en avez plus besoin.

```azurecli 
az group delete --name openshiftrg
```

## <a name="next-steps"></a>Étapes suivantes

- [Tâches de post-déploiement](./openshift-container-platform-3x-post-deployment.md)
- [Résoudre les problèmes liés au déploiement d’OpenShift](./openshift-container-platform-3x-troubleshooting.md)
- [Bien démarrer avec OKD](https://docs.okd.io)

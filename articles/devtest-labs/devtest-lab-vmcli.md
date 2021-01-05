---
title: Créer et gérer des machines virtuelles dans DevTest Labs avec Azure CLI
description: Apprenez à utiliser Azure DevTest Labs pour créer et gérer des machines virtuelles avec Azure CLI
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 57510a722ab6a34b9d498e74a140d6f350c8fc02
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85482834"
---
# <a name="create-and-manage-virtual-machines-with-devtest-labs-using-the-azure-cli"></a>Créer et gérer des machines virtuelles avec DevTest Labs à l’aide de l’interface de ligne de commande Azure
Ce guide de démarrage rapide vous montre pas à pas comment créer, démarrer, vous connecter à, mettre à jour et nettoyer une machine de développement dans votre lab. 

Avant de commencer :

* Si aucun laboratoire n’a été créé, la procédure à suivre se trouve [ici](devtest-lab-create-lab.md).

* [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli). Pour démarrer, exécutez az login pour créer une connexion avec Azure. 

## <a name="create-and-verify-the-virtual-machine"></a>Créer et vérifier la machine virtuelle 
Avant d’exécuter les commandes DevTest Labs, définissez le contexte Azure approprié à l’aide de la commande `az account set` :

```azurecli
az account set --subscription 11111111-1111-1111-1111-111111111111
```

La commande de création d’une machine virtuelle est celle-ci : `az lab vm create`. Le nom du groupe de ressources du lab, le nom du lab et le nom de la machine virtuelle sont des arguments obligatoires. Les autres arguments varient selon le type de la machine virtuelle.

La commande suivante crée une image basée sur Windows à partir de la Place de marché Azure. Le nom de l’image est identique à celui que vous voyez quand vous créez une machine virtuelle par le biais du portail Azure. 

```azurecli
az lab vm create --resource-group DtlResourceGroup --lab-name MyLab --name 'MyTestVm' --image "Visual Studio Community 2017 on Windows Server 2016 (x64)" --image-type gallery --size 'Standard_D2s_v3' --admin-username 'AdminUser' --admin-password 'Password1!'
```

La commande suivante crée une machine virtuelle basée sur une image personnalisée disponible dans le lab :

```azurecli
az lab vm create --resource-group DtlResourceGroup --lab-name MyLab --name 'MyTestVm' --image "My Custom Image" --image-type custom --size 'Standard_D2s_v3' --admin-username 'AdminUser' --admin-password 'Password1!'
```

L’argument **image-type** est passé de **gallery** à **custom**. Le nom de l’image est identique à celui que vous voyez quand vous créez une machine virtuelle par le biais du portail Azure.

La commande suivante crée une machine virtuelle à partir d’une image de la Place du marché avec authentification SSH :

```azurecli
az lab vm create --lab-name sampleLabName --resource-group sampleLabResourceGroup --name sampleVMName --image "Ubuntu Server 16.04 LTS" --image-type gallery --size Standard_DS1_v2 --authentication-type  ssh --generate-ssh-keys --ip-configuration public 
```

Vous pouvez également créer des machines virtuelles basées sur des formules en définissant le paramètre **image-type** sur **formula**. Si vous devez choisir un réseau virtuel spécifique pour votre machine virtuelle, utilisez les paramètres **vnet-name** et **subnet**. Pour plus d’informations, consultez [az lab vm create](/cli/azure/lab/vm#az-lab-vm-create).

## <a name="verify-that-the-vm-is-available"></a>Vérifiez la disponibilité de la machine virtuelle.
Utilisez la commande `az lab vm show` pour vérifier que la machine virtuelle est disponible avant de la démarrer et de vous y connecter. 

```azurecli
az lab vm show --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup --expand 'properties($expand=ComputeVm,NetworkInterface)' --query '{status: computeVm.statuses[0].displayStatus, fqdn: fqdn, ipAddress: networkInterface.publicIpAddress}'
```
```json
{
  "fqdn": "lisalabvm.southcentralus.cloudapp.azure.com",
  "ipAddress": "13.85.228.112",
  "status": "Provisioning succeeded"
}
```

## <a name="start-and-connect-to-the-virtual-machine"></a>Démarrer et se connecter à la machine virtuelle
La commande suivante démarre une machine virtuelle :

```azurecli
az lab vm start --lab-name sampleLabName --name sampleVMName --resource-group sampleLabResourceGroup
```

Connectez-vous à une machine virtuelle : [SSH](../virtual-machines/linux/mac-create-ssh-keys.md) ou [Bureau à distance](../virtual-machines/windows/connect-logon.md).
```bash
ssh userName@ipAddressOrfqdn 
```

## <a name="update-the-virtual-machine"></a>Mettre à jour la machine virtuelle
La commande suivante applique des artefacts à une machine virtuelle :

```azurecli
az lab vm apply-artifacts --lab-name  sampleLabName --name sampleVMName  --resource-group sampleResourceGroup  --artifacts @/artifacts.json
```

```json
[
  {
    "artifactId": "/artifactSources/public repo/artifacts/linux-java",
    "parameters": []
  },
  {
    "artifactId": "/artifactSources/public repo/artifacts/linux-install-nodejs",
    "parameters": []
  },
  {
    "artifactId": "/artifactSources/public repo/artifacts/linux-apt-package",
    "parameters": [
      {
        "name": "packages",
        "value": "abcd"
      },
      {
        "name": "update",
        "value": "true"
      },
      {
        "name": "options",
        "value": ""
      }
    ]
  } 
]
```

### <a name="list-artifacts-available-in-the-lab"></a>Lister les artefacts disponibles dans le laboratoire

Pour lister les artefacts disponibles dans une machine virtuelle d'un laboratoire, exécutez les commandes suivantes.

**Cloud Shell - PowerShell** : notez l’utilisation de l’accent grave (\`) devant $ dans $expand (`$expand) :

```azurecli-interactive
az lab vm show --resource-group <resourcegroupname> --lab-name <labname> --name <vmname> --expand "properties(`$expand=artifacts)" --query "artifacts[].{artifactId: artifactId, status: status}"
```

**Cloud Shell - Bash** : notez l’utilisation du caractère barre oblique (\\) devant $ dans la commande. 

```azurecli-interactive
az lab vm show --resource-group <resourcegroupname> --lab-name <labname> --name <vmname> --expand "properties(\$expand=artifacts)" --query "artifacts[].{artifactId: artifactId, status: status}"
```

Exemple de sortie : 

```json
[
  {
    "artifactId": "/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.DevTestLab/labs/<lab name>/artifactSources/public repo/artifacts/windows-7zip",
    "status": "Succeeded"
  }
]
```

## <a name="stop-and-delete-the-virtual-machine"></a>Arrêter et supprimer la machine virtuelle    
La commande suivante arrête une machine virtuelle.

```azurecli
az lab vm stop --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup
```

Supprimez une machine virtuelle.
```azurecli
az lab vm delete --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes
Consultez la [documentation Azure CLI pour Azure DevTest Labs](/cli/azure/lab?view=azure-cli-latest). 

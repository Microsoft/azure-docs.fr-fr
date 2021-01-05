---
title: Créer une machine virtuelle Linux à l’aide de l’API REST
description: Découvrez comment créer une machine virtuelle Linux dans Azure utilisant la fonctionnalité Disques managés et l’authentification SSH avec l’API REST Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 06/05/2018
ms.author: cynthn
ms.openlocfilehash: d6e5195f43991f4d40af57c1ab4b87aaca475b64
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87373400"
---
# <a name="create-a-linux-virtual-machine-that-uses-ssh-authentication-with-the-rest-api"></a>Créer une machine virtuelle Linux utilisant l’authentification SSH avec l’API REST

Une machine virtuelle Linux dans Azure se compose de diverses ressources telles que des disques et des interfaces réseau, et définit des paramètres tels que l’emplacement, la taille, l’image du système d’exploitation et l’authentification.

Vous pouvez créer une machine virtuelle Linux via le portail Azure, Azure CLI 2.0, de nombreux kits SDK Azure, les modèles Azure Resource Manager et un grand nombre d’outils tiers comme Ansible ou Terraform. Tous ces outils utilisent à la fin l’API REST pour créer la machine virtuelle Linux.

Cet article explique comment utiliser l’API REST pour créer une machine virtuelle Linux exécutant Ubuntu 18.04-LTS avec des disques managés et l’authentification SSH.

## <a name="before-you-start"></a>Avant de commencer

Avant de créer et d’envoyer la requête, vous avez besoin des éléments suivants :

* `{subscription-id}` pour votre abonnement
  * Si vous avez plusieurs abonnements, consultez [Utilisation de plusieurs abonnements](/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest)
* `{resourceGroupName}` que vous avez créé à l’avance
* [Interface de réseau virtuel](../../virtual-network/virtual-network-network-interface.md) dans le même groupe de ressources
* Paire de clés SSH (vous pouvez en [générer une](mac-create-ssh-keys.md) si vous n’en avez pas)

## <a name="request-basics"></a>Informations de base de la requête

Pour créer ou mettre à jour une machine virtuelle, utilisez l’opération *PUT* suivante :

``` http
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2017-12-01
```

Outre les paramètres `{subscription-id}` et `{resourceGroupName}`, vous devez spécifier `{vmName}` (`api-version` est facultatif, mais cet article a été testé avec `api-version=2017-12-01`)

Les en-têtes suivants sont requis :

| En-tête de requête   | Description |
|------------------|-----------------|
| *Content-Type :*  | Obligatoire. Défini sur `application/json`. |
| *Authorization :* | Obligatoire. Défini sur un [jeton d’accès](/rest/api/azure/#authorization-code-grant-interactive-clients) `Bearer` valide. |

Pour obtenir des informations générales sur l’utilisation des requêtes d’API REST, consultez [Composants d’une requête/réponse d’API REST](/rest/api/azure/#components-of-a-rest-api-requestresponse).

## <a name="create-the-request-body"></a>Créer le corps de la demande

Les définitions courantes suivantes permettent de générer un corps de demande :

| Nom                       | Obligatoire | Type                                                                                | Description  |
|----------------------------|----------|-------------------------------------------------------------------------------------|--------------|
| location                   | True     | string                                                                              | Emplacement de la ressource. |
| name                       |          | string                                                                              | Nom de la machine virtuelle. |
| properties.hardwareProfile |          | [HardwareProfile](/rest/api/compute/virtualmachines/createorupdate#hardwareprofile) | Spécifie les paramètres matériels de la machine virtuelle. |
| properties.storageProfile  |          | [StorageProfile](/rest/api/compute/virtualmachines/createorupdate#storageprofile)   | Spécifie les paramètres de stockage des disques de machine virtuelle. |
| properties.osProfile       |          | [OSProfile](/rest/api/compute/virtualmachines/createorupdate#osprofile)             | Spécifie les paramètres du système d’exploitation de la machine virtuelle. |
| properties.networkProfile  |          | [NetworkProfile](/rest/api/compute/virtualmachines/createorupdate#networkprofile)   | Spécifie les interfaces réseau de la machine virtuelle. |

Un exemple de corps de requête figure ci-dessous. Veillez à spécifier le nom de la machine virtuelle dans les paramètres `{computerName}` et `{name}`, le nom de l’interface réseau que vous avez créée sous `networkInterfaces`, votre nom d’utilisateur dans `adminUsername` et `path`, et la partie *public* de votre paire de clés SSH (qui se trouve dans, par exemple, `~/.ssh/id_rsa.pub`) dans `keyData`. `location` et `vmSize` figurent parmi les autres paramètres que vous pouvez modifier.  

```json
{
  "location": "eastus",
  "name": "{vmName}",
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_DS1_v2"
    },
    "storageProfile": {
      "imageReference": {
        "sku": "18.04-LTS",
        "publisher": "Canonical",
        "version": "latest",
        "offer": "UbuntuServer"
      },
      "osDisk": {
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS"
        },
        "name": "myVMosdisk",
        "createOption": "FromImage"
      }
    },
    "osProfile": {
      "adminUsername": "{your-username}",
      "computerName": "{vmName}",
      "linuxConfiguration": {
        "ssh": {
          "publicKeys": [
            {
              "path": "/home/{your-username}/.ssh/authorized_keys",
              "keyData": "ssh-rsa AAAAB3NzaC1{snip}mf69/J1"
            }
          ]
        },
        "disablePasswordAuthentication": true
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/{subscription-id}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkInterfaces/{existing-nic-name}",
          "properties": {
            "primary": true
          }
        }
      ]
    }
  }
}
```

Pour obtenir la liste complète des définitions disponibles dans le corps de la demande, consultez [Création ou mise à jour des définitions de corps de demande par les machines virtuelles](/rest/api/compute/virtualmachines/createorupdate#definitions).

## <a name="sending-the-request"></a>Envoi de la requête

Vous pouvez utiliser le client de votre choix pour l’envoi de cette requête HTTP. Vous pouvez également utiliser un [outil dans le navigateur](/rest/api/compute/virtualmachines/createorupdate) en cliquant sur le bouton **Essayer**.

### <a name="responses"></a>Réponses

Il existe deux réponses de réussite pour l’opération de création ou de mise à jour d’une machine virtuelle :

| Nom        | Type                                                                              | Description |
|-------------|-----------------------------------------------------------------------------------|-------------|
| 200 OK      | [VirtualMachine](/rest/api/compute/virtualmachines/createorupdate#virtualmachine) | OK          |
| 201 Créé | [VirtualMachine](/rest/api/compute/virtualmachines/createorupdate#virtualmachine) | Date de création     |

Une réponse *201 Créé* condensée à partir de l’exemple de corps de demande précédent qui crée une machine virtuelle affiche un ID *vmId* assigné et l’état *provisioningState* *Creating* :

```json
{
    "vmId": "e0de9b84-a506-4b95-9623-00a425d05c90",
    "provisioningState": "Creating"
}
```

Pour plus d’informations sur les réponses des API REST, consultez [Process the response message](/rest/api/azure/#process-the-response-message) (Traiter le message de réponse).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les API REST Azure ou d’autres outils de gestion comme Azure CLI ou Azure PowerShell, consultez les rubriques suivantes :

- [Azure Compute provider REST API](/rest/api/compute/) (API REST du fournisseur Calcul Azure)
- [Bien démarrer avec l’API REST Azure](/rest/api/azure/)
- [Azure CLI](/cli/azure/)
- [Module Azure PowerShell](/powershell/azure/)

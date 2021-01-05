---
title: Extension Chef pour les machines virtuelles Azure
description: Déployez le client Chef sur une machine virtuelle à l’aide de l’extension de machine virtuelle Chef.
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 09/21/2018
ms.author: akjosh
ms.openlocfilehash: 8977563c6b19754eda53686baf85f840a7583e77
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968364"
---
# <a name="chef-vm-extension-for-linux-and-windows"></a>Extension de machine virtuelle Chef pour Linux et Windows

Chef Software fournit une plateforme d’automatisation DevOps pour Linux et Windows qui permet la gestion de configurations de serveurs virtuels et physiques. L’extension de machine virtuelle Chef est une extension qui active Chef sur des machines virtuelles.

## <a name="prerequisites"></a>Conditions préalables requises

### <a name="operating-system"></a>Système d’exploitation

L’extension de machine virtuelle Chef est prise en charge sur tous les [systèmes d’exploitation pris en charge par l’extension](https://support.microsoft.com/help/4078134/azure-extension-supported-operating-systems) dans Azure.

### <a name="internet-connectivity"></a>Connectivité Internet

L’extension de machine virtuelle Chef nécessite que la machine virtuelle cible soit connectée à Internet afin de récupérer la charge utile du client Chef à partir du réseau de distribution de contenu (CDN).  

## <a name="extension-schema"></a>Schéma d’extensions

Le JSON suivant illustre le schéma de l’extension de machine virtuelle Chef. L’extension nécessite au minimum l’URL du serveur Chef, le nom du client de validation et la clé de validation pour le serveur Chef. Ces valeurs sont disponibles dans le fichier `knife.rb` dans starter-kit.zip qui est téléchargé quand vous installez [Chef Automate](https://azuremarketplace.microsoft.com/marketplace/apps/chef-software.chef-automate) ou un [serveur Chef](https://downloads.chef.io/chef-server) autonome. Comme la clé de validation doit être traitée comme des données sensibles, elle doit être configurée sous l’élément **protectedSettings**, ce qui signifie qu’elle est déchiffrée uniquement sur la machine virtuelle cible.

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(variables('vmName'),'/', parameters('chef_vm_extension_type'))]",
  "apiVersion": "2017-12-01",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
  ],
  "properties": {
    "publisher": "Chef.Bootstrap.WindowsAzure",
    "type": "[parameters('chef_vm_extension_type')]",
    "typeHandlerVersion": "1210.13",
    "settings": {
      "bootstrap_options": {
        "chef_server_url": "[parameters('chef_server_url')]",
        "validation_client_name": "[parameters('chef_validation_client_name')]"
      },
      "runlist": "[parameters('chef_runlist')]"
    },
    "protectedSettings": {
      "validation_key": "[parameters('chef_validation_key')]"
    }
  }
}  
```

### <a name="core-property-values"></a>Valeurs de propriétés principales

| Name | Valeur/Exemple | Type de données
| ---- | ---- | ----
| apiVersion | `2017-12-01` | string (date) |
| publisher | `Chef.Bootstrap.WindowsAzure` | string |
| type | `LinuxChefClient` (Linux), `ChefClient` (Windows) | string |
| typeHandlerVersion | `1210.13` | string (double) |

### <a name="settings"></a>Paramètres

| Name | Valeur/Exemple | Type de données | Requis ?
| ---- | ---- | ---- | ----
| settings/bootstrap_options/chef_server_url | `https://api.chef.io/organizations/myorg` | string (url) | O |
| settings/bootstrap_options/validation_client_name | `myorg-validator` | string | O |
| settings/runlist | `recipe[mycookbook::default]` | string | O |

### <a name="protected-settings"></a>Paramètres protégés

| Name | Exemple | Type de données | Requis ?
| ---- | ---- | ---- | ---- |
| protectedSettings/validation_key | `-----BEGIN RSA PRIVATE KEY-----\nKEYDATA\n-----END RSA PRIVATE KEY-----` | string | O |

<!--
### Linux-specific settings

| Name | Value / Example | Data Type |
| ---- | ---- | ---- |

### Windows-specific settings

| Name | Value / Example | Data Type |
| ---- | ---- | ---- |
-->

## <a name="template-deployment"></a>Déploiement de modèle

Les extensions de machines virtuelles Azure peuvent être déployées avec des modèles Azure Resource Manager. Vous pouvez utiliser des modèles pour déployer une ou plusieurs machines virtuelles, installer le client Chef, vous connecter au serveur Chef et effectuer la configuration initiale sur le serveur, comme définie par [Run-list](https://docs.chef.io/run_lists.html)

Un exemple de modèle Resource Manager qui inclut l’extension de machine virtuelle Chef est disponible dans la [galerie de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/chef-json-parameters-linux-vm).

La configuration JSON pour une extension de machine virtuelle peut être imbriquée à l’intérieur de la ressource de machine virtuelle ou placée à la racine ou au niveau supérieur d’un modèle de Resource Manager JSON. Le positionnement de la configuration JSON affecte la valeur du nom de la ressource et son type. Pour plus d’informations, consultez [Définition du nom et du type des ressources enfants](../../azure-resource-manager/templates/child-resource-name-type.md).

## <a name="azure-cli-deployment"></a>Déploiement de l’interface de ligne de commande Azure

Vous pouvez utiliser Azure CLI pour déployer l’extension de machine virtuelle Chef sur une machine virtuelle existante. Remplacez **validation_key** par le contenu de votre clé de validation (ce fichier a l’extension `.pem`).  Remplacez **validation_client_name**, **chef_server_url** et **run_list** par les valeurs du fichier `knife.rb` dans votre starter kit.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myExistingVM \
  --name LinuxChefClient \
  --publisher Chef.Bootstrap.WindowsAzure \
  --version 1210.13 --protected-settings '{"validation_key": "<validation_key>"}' \
  --settings '{ "bootstrap_options": { "chef_server_url": "<chef_server_url>", "validation_client_name": "<validation_client_name>" }, "runlist": "<run_list>" }'
```

## <a name="troubleshooting-and-support"></a>Résolution des problèmes et support

Vous pouvez récupérer les données sur l’état des déploiements d’extension à partir du portail Azure et à l’aide de l’interface de ligne de commande Azure. Pour afficher l’état du déploiement des extensions pour une machine virtuelle donnée, exécutez la commande suivante à l’aide de l’interface de ligne de commande Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myExistingVM -o table
```

La sortie de l’exécution de l’extension est enregistrée dans le fichier suivant :

### <a name="linux"></a>Linux

```bash
/var/lib/waagent/Chef.Bootstrap.WindowsAzure.LinuxChefClient
```

### <a name="windows"></a>Windows

```powershell
C:\Packages\Plugins\Chef.Bootstrap.WindowsAzure.ChefClient\
```

### <a name="error-codes-and-their-meanings"></a>Codes d’erreur et signification

| Code d'erreur | Signification | Action possible |
| :---: | --- | --- |
| 51 | Cette extension n’est pas prise en charge sur le système d’exploitation de la machine virtuelle | |

Des informations de dépannage supplémentaires sont disponibles dans le fichier [Readme de l’extension de machine virtuelle Chef](https://github.com/chef-partners/azure-chef-extension).

> [!NOTE]
> Pour toute autre question se rapportant à Chef, contactez le [Support Chef](https://www.chef.io/support/).

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur Obtenir un support. Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

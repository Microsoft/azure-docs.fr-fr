---
title: Extension de l’agent Azure Linux pour Stackify Retrace
description: Déployer l’agent Linux pour Stackify Retrace sur une machine virtuelle Linux.
services: virtual-machines-linux
documentationcenter: ''
author: darinhoward
manager: gwallace
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/12/2018
ms.author: akjosh
ms.openlocfilehash: 0f9ac4f7c88e72cb6ddadc2450947697b0e3c6ef
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961972"
---
# <a name="stackify-retrace-linux-agent-extension"></a>Extension de l’agent Linux pour Stackify Retrace

## <a name="overview"></a>Vue d’ensemble

Stackify fournit des produits qui assurent le suivi d’informations se rapportant à votre application, afin de pouvoir rapidement identifier et résoudre les problèmes. Entièrement intégrée et présente dans de nombreux environnements, Retrace est une superpuissance en matière de performances d’application pour les équipes de développeurs. Elle associe différents outils indispensables à toutes les équipes de développement.

Retrace est le SEUL outil capable de fournir l’ensemble des fonctionnalités suivantes, dans tous les environnements et sur une seule plateforme.

* Gestion des performances des applications (APM)
* Journalisation des applications et des serveurs
* Suivi et analyse des erreurs
* Métriques : des serveurs, des applications et personnalisés

**À propos de l’extension de l’agent Linux Stackify**

Cette extension fournit un chemin d’installation à l’agent Linux pour Retrace. 

## <a name="prerequisites"></a>Conditions préalables requises

### <a name="operating-system"></a>Système d’exploitation 

Vous pouvez exécuter l’agent Retrace par rapport à ces distributions de Linux

| Distribution | Version |
|---|---|
| Ubuntu | 16.04 LTS, 14.04 LTS, 16.10 et 17.04 |
| Debian | 7.9+ et 8.2+, 9 |
| Red Hat | 6.7+, 7.1+ |
| CentOS | 6.3+, 7.0+ |

### <a name="internet-connectivity"></a>Connectivité Internet

La machine virtuelle cible doit être connectée à Internet afin de pouvoir utiliser l’extension de l’agent Stackify pour Linux. 

Vous devrez peut-être ajuster la configuration de votre réseau pour autoriser les connexions à Stackify, consultez https://support.stackify.com/hc/en-us/articles/207891903-Adding-Exceptions-to-a-Firewall. 


## <a name="extension-schema"></a>Schéma d’extensions

---

L’extrait JSON suivant illustre le schéma de l’extension de l’agent Stackify Retrace. L’extension nécessite `environment` et `activationKey`.

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }      
```

## <a name="template-deployment"></a>Déploiement de modèle 

Les extensions de machines virtuelles Azure peuvent être déployées avec des modèles Azure Resource Manager. Le schéma JSON détaillé à la section précédente peut être utilisé dans un modèle Azure Resource Manager, afin d’exécuter l’extension de l’agent Linux pour Stackify Retrace durant un déploiement de modèle Azure Resource Manager.  

Le code JSON pour une extension de machine virtuelle peut être imbriqué à l’intérieur de la ressource de machine virtuelle ou placé à la racine ou au niveau supérieur d’un modèle de Resource Manager JSON. Le positionnement du JSON affecte la valeur du nom de la ressource et son type. Pour plus d’informations, consultez Définir le nom et le type des ressources enfants.

L’exemple suivant suppose que l’extension Linux pour Stackify Retrace est imbriquée dans la ressource de machine virtuelle. Lors de l’imbrication de la ressource d’extension, le code JSON est inséré dans l’objet "resources": [] de la machine virtuelle.

L’extension nécessite `environment` et `activationKey`.

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }      
```

Lorsque vous placez l’extension JSON à la racine du modèle, le nom de ressource inclut une référence à la machine virtuelle parente, et le type reflète la configuration imbriquée.

```json
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "<parentVmResource>/StackifyExtension",
        "apiVersion": "[variables('apiVersion')]",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "properties": {
            "publisher": "Stackify.LinuxAgent.Extension",
            "type": "StackifyLinuxAgentExtension",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
              "environment": "myEnvironment"
            },
            "protectedSettings": {
              "activationKey": "myActivationKey"
            }
        }
    }
```


## <a name="powershell-deployment"></a>Déploiement PowerShell

Vous pouvez utiliser la commande `Set-AzVMExtension` pour déployer l’extension de machine virtuelle de l’agent Stackify Retrace pour Linux sur une machine virtuelle existante. Avant d’exécuter la commande, les configurations publique et privée doivent être stockées dans une table de hachage PowerShell.

L’extension nécessite `environment` et `activationKey`.

```powershell
$PublicSettings = @{"environment" = "myEnvironment"}
$ProtectedSettings = @{"activationKey" = "myActivationKey"}

Set-AzVMExtension -ExtensionName "Stackify.LinuxAgent.Extension" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Stackify.LinuxAgent.Extension" `
    -ExtensionType "StackifyLinuxAgentExtension" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS `
```

## <a name="azure-cli-deployment"></a>Déploiement de l’interface de ligne de commande Azure 

Vous pouvez utiliser l’outil Azure CLI pour déployer l’extension de machine virtuelle de l’agent Stackify Retrace pour Linux sur une machine virtuelle existante.  

L’extension nécessite `environment` et `activationKey`.

```azurecli
az vm extension set --publisher 'Stackify.LinuxAgent.Extension' --version 1.0 --name 'StackifyLinuxAgentExtension' --protected-settings '{"activationKey":"myActivationKey"}' --settings '{"environment":"myEnvironment"}'  --resource-group 'myResourceGroup' --vm-name 'myVmName'
```

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="error-codes"></a>Codes d’erreur

| Code d'erreur | Signification | Action possible |
| :---: | --- | --- |
| 10 | Erreur d’installation | wget est nécessaire |
| 20 | Erreur d’installation | python est nécessaire |
| 30 | Erreur d’installation | sudo est nécessaire |
| 40 | Erreur d’installation | activationKey est nécessaire |
| 51 | Erreur d’installation | Distribution du système d’exploitation non prise en charge |
| 60 | Erreur d’installation | environment est nécessaire |
| 70 | Erreur d’installation | Unknown |
| 80 | Erreur d’activation | Échec d’installation du service |
| 90 | Erreur d’activation | Échec de démarrage du service |
| 100 | Erreur de désactivation | Échec d’arrêt du service |
| 110 | Erreur de désactivation | Échec de suppression du service |
| 120 | Erreur de désinstallation | Échec d’arrêt du service |

Si vous avez besoin d’aide, vous pouvez contacter le support de Stackify sur le site https://support.stackify.com.
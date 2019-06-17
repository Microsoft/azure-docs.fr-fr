---
title: Azure Disk Encryption pour Windows | Microsoft Docs
description: Déploiement d’Azure Disk Encryption sur une machine virtuelle Windows à l’aide d’une extension de machine virtuelle.
services: virtual-machines-windows
documentationcenter: ''
author: ejarvi
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 06/12/2018
ms.author: ejarvi
ms.openlocfilehash: 46699fb1add42d23a11234d5cd05e4a9627a91fd
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/23/2019
ms.locfileid: "60800046"
---
# <a name="azure-disk-encryption-for-windows-microsoftazuresecurityazurediskencryption"></a>Azure Disk Encryption pour Windows (Microsoft.Azure.Security.AzureDiskEncryption)

## <a name="overview"></a>Présentation

Azure Disk Encryption s’appuie sur BitLocker pour fournir un chiffrement de disque complet des machines virtuelles Azure exécutant Windows.  La solution est intégrée à Azure Key Vault, ce qui vous permet de gérer les clés de chiffrement de disque et les secrets dans votre abonnement Key Vault. 

## <a name="prerequisites"></a>Conditions préalables

Pour obtenir la liste complète des conditions requises, consultez la page de [Prérequis pour Azure Disk Encryption](
../../security/azure-security-disk-encryption-prerequisites.md).

### <a name="operating-system"></a>Système d’exploitation

Pour obtenir la liste des versions de Windows actuellement prises en charge, consultez [Prérequis pour Azure Disk Encryption](../../security/azure-security-disk-encryption-prerequisites.md).

### <a name="internet-connectivity"></a>Connectivité Internet

Azure Disk Encryption doit être connecté à Internet pour accéder à Active Directory, à Key Vault, au stockage et aux points de terminaison de gestion des packages.  Pour plus d’informations sur les paramètres de sécurité réseau, consultez [Prérequis pour Azure Disk Encryption](
../../security/azure-security-disk-encryption-prerequisites.md).

## <a name="extension-schema"></a>Schéma d’extensions

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2015-06-15",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientSecret": "[aadClientSecret]",
    },
    "publisher": "Microsoft.Azure.Security",
    "settings": {
      "AADClientID": "[aadClientID]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
          "KekVaultResourceId": "[keyVaultResourceID]",
      
      "KeyVaultURL": "[keyVaultURL]",
          "KeyVaultResourceId": "[keyVaultResourceID]",

      "EncryptionOperation": "[encryptionOperation]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    },
    "type": "AzureDiskEncryption",
    "typeHandlerVersion": "[extensionVersion]"
  }
}
```

### <a name="property-values"></a>Valeurs de propriétés

| Nom | Valeur/Exemple | Type de données |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| publisher | Microsoft.Azure.Security | string |
| type | AzureDiskEncryptionForWindows| string |
| typeHandlerVersion | 1.0, 1.1, 2.2 (VMSS) | int |
| (facultatif) AADClientID | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | GUID | 
| (facultatif) AADClientSecret | password | string |
| (facultatif) AADClientCertificate | thumbprint | string |
| EncryptionOperation | EnableEncryption | string | 
| KeyEncryptionAlgorithm | RSA-OAEP, RSA1_5 | string |
| KeyEncryptionKeyURL | url | string |
| KeyVaultResourceId | uri de ressource | string |
| KekVaultResourceId | uri de ressource | string |
| KeyVaultURL | url | string |
| (facultatif) Passphrase | password | string | 
| SequenceVersion | uniqueidentifier | string |
| VolumeType | Système d’exploitation, données, tout | string |

## <a name="template-deployment"></a>Déploiement de modèle
Pour obtenir un exemple de déploiement de modèle, consultez [ Create a new encrypted Windows VM from gallery image](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-new-vm-gallery-image) (créer une machine virtuelle Windows chiffrée à partir de l’image de la galerie).

## <a name="azure-cli-deployment"></a>Déploiement de l’interface de ligne de commande Azure

Vous trouverez des instructions dans la dernière version de la [documentation Azure CLI](/cli/azure/vm/encryption?view=azure-cli-latest). 

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Résolution des problèmes

Consultez le [Guide de dépannage Azure Disk Encryption](../../security/azure-security-disk-encryption-tsg.md).

### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/community/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur Obtenir un support. Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les extensions de machine virtuelle, consultez la page [Extensions et fonctionnalités de machine virtuelle pour Windows](features-windows.md).

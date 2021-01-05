---
title: Azure Disk Encryption pour Windows
description: Déploiement d’Azure Disk Encryption sur une machine virtuelle Windows à l’aide d’une extension de machine virtuelle.
services: virtual-machines-windows
documentationcenter: ''
author: ejarvi
manager: gwallace
editor: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/19/2020
ms.author: ejarvi
ms.openlocfilehash: e5a0069e02c5285a950d23abc0ec4bee6e9e467b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968381"
---
# <a name="azure-disk-encryption-for-windows-microsoftazuresecurityazurediskencryption"></a>Azure Disk Encryption pour Windows (Microsoft.Azure.Security.AzureDiskEncryption)

## <a name="overview"></a>Vue d’ensemble

Azure Disk Encryption s’appuie sur BitLocker pour fournir un chiffrement de disque complet des machines virtuelles Azure exécutant Windows.  La solution est intégrée à Azure Key Vault, ce qui vous permet de gérer les clés de chiffrement de disque et les secrets dans votre abonnement Key Vault. 

## <a name="prerequisites"></a>Prérequis

Pour obtenir la liste complète des conditions préalables requises, consultez [Azure Disk Encryption pour les machines virtuelles Windows](../windows/disk-encryption-overview.md), en particulier les sections suivantes :

- [Machines virtuelles et systèmes d’exploitation pris en charge](../windows/disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Exigences réseau](../windows/disk-encryption-overview.md#networking-requirements)
- [Exigences de stratégies de groupe](../windows/disk-encryption-overview.md#group-policy-requirements)

## <a name="extension-schema"></a>Schéma de l’extension

Il existe deux versions du schéma d’extension pour Azure Disk Encryption (ADE) :
- v2.2 : nouveau schéma recommandé qui n’utilise pas les propriétés Azure Active Directory (AAD).
- v1.1 : ancien schéma qui exige des propriétés Azure Active Directory (AAD). 

Pour pouvoir sélectionner un schéma cible, vous devez définir la propriété `typeHandlerVersion` sur la version du schéma que vous souhaitez utiliser.

### <a name="schema-v22-no-aad-recommended"></a>Schéma v2.2 : sans AAD (recommandé)

Le schéma v2.2 est recommandé pour toutes les nouvelles machines virtuelles et ne nécessite pas de propriétés Azure Active Directory.

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
        "publisher": "Microsoft.Azure.Security",
        "type": "AzureDiskEncryption",
        "typeHandlerVersion": "2.2",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "EncryptionOperation": "[encryptionOperation]",
          "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
          "KeyVaultURL": "[keyVaultURL]",
          "KekVaultResourceId": "[keyVaultResourceID]",
          "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
          "KeyVaultResourceId": "[keyVaultResourceID]",
          "SequenceVersion": "sequenceVersion]",
          "VolumeType": "[volumeType]"
        }
  }
}
```


### <a name="schema-v11-with-aad"></a>Schéma v1.1 : avec AAD 

Le schéma 1.1 nécessite `aadClientID` et `aadClientSecret` ou `AADClientCertificate`. Il n’est pas recommandé pour les nouvelles machines virtuelles.

Utilisation de `aadClientSecret`:

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientSecret": "[aadClientSecret]"
    },    
    "publisher": "Microsoft.Azure.Security",
    "type": "AzureDiskEncryption",
    "typeHandlerVersion": "1.1",
    "settings": {
      "AADClientID": "[aadClientID]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyVaultURL": "[keyVaultURL]",
      "KeyVaultResourceId": "[keyVaultResourceID]",
      "KekVaultResourceId": "[keyVaultResourceID]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    }
  }
}
```

Utilisation de `AADClientCertificate`:

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientCertificate": "[aadClientCertificate]"
    },    
    "publisher": "Microsoft.Azure.Security",
    "type": "AzureDiskEncryption",
    "typeHandlerVersion": "1.1",
    "settings": {
      "AADClientID": "[aadClientID]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyVaultURL": "[keyVaultURL]",
      "KeyVaultResourceId": "[keyVaultResourceID]",
      "KekVaultResourceId": "[keyVaultResourceID]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    }
  }
}
```


### <a name="property-values"></a>Valeurs de propriétés

| Nom | Valeur/Exemple | Type de données |
| ---- | ---- | ---- |
| apiVersion | 2019-07-01 | Date |
| publisher | Microsoft.Azure.Security | string |
| type | AzureDiskEncryption | string |
| typeHandlerVersion | 2.2, 1.1 | string |
| (schéma 1.1) AADClientID | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | guid | 
| (schéma 1.1) AADClientSecret | mot de passe | string |
| (schéma 1.1) AADClientCertificate | thumbprint | string |
| EncryptionOperation | EnableEncryption, EnableEncryptionFormatAll | string | 
| (facultatif – par défaut, RSA-OAEP) KeyEncryptionAlgorithm | « RSA-OAEP », « RSA-OAEP-256 », « RSA1_5 » | string |
| KeyVaultURL | url | string |
| KeyVaultResourceId | url | string |
| (facultatif) KeyEncryptionKeyURL | url | string |
| (facultatif) KekVaultResourceId | url | string |
| (facultatif) SequenceVersion | UNIQUEIDENTIFIER | string |
| VolumeType | Système d’exploitation, données, tout | string |

## <a name="template-deployment"></a>Déploiement de modèle

Pour un exemple de déploiement de modèle basé sur le schéma v2.2, consultez le modèle de démarrage rapide Azure [201-encrypt-running-windows-vm-without-aad](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-without-aad).

Pour un exemple de déploiement de modèle basé sur le schéma v1.1, consultez le modèle de démarrage rapide Azure [201-encrypt-running-windows-vm](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm).

>[!NOTE]
> Par ailleurs, si le paramètre `VolumeType` a la valeur All, les disques de données ne sont chiffrés que s’ils sont correctement formatés. 

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Dépanner

Pour la résolution des problèmes, reportez-vous au [guide de dépannage Azure Disk Encryption](../windows/disk-encryption-troubleshooting.md).

### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/community/). 

Vous pouvez également signaler un incident au support Azure. Accédez à [Support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez Obtenir de l’aide. Pour plus d’informations sur l’utilisation du support Azure, lisez la [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les extensions de machine virtuelle, consultez la page [Extensions et fonctionnalités de machine virtuelle pour Windows](features-windows.md).
* Pour plus d’informations sur Azure Disk Encryption pour Windows, consultez [Machines virtuelles Windows](../../security/fundamentals/azure-disk-encryption-vms-vmss.md#windows-virtual-machines).

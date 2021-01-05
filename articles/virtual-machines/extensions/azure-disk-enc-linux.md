---
title: Azure Disk Encryption pour Linux
description: Déploiement d’Azure Disk Encryption pour Linux sur une machine virtuelle à l’aide d’une extension de machine virtuelle.
services: virtual-machines-linux
documentationcenter: ''
author: ejarvi
manager: gwallace
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/19/2020
ms.author: ejarvi
ms.openlocfilehash: 61f8fd3d671bbd66d2dc5656e9a467b06798d280
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968415"
---
# <a name="azure-disk-encryption-for-linux-microsoftazuresecurityazurediskencryptionforlinux"></a>Azure Disk Encryption pour Linux (Microsoft.Azure.Security.AzureDiskEncryptionForLinux)

## <a name="overview"></a>Vue d’ensemble

Azure Disk Encryption tire profit du sous-système dm-crypt de Linux pour fournir un chiffrement de disque complet sur [une sélection de distributions Linux Azure](../linux/disk-encryption-overview.md).  Cette solution est intégrée à Azure Key Vault pour gérer les secrets et clés de chiffrement de disque.

## <a name="prerequisites"></a>Prérequis

Pour obtenir la liste complète des conditions préalables requises, consultez [Azure Disk Encryption pour les machines virtuelles Linux](../linux/disk-encryption-overview.md), en particulier les sections suivantes :

- [Machines virtuelles et systèmes d’exploitation pris en charge](../linux/disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Configuration requise supplémentaire pour les machines virtuelles](../linux/disk-encryption-overview.md#additional-vm-requirements)
- [Exigences réseau](../linux/disk-encryption-overview.md#networking-requirements)
- [Exigences liées au stockage des clés de chiffrement](../linux/disk-encryption-overview.md#encryption-key-storage-requirements)

## <a name="extension-schema"></a>Schéma de l’extension

Il existe deux versions du schéma d’extension pour Azure Disk Encryption (ADE) :
- v1.1 : nouveau schéma recommandé qui n’utilise pas les propriétés Azure Active Directory (AAD).
- v0.1 : ancien schéma qui exige des propriétés Azure Active Directory (AAD). 

Pour pouvoir sélectionner un schéma cible, vous devez définir la propriété `typeHandlerVersion` sur la version du schéma que vous souhaitez utiliser.

### <a name="schema-v11-no-aad-recommended"></a>Schéma v1.1 : sans AAD (recommandé)

Le schéma v1.1 est recommandé et n’exige pas de propriétés Azure Active Directory (AAD).

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
        "publisher": "Microsoft.Azure.Security",
        "type": "AzureDiskEncryptionForLinux",
        "typeHandlerVersion": "1.1",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "DiskFormatQuery": "[diskFormatQuery]",
          "EncryptionOperation": "[encryptionOperation]",
          "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
          "KeyVaultURL": "[keyVaultURL]",
          "KeyVaultResourceId": "[KeyVaultResourceId]",
          "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
          "KekVaultResourceId": "[KekVaultResourceId",
          "SequenceVersion": "sequenceVersion]",
          "VolumeType": "[volumeType]"
        }
  }
}
```


### <a name="schema-v01-with-aad"></a>Schéma v0.1 : avec AAD 

Le schéma 0.1 exige `AADClientID` et `AADClientSecret` ou `AADClientCertificate`.

Utilisation de `AADClientSecret`:

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientSecret": "[aadClientSecret]",
      "Passphrase": "[passphrase]"
    },
    "publisher": "Microsoft.Azure.Security",
    "type": "AzureDiskEncryptionForLinux",
    "typeHandlerVersion": "0.1",
    "settings": {
      "AADClientID": "[aadClientID]",
      "DiskFormatQuery": "[diskFormatQuery]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "KeyVaultURL": "[keyVaultURL]",
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
      "AADClientCertificate": "[aadClientCertificate]",
      "Passphrase": "[passphrase]"
    },
    "publisher": "Microsoft.Azure.Security",
    "type": "AzureDiskEncryptionForLinux",
    "typeHandlerVersion": "0.1",
    "settings": {
      "AADClientID": "[aadClientID]",
      "DiskFormatQuery": "[diskFormatQuery]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "KeyVaultURL": "[keyVaultURL]",
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
| type | AzureDiskEncryptionForLinux | string |
| typeHandlerVersion | 1.1, 0.1 | int |
| (schéma 0.1) AADClientID | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | guid | 
| (schéma 0.1) AADClientSecret | mot de passe | string |
| (schéma 0.1) AADClientCertificate | thumbprint | string |
| (facultatif) (schéma 0.1) Passphrase | mot de passe | string |
| DiskFormatQuery | {"dev_path":"","name":"","file_system":""} | Dictionnaire JSON |
| EncryptionOperation | EnableEncryption, EnableEncryptionFormatAll | string | 
| (facultatif – par défaut, RSA-OAEP) KeyEncryptionAlgorithm | « RSA-OAEP », « RSA-OAEP-256 », « RSA1_5 » | string |
| KeyVaultURL | url | string |
| KeyVaultResourceId | url | string |
| (facultatif) KeyEncryptionKeyURL | url | string |
| (facultatif) KekVaultResourceId | url | string |
| (facultatif) SequenceVersion | UNIQUEIDENTIFIER | string |
| VolumeType | Système d’exploitation, données, tout | string |

## <a name="template-deployment"></a>Déploiement de modèle

Pour un exemple de déploiement de modèle basé sur le schéma v1.1, consultez le modèle de démarrage rapide Azure [201-encrypt-running-linux-vm-without-aad](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad).

Pour un exemple de déploiement de modèle basé sur le schéma v0.1, consultez le modèle de démarrage rapide Azure [201-encrypt-running-linux-vm](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm).

>[!WARNING]
> - Si vous avez déjà utilisé Azure Disk Encryption avec Azure AD pour chiffrer une machine virtuelle, vous devez continuer à utiliser cette option pour chiffrer votre machine virtuelle.
> - Lors du chiffrement de volumes de système d’exploitation Linux, la machine virtuelle ne devrait pas être disponible. Nous vous recommandons vivement d'éviter les connexions SSH pendant le chiffrement afin d'éviter tout problème risquant de bloquer les fichiers ouverts qui devront être accessibles pendant le processus de chiffrement. Pour vérifier la progression, utilisez la cmdlet PowerShell [Get-AzVMDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) ou la commande CLI [vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). Ce processus peut prendre quelques heures pour un volume de système d’exploitation de 30 Go, et du temps supplémentaire pour le chiffrement des volumes de données. La durée de chiffrement de volume de données est proportionnelle à la taille et à la quantité des volumes de données, sauf si l’option « EncryptFormatAll » est utilisée. 
> - La désactivation du chiffrement sur les machines virtuelles Linux est prise en charge seulement pour les volumes de données. Elle n’est pas prise en charge sur les volumes de données ou de système d’exploitation si le volume du système d’exploitation a été chiffré. 

>[!NOTE]
> Par ailleurs, si le paramètre `VolumeType` a la valeur All, les disques de données ne sont chiffrés que s’ils sont correctement montés.

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Dépanner

Pour la résolution des problèmes, reportez-vous au [guide de dépannage Azure Disk Encryption](../linux/disk-encryption-troubleshooting.md).

### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/community/). 

Vous pouvez également signaler un incident au support Azure. Accédez à [Support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez Obtenir de l’aide. Pour plus d’informations sur l’utilisation du support Azure, lisez la [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les extensions de machine virtuelle, consultez la page [Extensions et fonctionnalités de machine virtuelle pour Linux](features-linux.md).
* Pour plus d’informations sur Azure Disk Encryption pour Linux, consultez [Machines virtuelles Linux](../../security/fundamentals/azure-disk-encryption-vms-vmss.md#linux-virtual-machines).
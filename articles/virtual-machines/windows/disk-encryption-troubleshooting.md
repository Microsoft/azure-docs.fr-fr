---
title: Guide de rési=olution des problèmes Azure Disk Encryption
description: Cet article contient des conseils de dépannage concernant Microsoft Azure Disk Encryption pour les machines virtuelles Windows.
author: msmbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: troubleshooting
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 45c64261b60d88557e63cb8f9af922fd7aa71ad4
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91977899"
---
# <a name="azure-disk-encryption-troubleshooting-guide"></a>Guide de rési=olution des problèmes Azure Disk Encryption

Ce guide s’adresse aux informaticiens professionnels, aux analystes de la sécurité des informations et aux administrateurs de cloud dont les organisations utilisent Azure Disk Encryption. Cet article est destiné à vous aider à résoudre les problèmes liés au chiffrement des disques.

Avant d’effectuer l’une des étapes ci-dessous, vérifiez que les machines virtuelles que vous tentez de chiffrer sont conformes aux [tailles et systèmes d’exploitation de machine virtuelle pris en charge](disk-encryption-overview.md#supported-vms-and-operating-systems) et que tous les prérequis sont respectés :

- [Exigences réseau](disk-encryption-overview.md#networking-requirements)
- [Exigences de stratégies de groupe](disk-encryption-overview.md#group-policy-requirements)
- [Exigences liées au stockage des clés de chiffrement](disk-encryption-overview.md#encryption-key-storage-requirements)

## <a name="troubleshooting-azure-disk-encryption-behind-a-firewall"></a>Résolution des problèmes Azure Disk Encryption derrière un pare-feu

Lorsque la connectivité est limitée par un pare-feu, une exigence de proxy ou des paramètres de groupe de sécurité réseau (NSG), cela peut interrompre la capacité de l’extension à effectuer les tâches nécessaires. Cette interruption peut entraîner les messages d’état de type « État de l’extension non disponible sur la machine virtuelle ». Dans les scénarios prévus, le processus de chiffrement échoue. Les sections qui suivent décrivent certains problèmes courants au niveau du pare-feu qui valent la peine d’être examinés.

### <a name="network-security-groups"></a>Groupes de sécurité réseau
Les paramètres de groupe de sécurité réseau appliqués doivent permettre au point de terminaison de remplir les [conditions requises](disk-encryption-overview.md#networking-requirements) de configuration réseau indiquées pour le chiffrement de disque.

### <a name="azure-key-vault-behind-a-firewall"></a>Azure Key Vault derrière un pare-feu

Lorsque le chiffrement est activé avec des [informations d’identification Azure AD](disk-encryption-windows-aad.md#), la machine virtuelle cible doit autoriser la connectivité aux points de terminaison Azure Active Directory et à ceux de Key Vault. Les points de terminaison d’authentification Azure Active Directory actuels sont gérés dans les sections 56 et 59 de la documentation [URL et plages d’adresses IP Microsoft 365](/microsoft-365/enterprise/urls-and-ip-address-ranges). Des instructions relatives à Key Vault sont fournies dans la documentation sur l’[accès à Azure Key Vault derrière un pare-feu](../../key-vault/general/access-behind-firewall.md).

### <a name="azure-instance-metadata-service"></a>Service de métadonnées d’instance Azure 
La machine virtuelle doit pouvoir accéder au point de terminaison [Azure Instance Metadata Service](../windows/instance-metadata-service.md) qui utilise une adresse IP non routable bien connue (`169.254.169.254`), accessible uniquement à partir de la machine virtuelle.  Les configurations de proxy qui modifient le trafic HTTP local à cette adresse (par exemple, en ajoutant un en-tête X-Forwarded-For) ne sont pas prises en charge.

## <a name="troubleshooting-windows-server-2016-server-core"></a>Résolution des problèmes Windows Server 2016 Server Core

Sur Windows Server 2016 Server Core, le composant bdehdcfg n’est pas disponible par défaut. Ce composant est requis par Azure Disk Encryption. Il est utilisé pour séparer le volume système du volume du système d’exploitation, ce qui est effectué une seule fois pendant toute la durée de vie de la machine virtuelle. Ces fichiers binaires ne sont pas nécessaires lors des opérations de chiffrement ultérieures.

Pour contourner ce problème, copiez les quatre fichiers suivants à partir d’une machine virtuelle Windows Server 2016 Data Center vers le même emplacement sur Server Core :

   ```
   \windows\system32\bdehdcfg.exe
   \windows\system32\bdehdcfglib.dll
   \windows\system32\en-US\bdehdcfglib.dll.mui
   \windows\system32\en-US\bdehdcfg.exe.mui
   ```

1. Entrez la commande suivante :

   ```
   bdehdcfg.exe -target default
   ```

1. Cette commande crée une partition de système de 550 Mo. Redémarrez le système.

1. Utilisez DiskPart pour vérifier les volumes, avant de continuer.  

Par exemple :

```
DISKPART> list vol

  Volume ###  Ltr  Label        Fs     Type        Size     Status     Info
  ----------  ---  -----------  -----  ----------  -------  ---------  --------
  Volume 0     C                NTFS   Partition    126 GB  Healthy    Boot
  Volume 1                      NTFS   Partition    550 MB  Healthy    System
  Volume 2     D   Temporary S  NTFS   Partition     13 GB  Healthy    Pagefile
```

## <a name="troubleshooting-encryption-status"></a>Résolution des problèmes de l’état du chiffrement 

Le portail peut afficher un disque sous forme chiffrée, même après qu’il a été déchiffré au sein de la machine virtuelle.  Cela peut se produire lorsque des commandes de bas niveau sont utilisées pour déchiffrer directement le disque à partir de la machine virtuelle, au lieu des commandes de gestion Azure Disk Encryption de niveau supérieur.  Les commandes de niveau supérieur déchiffrent le disque à partir de la machine virtuelle. Toutefois, en dehors de la machine virtuelle, elles mettent aussi à jour les paramètres importants de chiffrement et d’extension au niveau de la plateforme, qui sont associés à la machine virtuelle.  S’ils ne sont pas alignés, la plateforme ne sera pas en mesure de rapporter l’état de chiffrement ou de configurer la machine virtuelle.

Pour désactiver Azure Disk Encryption avec PowerShell, utilisez [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption), puis [Remove-AzVMDiskEncryptionExtension](/powershell/module/az.compute/remove-azvmdiskencryptionextension). L’exécution de Remove-AzVMDiskEncryptionExtension en avant la désactivation du chiffrement échouera.

Pour désactiver Azure Disk Encryption avec l’interface CLI, utilisez [az vm encryption disable](/cli/azure/vm/encryption). 

## 



## <a name="next-steps"></a>Étapes suivantes

Ce document vous a fait découvrir certains problèmes couramment rencontrés dans Azure Disk Encryption et en a décrit la résolution. Pour plus d’informations sur ce service et ses fonctionnalités, consultez les articles suivants :

- [Appliquer le chiffrement de disque dans Azure Security Center](../../security-center/asset-inventory.md)
- [Chiffrement des données au repos Azure](../../security/fundamentals/encryption-atrest.md)
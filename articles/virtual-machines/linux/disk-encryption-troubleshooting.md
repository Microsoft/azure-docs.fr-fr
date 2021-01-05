---
title: Dépannage d'Azure Disk Encryption pour les machines virtuelles Linux
description: Cet article contient des conseils de dépannage concernant Microsoft Azure Disk Encryption pour les machines virtuelles Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: troubleshooting
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 0a1bdce0478961b27bde7dd32733d95c58ba2361
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91972235"
---
# <a name="azure-disk-encryption-for-linux-vms-troubleshooting-guide"></a>Guide de résolution des problèmes d’Azure Disk Encryption pour les machines virtuelles Linux

Ce guide s’adresse aux informaticiens professionnels, aux analystes de la sécurité des informations et aux administrateurs de cloud dont les organisations utilisent Azure Disk Encryption. Cet article est destiné à vous aider à résoudre les problèmes liés au chiffrement des disques.

Avant d’effectuer l’une des étapes ci-dessous, vérifiez que les machines virtuelles que vous tentez de chiffrer sont conformes aux [tailles et systèmes d’exploitation de machine virtuelle pris en charge](disk-encryption-overview.md#supported-vms-and-operating-systems) et que tous les prérequis sont respectés :

- [Conditions supplémentaires pour les machines virtuelles](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Exigences réseau](disk-encryption-overview.md#networking-requirements)
- [Exigences liées au stockage des clés de chiffrement](disk-encryption-overview.md#encryption-key-storage-requirements)

 

## <a name="troubleshooting-linux-os-disk-encryption"></a>Résolution des problèmes de chiffrement de disque de système d’exploitation Linux

Pour chiffrer le disque du système d’exploitation (SE) Linux, il faut démonter le lecteur du système d’exploitation avant l’exécution du processus de chiffrement de disque complet. Si le service ne peut pas démonter le lecteur, un message d’erreur du type « Échec du démontage après... » apparaît généralement.

Cette erreur peut se produire lors d’une tentative de chiffrement de disque de système d’exploitation sur une machine virtuelle dont l’image de l’environnement a été modifiée par rapport à celle de la galerie d’images stock (officielles) prises en charge. Les écarts de l’image prise en charge peuvent interférer avec la capacité de l’extension à démonter le lecteur du système d’exploitation. Les exemples d’écarts peuvent inclure les éléments suivants :
- Images personnalisées qui ne correspondent plus au système de fichiers ou au schéma de partitionnement pris en charge.
- Les applications volumineuses, telles que SAP, MongoDB, Apache Cassandra et Docker, ne sont pas prises en charge quand elles sont installées et exécutées dans le système d’exploitation avant le chiffrement. Azure Disk Encryption ne peut pas arrêter en toute sécurité ces processus, comme cela est nécessaire pour la préparation du lecteur du système d’exploitation pour le chiffrement du disque. S’il existe toujours des processus actifs détenant des handles de fichier ouverts sur le lecteur du système d’exploitation, ce lecteur ne peut pas être démonté, ce qui entraîne l’échec du chiffrement du lecteur du système d’exploitation. 
- Les scripts personnalisés qui s’exécutent au moment de l’activation du chiffrement ou en cas de modification apportée sur la machine virtuelle pendant le processus de chiffrement. Ce conflit peut se produire lorsqu’un modèle Azure Resource Manager définit plusieurs extensions à exécuter simultanément ou lorsqu’une extension de script personnalisé ou une autre action est exécutée en même temps que le chiffrement de disque. L’exécution de ces étapes de manière sérialisée et isolée peut résoudre le problème.
- Comme Security Enhanced Linux (SELinux) n’a pas été désactivé avant l’activation du chiffrement, l’étape de démontage échoue. SELinux peut être réactivé à la fin du chiffrement.
- Le disque du système d’exploitation utilise un schéma de Gestionnaire de volume logique (LVM). Même si une prise en charge limitée des disques de données LVM existe, ce n’est pas le cas pour le disque du système d’exploitation LVM.
- Les configurations requises en matière de capacité de mémoire minimale ne sont pas remplies (7 Go sont recommandés pour le chiffrement de disque de système d’exploitation).
- Les lecteurs de données ont été montés de manière récursive sous le répertoire /mnt/ ou les uns sous les autres (par exemple /mnt/data1, /mnt/data2, /data3 + /data3/data4, etc.).

## <a name="update-the-default-kernel-for-ubuntu-1404-lts"></a>Mettre à jour le noyau par défaut pour Ubuntu 14.04 LTS

L’image Ubuntu 14.04 LTS est fournie avec une version de noyau par défaut de 4.4. Cette version de noyau présente un problème connu : le mécanisme Out of Memory Killer met fin de façon inappropriée à la commande dd lors du processus de chiffrement du système d’exploitation. Ce bogue a été corrigé dans le noyau Linux réglé sur Azure le plus récent. Pour éviter cette erreur, avant d’activer le chiffrement sur l’image, effectuez une mise à jour sur le [noyau réglé sur Azure 4.15](https://packages.ubuntu.com/trusty/linux-azure) ou version ultérieure à l’aide des commandes suivantes :

```
sudo apt-get update
sudo apt-get install linux-azure
sudo reboot
```

Après le redémarrage de la machine virtuelle dans le nouveau noyau, la nouvelle version du noyau peut être confirmée à l’aide de :

```
uname -a
```

## <a name="update-the-azure-virtual-machine-agent-and-extension-versions"></a>Mettre à jour l’agent de machine virtuelle Azure et les versions d’extension

Les opérations Azure Disk Encryption peuvent échouer sur les images de machine virtuelle avec des versions non prises en charge de l’agent de machine virtuelle Azure. Les images Linux avec des versions d’agent antérieures à la version 2.2.38 doivent être mises à jour avant l’activation du chiffrement. Pour plus d’informations, consultez [Guide pratique pour mettre à jour l’agent Linux Azure sur une machine virtuelle](../extensions/update-linux-agent.md) et [Prise en charge de version minimale pour les agents de machine virtuelle dans Azure](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

La version correcte de l’extension de l’agent invité Microsoft.Azure.Security.AzureDiskEncryption ou Microsoft.Azure.Security.AzureDiskEncryptionForLinux est également requise. Les versions d’extension sont conservées et mises à jour automatiquement par la plateforme lorsque la configuration requise de l’agent de machine virtuelle Azure est satisfaite et qu’une version prise en charge de l’agent de machine virtuelle est utilisée.

L’extension Microsoft.OSTCExtensions.AzureDiskEncryptionForLinux est déconseillée et n’est plus prise en charge.  

## <a name="unable-to-encrypt-linux-disks"></a>Impossible de chiffrer des disques Linux

Dans certains cas, le chiffrement de disque Linux semble être bloqué à l’étape « OS disk encryption started » (le chiffrement du disque du système d’exploitation a démarré) et SSH est désactivé. Ce processus de chiffrement peut prendre entre 3 et 16 heures pour se terminer sur une image de galerie de stock. Si des disques de données de plusieurs To sont ajoutés, le processus peut prendre des jours.

La séquence de chiffrement de disque du système d’exploitation Linux démonte le lecteur du système d’exploitation temporairement. Il effectue ensuite un chiffrement bloc par bloc de la totalité du disque du système d’exploitation avant de le remonter dans son état chiffré. Le chiffrement de disque Linux n’autorise pas l’utilisation simultanée de la machine virtuelle pendant le chiffrement. Les caractéristiques de performances de la machine virtuelle peuvent faire la différence de façon significative dans le temps nécessaire pour exécuter le chiffrement. Ces caractéristiques incluent la taille du disque et si le compte de stockage est standard ou premium (SSD).

Pendant le chiffrement du lecteur de système d’exploitation, la machine virtuelle passe en état de maintenance et désactive le protocole SSH pour éviter toute interruption du processus en cours.  Pour vérifier l’état du chiffrement, utilisez la commande Azure PowerShell [AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus), puis consultez le champ **ProgressMessage**. **ProgressMessage** rapporte une série d’états au fur et à mesure du chiffrement des disques de données et de système d’exploitation :

```azurepowershell
PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings :
ProgressMessage            : Transitioning

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Encryption succeeded for data volumes

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Provisioning succeeded

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : OS disk encryption started
```

**ProgressMessage** reste dans **Chiffrement du disque de système d’exploitation commencé** pendant la majeure partie du processus de chiffrement.  Une fois le chiffrement terminé et réussi, **ProgressMessage** retourne :

```azurepowershell
PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : Encrypted
DataVolumesEncrypted       : NotMounted
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Encryption succeeded for all volumes
```

Lorsque ce message s’affiche, le lecteur de système d’exploitation chiffré est prêt à être utilisé et la machine virtuelle est de nouveau disponible.

Si les informations de démarrage, le message de progression ou une erreur signalent que le chiffrement du système d’exploitation a échoué au cours de ce processus, restaurez la machine virtuelle sur l’instantané ou la sauvegarde effectuée juste avant le chiffrement. Le message peut-être par exemple l’erreur « Échec du démontage » qui est décrite dans ce guide.

Avant de retenter le chiffrement, réévaluez les caractéristiques de la machine virtuelle et vérifiez que tous les prérequis sont remplis.

## <a name="troubleshooting-azure-disk-encryption-behind-a-firewall"></a>Résolution des problèmes Azure Disk Encryption derrière un pare-feu

Voir [Chiffrement de disque sur un réseau isolé](disk-encryption-isolated-network.md)

## <a name="troubleshooting-encryption-status"></a>Résolution des problèmes de l’état du chiffrement 

Le portail peut afficher un disque sous forme chiffrée, même après qu’il a été déchiffré au sein de la machine virtuelle.  Cela peut se produire lorsque des commandes de bas niveau sont utilisées pour déchiffrer directement le disque à partir de la machine virtuelle, au lieu des commandes de gestion Azure Disk Encryption de niveau supérieur.  Les commandes de niveau supérieur déchiffrent le disque à partir de la machine virtuelle. Toutefois, en dehors de la machine virtuelle, elles mettent aussi à jour les paramètres importants de chiffrement et d’extension au niveau de la plateforme, qui sont associés à la machine virtuelle.  S’ils ne sont pas alignés, la plateforme ne sera pas en mesure de rapporter l’état de chiffrement ou de configurer la machine virtuelle.

Pour désactiver Azure Disk Encryption avec PowerShell, utilisez [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption), puis [Remove-AzVMDiskEncryptionExtension](/powershell/module/az.compute/remove-azvmdiskencryptionextension). L’exécution de Remove-AzVMDiskEncryptionExtension en avant la désactivation du chiffrement échouera.

Pour désactiver Azure Disk Encryption avec l’interface CLI, utilisez [az vm encryption disable](/cli/azure/vm/encryption).

## <a name="next-steps"></a>Étapes suivantes

Ce document vous a fait découvrir certains problèmes couramment rencontrés dans Azure Disk Encryption et en a décrit la résolution. Pour plus d’informations sur ce service et ses fonctionnalités, consultez les articles suivants :

- [Appliquer le chiffrement de disque dans Azure Security Center](../../security-center/asset-inventory.md)
- [Chiffrement des données au repos Azure](../../security/fundamentals/encryption-atrest.md)
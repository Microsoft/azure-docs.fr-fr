---
title: Vérifier l’état du chiffrement pour Linux - Azure Disk Encryption
description: Cet article fournit des instructions sur la vérification de l’état du chiffrement au niveau de la plateforme et du système d’exploitation.
author: kailashmsft
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: how-to
ms.author: kaib
ms.date: 03/11/2020
ms.custom: seodec18, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 7f51aae39c2cb60d8b60d4fb496f74eadb91b33b
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92487651"
---
# <a name="verify-encryption-status-for-linux"></a>Vérifier l’état du chiffrement pour Linux 

L’objectif de cet article est de vérifier l’état du chiffrement d’une machine virtuelle en utilisant différentes méthodes : le portail Azure, PowerShell, Azure CLI ou le système d’exploitation de la machine virtuelle. 

Vous pouvez valider l’état du chiffrement pendant ou après le chiffrement, de l’une des manières suivantes :

- En vérifiant les disques attachés à une machine virtuelle spécifique. 
- En interrogeant les paramètres de chiffrement sur chaque disque, que ce disque soit attaché ou non.

Ce scénario s’applique aux extensions Azure Disk Encryption en une seule passe ou en deux passes. Les distributions Linux sont le seul environnement pour ce scénario.

>[!NOTE] 
>Nous utilisons des variables dans l’ensemble de l’article. Remplacez les valeurs en conséquence.

## <a name="portal"></a>Portail

Dans le portail Azure, à l’intérieur de la section **Extensions** , sélectionnez l’extension Azure Disk Encryption dans la liste. Les informations du **message d’état** indiquent l’état actuel du chiffrement :

![Vérification dans le portail montrant l’état, la version et le message d’état](./media/disk-encryption/verify-encryption-linux/portal-check-001.png)

Dans la liste des extensions, vous verrez la version de l’extension Azure Disk Encryption correspondante. La version 0.x correspond à Azure Disk Encryption en deux passes et la version 1.x correspond à Azure Disk Encryption en une seule passe.

Vous pouvez obtenir plus d’informations en sélectionnant l’extension, puis en sélectionnant **Afficher l’état détaillé** . L’état détaillé du processus de chiffrement apparaît au format JSON.

![Vérification dans le portail montrant le lien « Afficher l’état détaillé »](./media/disk-encryption/verify-encryption-linux/portal-check-002.png)

![État détaillé au format JSON](./media/disk-encryption/verify-encryption-linux/portal-check-003.png)

Pour valider l’état de chiffrement, vous pouvez également consulter la section **Paramètres de disque** .

![État du chiffrement pour le disque du système d’exploitation et les disques de données](./media/disk-encryption/verify-encryption-linux/portal-check-004.png)

>[!NOTE] 
> Cet état indique que les disques ont des paramètres de chiffrement marqués, pas qu’ils ont été réellement chiffrés au niveau du système d’exploitation.
>
> Par défaut, les disques sont d’abord marqués, puis chiffrés ultérieurement. Si le processus de chiffrement échoue, les disques peuvent finir par être marqués, mais pas chiffrés. 
>
> Pour vérifier si les disques sont réellement chiffrés, vous pouvez vérifier le chiffrement de chaque disque au niveau du système d’exploitation.

## <a name="powershell"></a>PowerShell

Vous pouvez valider l’état *général* du chiffrement d’une machine virtuelle chiffrée à l’aide des commandes PowerShell suivantes :

```azurepowershell
   $VMNAME="VMNAME"
   $RGNAME="RGNAME"
   Get-AzVmDiskEncryptionStatus -ResourceGroupName  ${RGNAME} -VMName ${VMNAME}
```
![État général du chiffrement dans PowerShell](./media/disk-encryption/verify-encryption-linux/verify-status-ps-01.png)

Vous pouvez capturer les paramètres de chiffrement de chaque disque à l’aide des commandes PowerShell suivantes.

### <a name="single-pass"></a>Une passe
En une seule passe, les paramètres de chiffrement sont marqués sur chacun des disques (système d’exploitation et données). Vous pouvez capturer les paramètres de chiffrement d’un disque de système d’exploitation en une seule passe comme suit :

``` powershell
$RGNAME = "RGNAME"
$VMNAME = "VMNAME"

$VM = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME}  
 $Sourcedisk = Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName $VM.StorageProfile.OsDisk.Name
 Write-Host "============================================================================================================================================================="
 Write-Host "Encryption Settings:"
 Write-Host "============================================================================================================================================================="
 Write-Host "Enabled:" $Sourcedisk.EncryptionSettingsCollection.Enabled
 Write-Host "Version:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettingsVersion
 Write-Host "Source Vault:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SourceVault.Id
 Write-Host "Secret URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl
 Write-Host "Key URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl
 Write-Host "============================================================================================================================================================="
```
![Paramètres de chiffrement d’un disque de système d’exploitation](./media/disk-encryption/verify-encryption-linux/verify-os-single-ps-001.png)

Si le disque ne comporte pas de paramètres de chiffrement marqués, la sortie est vide :

![Sortie vide](./media/disk-encryption/verify-encryption-linux/os-encryption-settings-2.png)

Utilisez les commandes suivantes pour capturer les paramètres de chiffrement pour les disques de données :

```azurepowershell
$RGNAME = "RGNAME"
$VMNAME = "VMNAME"

$VM = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME}
 clear
 foreach ($i in $VM.StorageProfile.DataDisks|ForEach-Object{$_.Name})
 {
 Write-Host "============================================================================================================================================================="
 Write-Host "Encryption Settings:"
 Write-Host "============================================================================================================================================================="
 Write-Host "Checking Disk:" $i
 $Disk=(Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName $i)
 Write-Host "Encryption Enable: " $Sourcedisk.EncryptionSettingsCollection.Enabled
 Write-Host "Encryption KeyEncryptionKey: " $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl;
 Write-Host "Encryption DiskEncryptionKey: " $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl;
 Write-Host "============================================================================================================================================================="
 }
```
![Paramètres de chiffrement pour les disques de données](./media/disk-encryption/verify-encryption-linux/verify-data-single-ps-001.png)

### <a name="dual-pass"></a>Deux passes
Dans le cas d’une double passe, les paramètres de chiffrement sont marqués sur le modèle de machine virtuelle et non sur chaque disque.

Pour vérifier que les paramètres de chiffrement ont été marqués dans une double passe, utilisez les commandes suivantes :

```azurepowershell
$RGNAME = "RGNAME"
$VMNAME = "VMNAME"

$vm = Get-AzVm -ResourceGroupName ${RGNAME} -Name ${VMNAME};
$Sourcedisk = Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName $VM.StorageProfile.OsDisk.Name
clear
Write-Host "============================================================================================================================================================="
Write-Host "Encryption Settings:"
Write-Host "============================================================================================================================================================="
Write-Host "Enabled:" $Sourcedisk.EncryptionSettingsCollection.Enabled
Write-Host "Version:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettingsVersion
Write-Host "Source Vault:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SourceVault.Id
Write-Host "Secret URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl
Write-Host "Key URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl
Write-Host "============================================================================================================================================================="
```
![Paramètres de chiffrement dans une double passe](./media/disk-encryption/verify-encryption-linux/verify-dual-ps-001.png)

### <a name="unattached-disks"></a>Disques non attachés

Vérifiez les paramètres de chiffrement pour les disques qui ne sont pas attachés à une machine virtuelle.

### <a name="managed-disks"></a>Disques managés
```powershell
$Sourcedisk = Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName ${TARGETDISKNAME}
Write-Host "============================================================================================================================================================="
Write-Host "Encryption Settings:"
Write-Host "============================================================================================================================================================="
Write-Host "Enabled:" $Sourcedisk.EncryptionSettingsCollection.Enabled
Write-Host "Version:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettingsVersion
Write-Host "Source Vault:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SourceVault.Id
Write-Host "Secret URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl
Write-Host "Key URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl
Write-Host "============================================================================================================================================================="
```
## <a name="azure-cli"></a>Azure CLI

Vous pouvez valider l’état *général* du chiffrement d’une machine virtuelle chiffrée à l’aide des commandes d’interface CLI Azure suivantes :

```bash
VMNAME="VMNAME"
RGNAME="RGNAME"
az vm encryption show --name ${VMNAME} --resource-group ${RGNAME} --query "substatus"
```
![État général du chiffrement général avec Azure CLI ](./media/disk-encryption/verify-encryption-linux/verify-gen-cli.png)

### <a name="single-pass"></a>Une passe
Vous pouvez valider les paramètres de chiffrement de chaque disque à l’aide des commandes Azure CLI suivantes :

```bash
az vm encryption show -g ${RGNAME} -n ${VMNAME} --query "disks[*].[name, statuses[*].displayStatus]"  -o table
```

![Paramètres de chiffrement des données](./media/disk-encryption/verify-encryption-linux/data-encryption-settings-2.png)

>[!IMPORTANT]
> Si le disque n’a pas de paramètres de chiffrement marqués, vous pouvez voir le texte **Le disque n’est pas chiffré** .

Utilisez les commandes suivantes pour obtenir l’état détaillé et les paramètres de chiffrement.

Disque de système d’exploitation :

```bash
RGNAME="RGNAME"
VMNAME="VNAME"

disk=`az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.osDisk.name -o tsv`
for disk in $disk; do \
echo "============================================================================================================================================================="
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${disk} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
done
```

![État détaillé et paramètres de chiffrement du disque de système d’exploitation](./media/disk-encryption/verify-encryption-linux/os-single-cli.png)

Disques de données :

```bash
RGNAME="RGNAME"
VMNAME="VMNAME"
az vm encryption show --name ${VMNAME} --resource-group ${RGNAME} --query "substatus"

for disk in `az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.dataDisks[].name -o tsv`; do \
echo "============================================================================================================================================================="; \
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${disk} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
done
```

![État détaillé et paramètres de chiffrement des disques de données](./media/disk-encryption/verify-encryption-linux/data-single-cli.png)

### <a name="dual-pass"></a>Deux passes

``` bash
az vm encryption show --name ${VMNAME} --resource-group ${RGNAME} -o table
```

![Paramètres généraux de chiffrement pour une double passe par le biais d’Azure CLI](./media/disk-encryption/verify-encryption-linux/verify-gen-dual-cli.png)

Vous pouvez également vérifier les paramètres de chiffrement sur le profil de stockage du modèle de machine virtuelle du disque du système d’exploitation :

```bash
disk=`az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.osDisk.name -o tsv`
for disk in $disk; do \
echo "============================================================================================================================================================="; \
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${disk} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
done
```

![Profil de machine virtuelle pour une double passe par le biais d’Azure CLI](./media/disk-encryption/verify-encryption-linux/verify-vm-profile-dual-cli.png)

### <a name="unattached-disks"></a>Disques non attachés

Vérifiez les paramètres de chiffrement pour les disques qui ne sont pas attachés à une machine virtuelle.

### <a name="managed-disks"></a>Disques managés

```bash
RGNAME="RGNAME"
TARGETDISKNAME="DISKNAME"
echo "============================================================================================================================================================="
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
```
### <a name="unmanaged-disks"></a>Disques non managés

Les disques non managés sont des fichiers VHD qui sont stockés en tant qu’objets blob de pages dans des comptes de stockage Azure.

Pour obtenir les détails sur un disque en particulier, vous devez fournir les éléments suivants :

- ID du compte de stockage qui contient le disque.
- Chaîne de connexion de ce compte de stockage.
- Nom du conteneur qui stocke le disque.
- Nom du disque.

Cette commande répertorie tous les ID de tous vos comptes de stockage :

```bash
az storage account list --query [].[id] -o tsv
```
Les ID de compte de stockage sont répertoriés sous la forme suivante :

/subscriptions/\<subscription id>/resourceGroups/\<resource group name>/providers/Microsoft.Storage/storageAccounts/\<storage account name>

Sélectionnez l’ID approprié et stockez-le sur une variable :
```bash
id="/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name>"
```

Cette commande obtient la chaîne de connexion pour un compte de stockage et la stocke sur une variable :

```bash
ConnectionString=$(az storage account show-connection-string --ids $id --query connectionString -o tsv)
```

La commande suivante répertorie tous les conteneurs sous un compte de stockage :
```bash
az storage container list --connection-string $ConnectionString --query [].[name] -o tsv
```
Le conteneur utilisé pour les disques s’appelle normalement « VHD ».

Stockez le nom du conteneur sur une variable : 
```bash
ContainerName="name of the container"
```

Utilisez cette commande pour lister tous les objets blob d’un conteneur particulier :
```bash 
az storage blob list -c ${ContainerName} --connection-string $ConnectionString --query [].[name] -o tsv
```
Choisissez le disque que vous voulez interroger et stockez son nom sur une variable :
```bash
DiskName="diskname.vhd"
```
Interrogez les paramètres de chiffrement du disque :
```bash
az storage blob show -c ${ContainerName} --connection-string ${ConnectionString} -n ${DiskName} --query metadata.DiskEncryptionSettings
```

## <a name="operating-system"></a>Système d’exploitation
Vérifiez si les partitions du disque de données sont chiffrées (et que le disque du système d’exploitation ne l’est pas).

Quand une partition ou un disque sont chiffrés, ils s’affichent en tant que type **crypt** . Non chiffrés, ils s’affichent en tant que type **part/disk** .

``` bash
lsblk
```

![Couche de chiffrement du système d’exploitation pour une partition](./media/disk-encryption/verify-encryption-linux/verify-os-crypt-layer.png)

Vous pouvez obtenir plus d’informations à l’aide de la variante **lsblk** suivante. 

Vous verrez une couche de type **crypt** , qui est montée par l’extension. L’exemple suivant présente des volumes logiques et des disques normaux avec **crypto\_LUKS FSTYPE** .

```bash
lsblk -o NAME,TYPE,FSTYPE,LABEL,SIZE,RO,MOUNTPOINT
```
![Couche de chiffrement du système d’exploitation pour des volumes logiques et des disques normaux](./media/disk-encryption/verify-encryption-linux/verify-os-crypt-layer-2.png)

En guise d’étape supplémentaire, vous pouvez vérifier si des clés sont chargées sur le disque de données :

``` bash
cryptsetup luksDump /dev/VGNAME/LVNAME
```

``` bash
cryptsetup luksDump /dev/sdd1
```

Vous pouvez aussi vérifier quels appareils **dm** sont listés en tant que **crypt**  :

```bash
dmsetup ls --target crypt
```

## <a name="next-steps"></a>Étapes suivantes

- [Résolution des problèmes liés à Azure Disk Encryption](disk-encryption-troubleshooting.md)

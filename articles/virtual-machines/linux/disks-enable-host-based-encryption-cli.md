---
title: Activer le chiffrement de bout en bout à l’aide du chiffrement sur l’hôte - Azure CLI - Disques managés
description: Utilisez le chiffrement sur l’hôte pour activer le chiffrement de bout en bout sur vos disques managés Azure.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 08/24/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 5c0336b80bee1cd5eb76d0ce3d5f99c7296a8467
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499762"
---
# <a name="use-the-azure-cli-to-enable-end-to-end-encryption-using-encryption-at-host"></a>Utilisez Azure CLI pour activer le chiffrement de bout en bout à l’aide du chiffrement sur l’hôte

Quand vous activez le chiffrement sur l’hôte, les données stockées sur l’hôte de machine virtuelle sont chiffrées au repos et les flux sont chiffrés dans le service de stockage. Pour obtenir des informations conceptuelles sur le chiffrement sur l'hôte ainsi que sur d'autres types de chiffrement de disques managés, consultez [Chiffrement sur l'hôte : chiffrement de bout en bout pour vos données de machine virtuelle](../disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

## <a name="restrictions"></a>Restrictions

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../../../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

### <a name="supported-regions"></a>Régions prises en charge

[!INCLUDE [virtual-machines-disks-encryption-at-host-regions](../../../includes/virtual-machines-disks-encryption-at-host-regions.md)]

### <a name="supported-vm-sizes"></a>Tailles des machines virtuelles prises en charge

[!INCLUDE [virtual-machines-disks-encryption-at-host-suported-sizes](../../../includes/virtual-machines-disks-encryption-at-host-suported-sizes.md)]

Vous pouvez également rechercher les tailles de machine virtuelle programmatiquement. Pour savoir comment les récupérer programmatiquement, reportez-vous à la section [Rechercher les tailles de machine virtuelle prises en charge](#finding-supported-vm-sizes).

## <a name="prerequisites"></a>Prérequis

Afin de pouvoir utiliser le chiffrement sur l'hôte pour vos machines virtuelles ou groupes de machines virtuelles identiques, vous devez activer la fonctionnalité dans votre abonnement. Envoyez un e-mail à encryptionAtHost@microsoft.com avec vos ID d’abonnement pour activer la fonctionnalité.

### <a name="create-an-azure-key-vault-and-diskencryptionset"></a>Créer un coffre de clés Azure Key Vault et un jeu de chiffrement DiskEncryptionSet

Une fois la fonctionnalité activée, vous devez configurer un coffre de clés Azure Key Vault et un jeu de chiffrement DiskEncryptionSet, si ce n'est déjà fait.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-cli](../../../includes/virtual-machines-disks-encryption-create-key-vault-cli.md)]

## <a name="examples"></a>Exemples

### <a name="create-a-vm-with-encryption-at-host-enabled-with-customer-managed-keys"></a>Créer une machine virtuelle avec chiffrement sur l’hôte activé avec des clés gérées par le client. 

Créez une machine virtuelle avec disques managés à l’aide de l’URI de ressource du DiskEncryptionSet créé précédemment afin de chiffrer le cache des disques de système d’exploitation et des disques de données avec des clés gérées par le client. Les disques temporaires sont chiffrés avec des clés gérées par la plateforme. 

```azurecli
rgName=yourRGName
vmName=yourVMName
location=eastus
vmSize=Standard_DS2_v2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskEncryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az vm create -g $rgName \
-n $vmName \
-l $location \
--encryption-at-host \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--os-disk-encryption-set $diskEncryptionSetId \
--data-disk-sizes-gb 128 128 \
--data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```

### <a name="create-a-vm-with-encryption-at-host-enabled-with-platform-managed-keys"></a>Créer une machine virtuelle avec chiffrement sur l’hôte activé avec des clés gérées par la plateforme. 

Créez une machine virtuelle avec chiffrement sur l’hôte activé afin de chiffrer le cache des disques de système d’exploitation/de données et des disques temporaires avec des clés gérées par la plateforme. 

```azurecli
rgName=yourRGName
vmName=yourVMName
location=eastus
vmSize=Standard_DS2_v2
image=UbuntuLTS 

az vm create -g $rgName \
-n $vmName \
-l $location \
--encryption-at-host \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--data-disk-sizes-gb 128 128 \
```

### <a name="update-a-vm-to-enable-encryption-at-host"></a>Mettre à jour une machine virtuelle pour activer le chiffrement sur l’hôte. 

```azurecli
rgName=yourRGName
vmName=yourVMName

az vm update -n $vmName \
-g $rgName \
--set securityProfile.encryptionAtHost=true
```

### <a name="check-the-status-of-encryption-at-host-for-a-vm"></a>Vérifier l’état du chiffrement sur l’hôte pour une machine virtuelle

```azurecli
rgName=yourRGName
vmName=yourVMName

az vm show -n $vmName \
-g $rgName \
--query [securityProfile.encryptionAtHost] -o tsv
```

### <a name="create-a-virtual-machine-scale-set-with-encryption-at-host-enabled-with-customer-managed-keys"></a>Créer un groupe de machines virtuelles identiques avec chiffrement sur l’hôte activé avec des clés gérées par le client. 

Créez un groupe de machines virtuelles identiques avec disques managés à l’aide de l’URI de ressource du DiskEncryptionSet créé précédemment afin de chiffrer le cache des disques de système d’exploitation et des disques de données avec des clés gérées par le client. Les disques temporaires sont chiffrés avec des clés gérées par la plateforme. 

```azurecli
rgName=yourRGName
vmssName=yourVMSSName
location=westus2
vmSize=Standard_DS3_V2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskEncryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az vmss create -g $rgName \
-n $vmssName \
--encryption-at-host \
--image UbuntuLTS \
--upgrade-policy automatic \
--admin-username azureuser \
--generate-ssh-keys \
--os-disk-encryption-set $diskEncryptionSetId \
--data-disk-sizes-gb 64 128 \
--data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```

### <a name="create-a-virtual-machine-scale-set-with-encryption-at-host-enabled-with-platform-managed-keys"></a>Créer un groupe de machines virtuelles identiques avec chiffrement sur l’hôte activé avec des clés gérées par la plateforme. 

Créez un groupe de machines virtuelles identiques avec chiffrement sur l’hôte activé afin de chiffrer le cache des disques de système d’exploitation/de données et des disques temporaires avec des clés gérées par la plateforme. 

```azurecli
rgName=yourRGName
vmssName=yourVMSSName
location=westus2
vmSize=Standard_DS3_V2
image=UbuntuLTS 

az vmss create -g $rgName \
-n $vmssName \
--encryption-at-host \
--image UbuntuLTS \
--upgrade-policy automatic \
--admin-username azureuser \
--generate-ssh-keys \
--data-disk-sizes-gb 64 128 \
```

### <a name="update-a-virtual-machine-scale-set-to-enable-encryption-at-host"></a>Mettre à jour un groupe de machines virtuelles identiques pour activer le chiffrement sur l’hôte. 

```azurecli
rgName=yourRGName
vmssName=yourVMName

az vmss update -n $vmssName \
-g $rgName \
--set virtualMachineProfile.securityProfile.encryptionAtHost=true
```

### <a name="check-the-status-of-encryption-at-host-for-a-virtual-machine-scale-set"></a>Vérifier l’état du chiffrement sur l’hôte pour un groupe de machines virtuelles identiques

```azurecli
rgName=yourRGName
vmssName=yourVMName

az vmss show -n $vmssName \
-g $rgName \
--query [virtualMachineProfile.securityProfile.encryptionAtHost] -o tsv
```

## <a name="finding-supported-vm-sizes"></a>Recherche des tailles de machine virtuelle prises en charge

Les tailles de machine virtuelle héritées ne sont pas prises en charge. Pour accéder à la liste des tailles de machines virtuelles prises en charge, procédez comme suit :

Appelez l'[API des références SKU de ressource](/rest/api/compute/resourceskus/list) et vérifiez que la capacité `EncryptionAtHostSupported` est définie sur **True**.

```json
    {
        "resourceType": "virtualMachines",
        "name": "Standard_DS1_v2",
        "tier": "Standard",
        "size": "DS1_v2",
        "family": "standardDSv2Family",
        "locations": [
        "CentralUSEUAP"
        ],
        "capabilities": [
        {
            "name": "EncryptionAtHostSupported",
            "value": "True"
        }
        ]
    }
```

Ou appelez la cmdlet PowerShell [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku?view=azps-3.8.0).

```powershell
$vmSizes=Get-AzComputeResourceSku | where{$_.ResourceType -eq 'virtualMachines' -and $_.Locations.Contains('CentralUSEUAP')} 

foreach($vmSize in $vmSizes)
{
    foreach($capability in $vmSize.capabilities)
    {
        if($capability.Name -eq 'EncryptionAtHostSupported' -and $capability.Value -eq 'true')
        {
            $vmSize

        }

    }
}
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez créé et configuré ces ressources, vous pouvez les utiliser pour sécuriser vos disques managés. Le lien suivant contient des exemples de scripts, chacun correspondant à un scénario différent, que vous pouvez utiliser pour sécuriser vos disques managés.

[Exemples de modèles Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/EncryptionAtHost)
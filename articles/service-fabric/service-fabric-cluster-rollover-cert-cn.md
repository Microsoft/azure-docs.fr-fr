---
title: Remplacer un certificat de cluster Azure Service Fabric
description: Découvrez comment renouveler un certificat de cluster Service Fabric identifié par le nom commun du certificat.
ms.topic: conceptual
ms.date: 09/06/2019
ms.openlocfilehash: 65ea4f463073c472ac6a31e62dcfdfd11cb28cc5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88853348"
---
# <a name="manually-roll-over-a-service-fabric-cluster-certificate"></a>Renouveler manuellement un certificat de cluster Service Fabric
Lorsqu’un certificat de cluster Service Fabric est proche de la date d’expiration, vous devez le mettre à jour.  Le renouvellement de certificat est un processus simple si le cluster a été [configuré pour utiliser des certificats basés sur un nom commun](service-fabric-cluster-change-cert-thumbprint-to-cn.md) (et pas une empreinte).  Obtenez un nouveau certificat auprès d’une autorité de certification avec une nouvelle date d’expiration.  Les certificats auto-signés ne sont pas pris en charge avec les clusters Service Fabric de production, afin d’inclure les certificats générés dans le cadre du workflow de création de cluster dans le portail Azure. Le nouveau certificat doit présenter le même nom commun que l’ancien certificat. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Lorsque plusieurs certificats valides sont installés sur l’ordinateur hôte, le cluster Service Fabric utilise automatiquement le certificat déclaré dont la date d’expiration est la plus éloignée. L’utilisation d’un modèle Resource Manager pour provisionner les ressources Azure constitue une bonne pratique. Dans un environnement autre que de production, le script suivant peut être utilisé pour charger un nouveau certificat dans un coffre de clés, puis pour installer le certificat sur le groupe de machines virtuelles identiques : 

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -Force

$SubscriptionId  =  <subscription ID>

# Sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId $SubscriptionId

$region = "southcentralus"
$KeyVaultResourceGroupName  = "keyvaultgroup"
$VaultName = "cntestvault2"
$certFilename = "C:\users\sfuser\sftutorialcluster20180419110824.pfx"
$certname = "cntestcert"
$Password  = "!P@ssw0rd321"
$VmssResourceGroupName     = "sfclustertutorialgroup"
$VmssName                  = "prnninnxj"

# Create new Resource Group 
New-AzResourceGroup -Name $KeyVaultResourceGroupName -Location $region

# Get the key vault.  The key vault must be enabled for deployment.
$keyVault = Get-AzKeyVault -VaultName $VaultName -ResourceGroupName $KeyVaultResourceGroupName 
$resourceId = $keyVault.ResourceId  

# Add the certificate to the key vault.
$PasswordSec = ConvertTo-SecureString -String $Password -AsPlainText -Force
$KVSecret = Import-AzKeyVaultCertificate -VaultName $vaultName -Name $certName  -FilePath $certFilename -Password $PasswordSec

$CertificateThumbprint = $KVSecret.Thumbprint
$CertificateURL = $KVSecret.SecretId
$SourceVault = $resourceId
$CommName    = $KVSecret.Certificate.SubjectName.Name

Write-Host "CertificateThumbprint    :"  $CertificateThumbprint
Write-Host "CertificateURL           :"  $CertificateURL
Write-Host "SourceVault              :"  $SourceVault
Write-Host "Common Name              :"  $CommName    

Set-StrictMode -Version 3
$ErrorActionPreference = "Stop"

$certConfig = New-AzVmssVaultCertificateConfig -CertificateUrl $CertificateURL -CertificateStore "My"

# Get current VM scale set 
$vmss = Get-AzVmss -ResourceGroupName $VmssResourceGroupName -VMScaleSetName $VmssName

# Add new secret to the VM scale set.
$vmss.VirtualMachineProfile.OsProfile.Secrets[0].VaultCertificates.Add($certConfig)

# Update the VM scale set 
Update-AzVmss -ResourceGroupName $VmssResourceGroupName -Name $VmssName -VirtualMachineScaleSet $vmss  -Verbose
```

>[!NOTE]
> Vous ne pouvez pas utiliser un même ID de ressource pour deux secrets de groupes de machines virtuelles identiques, puisque chaque secret constitue une ressource unique avec sa propre version. 

## <a name="next-steps"></a>Étapes suivantes

* Découvrez plus en détail la [sécurité des clusters](service-fabric-cluster-security.md).
* [Mettre à jour et gérer les certificats de cluster](service-fabric-cluster-security-update-certs-azure.md)

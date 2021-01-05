---
title: Activer le chiffrement de disque pour des clusters Windows
description: Cet article décrit comment activer le chiffrement de disque pour les nœuds de cluster Azure Service Fabric en utilisant Azure Key Vault dans Azure Resource Manager.
ms.topic: article
ms.date: 03/22/2019
ms.openlocfilehash: b08cdb63aa6f334c5a6f7c230b1624d232206c3b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78251816"
---
# <a name="enable-disk-encryption-for-azure-service-fabric-cluster-nodes-in-windows"></a>Activer le chiffrement de disque pour les nœuds de cluster Azure Service Fabric dans Windows 
> [!div class="op_single_selector"]
> * [Chiffrement de disque pour Windows](service-fabric-enable-azure-disk-encryption-windows.md)
> * [Chiffrement de disque pour Linux](service-fabric-enable-azure-disk-encryption-linux.md)
>
>

Dans ce didacticiel, vous allez apprendre à activer le chiffrement de disque sur les nœuds de cluster Azure Service Fabric dans Windows. Vous devez effectuer ces étapes pour chacun des types de nœuds et groupes de machines virtuelles identiques. Pour chiffrer les nœuds, nous allons utiliser les fonctionnalités d’Azure Disk Encryption sur les groupes de machines virtuelles identiques.

Le guide aborde les thèmes suivants :

* Concepts clés à connaître lors de l’activation du chiffrement de disque sur des groupes de machines virtuelles identiques de cluster Service Fabric dans Windows.
* Étapes à suivre avant d’activer le chiffrement de disque sur les nœuds de cluster Service Fabric dans Windows.
* Étapes à suivre pour activer le chiffrement de disque sur les nœuds de cluster Service Fabric dans Windows.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Conditions préalables requises

**Inscription automatique** 

La préversion du chiffrement de disque des groupes de machines virtuelles identiques nécessite une inscription automatique. Utiliser les étapes suivantes : 

1. Tout d’abord, exécutez la commande ci-dessous :
    ```powershell
    Register-AzProviderFeature -ProviderNamespace Microsoft.Compute -FeatureName "UnifiedDiskEncryption"
    ```
2. Attendez 10 minutes environ jusqu’à ce que l’état indique *Inscrit*. Vous pouvez vérifier l’état en exécutant la commande suivante : 
    ```powershell
    Get-AzProviderFeature -ProviderNamespace "Microsoft.Compute" -FeatureName "UnifiedDiskEncryption"
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    ```
**Azure Key Vault** 

1. Créez un coffre de clés dans le même abonnement et la même région que le groupe identique, puis sélectionnez la stratégie d’accès **EnabledForDiskEncryption** dans le coffre de clés à l’aide de son applet de commande PowerShell. Vous pouvez également définir la stratégie à l’aide de l’interface utilisateur Key Vault dans le Portail Azure à l’aide de la commande suivante :
    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -EnabledForDiskEncryption
    ```
2. Installez la dernière version d’[Azure CLI](/cli/azure/install-azure-cli), qui inclut les nouvelles commandes de chiffrement.
3. Installez la dernière version du [Kit de développement logiciel (SDK) Azure d’Azure PowerShell](https://github.com/Azure/azure-powershell/releases). Voici les cmdlets Azure Disk Encryption de groupe de machines virtuelles identiques pour activer ([set](/powershell/module/az.compute/set-azvmssdiskencryptionextension)) le chiffrement, récupérer ([get](/powershell/module/az.compute/get-azvmssvmdiskencryption)) l’état du chiffrement et supprimer ([disable](/powershell/module/az.compute/disable-azvmssdiskencryption)) le chiffrement sur une instance de groupe identique.

| Commande | Version |  Source  |
| ------------- |-------------| ------------|
| Get-AzVmssDiskEncryptionStatus   | 1.0.0 ou ultérieure | Az.Compute |
| Get-AzVmssVMDiskEncryptionStatus   | 1.0.0 ou ultérieure | Az.Compute |
| Disable-AzVmssDiskEncryption   | 1.0.0 ou ultérieure | Az.Compute |
| Get-AzVmssDiskEncryption   | 1.0.0 ou ultérieure | Az.Compute |
| Get-AzVmssVMDiskEncryption   | 1.0.0 ou ultérieure | Az.Compute |
| Set-AzVmssDiskEncryptionExtension   | 1.0.0 ou ultérieure | Az.Compute |


## <a name="supported-scenarios-for-disk-encryption"></a>Scénarios pris en charge pour le chiffrement de disque
* Le chiffrement de groupes de machines virtuelles identiques est pris en charge uniquement pour les groupes identiques créés avec des disques managés. Il ne l’est pas pour les groupes identiques de disques natifs (ou non managés).
* Le chiffrement est pris en charge pour les volumes de données et de systèmes d’exploitation dans les groupes de machines virtuelles identiques dans Windows. Le chiffrement de désactivation est également pris en charge pour les volumes de données et de systèmes d’exploitation dans les groupes de machines virtuelles identiques dans Windows.
* Les opérations de réimageage et de mise à niveau de machine virtuelle de groupes de machines virtuelles identiques ne sont pas prises en charge dans la préversion actuelle.


## <a name="create-a-new-cluster-and-enable-disk-encryption"></a>Créer un cluster et activer le chiffrement de disque

Utilisez les commandes suivantes pour créer un cluster et activer le chiffrement de disque à l’aide d’un modèle Azure Resource Manager et d’un certificat auto-signé.

### <a name="sign-in-to-azure"></a>Connexion à Azure 
Connectez-vous à l’aide des commandes suivantes :
```powershell
Login-AzAccount
Set-AzContext -SubscriptionId <guid>

```

```azurecli

azure login
az account set --subscription $subscriptionId

```

### <a name="use-the-custom-template-that-you-already-have"></a>Utiliser le modèle personnalisé que vous possédez déjà 

Si vous avez besoin de créer un modèle personnalisé adapté à vos besoins, nous vous recommandons vivement de commencer avec l’un des modèles disponibles dans la page [Exemples de modèles de création de cluster Azure Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master). Pour [personnaliser votre modèle de cluster][customize-your-cluster-template], suivez les instructions ci-après.

Si vous disposez déjà d’un modèle personnalisé, vérifiez bien que les trois paramètres liés au certificat dans le modèle et le fichier de paramètres portent les noms indiqués ci-après et que ces valeurs sont null comme suit :

```Json
   "certificateThumbprint": {
      "value": ""
    },
    "sourceVaultValue": {
      "value": ""
    },
    "certificateUrlValue": {
      "value": ""
    },
```


```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$CertSubjectName="mycluster.westus.cloudapp.azure.com"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$certOutputFolder="c:\certificates"

$parameterFilePath="c:\templates\templateparam.json"
$templateFilePath="c:\templates\template.json"


New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -CertificateOutputFolder $certOutputFolder -CertificatePassword $certpassword -CertificateSubjectName $CertSubjectName -TemplateFile $templateFilePath -ParameterFile $parameterFilePath 

```


```azurecli

declare certPassword=""
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare certSubjectName="mylinuxsecure.westus.cloudapp.azure.com"
declare parameterFilePath="c:\mytemplates\linuxtemplateparm.json"
declare templateFilePath="c:\mytemplates\linuxtemplate.json"
declare certOutputFolder="c:\certificates"


az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-output-folder $certOutputFolder --certificate-password $certPassword  \
    --certificate-subject-name $certSubjectName \
    --template-file $templateFilePath --parameter-file $parametersFilePath

```

### <a name="deploy-an-application-to-a-service-fabric-cluster-in-windows"></a>Déployer une application sur un cluster Service Fabric dans Windows
Pour déployer une application sur votre cluster, suivez les étapes et les instructions dans [Déployer et supprimer des applications avec PowerShell](service-fabric-deploy-remove-applications.md).


### <a name="enable-disk-encryption-for-the-virtual-machine-scale-sets-created-previously"></a>Activer le chiffrement de disque pour le groupe de machines virtuelles identiques créé précédemment

Pour activer le chiffrement de disque pour les groupes de machines virtuelles identiques que vous avez créés dans les étapes précédentes, exécutez les commandes suivantes :
 
```powershell

$VmssName = "nt1vm"
$vaultName = "mykeyvault"
$resourceGroupName = "mycluster"
$KeyVault = Get-AzKeyVault -VaultName $vaultName -ResourceGroupName $rgName
$DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri
$KeyVaultResourceId = $KeyVault.ResourceId

Set-AzVmssDiskEncryptionExtension -ResourceGroupName $resourceGroupName -VMScaleSetName $VmssName -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType All

```

```azurecli

az vmss encryption enable -g <resourceGroupName> -n <VMSS name> --disk-encryption-keyvault <KeyVaultResourceId>

```


### <a name="validate-if-disk-encryption-is-enabled-for-a-virtual-machine-scale-set-in-windows"></a>Valider si le chiffrement de disque est activé pour un groupe de machines virtuelles identiques dans Windows
Obtenez l’état de l’ensemble d’un groupe de machines virtuelles identiques ou d’une instance d’un groupe identique en exécutant les commandes suivantes.

```powershell

$VmssName = "nt1vm"
$resourceGroupName = "mycluster"
Get-AzVmssDiskEncryption -ResourceGroupName $resourceGroupName -VMScaleSetName $VmssName

Get-AzVmssVMDiskEncryption -ResourceGroupName $resourceGroupName -VMScaleSetName $VmssName -InstanceId "0"

```

```azurecli

az vmss encryption show -g <resourceGroupName> -n <VMSS name>

```


De plus, vous pouvez vous connecter au groupe de machines virtuelles identiques et vous assurer que les disques sont chiffrés.

### <a name="disable-disk-encryption-for-a-virtual-machine-scale-set-in-a-service-fabric-cluster"></a>Désactiver le chiffrement de disque pour un groupe de machines virtuelles identiques dans un cluster Service Fabric 
Désactivez le chiffrement de disque pour un groupe de machines virtuelles identiques en exécutant les commandes suivantes. Notez que la désactivation du chiffrement de disque s’applique à l’ensemble du groupe de machines virtuelles identiques et non par instance individuelle.

```powershell

$VmssName = "nt1vm"
$resourceGroupName = "mycluster"
Disable-AzVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $VmssName

```

```azurecli

az vmss encryption disable -g <resourceGroupName> -n <VMSS name>

```


## <a name="next-steps"></a>Étapes suivantes
À ce stade, vous devez avoir un cluster sécurisé et savoir comment activer et désactiver le chiffrement de disque pour des nœuds de cluster Service Fabric et des groupes de machines virtuelles identiques. Pour obtenir des recommandations similaires sur les nœuds de cluster Service Fabric dans Windows, consultez [Chiffrement de disque pour Linux](service-fabric-enable-azure-disk-encryption-linux.md).

[customize-your-cluster-template]: https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure#creating-a-custom-arm-template

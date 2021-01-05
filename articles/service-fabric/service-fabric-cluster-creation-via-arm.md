---
title: Créer un cluster Azure Service Fabric
description: Découvrez comment configurer un cluster Service Fabric sécurisé dans Azure à l’aide d’Azure Resource Manager.  Vous pouvez créer un cluster à l’aide d’un modèle par défaut ou à l’aide de votre propre modèle de cluster.
ms.topic: conceptual
ms.date: 08/16/2018
ms.openlocfilehash: fd2c472d24f305e42f1706e5fc49168ccde2a580
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008665"
---
# <a name="create-a-service-fabric-cluster-using-azure-resource-manager"></a>Créer un cluster Service Fabric à l’aide d’Azure Resource Manager 
> [!div class="op_single_selector"]
> * [Azure Resource Manager](service-fabric-cluster-creation-via-arm.md)
> * [Azure portal](service-fabric-cluster-creation-via-portal.md)
>
>

Un [cluster Azure Service Fabric](service-fabric-deploy-anywhere.md) est un ensemble de machines virtuelles connecté au réseau, sur lequel vos microservices sont déployés et gérés.  Un cluster Service Fabric s’exécutant dans Azure est une ressource Azure et il est déployé à l’aide d’Azure Resource Manager. Cet article décrit comment déployer un cluster Service Fabric sécurisé dans Azure à l’aide du gestionnaire des ressources. Vous pouvez utiliser un modèle de cluster par défaut ou personnalisé.  Si vous ne disposez pas d’un modèle personnalisé, vous pouvez [apprendre à en créer un](service-fabric-cluster-creation-create-template.md).

Le type de sécurité choisi pour sécuriser le cluster (p. ex. : identité Windows, X509 etc.) doit être spécifié lors de la création initiale du cluster, et ne peut être modifié par la suite. Avant de configurer un cluster, lisez les [scénarios de sécurité du cluster Service Fabric][service-fabric-cluster-security]. Dans Azure, Service Fabric utilise un certificat x509 pour sécuriser votre cluster et ses points de terminaison, authentifier les clients et chiffrer les données. Azure Active Directory est également recommandé pour sécuriser l’accès aux points de terminaison de gestion. Pour plus d’informations, consultez [Configurer Azure AD pour authentifier les clients](service-fabric-cluster-creation-setup-aad.md).

Si vous créez un cluster de production pour exécuter des charges de travail de production, nous vous recommandons de commencer par lire la [liste de vérification de disponibilité de la production](service-fabric-production-readiness-checklist.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis 
Dans cet article, utilisez les modules de gestionnaire de ressources Service Fabric Powershell ou Azure CLI pour déployer un cluster :

* [Azure PowerShell 4.1 et versions ultérieures][azure-powershell]
* [Azure CLI 2.0 et versions ultérieures][azure-CLI]

Vous trouverez la documentation de référence pour les modules Service Fabric ici :
* [Az.ServiceFabric](/powershell/module/az.servicefabric)
* [module CLI AZ SF](/cli/azure/sf?view=azure-cli-latest)

### <a name="sign-in-to-azure"></a>Connexion à Azure

Avant d’exécuter les commandes dans cet article, connectez-vous à Azure.

```powershell
Connect-AzAccount
Set-AzContext -SubscriptionId <subscriptionId>
```

```azurecli
az login
az account set --subscription $subscriptionId
```

## <a name="create-a-new-cluster-using-a-system-generated-self-signed-certificate"></a>Créer un cluster à l’aide d’un certificat auto-signé généré par le système

Utilisez les commandes suivantes pour créer un cluster sécurisé avec un certificat auto-signé généré par le système. Cette commande configure un certificat de cluster principal qui est utilisé pour la sécurité du cluster et pour configurer l’accès administrateur et effectuer des opérations de gestion à l’aide de ce certificat.  Les certificats auto-signés sont utiles pour sécuriser les clusters de test.  Les clusters de production doivent être sécurisés avec un certificat délivré par une autorité de certification (AC).

### <a name="use-the-default-cluster-template-that-ships-in-the-module"></a>Utiliser le modèle de cluster par défaut qui est fourni dans le module

Utiliser la commande suivante pour créer un cluster rapidement, en spécifiant des paramètres minimums et en utilisant le modèle par défaut.

Le modèle utilisé est disponible dans les [exemples de modèles Azure Service Fabric : modèle Windows](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure-NSG) et le [modèle Ubuntu](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Ubuntu-1-NodeTypes-Secure)

La commande suivante peut créer des clusters Windows ou Linux, vous devez spécifier le système d’exploitation. Les commandes PowerShell/CLI génèrent également le certificat dans *CertificateOutputFolder* spécifié. Vérifiez toutefois que le dossier Certificat est déjà créé. La commande accepte d’autres paramètres comme Référence de la machine virtuelle.

> [!NOTE]
> La commande PowerShell suivante fonctionne uniquement avec le module Azure PowerShell `Az`. Pour vérifier la version actuelle de PowerShell Azure Resource Manager, exécutez la commande PowerShell « Get-Module Az ». Suivez [ce lien](/powershell/azure/install-Az-ps) pour mettre à niveau votre version de PowerShell Azure Resource Manager. 
>
>

Déployer le cluster à l’aide de PowerShell :

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$vaultName="myvault"
$vaultResourceGroupName="myvaultrg"
$CertSubjectName="mycluster.westus.cloudapp.azure.com"
$certPassword="Password123!@#" | ConvertTo-SecureString -AsPlainText -Force 
$vmpassword="Password4321!@#" | ConvertTo-SecureString -AsPlainText -Force
$vmuser="myadmin"
$os="WindowsServer2016DatacenterwithContainers"
$certOutputFolder="c:\certificates"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -Location $resourceGroupLocation -CertificateOutputFolder $certOutputFolder -CertificatePassword $certpassword -CertificateSubjectName $CertSubjectName -OS $os -VmPassword $vmpassword -VmUserName $vmuser
```

Déployer le cluster à l’aide d’Azure CLI :

```azurecli
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare vaultResourceGroupName="myvaultrg"
declare vaultName="myvault"
declare CertSubjectName="mylinux.westus.cloudapp.azure.com"
declare vmpassword="Password!1"
declare certpassword="Password!4321"
declare vmuser="myadmin"
declare vmOs="UbuntuServer1604"
declare certOutputFolder="c:\certificates"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-output-folder $certOutputFolder --certificate-password $certpassword  \
    --vault-name $vaultName --vault-resource-group $resourceGroupName  \
    --template-file $templateFilePath --parameter-file $parametersFilePath --vm-os $vmOs  \
    --vm-password $vmpassword --vm-user-name $vmuser
```

### <a name="use-your-own-custom-template"></a>Utiliser votre propre modèle personnalisé

Si vous avez besoin de créer un modèle personnalisé adapté à vos besoins, il est vivement recommandé de commencer par l’un des modèles disponibles dans les [exemples de modèles Azure Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master). Découvrez comment [personnaliser votre modèle de cluster][customize-your-cluster-template].

Si vous disposez déjà d’un modèle personnalisé, vérifiez bien que les trois paramètres liés au certificat dans le modèle et le fichier de paramètres portent les noms indiqués ci-après et que les valeurs sont null comme suit :

```json
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

Déployer le cluster à l’aide de PowerShell :

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$CertSubjectName="mycluster.westus.cloudapp.azure.com"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$certOutputFolder="c:\certificates"

$parameterFilePath="c:\mytemplates\mytemplateparm.json"
$templateFilePath="c:\mytemplates\mytemplate.json"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -CertificateOutputFolder $certOutputFolder -CertificatePassword $certpassword -CertificateSubjectName $CertSubjectName -TemplateFile $templateFilePath -ParameterFile $parameterFilePath 
```

Déployer le cluster à l’aide d’Azure CLI :

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

## <a name="create-a-new-cluster-using-your-own-x509-certificate"></a>Créer un cluster à l’aide de votre propre certificat X.509

Utilisez la commande suivante pour créer un cluster, si vous disposez d’un certificat que vous souhaitez utiliser pour sécuriser votre cluster.

S’il s’agit d’un certificat signé d’une autorité de certification que vous allez également utiliser à d’autres fins, il est recommandé de fournir un groupe de ressources distinct pour votre coffre de clés. Nous vous recommandons de placer ce coffre de clés dans son propre groupe de ressources. Vous pouvez ainsi supprimer les groupes de ressources de calcul et de stockage, y compris le groupe de ressources contenant votre cluster Service Fabric, sans risquer de perdre vos clés et secrets. **Le groupe de ressources qui contient votre coffre de clés *doit se trouver dans la même région* que le cluster qui l’utilise.**

### <a name="use-the-default-five-node-one-node-type-template-that-ships-in-the-module"></a>Utiliser le modèle de type de nœud Node 1 5 par défaut qui est fourni dans le module
Le modèle utilisé est disponible dans les [exemples Azure : modèle Windows](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure-NSG) et [modèle Ubuntu](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Ubuntu-1-NodeTypes-Secure)

Déployer le cluster à l’aide de PowerShell :

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$vaultName="myvault"
$vaultResourceGroupName="myvaultrg"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$vmpassword=("Password!4321" | ConvertTo-SecureString -AsPlainText -Force) 
$vmuser="myadmin"
$os="WindowsServer2016DatacenterwithContainers"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -Location $resourceGroupLocation -KeyVaultResourceGroupName $vaultResourceGroupName -KeyVaultName $vaultName -CertificateFile C:\MyCertificates\chackocertificate3.pfx -CertificatePassword $certPassword -OS $os -VmPassword $vmpassword -VmUserName $vmuser 
```

Déployer le cluster à l’aide d’Azure CLI :

```azurecli
declare vmPassword="Password!1"
declare certPassword="Password!1"
declare vmUser="myadmin"
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare vaultResourceGroupName="myvaultrg"
declare vaultName="myvault"
declare certificate-file="c:\certificates\mycert.pem"
declare vmOs="UbuntuServer1604"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-file $certificate-file --certificate-password $certPassword  \
    --vault-name $vaultName --vault-resource-group $vaultResourceGroupName  \
    --vm-os vmOs \
    --vm-password $vmPassword --vm-user-name $vmUser
```

### <a name="use-your-own-custom-cluster-template"></a>Utiliser votre propre modèle de cluster personnalisé
Si vous avez besoin de créer un modèle personnalisé adapté à vos besoins, il est vivement recommandé de commencer par l’un des modèles disponibles dans les [exemples de modèles Azure Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master). Découvrez comment [personnaliser votre modèle de cluster][customize-your-cluster-template].

Si vous disposez déjà d’un modèle personnalisé, vérifiez une nouvelle fois que les trois paramètres du modèle qui sont liés au certificat et le fichier de paramètres portent les noms indiqués ci-après et que les valeurs sont null comme suit.

```json
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

Déployer le cluster à l’aide de PowerShell :

```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$vaultName="myvault"
$vaultResourceGroupName="myvaultrg"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$os="WindowsServer2016DatacenterwithContainers"
$parameterFilePath="c:\mytemplates\mytemplateparm.json"
$templateFilePath="c:\mytemplates\mytemplate.json"
$certificateFile="C:\MyCertificates\chackonewcertificate3.pem"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -Location $resourceGroupLocation -TemplateFile $templateFilePath -ParameterFile $parameterFilePath -KeyVaultResourceGroupName $vaultResourceGroupName -KeyVaultName $vaultName -CertificateFile $certificateFile -CertificatePassword $certPassword
```

Déployer le cluster à l’aide d’Azure CLI :

```azurecli
declare certPassword="Password!1"
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare vaultResourceGroupName="myvaultrg"
declare vaultName="myvault"
declare parameterFilePath="c:\mytemplates\linuxtemplateparm.json"
declare templateFilePath="c:\mytemplates\linuxtemplate.json"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-file $certificate-file --certificate-password $password  \
    --vault-name $vaultName --vault-resource-group $vaultResourceGroupName  \
    --template-file $templateFilePath --parameter-file $parametersFilePath 
```

### <a name="use-a-pointer-to-a-secret-uploaded-into-a-key-vault"></a>Utiliser un pointeur vers un secret chargé dans un coffre de clés

Pour utiliser un coffre de clés existant, il doit être [activé pour le déploiement](../key-vault/general/manage-with-cli2.md#bkmk_KVperCLI) afin d’autoriser le fournisseur de ressources de calcul à obtenir des certificats à partir de ce coffre et à les installer sur des nœuds de cluster.

Déployer le cluster à l’aide de PowerShell :

```powershell
Set-AzKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -EnabledForDeployment

$parameterFilePath="c:\mytemplates\mytemplate.json"
$templateFilePath="c:\mytemplates\mytemplateparm.json"
$secretID="https://test1.vault.azure.net:443/secrets/testcertificate4/55ec7c4dc61a462bbc645ffc9b4b225f"

New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -SecretIdentifier $secretID -TemplateFile $templateFilePath -ParameterFile $parameterFilePath 
```

Déployer le cluster à l’aide d’Azure CLI :

```azurecli
declare $resourceGroupName = "testRG"
declare $parameterFilePath="c:\mytemplates\mytemplate.json"
declare $templateFilePath="c:\mytemplates\mytemplateparm.json"
declare $secretID="https://test1.vault.azure.net:443/secrets/testcertificate4/55ec7c4dc61a462bbc645ffc9b4b225f"

az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --secret-identifier $secretID  \
    --template-file $templateFilePath --parameter-file $parameterFilePath 
```

## <a name="next-steps"></a>Étapes suivantes
À ce stade, vous avez un cluster sécurisé exécuté sous Azure. Ensuite, [connectez-vous à votre cluster](service-fabric-connect-to-secure-cluster.md) et découvrez comment [gérer les secrets d’application](service-fabric-application-secret-management.md).

Pour connaître la syntaxe JSON et les propriétés à utiliser dans un modèle, consultez les informations de référence sur les modèles [Microsoft.ServiceFabric/clusters](/azure/templates/microsoft.servicefabric/clusters).

<!-- Links -->
[azure-powershell]:https://docs.microsoft.com/powershell/azure/install-Az-ps
[azure-CLI]:https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[customize-your-cluster-template]: service-fabric-cluster-creation-create-template.md

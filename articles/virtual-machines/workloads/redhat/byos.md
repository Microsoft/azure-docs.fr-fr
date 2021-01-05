---
title: Images Azure BYOS (Apportez votre propre abonnement) de Red Hat Enterprise Linux | Microsoft Docs
description: Découvrez les images BYOS (Apportez votre propre abonnement) pour Red Hat Enterprise Linux sur Azure.
author: asinn826
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 06/10/2020
ms.author: alsin
ms.reviewer: cynthn
ms.openlocfilehash: 9a53eb1004d68045781903b71d02c3416f494be2
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97803814"
---
# <a name="red-hat-enterprise-linux-bring-your-own-subscription-gold-images-in-azure"></a>Images Gold BYOS (Apportez votre propre abonnement) de Red Hat Enterprise Linux dans Azure

Les images RHEL (Red Hat Enterprise Linux) sont disponibles dans Azure sous la forme d’un modèle Paiement à l’utilisation ou Apportez votre propre abonnement (BYOS) (Red Hat Gold Image). Cet article donne une vue d’ensemble des images Red Hat Gold dans Azure.

>[!NOTE]
> Toutes les images Gold BYOS RHEL sont disponibles dans les clouds Azure publics (commerciaux) et Azure Government. Elles ne sont pas disponibles dans les clouds Azure Chine et Azure Blackforest.

## <a name="important-points-to-consider"></a>Points importants à prendre en compte

- Les images Red Hat Gold fournies dans ce programme sont des images RHEL prêtes pour la production, similaires aux images RHEL Paiement à l’utilisation dans la Place de marché Azure.
- Les images suivent les stratégies actuelles, décrites dans [Images Red Hat Enterprise Linux sur Azure](./redhat-images.md).
- Les stratégies de support standard s’appliquent aux machines virtuelles créées à partir de ces images.
- Les machines virtuelles provisionnées à partir d’images Red Hat Gold ne sont pas soumises aux redevances RHEL associées aux images RHEL Paiement à l’utilisation.
- Les images ne sont pas autorisées. Vous devez donc utiliser Red Hat Subscription-Manager pour vous inscrire et vous abonner aux machines virtuelles afin d’obtenir directement les mises à jour auprès de Red Hat.
- Il est possible de passer d’images assortie d’un paiement à l’utilisation à une stratégie BYOS à l’aide d’[Azure Hybrid Benefit](../../linux/azure-hybrid-benefit-linux.md). En revanche, il n’est pas possible de passer d’une stratégie BYOS déployée initialement à un modèle de facturation de paiement à l’utilisation pour des images Linux. Pour changer de modèle de facturation afin de passer d’une stratégie BYOS au paiement à l’utilisation, vous devez redéployer la machine virtuelle à partir de l’image correspondante.

>[!NOTE]
> Les images BYOS RHEL génération 2 ne sont actuellement pas disponibles via l’offre de la Place de marché. Si vous avez besoin d’une image BYOS RHEL génération 2, consultez le tableau de bord Cloud Access dans Red Hat Subscription Management. Pour plus d’informations, consultez la [documentation de Red Hat](https://access.redhat.com/articles/4847681).

## <a name="requirements-and-conditions-to-access-the-red-hat-gold-images"></a>Exigences et conditions pour accéder aux images Red Hat Gold

1. Familiarisez-vous avec les conditions du [programme Red Hat Cloud Access](https://www.redhat.com/en/technologies/cloud-computing/cloud-access). Activez vos abonnements Red Hat pour Cloud Access sur le site [Red Hat Subscription-Manager](https://access.redhat.com/management/cloud). Vous devez disposer des abonnements Azure qui vont être inscrits pour Cloud Access.

1. Si les abonnements Red Hat que vous avez activés pour pour Cloud Access répondent aux conditions d’éligibilité, vos abonnements Azure sont automatiquement activés pour l’accès aux images Gold.

### <a name="expected-time-for-image-access"></a>Délai attendu pour l’accès aux images

Une fois les étapes d’activation de Cloud Access effectuées, Red Hat valide votre éligibilité pour les images Red Hat Gold. Si la validation réussit, vous recevez l’accès aux images Gold dans un délai de trois heures.

## <a name="use-the-red-hat-gold-images-from-the-azure-portal"></a>Utiliser les images Red Hat Gold depuis le portail Azure

1. Une fois que votre abonnement a reçu l’accès aux images Red Hat Gold, vous pouvez les trouver dans le [portail Azure](https://portal.azure.com). Accédez à **Créer une ressource** > **Afficher tout**.

1. En haut de la page, vous verrez que vous disposez d’offres privées.

    ![Offres privées de la place de marché](./media/rhel-byos-privateoffers.png)

1. Sélectionnez le lien violet ou faites défiler la page vers le bas pour voir vos offres privées.

1. La suite du provisionnement dans l’interface utilisateur n’est pas différente de celui d’une image Red Hat existante. Choisissez votre version RHEL et suivez les invites pour provisionner votre machine virtuelle. Ce processus vous permet également d’accepter les conditions de l’image à la dernière étape.

>[!NOTE]
>Les étapes effectuées jusqu’à présent n’activeront pas votre image Red Hat Gold pour le déploiement par programmation. Une étape supplémentaire est nécessaire, comme décrit dans la section « Informations supplémentaires ».

La suite de ce document se concentre sur la méthode CLI pour provisionner et accepter les conditions sur l’image. L’interface utilisateur et l’interface CLI sont entièrement interchangeables en ce qui concerne le résultat final (une machine virtuelle RHEL Gold Image provisionnée).

## <a name="use-the-red-hat-gold-images-from-the-azure-cli"></a>Utiliser les images Red Hat Gold depuis Azure CLI

Les instructions suivantes vous guident tout au long du processus de déploiement initial pour une machine virtuelle RHEL avec Azure CLI. Elle part du principe [qu’Azure CLI est installé](/cli/azure/install-azure-cli).

>[!IMPORTANT]
>Veillez à mettre toutes les lettres en minuscules dans les références à l’éditeur, à l’offre, à l’abonnement et à l’image pour toutes les commandes suivantes.

1. Vérifiez que vous êtes dans l’abonnement souhaité.

    ```azurecli
    az account show -o=json
    ```

1. Créez un groupe de ressources pour votre machine virtuelle Red Hat Gold Image.

    ```azurecli
    az group create --name <name> --location <location>
    ```

1. Acceptez les conditions de l’image.

    ```azurecli
    az vm image terms accept --publisher redhat --offer rhel-byos --plan <SKU value here> -o=jsonc

    # Example:
    az vm image terms accept --publisher redhat --offer rhel-byos --plan rhel-lvm75 -o=jsonc

    OR

    az vm image terms accept --urn redhat:rhel-byos:rhel-lvm8:8.0.20190620
    ```

    >[!NOTE]
    >Ces conditions doivent être acceptées *une fois par abonnement Azure, par référence SKU d’image*.

1. (Facultatif) Validez le déploiement de votre machine virtuelle avec la commande suivante :

    ```azurecli
    az vm create -n <VM name> -g <resource group name> --image <image urn> --validate

    # Example:
    az vm create -n rhel-byos-vm -g rhel-byos-group --image redhat:rhel-byos:rhel-lvm8:latest --validate
    ```

1. Provisionnez votre machine virtuelle en exécutant la même commande que dans l’exemple précédent sans l’argument `--validate`.

    ```azurecli
    az vm create -n <VM name> -g <resource group name> --image <image urn>

    # Example:
    az vm create -n rhel-byos-vm -g rhel-byos-group --image redhat:rhel-byos:rhel-lvm8:latest
    ```

1. Établissez une liaison SSH sur votre machine virtuelle et vérifiez que vous disposez d’une image non autorisée. Pour effectuer cette étape, exécutez `sudo yum repolist`. Pour RHEL 8, utilisez `sudo dnf repolist`. La sortie vous invite à utiliser Subscription-Manager pour inscrire la machine virtuelle auprès de Red Hat.

>[!NOTE]
>Sur RHEL 8, `dnf` et `yum` sont interchangeables. Pour plus d’informations, consultez le [Guide d’administration de RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/packaging_and_distributing_software/index).

## <a name="use-the-red-hat-gold-images-from-powershell"></a>Utiliser les images Red Hat Gold depuis PowerShell

Le script suivant en est un exemple. Remplacez le groupe de ressources, l’emplacement, le nom de la machine virtuelle, les informations de connexion et d’autres variables par la configuration de votre choix. Les informations du serveur de publication et du plan doivent être en minuscules.

```powershell-interactive
    # Variables for common values
    $resourceGroup = "testbyos"
    $location = "canadaeast"
    $vmName = "test01"

    # Define user name and blank password
    $securePassword = ConvertTo-SecureString 'TestPassword1!' -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential("azureuser",$securePassword)
    Get-AzureRmMarketplaceTerms -Publisher redhat -Product rhel-byos -Name rhel-lvm75 | SetAzureRmMarketplaceTerms -Accept

    # Create a resource group
    New-AzureRmResourceGroup -Name $resourceGroup -Location $location

    # Create a subnet configuration
    $subnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

    # Create a virtual network
    $vnet = New-AzureRmVirtualNetwork -ResourceGroupName $resourceGroup -Location
    $location `-Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

    # Create a public IP address and specify a DNS name
    $pip = New-AzureRmPublicIpAddress -ResourceGroupName $resourceGroup -Location
    $location `-Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4

    # Create an inbound network security group rule for port 22
    $nsgRuleSSH = New-AzureRmNetworkSecurityRuleConfig -Name
    myNetworkSecurityGroupRuleSSH -Protocol Tcp `
    -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -
    DestinationAddressPrefix * `-DestinationPortRange 22 -Access Allow

    # Create a network security group
    $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location
    $location `-Name myNetworkSecurityGroup -SecurityRules $nsgRuleSSH

    # Create a virtual network card and associate with public IP address and NSG
    $nic = New-AzureRmNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -
    Location $location `-SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

    # Create a virtual machine configuration
    $vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize Standard_D3_v2 |
    Set-AzureRmVMOperatingSystem -Linux -ComputerName $vmName -Credential $cred |
    Set-AzureRmVMSourceImage -PublisherName redhat -Offer rhel-byos -Skus rhel-lvm75 -Version latest | Add-     AzureRmVMNetworkInterface -Id $nic.Id
    Set-AzureRmVMPlan -VM $vmConfig -Publisher redhat -Product rhel-byos -Name "rhel-lvm75"

    # Configure SSH Keys
    $sshPublicKey = Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub"
    Add-AzureRmVMSshPublicKey -VM $vmconfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"

    # Create a virtual machine
    New-AzureRmVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```

## <a name="encrypt-red-hat-enterprise-linux-bring-your-own-subscription-gold-images"></a>Chiffrer des images Gold BYOS de Red Hat Enterprise Linux

Les images Gold BYOS de Red Hat Enterprise Linux peuvent être sécurisées à l’aide d’[Azure Disk Encryption](../../linux/disk-encryption-overview.md). Vous *devez* d’abord inscrire l’abonnement pour pouvoir activer le chiffrement. Pour plus d’informations sur la façon d’inscrire une image Gold BYOS RHEL, consultez [How to register and subscribe a system to the Red Hat Customer Portal using Red Hat Subscription-Manager](https://access.redhat.com/solutions/253273) (Guide pratique pour inscrire et abonner un système au portail client Red Hat à l’aide de Red Hat Subscription-Manager). Si vous disposez d’un abonnement Red Hat actif, vous pouvez également lire [Creating Red Hat Customer Portal Activation Keys](https://access.redhat.com/articles/1378093) (Création de clés d’activation du portail client Red Hat).

Azure Disk Encryption n’est pas pris en charge sur les [images personnalisées Red Hat](../../linux/redhat-create-upload-vhd.md). D’autres conditions et prérequis Azure Disk Encryption sont documentés dans [Azure Disk Encryption pour les machines virtuelles Linux](../../linux/disk-encryption-overview.md#additional-vm-requirements).

Pour connaître les étapes à suivre pour appliquer Azure Disk Encryption, consultez [Scénarios Azure Disk Encryption sur les machines virtuelles Linux](../../linux/disk-encryption-linux.md) et les articles connexes.

## <a name="additional-information"></a>Informations supplémentaires

- Si vous tentez de provisionner une machine virtuelle sur un abonnement non activé pour cette offre, le message suivant s’affiche :

    ```
    "Offer with PublisherId: redhat, OfferId: rhel-byos, PlanId: rhel-lvm75 is private and can not be purchased by subscriptionId: GUID"
    ```

    Dans ce cas, contactez Microsoft ou Red Hat pour activer votre abonnement.

- Si vous modifiez un instantané à partir d’une image BYOS RHEL et que vous tentez de publier cette image personnalisée sur [Shared Image Gallery](../../linux/shared-image-galleries.md), vous devez fournir des informations d’abonnement qui correspondent à la source d’origine de l’instantané. Par exemple, la commande peut se présenter comme suit :

    ```azurecli
    az vm create –image \
    "/subscriptions/GUID/resourceGroups/GroupName/providers/Microsoft.Compute/galleries/GalleryName/images/ImageName/versions/1.0.0" \
    -g AnotherGroupName --location EastUS2 -n VMName \
    --plan-publisher redhat --plan-product rhel-byos --plan-name rhel-lvm75
    ```

    Notez les paramètres d’abonnement sur la dernière ligne.

    [Azure Disk Encryption](#encrypt-red-hat-enterprise-linux-bring-your-own-subscription-gold-images) n’est pas pris en charge sur les images personnalisées.

- Si vous utilisez l’automatisation pour provisionner des machines virtuelles à partir des images BYOS RHEL, vous devez fournir des paramètres d’abonnement similaires à ce qui est indiqué dans les exemples de commandes. Par exemple, avec Terraform, les informations d’abonnement sont indiquées dans un [bloc d’abonnement](https://www.terraform.io/docs/providers/azurerm/r/virtual_machine.html#plan).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur Red Hat Cloud Access, consultez la [documentation du cloud public Red Hat](https://access.redhat.com/public-cloud)
- Pour obtenir des guides pas-à-pas et des détails sur les programmes pour Cloud Access, consultez la [documentation de Red Hat Cloud Access](https://access.redhat.com/documentation/en-us/red_hat_subscription_management/1/html/red_hat_cloud_access_reference_guide/index).
- Pour en savoir plus sur Azure Red Hat Update Infrastructure, consultez [Azure Red Hat Update Infrastructure](./redhat-rhui.md).
- Pour plus d’informations sur les images Red Hat dans Azure, consultez la [page de documentation](./redhat-images.md).
- Pour obtenir des informations sur les stratégies de prise en charge de Red Hat pour toutes les versions de RHEL, consultez la page [Cycle de vie Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).
- Vous trouverez de la documentation supplémentaire sur les images RHEL Gold dans la [documentation de Red Hat](https://access.redhat.com/documentation/en-us/red_hat_subscription_management/1/html/red_hat_cloud_access_reference_guide/cloud-access-gold-images_cloud-access#proc_using-gold-images-azure_cloud-access).

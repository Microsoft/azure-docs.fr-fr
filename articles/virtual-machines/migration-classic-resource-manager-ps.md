---
title: Migrer vers Resource Manager avec PowerShell
description: Cet article décrit pas à pas la procédure de migration de ressources IaaS, comme les machines virtuelles, les réseaux virtuels et les comptes de stockage, prise en charge par la plateforme de l’environnement Classic vers Azure Resource Manager à l’aide de commandes Azure PowerShell.
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 02/06/2020
ms.author: tagore
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 07b511182950095c356887d72ffde75ac098674d
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94904625"
---
# <a name="migrate-iaas-resources-from-classic-to-azure-resource-manager-by-using-powershell"></a>Migration de ressources IaaS d’un environnement classique vers Azure Resource Manager à l’aide de PowerShell

> [!IMPORTANT]
> Aujourd'hui, environ 90 % des machines virtuelles IaaS utilisent [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). Depuis le 28 février 2020, les machines virtuelles classiques sont dépréciées. Elles seront entièrement mises hors service le 1er mars 2023. [Apprenez-en davantage]( https://aka.ms/classicvmretirement) sur cette désapprobation et [son impact sur vous](classic-vm-deprecation.md#how-does-this-affect-me).

Ces étapes vous montrent comment utiliser les commandes Azure PowerShell pour migrer des ressources d’infrastructure en tant que service (IaaS) à partir du modèle de déploiement Classic vers le modèle de déploiement Azure Resource Manager.

Si vous le souhaitez, vous pouvez également migrer des ressources à l’aide d’[Azure CLI](migration-classic-resource-manager-cli.md).

* Pour plus d’informations sur les scénarios de migration pris en charge, consultez [Migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md).
* Pour un guide détaillé et une procédure pas à pas pour la migration, consultez [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md).
* [Passer en revue les erreurs de migration les plus courantes](migration-classic-resource-manager-errors.md).

<br>
Voici un organigramme permettant d’identifier l’ordre dans lequel les étapes doivent être exécutées lors d’un processus de migration.

![Screenshot that shows the migration steps](media/migration-classic-resource-manager/migration-flow.png)

 

## <a name="step-1-plan-for-migration"></a>Étape 1 : Planifier la migration
Voici quelques-unes des meilleures pratiques recommandées pour déterminer si vous devez migrer des ressources IaaS d’un environnement classique vers Resource Manager :

* Lisez les [Fonctionnalités et configurations prises en charge et non prises en charge](migration-classic-resource-manager-overview.md). Si vous avez des machines virtuelles qui utilisent des configurations ou fonctionnalités non prises en charge, attendez que leur prise en charge soit annoncée. Vous pouvez également supprimer cette fonctionnalité ou modifier cette configuration pour permettre la migration si cela répond à vos besoins.
* Si vous avez déjà des scripts automatisés qui déploient votre infrastructure et vos applications, essayez de créer une configuration de test similaire à l’aide de ces scripts pour la migration. Vous pouvez également configurer des environnements de test à l’aide du portail Azure.

> [!IMPORTANT]
> Les passerelles d’application ne sont actuellement pas prises en charge pour la migration de déploiement classique vers Resource Manager. Pour migrer un réseau virtuel à l’aide d’une passerelle d’application, supprimez la passerelle avant d’exécuter une opération de préparation pour déplacer le réseau. Après avoir effectué la migration, reconnectez la passerelle dans Azure Resource Manager.
>
> Les passerelles Azure ExpressRoute se connectant à des circuits ExpressRoute dans un autre abonnement ne peuvent pas être migrées automatiquement. Dans ce cas, supprimez la passerelle ExpressRoute, migrez le réseau virtuel et recréez la passerelle. Pour plus d’informations, consultez [Migrer des circuits ExpressRoute et les réseaux virtuels associés du modèle de déploiement classique au modèle de déploiement Resource Manager](../expressroute/expressroute-migration-classic-resource-manager.md).

## <a name="step-2-install-the-latest-version-of-powershell"></a>Étape 2 : Installer la version la plus récente de PowerShell
Il existe deux options principales pour l’installation d’Azure PowerShell : [PowerShell Gallery](https://www.powershellgallery.com/profiles/azure-sdk/) ou [Web Platform Installer (WebPI)](https://aka.ms/webpi-azps). WebPI reçoit des mises à jour mensuelles. PowerShell Gallery reçoit des mises à jour en continu. Cet article est basé sur Azure PowerShell version 2.1.0.

Pour connaître la procédure d’installation, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/).

## <a name="step-3-ensure-that-youre-an-administrator-for-the-subscription"></a>Étape 3 : Vérifier que vous êtes administrateur de l’abonnement
Pour effectuer cette migration, vous devez être ajouté en tant que coadministrateur de l’abonnement dans le [Portail Azure](https://portal.azure.com).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu **Hub**, sélectionnez **Abonnement**. Si vous ne le voyez pas, sélectionnez **Tous les services**.
3. Recherchez l’entrée d’abonnement appropriée, puis examinez le champ **MON RÔLE**. Pour un coadministrateur, la valeur doit être _Administrateur de compte_.

Si vous n'êtes pas en mesure d'ajouter un coadministrateur, contactez un administrateur de service ou un coadministrateur de l'abonnement afin qu'il vous ajoute.

## <a name="step-4-set-your-subscription-and-sign-up-for-migration"></a>Étape 4 : Définir votre abonnement et s’inscrire pour la migration
Commencez par démarrer une invite de commandes PowerShell. Pour la migration, configurez votre environnement à la fois pour l’environnement classique et pour Resource Manager.

Connectez-vous à votre compte pour le modèle Resource Manager.

```powershell
    Connect-AzAccount
```

Obtenez tous les abonnements disponibles à l’aide de la commande suivante :

```powershell
    Get-AzSubscription | Sort Name | Select Name
```

Définissez votre abonnement Azure pour la session active. Cet exemple définit le nom d’abonnement par défaut sur **My Azure Subscription**. Remplacez l’exemple de nom d’abonnement par le vôtre.

```powershell
    Select-AzSubscription –SubscriptionName "My Azure Subscription"
```

> [!NOTE]
> L’inscription constitue une étape unique, mais vous devez la faire une fois avant de tenter la migration. Sans vous inscrire, vous verrez le message d’erreur suivant :
>
> *BadRequest : L'abonnement n'est pas inscrit pour la migration.*

Inscrivez-vous auprès du fournisseur de ressources de migration à l’aide de la commande suivante :

```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

Patientez cinq minutes le temps que l’inscription se termine. Vérifiez l’état de l’approbation à l’aide de la commande suivante :

```powershell
    Get-AzResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

Assurez-vous que RegistrationState est `Registered` avant de continuer.

Avant de basculer vers le modèle de déploiement Classic, vérifiez que vous avez un nombre suffisant de processeurs virtuels de machines virtuelles Azure Resource Manager dans la région Azure de votre déploiement ou réseau virtuel actuel. Vous pouvez utiliser la commande PowerShell suivante pour vérifier la quantité de processeurs virtuels dont vous disposez actuellement dans Azure Resource Manager. Pour en savoir plus sur les quotas de processeurs virtuels, consultez [Limites et Azure Resource Manager](../azure-resource-manager/management/azure-subscription-service-limits.md#managing-limits).

Cet exemple vérifie la disponibilité dans la région **USA Ouest**. Remplacez l’exemple de nom de région par le vôtre.

```powershell
    Get-AzVMUsage -Location "West US"
```

À présent, connectez-vous à votre compte pour le modèle de déploiement classique.

```powershell
    Add-AzureAccount
```

Obtenez tous les abonnements disponibles à l’aide de la commande suivante :

```powershell
    Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName
```

Définissez votre abonnement Azure pour la session active. Cet exemple définit l’abonnement par défaut sur **My Azure Subscription**. Remplacez l’exemple de nom d’abonnement par le vôtre.

```powershell
    Select-AzureSubscription –SubscriptionName "My Azure Subscription"
```


## <a name="step-5-run-commands-to-migrate-your-iaas-resources"></a>Étape 5 : exécuter des commandes pour effectuer la migration de vos ressources IaaS
* [Migration de machines virtuelles dans un service cloud (ne figurant pas dans un réseau virtuel)](#step-51-option-1---migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network)
* [Migration de machines virtuelles dans un réseau virtuel](#step-51-option-2---migrate-virtual-machines-in-a-virtual-network)
* [Migration d’un compte de stockage](#step-52-migrate-a-storage-account)

> [!NOTE]
> Toutes les opérations décrites ici sont idempotentes. Si vous rencontrez un problème autre qu’une fonctionnalité non prise en charge ou qu’une erreur de configuration, nous vous recommandons de réexécuter la procédure de préparation, d’abandon ou de validation. La plateforme réessaie alors d’exécuter l’action.


### <a name="step-51-option-1---migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network"></a>Étape 5.1 : Option 1 - migrer les machines virtuelles vers un service cloud (pas un réseau virtuel)
Récupérez la liste des services cloud à l’aide de la commande suivante. Sélectionnez ensuite le service cloud que vous souhaitez migrer. La commande renvoie un message d’erreur si les machines virtuelles du service cloud sont dans un réseau virtuel ou si elles ont des rôles web/de travail.

```powershell
    Get-AzureService | ft Servicename
```

Obtenez le nom du déploiement du service cloud. Dans cet exemple, le nom du service est **Mon service**. Remplacez l’exemple de nom de service avec votre propre nom de service.

```powershell
    $serviceName = "My Service"
    $deployment = Get-AzureDeployment -ServiceName $serviceName
    $deploymentName = $deployment.DeploymentName
```

Préparez les machines virtuelles dans le service cloud pour la migration. Vous disposez de deux options.

* **Option n°1 : Migrer les machines virtuelles vers un réseau virtuel créé à partir d’une plateforme.**

    La première étape consiste à vérifier si vous pouvez migrer le service cloud à l’aide des commandes suivantes :

    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    $validate.ValidationMessages
    ```

    La commande suivante affiche les avertissements et erreurs qui bloquent la migration. Si la validation se déroule correctement, vous pouvez passer à l’étape de préparation.

    ```powershell
    Move-AzureService -Prepare -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    ```
* **Option n°2 : Migrer vers un réseau virtuel existant dans le modèle de déploiement Resource Manager.**

    Cet exemple définit le nom de groupe de ressources sur **myResourceGroup**, le nom de réseau virtuel sur **myVirtualNetwork** et le nom du sous-réseau sur **mySubNet**. Remplacez les noms dans l’exemple par les noms de vos propres ressources.

    ```powershell
    $existingVnetRGName = "myResourceGroup"
    $vnetName = "myVirtualNetwork"
    $subnetName = "mySubNet"
    ```

    La première étape consiste à vérifier si vous pouvez migrer le réseau virtuel à l’aide de la commande suivante :

    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName
    $validate.ValidationMessages
    ```

    La commande suivante affiche les avertissements et erreurs qui bloquent la migration. Si la validation se déroule correctement, vous pouvez passer à l’étape de la préparation suivante :

    ```powershell
        Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName `
        -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName `
        -VirtualNetworkName $vnetName -SubnetName $subnetName
    ```

Une fois l’opération de préparation effectuée avec une des options précédentes, interrogez l’état de la migration des machines virtuelles. Assurez-vous qu’elles ont l’état `Prepared`.

Cet exemple définit le nom de la machine virtuelle sur **myVM**. Remplacez l’exemple de nom avec votre propre nom de machine virtuelle.

```powershell
    $vmName = "myVM"
    $vm = Get-AzureVM -ServiceName $serviceName -Name $vmName
    $vm.VM.MigrationState
```

Vérifiez la configuration pour les ressources préparées à l’aide de PowerShell ou du portail Azure. Si vous n’êtes pas prêt pour la migration et que vous souhaitez revenir à l’ancien état, utilisez la commande suivante :

```powershell
    Move-AzureService -Abort -ServiceName $serviceName -DeploymentName $deploymentName
```

Si la configuration préparée semble correcte, vous pouvez continuer et valider les ressources à l’aide de la commande suivante :

```powershell
    Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName
```

### <a name="step-51-option-2---migrate-virtual-machines-in-a-virtual-network"></a>Étape 5.1 : Option 2 - migrer les machines virtuelles vers un réseau virtuel

Pour migrer des machines virtuelles dans un service cloud, vous migrez le réseau virtuel. Les machines virtuelles migrent automatiquement avec le réseau virtuel. Sélectionnez le réseau virtuel dont vous souhaitez effectuer la migration.
> [!NOTE]
> [Migrez une machine virtuelle unique](./windows/create-vm-specialized-portal.md) créée à l’aide du modèle de déploiement classique en créant une nouvelle machine virtuelle Resource Manager avec la fonctionnalité Disques managés en utilisant les fichiers de disque dur virtuel (système d’exploitation et données) de la machine virtuelle.
<br>

> [!NOTE]
> Le nom du réseau virtuel peut différer de ce qui est affiché dans le nouveau portail. Le nouveau Portail Azure affiche le nom sous la forme `[vnet-name]`, mais le nom réel du réseau virtuel est du type `Group [resource-group-name] [vnet-name]`. Avant de migrer, recherchez le nom réel du réseau virtuel à l’aide de la commande `Get-AzureVnetSite | Select -Property Name`, ou affichez-le dans l’ancien Portail Azure. 

Cet exemple définit le nom de réseau virtuel sur **myVnet**. Remplacez l’exemple de nom de réseau virtuel par le vôtre.

```powershell
    $vnetName = "myVnet"
```

> [!NOTE]
> Vous obtenez un message d’erreur pour la validation si le réseau virtuel contient des rôles web/de travail ou des machines virtuelles avec des configurations non prises en charge.

La première étape consiste à vérifier si vous pouvez migrer le réseau virtuel à l’aide de la commande suivante :

```powershell
    Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName
```

La commande suivante affiche les avertissements et erreurs qui bloquent la migration. Si la validation se déroule correctement, vous pouvez passer à l’étape de la préparation suivante :

```powershell
    Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName
```

Vérifiez la configuration pour les machines virtuelles préparées à l’aide d’Azure PowerShell ou du portail Azure. Si vous n’êtes pas prêt pour la migration et que vous souhaitez revenir à l’ancien état, utilisez la commande suivante :

```powershell
    Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName
```

Si la configuration préparée semble correcte, vous pouvez continuer et valider les ressources à l’aide de la commande suivante :

```powershell
    Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName
```

### <a name="step-52-migrate-a-storage-account"></a>Étape 5.2 : Migrer un compte de stockage
Après avoir terminé la migration des machines virtuelles, effectuez les vérifications prérequises suivantes avant de migrer les comptes de stockage.

> [!NOTE]
> Si votre compte de stockage ne dispose d’aucun disque ou de données de machine virtuelle associés, vous pouvez passer directement à la section « Valider un compte de stockage et démarrer la migration ».

* Vérifications prérequises en cas de migration de machines virtuelles ou si votre compte de stockage dispose de ressources de disque :
    * Migrez des machines virtuelles dont les disques sont stockés dans le compte de stockage.

        La commande suivante renvoie les propriétés RoleName et DiskName de tous les disques de machine virtuelle dans le compte de stockage. RoleName est le nom de la machine virtuelle à laquelle un disque est joint. Si cette commande renvoie des disques, assurez-vous que les machines virtuelles sur lesquelles ces disques sont attachés sont migrées avant de migrer le compte de stockage.
        ```powershell
         $storageAccountName = 'yourStorageAccountName'
          Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Select-Object -ExpandProperty AttachedTo -Property `
          DiskName | Format-List -Property RoleName, DiskName

        ```
    * Supprimez les disques de machine virtuelle détachés stockés dans le compte de stockage.

        Recherchez les disques de machine virtuelle détachés dans le compte de stockage à l’aide de la commande suivante :

        ```powershell
            $storageAccountName = 'yourStorageAccountName'
            Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Where-Object -Property AttachedTo -EQ $null | Format-List -Property DiskName  

        ```
        Si la commande précédente renvoie des disques, supprimez-les en utilisant la commande suivante :

        ```powershell
           Remove-AzureDisk -DiskName 'yourDiskName'
        ```
    * Supprimez les images de machine virtuelle stockées dans le compte de stockage.

        La commande suivante renvoie toutes les images de machine virtuelle avec un disque de système d’exploitation stocké dans le compte de stockage.
         ```powershell
            Get-AzureVmImage | Where-Object { $_.OSDiskConfiguration.MediaLink -ne $null -and $_.OSDiskConfiguration.MediaLink.Host.Contains($storageAccountName)`
                                    } | Select-Object -Property ImageName, ImageLabel
         ```
         La commande suivante renvoie toutes les images de machine virtuelle avec un disque de données stocké dans le compte de stockage.
         ```powershell

            Get-AzureVmImage | Where-Object {$_.DataDiskConfigurations -ne $null `
                                             -and ($_.DataDiskConfigurations | Where-Object {$_.MediaLink -ne $null -and $_.MediaLink.Host.Contains($storageAccountName)}).Count -gt 0 `
                                            } | Select-Object -Property ImageName, ImageLabel
         ```
        Supprimez toutes les images de machine virtuelle renvoyées par les commandes précédentes à l’aide de cette commande :
        ```powershell
        Remove-AzureVMImage -ImageName 'yourImageName'
        ```
* Validez les comptes de stockage et démarrez la migration.

    Validez chaque compte de stockage pour la migration à l’aide de la commande suivante. Dans cet exemple, le nom de compte de stockage est **myStorageAccount**. Remplacez le nom d’exemple par celui de votre propre compte de stockage.

    ```powershell
        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Validate -StorageAccountName $storageAccountName
    ```

    L’étape suivante consiste à préparer le compte de stockage pour la migration.

    ```powershell
        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName
    ```

    Vérifiez la configuration pour le compte de stockage préparé à l’aide d’Azure PowerShell ou du portail Azure. Si vous n’êtes pas prêt pour la migration et que vous souhaitez revenir à l’ancien état, utilisez la commande suivante :

    ```powershell
        Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName
    ```

    Si la configuration préparée semble correcte, vous pouvez continuer et valider les ressources à l’aide de la commande suivante :

    ```powershell
        Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName
    ```

## <a name="next-steps"></a>Étapes suivantes
* [Vue d’ensemble de la migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager](migration-classic-resource-manager-plan.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure](migration-classic-resource-manager-cli.md)
* [Outils de la communauté pour aider à la migration de ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-community-tools.md)
* [Passer en revue les erreurs courantes de migration](migration-classic-resource-manager-errors.md)
* [Passez en revue les questions fréquemment posées sur la migration des ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-faq.md)

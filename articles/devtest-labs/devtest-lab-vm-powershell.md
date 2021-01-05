---
title: Créer une machine virtuelle dans DevTest Labs avec Azure PowerShell
description: Découvrez comment utiliser Azure DevTest Labs pour créer et gérer des machines virtuelles avec Azure PowerShell.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 6bcc1b8cfe1da7e5eafbee0af008a1defbe6f49d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85484109"
---
# <a name="create-a-virtual-machine-with-devtest-labs-using-azure-powershell"></a>Créer une machine virtuelle avec DevTest Labs en utilisant Azure PowerShell
Cet article vous montre comment créer une machine virtuelle dans Azure DevTest Labs avec Azure PowerShell. Vous pouvez utiliser des scripts PowerShell pour automatiser la création de machines virtuelles dans un labo dans Azure DevTest Labs. 

## <a name="prerequisites"></a>Prérequis
Avant de commencer :

- [Créez un labo](devtest-lab-create-lab.md) si vous ne voulez pas utiliser un labo existant pour tester le script ou les commandes de cet article. 
- [Installez Azure PowerShell](/powershell/azure/install-az-ps?view=azps-1.7.0) ou utilisez Azure Cloud Shell, qui est intégré dans le portail Azure. 

## <a name="powershell-script"></a>Script PowerShell
L’exemple de script de cette section utilise l’applet de commande [Invoke-AzResourceAction](/powershell/module/az.resources/invoke-azresourceaction?view=azps-1.7.0).  Cette applet de commande prend l’ID de ressource du labo, le nom de l’action à effectuer (`createEnvironment`) et les paramètres nécessaires pour effectuer cette action. Les paramètres sont dans une table de hachage qui contient toutes les propriétés de description de la machine virtuelle. 

```powershell
[CmdletBinding()]

Param(
[Parameter(Mandatory = $false)]  $SubscriptionId,
[Parameter(Mandatory = $true)]   $LabResourceGroup,
[Parameter(Mandatory = $true)]   $LabName,
[Parameter(Mandatory = $true)]   $NewVmName,
[Parameter(Mandatory = $true)]   $UserName,
[Parameter(Mandatory = $true)]   $Password
)
 
pushd $PSScriptRoot

try {
    if ($SubscriptionId -eq $null) {
        $SubscriptionId = (Get-AzContext).Subscription.SubscriptionId
    }

    $API_VERSION = '2016-05-15'
    $lab = Get-AzResource -ResourceId "/subscriptions/$SubscriptionId/resourceGroups/$LabResourceGroup/providers/Microsoft.DevTestLab/labs/$LabName"

    if ($lab -eq $null) {
       throw "Unable to find lab $LabName resource group $LabResourceGroup in subscription $SubscriptionId."
    }

    #For this example, we are getting the first allowed subnet in the first virtual network
    #  for the lab.
    #If a specific virtual network is needed use | to find it. 
    #ie $virtualNetwork = @(Get-AzResource -ResourceType  'Microsoft.DevTestLab/labs/virtualnetworks' -ResourceName $LabName -ResourceGroupName $lab.ResourceGroupName -ApiVersion $API_VERSION) | Where-Object Name -EQ "SpecificVNetName"

    $virtualNetwork = @(Get-AzResource -ResourceType  'Microsoft.DevTestLab/labs/virtualnetworks' -ResourceName $LabName -ResourceGroupName $lab.ResourceGroupName -ApiVersion $API_VERSION)[0]

    $labSubnetName = $virtualNetwork.properties.allowedSubnets[0].labSubnetName

    #Prepare all the properties needed for the createEnvironment
    # call used to create the new VM.
    # The properties will be slightly different depending on the base of the vm
    # (a marketplace image, custom image or formula).
    # The setup of the virtual network to be used may also affect the properties.
    # This sample includes the properties to add an additional disk under dataDiskParameters
    
    $parameters = @{
       "name"      = $NewVmName;
       "location"  = $lab.Location;
       "properties" = @{
          "labVirtualNetworkId"     = $virtualNetwork.ResourceId;
          "labSubnetName"           = $labSubnetName;
          "notes"                   = "Windows Server 2016 Datacenter";
          "osType"                  = "windows"
          "expirationDate"          = "2019-12-01"
          "galleryImageReference"   = @{
             "offer"     = "WindowsServer";
             "publisher" = "MicrosoftWindowsServer";
             "sku"       = "2016-Datacenter";
             "osType"    = "Windows";
             "version"   = "latest"
          };
          "size"                    = "Standard_DS2_v2";
          "userName"                = $UserName;
          "password"                = $Password;
          "disallowPublicIpAddress" = $true;
          "dataDiskParameters" = @(@{
            "attachNewDataDiskOptions" = @{
                "diskName" = "adddatadisk"
                "diskSizeGiB" = "1023"
                "diskType" = "Standard"
                }
          "hostCaching" = "ReadWrite"
          })
       }
    }
    
    #The following line is the same as invoking
    # https://azure.github.io/projects/apis/#!/Labs/Labs_CreateEnvironment rest api

    Invoke-AzResourceAction -ResourceId $lab.ResourceId -Action 'createEnvironment' -Parameters $parameters -ApiVersion $API_VERSION -Force -Verbose
}
finally {
   popd
}
```

Les propriétés de la machine virtuelle dans le script ci-dessus nous permettent de créer une machine virtuelle avec le système d’exploitation Windows Server 2016 DataCenter. Pour chaque type de machine virtuelle, ces propriétés seront légèrement différentes. La section [Définir une machine virtuelle](#define-virtual-machine) vous montre comment déterminer quelles propriétés utiliser dans ce script.

La commande suivante donne un exemple d’exécution du script enregistré dans un nom de fichier : Create-LabVirtualMachine.ps1. 

```powershell
 PS> .\Create-LabVirtualMachine.ps1 -ResourceGroupName 'MyLabResourceGroup' -LabName 'MyLab' -userName 'AdminUser' -password 'Password1!' -VMName 'MyLabVM'
```

## <a name="define-virtual-machine"></a>Définir une machine virtuelle
Cette section vous montre comment obtenir les propriétés qui sont spécifiques à un type de machine virtuelle que vous voulez créer. 

### <a name="use-azure-portal"></a>Utiliser le portail Azure
Vous pouvez générer un modèle Azure Resource Manager lors de la création d’une machine virtuelle dans le portail Azure. Vous n’avez pas besoin de terminer le processus de création de la machine virtuelle. Vous suivez seulement les étapes jusqu’au moment où vous voyez le modèle. Il s’agit de la meilleure façon d’obtenir la description JSON nécessaire si vous n’avez pas une machine virtuelle de labo déjà créée. 

1. Accédez au [portail Azure](https://portal.azure.com).
2. Dans le menu de navigation de gauche, sélectionnez **Tous les services**.
3. Recherchez et sélectionnez **DevTest Labs** dans la liste des services. 
4. Dans la page **DevTest Labs**, sélectionnez votre labo dans la liste des laboratoires.
5. Dans la page d’accueil de votre labo, sélectionnez **+ Ajouter** dans la barre d’outils. 
6. Sélectionnez une **image de base** pour la machine virtuelle. 
7. Sélectionnez **Options d’automatisation** en bas de l’écran, au-dessus du bouton **Envoyer**. 
8. Vous voyez le **modèle Azure Resource Manager** pour la création de la machine virtuelle. 
9. Le segment JSON dans la section **ressources** a la définition pour le type d’image que vous avez sélectionné précédemment. 

    ```json
    {
      "apiVersion": "2018-10-15-preview",
      "type": "Microsoft.DevTestLab/labs/virtualmachines",
      "name": "[variables('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "labVirtualNetworkId": "[variables('labVirtualNetworkId')]",
        "notes": "Windows Server 2019 Datacenter",
        "galleryImageReference": {
          "offer": "WindowsServer",
          "publisher": "MicrosoftWindowsServer",
          "sku": "2019-Datacenter",
          "osType": "Windows",
          "version": "latest"
        },
        "size": "[parameters('size')]",
        "userName": "[parameters('userName')]",
        "password": "[parameters('password')]",
        "isAuthenticationWithSshKey": false,
        "labSubnetName": "[variables('labSubnetName')]",
        "disallowPublicIpAddress": true,
        "storageType": "Standard",
        "allowClaim": false,
        "networkInterface": {
          "sharedPublicIpAddressConfiguration": {
            "inboundNatRules": [
              {
                "transportProtocol": "tcp",
                "backendPort": 3389
              }
            ]
          }
        }
      }
    }
    ```

Dans cet exemple, vous voyez comment obtenir une définition d’une image de la Place de marché Azure. Vous pouvez obtenir une définition d’une image personnalisée, une formule ou un environnement de la même façon. Ajoutez les artefacts nécessaires pour la machine virtuelle et définissez les paramètres avancés nécessaires. Après avoir entré les valeurs pour les champs obligatoires et les champs facultatifs, avant de sélectionner le bouton **Options d’automatisation**.

### <a name="use-azure-rest-api"></a>Utiliser l’API REST Azure
La procédure suivante vous indique les étapes permettant d’obtenir les propriétés d’une image avec l’API REST : Ces étapes fonctionnent seulement pour une machine virtuelle existante dans un labo. 

1. Accédez à la page [Machines virtuelles - Liste](/rest/api/dtl/virtualmachines/list), puis sélectionnez le bouton **Essayer**. 
2. Sélectionnez votre **abonnement Azure**.
3. Entrez le **groupe de ressources pour le labo**.
4. Entrez le **nom du labo**. 
5. Sélectionnez **Exécuter**.
6. Vous voyez les **propriétés de l’image** à partir de laquelle la machine virtuelle a été créée. 

## <a name="set-expiration-date"></a>Définir la date d’expiration
Dans des scénarios tels que la formation, les démonstrations et les essais, vous pouvez créer des machines virtuelles et les supprimer automatiquement après une durée fixe, afin de ne pas occasionner de coûts inutiles. Vous pouvez définir une date d’expiration pour une machine virtuelle lors de sa création à l’aide de PowerShell, comme indiqué dans la section [Script PowerShell](#powershell-script) de l’exemple.

Voici un exemple de script PowerShell qui définit la date d’expiration de toutes les machines virtuelles existantes dans un laboratoire :

```powershell
# Values to change
$subscriptionId = '<Enter the subscription Id that contains lab>'
$labResourceGroup = '<Enter the lab resource group>'
$labName = '<Enter the lab name>'
$VmName = '<Enter the VmName>'
$expirationDate = '<Enter the expiration date e.g. 2019-12-16>'

# Log into your Azure account
Login-AzureRmAccount

Select-AzureRmSubscription -SubscriptionId $subscriptionId
$VmResourceId = "subscriptions/$subscriptionId/resourcegroups/$labResourceGroup/providers/microsoft.devtestlab/labs/$labName/virtualmachines/$VmName"

$vm = Get-AzureRmResource -ResourceId $VmResourceId -ExpandProperties

# Get all the Vm properties
$VmProperties = $vm.Properties

# Set the expirationDate property
If ($VmProperties.expirationDate -eq $null) {
    $VmProperties | Add-Member -MemberType NoteProperty -Name expirationDate -Value $expirationDate
} Else {
    $VmProperties.expirationDate = $expirationDate
}

Set-AzureRmResource -ResourceId $VmResourceId -Properties $VmProperties -Force
```


## <a name="next-steps"></a>Étapes suivantes
Consultez la [Documentation Azure PowerShell pour Azure DevTest Labs](/powershell/module/az.devtestlabs/)

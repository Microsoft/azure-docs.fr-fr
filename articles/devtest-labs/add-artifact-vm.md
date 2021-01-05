---
title: Ajouter un artefact à une machine virtuelle dans Azure DevTest Labs | Microsoft Docs
description: Apprenez à ajouter un artefact à une machine virtuelle dans un labo d'Azure DevTest Labs
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b4772755d8077f7a659c4d403961ffaeb9e1d483
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85483888"
---
# <a name="add-an-artifact-to-a-vm"></a>Ajouter un artefact à une machine virtuelle
Lors de la création d'une machine virtuelle, vous pouvez y ajouter des artefacts existants. Ces artefacts peuvent provenir du [référentiel Git public de DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts) ou de votre propre référentiel Git. Cet article explique comment ajouter des artefacts sur le portail Azure et à l'aide d'Azure PowerShell. 

Les *artefacts* Azure DevTest Labs vous permettent de spécifier les *actions* qui sont effectuées lorsque la machine virtuelle est approvisionnée (exécution de scripts Windows PowerShell, exécution de commandes Bash, installation de logiciel, etc.). Les *paramètres* des artefacts vous permettent de personnaliser l'artefact pour votre scénario spécifique.

Pour en savoir plus sur la création d'artefacts personnalisés, consultez l'article suivant : [Créer des artefacts personnalisés](devtest-lab-artifact-author.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="use-azure-portal"></a>Utiliser le portail Azure 
1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
1. Dans la liste des laboratoires, sélectionnez le laboratoire contenant la machine virtuelle avec laquelle vous souhaitez travailler.  
1. Sélectionnez **Mes machines virtuelles**.
1. Sélectionnez la machine virtuelle qui vous intéresse.
1. Sélectionnez **Gérer les artefacts**. 
1. Sélectionnez **Appliquer des artefacts**.
1. Dans le volet **Appliquer des artefacts**, sélectionnez l’artefact que vous souhaitez ajouter à la machine virtuelle.
1. Dans le volet **Ajouter un artefact**, entrez les valeurs de paramètre requises et tous les paramètres facultatifs dont vous avez besoin.  
1. Sélectionnez **Ajouter** pour ajouter l’artefact et revenir au volet **Appliquer des artefacts**.
1. Continuez à ajouter des artefacts en fonction des besoins de votre machine virtuelle.
1. Une fois que vous avez ajouté vos artefacts, vous pouvez [modifier l’ordre dans lequel les artefacts sont exécutés](#change-the-order-in-which-artifacts-are-run). Vous pouvez également revenir à [Afficher ou modifier un artefact](#view-or-modify-an-artifact).
1. Lorsque vous avez terminé d’ajouter des artefacts, sélectionnez **Appliquer**.

### <a name="change-the-order-in-which-artifacts-are-run"></a>Modification de l'ordre dans lequel les artefacts sont exécutés
Par défaut, les actions des artefacts sont exécutées dans l'ordre dans lequel ceux-ci sont ajoutés à la machine virtuelle. Les étapes suivantes montrent comment modifier l’ordre dans lequel les artefacts sont exécutés.

1. En haut du volet **Appliquer des artefacts**, sélectionnez le lien indiquant le nombre d’artefacts ajoutés à la machine virtuelle.
   
    ![Nombre d’artefacts ajoutés à la machine virtuelle](./media/devtest-lab-add-vm-with-artifacts/devtestlab-add-artifacts-blade-selected-artifacts.png)
1. Dans le volet **Artefacts sélectionnés**, effectuez un glisser-déplacer sur les artefacts dans l’ordre souhaité. Si vous avez des difficultés à faire glisser l’artefact, vérifiez que vous effectuez cette opération à partir du côté gauche de l’artefact. 
1. Cliquez sur **OK** quand vous avez terminé.  

### <a name="view-or-modify-an-artifact"></a>Afficher ou modifier un artefact
Les étapes suivantes montrent comment afficher ou modifier les paramètres d’un artefact :

1. En haut du volet **Appliquer des artefacts**, sélectionnez le lien indiquant le nombre d’artefacts ajoutés à la machine virtuelle.
   
    ![Nombre d’artefacts ajoutés à la machine virtuelle](./media/devtest-lab-add-vm-with-artifacts/devtestlab-add-artifacts-blade-selected-artifacts.png)
1. Dans le volet **Artefacts sélectionnés**, sélectionnez l’artefact que vous voulez visualiser ou modifier.  
1. Dans le volet **Ajouter un artefact**, apportez les modifications nécessaires, puis cliquez sur **OK** pour fermer le volet **Ajouter un artefact**.
1. Cliquez sur **OK** pour fermer le volet **Artefacts sélectionnés**.

## <a name="use-powershell"></a>Utiliser PowerShell
Le script suivant applique l'artefact spécifié à la machine virtuelle spécifiée. La commande [Invoquer-AzResourceAction](/powershell/module/az.resources/invoke-azresourceaction) exécute l'opération.  

```powershell
#Requires -Module Az.Resources

param
(
[Parameter(Mandatory=$true, HelpMessage="The ID of the subscription that contains the lab")]
   [string] $SubscriptionId,
[Parameter(Mandatory=$true, HelpMessage="The name of the lab containing the virtual machine")]
   [string] $DevTestLabName,
[Parameter(Mandatory=$true, HelpMessage="The name of the virtual machine")]
   [string] $VirtualMachineName,
[Parameter(Mandatory=$true, HelpMessage="The repository where the artifact is stored")]
   [string] $RepositoryName,
[Parameter(Mandatory=$true, HelpMessage="The artifact to apply to the virtual machine")]
   [string] $ArtifactName,
[Parameter(ValueFromRemainingArguments=$true)]
   $Params
)

# Set the appropriate subscription
Set-AzContext -SubscriptionId $SubscriptionId | Out-Null
 
# Get the lab resource group name
$resourceGroupName = (Get-AzResource -ResourceType 'Microsoft.DevTestLab/labs' | Where-Object { $_.Name -eq $DevTestLabName}).ResourceGroupName
if ($resourceGroupName -eq $null) { throw "Unable to find lab $DevTestLabName in subscription $SubscriptionId." }

# Get the internal repo name
$repository = Get-AzResource -ResourceGroupName $resourceGroupName `
                    -ResourceType 'Microsoft.DevTestLab/labs/artifactsources' `
                    -ResourceName $DevTestLabName `
                    -ApiVersion 2016-05-15 `
                    | Where-Object { $RepositoryName -in ($_.Name, $_.Properties.displayName) } `
                    | Select-Object -First 1

if ($repository -eq $null) { "Unable to find repository $RepositoryName in lab $DevTestLabName." }

# Get the internal artifact name
$template = Get-AzResource -ResourceGroupName $resourceGroupName `
                -ResourceType "Microsoft.DevTestLab/labs/artifactSources/artifacts" `
                -ResourceName "$DevTestLabName/$($repository.Name)" `
                -ApiVersion 2016-05-15 `
                | Where-Object { $ArtifactName -in ($_.Name, $_.Properties.title) } `
                | Select-Object -First 1

if ($template -eq $null) { throw "Unable to find template $ArtifactName in lab $DevTestLabName." }

# Find the virtual machine in Azure
$FullVMId = "/subscriptions/$SubscriptionId/resourceGroups/$resourceGroupName`
                /providers/Microsoft.DevTestLab/labs/$DevTestLabName/virtualmachines/$virtualMachineName"

$virtualMachine = Get-AzResource -ResourceId $FullVMId

# Generate the artifact id
$FullArtifactId = "/subscriptions/$SubscriptionId/resourceGroups/$resourceGroupName`
                        /providers/Microsoft.DevTestLab/labs/$DevTestLabName/artifactSources/$($repository.Name)`
                        /artifacts/$($template.Name)"

# Handle the inputted parameters to pass through
$artifactParameters = @()

# Fill artifact parameter with the additional -param_ data and strip off the -param_
$Params | ForEach-Object {
   if ($_ -match '^-param_(.*)') {
      $name = $_.TrimStart('^-param_')
   } elseif ( $name ) {
      $artifactParameters += @{ "name" = "$name"; "value" = "$_" }
      $name = $null #reset name variable
   }
}

# Create structure for the artifact data to be passed to the action

$prop = @{
artifacts = @(
    @{
        artifactId = $FullArtifactId
        parameters = $artifactParameters
    }
    )
}

# Check the VM
if ($virtualMachine -ne $null) {
   # Apply the artifact by name to the virtual machine
   $status = Invoke-AzResourceAction -Parameters $prop -ResourceId $virtualMachine.ResourceId -Action "applyArtifacts" -ApiVersion 2016-05-15 -Force
   if ($status.Status -eq 'Succeeded') {
      Write-Output "##[section] Successfully applied artifact: $ArtifactName to $VirtualMachineName"
   } else {
      Write-Error "##[error]Failed to apply artifact: $ArtifactName to $VirtualMachineName"
   }
} else {
   Write-Error "##[error]$VirtualMachine was not found in the DevTest Lab, unable to apply the artifact"
}

```

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants relatifs aux artefacts :

- [Spécifier des artefacts obligatoires pour votre labo](devtest-lab-mandatory-artifacts.md)
- [Créer des artefacts personnalisés](devtest-lab-artifact-author.md)
- [Ajouter un dépôt d’artefacts à un labo](devtest-lab-artifact-author.md)
- [Diagnostiquer les échecs d’artefact](devtest-lab-troubleshoot-artifact-failure.md)

---
title: Ajouter un référentiel d’artefacts à votre laboratoire dans Azure DevTest Labs | Microsoft Docs
description: Découvrez comment spécifier votre propre dépôt d’artefacts pour votre laboratoire dans Azure DevTest Labs afin de stocker des outils non disponibles dans le dépôt d’artefacts public.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 5dd7d89020bf077e29b177f6871f43b52467b0d8
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97512009"
---
# <a name="add-an-artifact-repository-to-your-lab-in-devtest-labs"></a>Ajouter un référentiel d’artefacts à votre laboratoire dans Azure DevTest Labs.
Azure DevTest Labs vous permet de spécifier l’ajout d’un artefact à une machine virtuelle pendant ou après la création de cette dernière. Cet artefact peut être un outil ou une application que vous souhaitez installer sur la machine virtuelle. Les artefacts sont définis dans un fichier JSON chargé à partir d’un référentiel Git Azure DevOps ou GitHub.

Le [référentiel d’artefacts public](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts), géré par DevTest Labs, fournit de nombreux outils courants pour Windows et Linux. Un lien vers ce référentiel est automatiquement ajouté à votre laboratoire. Vous pouvez créer votre propre référentiel d’artefacts avec des outils spécifiques qui ne sont pas disponibles dans le référentiel d’artefacts public. Pour en savoir plus sur la création d’artefacts personnalisés, voir [Create custom artefacts](devtest-lab-artifact-author.md) (Créer des artefacts personnalisés).

Cet article fournit des informations sur l’ajout de votre référentiel d’artefacts personnalisé via le Portail Microsoft Azure, les modèles Azure Resource Manager et Azure PowerShell. Vous pouvez automatiser l’ajout d’un référentiel d’artefacts à un laboratoire en écrivant des scripts PowerShell ou CLI.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis
Pour ajouter un dépôt à votre laboratoire, obtenez d’abord des informations essentielles de votre dépôt. Les sections suivantes expliquent comment obtenir les informations requises pour les référentiels hébergés sur **GitHub** ou **Azure DevOps**.

### <a name="get-the-github-repository-clone-url-and-personal-access-token"></a>Obtenir l’URL de clonage du dépôt GitHub et le jeton d’accès personnel

1. Accédez à la page d’accueil du dépôt GitHub contenant l’artefact ou les définitions de modèle Resource Manager.
2. Sélectionnez **Cloner ou télécharger**.
3. Pour copier l’URL dans le Presse-papiers, sélectionnez le bouton **URL de clonage HTTPS**. Enregistrez l’URL pour une utilisation ultérieure.
4. Dans le coin supérieur droit de GitHub, Sélectionnez l’image de profil, puis sélectionnez **Paramètres**.
5. Dans le menu **Paramètres personnels** situé à gauche, sélectionnez **Paramètres de développeur**.
6. Sélectionnez **Jetons d’accès personnels** dans le menu de gauche.
7. Sélectionnez **Générer un nouveau jeton**.
8. Dans la page **Nouveau jeton d’accès personnel**, sous **Description du jeton**, entrez une description. Accepter les éléments par défaut sous **Sélectionner des étendues**, puis sélectionnez **Générer un jeton**.
9. Enregistrez le jeton généré. Vous l’utiliserez ultérieurement.
10. Fermez GitHub.   

### <a name="get-the-azure-repos-clone-url-and-personal-access-token"></a>Obtenir l’URL du clone Azure Repos et le jeton d’accès personnel
1. Accédez à la page d’accueil de votre collection d’équipe (par exemple, `https://contoso-web-team.visualstudio.com`), puis sélectionnez votre projet.
2. Sur la page d’accueil du projet, sélectionnez **Code**.
3. Pour afficher l’URL de clonage, sélectionnez dans la page **Code** du projet l’option **Cloner**.
4. Enregistrez l’URL. Vous l’utiliserez ultérieurement.
5. Pour créer un jeton d’accès personnel, dans le menu déroulant du compte d’utilisateur, sélectionnez **Mon profil**.
6. Sur la page d’informations du profil, sélectionnez **Sécurité**.
7. Dans l’onglet **Sécurité > Jetons d’accès personnels**, sélectionnez **+ Nouveau jeton**.
8. Sur la page **Créer un jeton d’accès personnel** :
   1. Entrez un **nom** pour le jeton.
   2. Dans la liste **Organisation**, sélectionnez **Toutes les organisations accessibles**.
   3. Dans la liste **Expiration (UTC)** , sélectionnez **90 jours** ou une période d’expiration personnalisée.
   4. Sélectionnez l’option **Accès complet** pour les étendues.
   5. Sélectionnez **Create** (Créer).
9. Le nouveau jeton apparaît dans la liste **Jetons d’accès personnels**. Sélectionnez **Copier le jeton** puis enregistrez la valeur du jeton pour utilisation ultérieure.
10. Passez à la section Connecter votre laboratoire au référentiel.

## <a name="use-azure-portal"></a>Utiliser le portail Azure
Cette section inclut les étapes permettant d’ajouter un référentiel d’artefacts à un laboratoire dans le Portail Microsoft Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Autres services**, puis **DevTest Labs** dans la liste de services.
3. Dans la liste de laboratoires, sélectionnez votre laboratoire.
4. Sélectionnez **Configuration et stratégies** dans le menu de gauche.
5. Sélectionnez **Référentiels** sous la section **Ressources externes**, dans le menu de gauche.
6. Sélectionnez **+Ajouter** dans la barre d’outils.

    ![Bouton pour ajouter un dépôt](./media/devtest-lab-add-repo/devtestlab-add-repo.png)
5. Sur la page **Référentiels**, spécifiez les informations suivantes :
   1. **Nom**. Entrez un nom pour le dépôt.
   2. **URL Git Clone**. Entrez l’URL de clonage Git HTTPS que vous avez précédemment copiée à partir de GitHub ou d’Azure DevOps Services.
   3. **Branche**. Pour obtenir vos définitions, entrez la branche.
   4. **Jeton d’accès personnel**. Entrez le jeton d’accès personnel que vous avez obtenu à partir de GitHub ou Azure DevOps Services.
   5. **Chemins de dossiers**. Entrez au moins un chemin de dossier relatif à l’URL du clone contenant votre artefact ou vos définitions de modèle Resource Manager. Quand vous spécifiez un sous-répertoire, veillez à inclure la barre oblique dans le chemin du dossier.

        ![Zone Référentiels](./media/devtest-lab-add-repo/devtestlab-repo-blade.png)
6. Sélectionnez **Enregistrer**.

## <a name="use-azure-resource-manager-template"></a>Utilisation d’un modèle Azure Resource Manager
Modèles Azure Resource Management (Azure Resource Manager) sont des fichiers JSON qui décrivent les ressources d’Azure que vous souhaitez créer. Pour en savoir plus sur ces modèles, voir [Création de modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

Cette section inclut les étapes permettant d’ajouter un référentiel d’artefacts à un laboratoire via un modèle Azure Resource Manager.  Ce modèle crée le laboratoire, s’il n’existe pas déjà.

### <a name="template"></a>Modèle
L’exemple de modèle utilisé dans cet article rassemble les informations suivantes par le biais de paramètres. La plupart des paramètres ont des valeurs par défaut intelligentes, mais certaines valeurs doivent être spécifiées. Vous devez spécifier le nom de laboratoire, l’URI du référentiel d’artefacts et le jeton de sécurité du référentiel.

- Nom du laboratoire.
- Nom d’affichage du référentiel d’artefacts dans l’interface utilisateur d’Azure DevTest Labs. La valeur par défaut est `Team Repository`.
- URI vers le référentiel (exemple : `https://github.com/<myteam>/<nameofrepo>.git` or `"https://MyProject1.visualstudio.com/DefaultCollection/_git/TeamArtifacts"`).
- Branche du référentiel qui contient les artefacts. La valeur par défaut est `master`.
- Nom du dossier contenant les artefacts. La valeur par défaut est `/Artifacts`.
- Type du référentiel. Valeurs autorisées : `VsoGit` ou `GitHub`.
- Jeton d’accès du référentiel.

    ```json
    {

        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "labName": {
                "type": "string"
            },
            "artifactRepositoryDisplayName": {
                "type": "string",
                "defaultValue": "Team Repository"
            },
            "artifactRepoUri": {
                "type": "string"
            },
            "artifactRepoBranch": {
                "type": "string",
                "defaultValue": "master"
            },
            "artifactRepoFolder": {
                "type": "string",
                "defaultValue": "/Artifacts"
            },
            "artifactRepoType": {
                "type": "string",
                "allowedValues": ["VsoGit", "GitHub"]
            },
            "artifactRepoSecurityToken": {
                "type": "securestring"
            }
        },
        "variables": {
            "artifactRepositoryName": "[concat('Repo-', uniqueString(subscription().subscriptionId))]"
        },
        "resources": [{
                "apiVersion": "2016-05-15",
                "type": "Microsoft.DevTestLab/labs",
                "name": "[parameters('labName')]",
                "location": "[resourceGroup().location]",
                "resources": [
                    {
                        "apiVersion": "2016-05-15",
                        "name": "[variables('artifactRepositoryName')]",
                        "type": "artifactSources",
                        "dependsOn": [
                            "[resourceId('Microsoft.DevTestLab/labs', parameters('labName'))]"
                        ],
                        "properties": {
                            "uri": "[parameters('artifactRepoUri')]",
                            "folderPath": "[parameters('artifactRepoFolder')]",
                            "branchRef": "[parameters('artifactRepoBranch')]",
                            "displayName": "[parameters('artifactRepositoryDisplayName')]",
                            "securityToken": "[parameters('artifactRepoSecurityToken')]",
                            "sourceType": "[parameters('artifactRepoType')]",
                            "status": "Enabled"
                        }
                    }
                ]
            }
        ]
    }
    ```


### <a name="deploy-the-template"></a>Déployer le modèle
Il existe quelques façons de déployer le modèle dans Azure et de créer la ressource, si elle n’existe pas, ou de la mettre à jour, le cas échéant. Pour plus de détails, consultez les articles suivants :

- [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
- [Déployer des ressources à l’aide de modèles Resource Manager et de l’interface de ligne de commande Azure](../azure-resource-manager/templates/deploy-cli.md)
- [Déployer des ressources à l’aide de modèles Resource Manager et du Portail Azure](../azure-resource-manager/templates/deploy-portal.md)
- [Déployer des ressources à l’aide de modèles Resource Manager et de l’API REST Resource Manager](../azure-resource-manager/templates/deploy-rest.md)

Poursuivons et voyons comment déployer le modèle dans PowerShell. Les cmdlets utilisées pour déployer le modèle dépendent du contexte, donc le locataire et l’abonnement actuels sont utilisés. Utilisez [Set-AzContext](/powershell/module/az.accounts/set-azcontext) avant de déployer le modèle, si nécessaire, pour modifier le contexte.

Commencez par créer un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Si le groupe de ressources que vous souhaitez utiliser existe déjà, ignorez cette étape.

```powershell
New-AzResourceGroup -Name MyLabResourceGroup1 -Location westus
```

Ensuite, créez un déploiement vers le groupe de ressources à l’aide de [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment). Cette cmdlet applique les modifications apportées aux ressources à Azure. Plusieurs déploiements de ressources peuvent être appliqués à n’importe quel groupe de ressources donné. Si vous effectuez plusieurs déploiements dans le même groupe de ressources, assurez-vous que le nom de chaque déploiement est unique.

```powershell
New-AzResourceGroupDeployment `
    -Name MyLabResourceGroup-Deployment1 `
    -ResourceGroupName MyLabResourceGroup1 `
    -TemplateFile azuredeploy.json `
    -TemplateParameterFile azuredeploy.parameters.json
```

Une fois la cmdlet AzResourceGroupDeployment de nouveau exécutée avec succès, la commande affiche des informations importantes, telles que l’état d’approvisionnement (qui doit être Réussi) et les différentes sorties pour le modèle.

## <a name="use-azure-powershell"></a>Utilisation d'Azure PowerShell
Cette section fournit un exemple de script PowerShell, qui peut être utilisé pour ajouter un référentiel d’artefacts à un laboratoire. Si vous n’avez pas Azure PowerShell, accédez à la section [How to install and configure Azure PowerShell](/powershell/azure/?view=azps-1.2.0) (Comment installer et configurer Azure PowerShell) pour obtenir des instructions détaillées afin de l’installer.

### <a name="full-script"></a>Script complet
Voici le script complet, y compris les commentaires et messages détaillés :

**New-DevTestLabArtifactRepository.ps1** :

```powershell

<#

.SYNOPSIS
This script creates a new custom repository and adds it to an existing DevTest Lab.

.PARAMETER LabName
The name of the lab.

.PARAMETER LabResourceGroupName
The name of the resource group that contains the lab.

.PARAMETER ArtifactRepositoryName
Name for the new artifact repository.
Script creates a random name for the repository if it is not specified.

.PARAMETER ArtifactRepositoryDisplayName
Display name for the artifact repository.
This is the name that shows up in the Azure portal (https://portal.azure.com) when viewing all the artifact repositories for a lab.

.PARAMETER RepositoryUri
Uri to the repository.

.PARAMETER RepositoryBranch
Branch in which artifact files can be found. Defaults to 'master'.

.PARAMETER FolderPath
Folder under which artifacts can be found. Defaults to '/Artifacts'

.PARAMETER PersonalAccessToken
Security token for access to GitHub or VSOGit repository.
See https://azure.microsoft.com/documentation/articles/devtest-lab-add-artifact-repo/ for instructions to get personal access token

.PARAMETER SourceType
Whether artifact is VSOGit or GitHub repository.

.EXAMPLE
Set-AzContext -SubscriptionId 11111111-1111-1111-1111-111111111111
.\New-DevTestLabArtifactRepository.ps1 -LabName "mydevtestlab" -LabResourceGroupName "mydtlrg" -ArtifactRepositoryName "MyTeam Repository" -RepositoryUri "https://github.com/<myteam>/<nameofrepo>.git" -PersonalAccessToken "1111...." -SourceType "GitHub"

.NOTES
Script uses the current Az context. To set the context, use the Set-AzContext cmdlet

#>


[CmdletBinding()]
Param(

    [Parameter(Mandatory=$true)]
    $LabName,

    [Parameter(Mandatory=$true)]
    $LabResourceGroupName,
    $ArtifactRepositoryName,
    $ArtifactRepositoryDisplayName  = 'Team Artifact Repository',

    [Parameter(Mandatory=$true)]
    $RepositoryUri,
    $RepositoryBranch = 'master',
    $FolderPath = '/Artifacts',

    [Parameter(Mandatory=$true)]
    $PersonalAccessToken ,

    [Parameter(Mandatory=$true)]
    [ValidateSet('VsoGit', 'GitHub')]
    $SourceType
)


#Set artifact repository internal name,
# if not set by user.

if ($ArtifactRepositoryName -eq $null){
    $ArtifactRepositoryName = "PrivateRepo" + (Get-Random -Maximum 999)
}

# Sign in to Azure
Connect-AzAccount


#Get Lab Resource
$LabResource = Get-AzResource -ResourceType 'Microsoft.DevTestLab/labs' -ResourceName $LabName -ResourceGroupName $LabResourceGroupName

Write-Verbose "Lab Name: $($LabResource.Name)"
Write-Verbose "Lab Resource Group Name: $($LabResource.ResourceGroupName)"
Write-Verbose "Lab Resource Location: $($LabResource.Location)"

Write-Verbose "Artifact Repository Internal Name: $ArtifactRepositoryName"

#Prepare properties object for call to New-AzResource
$propertiesObject = @{
    uri = $RepositoryUri;
    folderPath = $FolderPath;
    branchRef = $RepositoryBranch;
    displayName = $ArtifactRepositoryDisplayName;
    securityToken = $PersonalAccessToken;
    sourceType = $SourceType;
    status = 'Enabled'
}

Write-Verbose @"Properties to be passed to New-AzResource:$($propertiesObject | Out-String)"

#Resource will be added to current subscription.
$resourcetype = 'Microsoft.DevTestLab/labs/artifactSources'
$resourceName = $LabName + '/' + $ArtifactRepositoryName
Write-Verbose "Az ResourceType: $resourcetype"
Write-Verbose "Az ResourceName: $resourceName"

Write-Verbose "Creating artifact repository '$ArtifactRepositoryDisplayName'..."
$result = New-AzResource -Location $LabResource.Location -ResourceGroupName $LabResource.ResourceGroupName -properties $propertiesObject -ResourceType $resourcetype -ResourceName $resourceName -ApiVersion 2016-05-15 -Force


#Alternate implementation:
# Use resourceId rather than resourcetype and resourcename parameters.
# Using resourceId allows you to specify the $SubscriptionId rather than using the
# subscription id of Get-AzContext.
#$resourceId = "/subscriptions/$SubscriptionId/resourceGroups/$($LabResource.ResourceGroupName)/providers/Microsoft.DevTestLab/labs/$LabName/artifactSources/$ArtifactRepositoryName"
#$result = New-AzResource -properties $propertiesObject -ResourceId $resourceId -ApiVersion 2016-05-15 -Force


# Check the result
if ($result.Properties.ProvisioningState -eq "Succeeded") {
    Write-Verbose ("Successfully added artifact repository source '$ArtifactRepositoryDisplayName'")
}
else {
    Write-Error ("Error adding artifact repository source '$ArtifactRepositoryDisplayName'")
}

#Return the newly created resource so it may be used in subsequent scripts
return $result
```

### <a name="run-the-powershell-script"></a>Exécution du script PowerShell
L’exemple suivant indique comment exécuter le script :

```powershell
Set-AzContext -SubscriptionId <Your Azure subscription ID>

.\New-DevTestLabArtifactRepository.ps1 -LabName "mydevtestlab" -LabResourceGroupName "mydtlrg" -ArtifactRepositoryName "MyTeam Repository" -RepositoryUri "https://github.com/<myteam>/<nameofrepo>.git" -PersonalAccessToken "1111...." -SourceType "GitHub"
```


### <a name="parameters"></a>Paramètres
L’exemple de script PowerShell de cet article prend les paramètres suivants :

| Paramètre | Description |
| --------- | ----------- |
| LabName | Nom du lab. |
| ArtifactRepositoryName | Nom du nouveau référentiel d’artefacts. Le script crée un nom aléatoire pour le référentiel, s’il n’est pas spécifié. |
| ArtifactRepositoryDisplayName | Nom d’affichage du référentiel d’artefacts. Il s’agit du nom qui s’affiche dans le Portail Microsoft Azure (https://portal.azure.com) lors de l’affichage de tous les référentiels d’artefacts d’un laboratoire. |
| RepositoryUri | URI vers le référentiel. Exemples : `https://github.com/<myteam>/<nameofrepo>.git` ou `"https://MyProject1.visualstudio.com/DefaultCollection/_git/TeamArtifacts"`.|
| RepositoryBranch | Branche dans laquelle vous pouvez trouver les fichiers d’artefacts. La valeur par défaut est « master ». |
| FolderPath | Dossier sous lequel vous pouvez trouver les artefacts. La valeur par défaut est « /Artifacts ». |
| PersonalAccessToken | Jeton de sécurité pour accéder au référentiel GitHub ou VSOGit. Consultez la section relative aux composants requis pour savoir comment obtenir un jeton accès personnel. |
| SourceType | Indique si l’artefact est un référentiel VSOGit ou GitHub. |

Le référentiel lui-même a besoin d’un nom interne pour l’identification, qui est différent du nom d’affichage affiché sur le Portail Azure. Vous n’affichez pas le nom interne à l’aide du Portail Microsoft Azure, mais vous le voyez lorsque vous utilisez les API REST Azure ou Azure PowerShell. Le script fournit un nom, s’il n’est pas spécifié par l’utilisateur de notre script.

```powershell
#Set artifact repository name, if not set by user
if ($ArtifactRepositoryName -eq $null){
    $ArtifactRepositoryName = "PrivateRepo" + (Get-Random -Maximum 999)
}
```

### <a name="powershell-commands-used-in-the-script"></a>Commandes PowerShell utilisées dans le script

| Commande PowerShell | Notes |
| ------------------ | ----- |
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Cette commande est utilisée pour obtenir des informations sur le laboratoire, telles que son emplacement. |
| [New-AzResource](/powershell/module/az.resources/new-azresource) | Il n’existe aucune commande spécifique pour l’ajout de référentiels d’artefact. La cmdlet [New-AzResource](/powershell/module/az.resources/new-azresource) effectue cette tâche. Cette cmdlet requiert la valeur **ResourceId** ou la paire de valeurs **ResourceName** et **ResourceType** pour connaître le type de ressource à créer. Cet exemple de script utilise le nom de la ressource et de la paire de types de ressource. <br/><br/>Notez que vous créez la source du référentiel d’artefacts au même emplacement et sous le même groupe de ressources que le laboratoire.|

Ce script ajoute une nouvelle ressource à l’abonnement actuel. Utilisez [Get-AzContext](/powershell/module/az.accounts/get-azcontext) pour afficher cette information. Utilisez [Set-AzContext](/powershell/module/az.accounts/set-azcontext) pour définir le l’abonnement et le locataire actuels.

La meilleure façon de découvrir le nom de la ressource et les informations sur le type de ressource consiste à utiliser le site web permettant d’effectuer un [Test Drive des API Azure REST](https://azure.github.io/projects/apis/). Extrayez le fournisseur [DevTest Labs – 2016-05-15](https://aka.ms/dtlrestapis) pour consulter la liste des API REST disponibles pour le fournisseur DevTest Labs. Le script utilise l’ID de ressource suivant.

```powershell
"/subscriptions/$SubscriptionId/resourceGroups/$($LabResource.ResourceGroupName)/providers/Microsoft.DevTestLab/labs/$LabName/artifactSources/$ArtifactRepositoryName"
```

Le type de ressource correspond à l’ensemble du contenu affiché après « providers » dans l’URI, à l’exception des éléments répertoriés entre les accolades. Le nom de ressource correspond à l’ensemble du contenu figurant entre les accolades. Si plusieurs éléments sont attendus pour le nom de ressource, séparez-les avec une barre oblique, comme nous l’avons fait.

```powershell
$resourcetype = 'Microsoft.DevTestLab/labs/artifactSources'
$resourceName = $LabName + '/' + $ArtifactRepositoryName
```


## <a name="next-steps"></a>Étapes suivantes
- [Spécifier des artefacts obligatoires pour une instance Azure DevTest Labs](devtest-lab-mandatory-artifacts.md)
- [Créer des artefacts personnalisés pour votre machine virtuelle DevTest Labs](devtest-lab-artifact-author.md)
- [Diagnostiquer les échecs d’artefacts dans le labo](devtest-lab-troubleshoot-artifact-failure.md)

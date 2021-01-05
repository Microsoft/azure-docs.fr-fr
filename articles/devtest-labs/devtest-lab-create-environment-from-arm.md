---
title: Créer des environnements de plusieurs machines virtuelles et des ressources PaaS à l'aide de modèles
description: Découvrez comment créer des environnements de plusieurs machines virtuelles et des ressources PaaS dans Azure DevTest Labs à partir d’un modèle Azure Resource Manager
ms.topic: article
ms.date: 08/12/2020
ms.openlocfilehash: 97659d4ab95fdbe75460161d0ceed71a1cb5cf82
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88182406"
---
# <a name="create-multi-vm-environments-and-paas-resources-with-azure-resource-manager-templates"></a>Créer des environnements de plusieurs machines virtuelles et des ressources PaaS avec les modèles Azure Resource Manager

Les environnements Azure DevTest Labs permettent aux utilisateurs de déployer facilement des infrastructures complexes de manière cohérente au sein du laboratoire. Vous pouvez utiliser des [modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md) pour créer des environnements avec des ensembles de ressources dans DevTest Labs. Ces environnements peuvent contenir toutes les ressources Azure que des modèles Resource Manager peuvent créer.

Vous pouvez facilement [ajouter une machine virtuelle à la fois](devtest-lab-add-vm.md) à un laboratoire à l’aide du [portail Azure](https://portal.azure.com). Toutefois, les scénarios tels que les applications web à plusieurs niveaux ou une batterie SharePoint ont besoin d’un mécanisme de création de plusieurs machines virtuelles en une seule étape. Les modèles Azure Resource Manager vous permettent de définir l’infrastructure et la configuration de votre solution Azure et de déployer de manière répétée plusieurs machines virtuelles dans un état cohérent.

Les modèles Azure Resource Manager offrent également les avantages suivants :

- Les modèles Azure Resource Manager sont chargés directement à partir de vos référentiels de contrôle de code source GitHub ou Azure Repos.
- Vos utilisateurs peuvent créer un environnement en choisissant un modèle Azure Resource Manager configuré à partir du portail Azure, comme avec les autres types de [bases de machines virtuelles](devtest-lab-comparing-vm-base-image-types.md).
- Vous pouvez approvisionner des ressources PaaS Azure, ainsi que des machines virtuelles IaaS, dans un environnement à partir d’un modèle Azure Resource Manager.
- Vous pouvez suivre le coût des environnements dans le laboratoire en plus des machines virtuelles créées par d’autres types de bases. Les ressources PaaS sont créées et s’affichent dans le suivi des coûts. Toutefois, l’arrêt automatique de machine virtuelle ne s’applique pas aux ressources PaaS.

Pour en savoir plus sur les avantages de l’utilisation de modèles Resource Manager pour déployer, mettre à jour et supprimer de nombreuses ressources de laboratoire en une seule opération, consultez [Avantages de l’utilisation de modèles Resource Manager](../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager).

> [!NOTE]
> Lorsque vous utilisez un modèle Resource Manager comme base pour créer des machines virtuelles de laboratoire, il existe des différences entre la création de plusieurs machines virtuelles et celle d’une machine virtuelle individuelle. Pour plus d’informations, consultez [Utiliser un modèle Azure Resource Manager de machine virtuelle](devtest-lab-use-resource-manager-template.md).
>

## <a name="use-devtest-labs-public-environments"></a>Utiliser des environnements publics DevTest Labs
Azure DevTest Labs a un [référentiel public de modèles Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/Environments) que vous pouvez utiliser pour créer des environnements sans devoir vous connecter vous-même à une source GitHub externe. Ce référentiel public est similaire au référentiel public d’artefacts qui est disponible dans le portail Azure pour chaque laboratoire que vous créez. Le référentiel de l’environnement vous permet de prendre en main rapidement des modèles d’environnement pré-créés qui ont peu de paramètres d’entrée. Ces modèles vous offrent une expérience de prise en main sans soucis pour les ressources PaaS dans les laboratoires.

Dans le référentiel public, l’équipe DevTest Labs et d’autres ont créé et partagé des modèles fréquemment utilisés comme Azure Web Apps, un cluster Service Fabric et un environnement de batterie SharePoint de développement. Vous pouvez utiliser ces modèles directement ou les personnaliser en fonction de vos besoins. Pour plus d’informations, consultez [Configurer et utiliser des environnements publics dans DevTest Labs](devtest-lab-configure-use-public-environments.md). Une fois que vous avez créé vos propres modèles, vous pouvez les stocker dans ce référentiel pour les partager avec d’autres personnes ou configurer votre propre référentiel Git.

<a name="configure-your-own-template-repositories"></a>
## <a name="create-your-own-template-repositories"></a>Créer vos propres référentiels de modèles

Dans le cadre des bonnes pratiques liées à l’Infrastructure as Code et à la Configuration as Code, vous devez gérer des modèles d’environnement dans le contrôle de code source. Azure DevTest Labs suit cette pratique et charge tous les modèles Azure Resource Manager directement à partir de vos référentiels GitHub ou Azure Repos. Par conséquent, vous pouvez utiliser les modèles Resource Manager tout le long du cycle de publication, de l’environnement de test jusqu’à l’environnement de production.

Plusieurs règles sont à prendre en compte pour organiser vos modèles Azure Resource Manager dans un référentiel :

- Vous devez nommer le fichier de modèle principal *azuredeploy.json*.

- Si vous souhaitez utiliser des valeurs de paramètres définies dans un fichier de paramètres, le fichier de paramètres doit être nommé *azuredeploy.parameters.json*.

  Vous pouvez utiliser les paramètres `_artifactsLocation` et `_artifactsLocationSasToken` pour créer la valeur de l’URI parametersLink et permettre à DevTest Labs de gérer automatiquement les modèles imbriqués. Pour plus d’informations, consultez [Déployer des modèles Azure Resource Manager imbriqués pour les environnements de test](deploy-nested-template-environments.md).

- Vous pouvez définir des métadonnées pour spécifier le nom complet et la description du modèle dans un fichier nommé *metadata.json*, comme suit :

  ```json
  {
    "itemDisplayName": "<your template name>",
    "description": "<description of the template>"
  }
  ```

![Fichiers de modèle Azure Resource Manager clés](./media/devtest-lab-create-environment-from-arm/master-template.png)

## <a name="add-template-repositories-to-the-lab"></a>Ajouter des référentiels de modèles au laboratoire

Après avoir créé et configuré votre référentiel, vous pouvez l’ajouter à votre laboratoire à l’aide du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
1. Dans la liste des laboratoires, sélectionnez le laboratoire souhaité.
1. Dans le volet **Vue d’ensemble** du laboratoire, sélectionnez **Configuration et stratégies**.

   ![Configuration et stratégies](./media/devtest-lab-create-environment-from-arm/configuration-and-policies-menu.png)

1. À partir de la liste de paramètres **Configuration et stratégies**, sélectionnez **Référentiels**. Le référentiel **Public Artifact Repo** est généré automatiquement pour tous les laboratoires et se connecte au [référentiel GitHub public de DevTest Labs](https://github.com/Azure/azure-devtestlab).

1. Pour ajouter votre référentiel de modèles Azure Resource Manager, sélectionnez **Ajouter**.

   ![Dépôt public](./media/devtest-lab-create-environment-from-arm/public-repo.png)

1. Dans le volet **Référentiel**, entrez les informations suivantes :

   - **Name** : Entrez un nom de référentiel à utiliser dans le laboratoire.
   - **URL de clonage Git** : Entrez l’URL de clonage Git HTTPS provenant de GitHub ou d’Azure Repos.
   - **Branche** (facultatif) : Entrez le nom de la branche pour accéder à vos définitions de modèle Azure Resource Manager.
   - **Jeton d’accès personnel** : Entrez le jeton d’accès personnel utilisé pour accéder en toute sécurité à votre référentiel.
     - Pour récupérer votre jeton à partir d’Azure Repos, sous votre profil, sélectionnez **Paramètres utilisateur** > **Sécurité** > **Jetons d’accès personnels**.
     - Pour récupérer votre jeton à partir de GitHub, sous votre profil, sélectionnez **Paramètres** > **Paramètres de développeur** > **Jetons d’accès personnels**.
   - **Chemins de dossiers** : Entrez le chemin d’accès du dossier relatif à votre URI de clonage Git pour vos définitions d’artefact ou vos définitions de modèle Azure Resource Manager.

1. Sélectionnez **Enregistrer**.

   ![Ajouter un nouveau référentiel](./media/devtest-lab-create-environment-from-arm/repo-values.png)

Une fois que vous avez ajouté un modèle Azure Resource Manager au laboratoire, vos utilisateurs de laboratoire peuvent créer des environnements à l’aide du modèle.

## <a name="configure-access-rights-for-lab-users"></a>Configurer les droits d’accès pour les utilisateurs du laboratoire

Le rôle **Lecteur** est attribué par défaut aux utilisateurs du laboratoire de sorte qu’ils ne puissent pas modifier les ressources dans un groupe de ressources d’environnement. Par exemple, ils ne peuvent pas arrêter ou démarrer leurs ressources.

Pour attribuer le rôle **Contributeur** aux utilisateurs du laboratoire pour qu’ils puissent modifier les ressources dans leurs environnements, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), dans le volet **Vue d’ensemble** de votre laboratoire, sélectionnez **Configuration et stratégies**, puis sélectionnez **Paramètres du lab**.

1. Dans le volet **Paramètres du lab**, sélectionnez **Contributeur**, puis sélectionnez **Enregistrer** pour accorder des autorisations d’écriture aux utilisateurs de laboratoire.

   ![Configurer les droits d’accès d’utilisateur de laboratoire](./media/devtest-lab-create-environment-from-arm/config-access-rights.png)

La section suivante vous guide dans la création d’environnements à partir d’un modèle Azure Resource Manager.

## <a name="create-environments-from-templates-in-the-azure-portal"></a>Créer des environnements à partir de modèles dans le portail Azure

Une fois que vous avez ajouté un modèle Azure Resource Manager au laboratoire, vos utilisateurs de laboratoire peuvent créer des environnements dans le portail Azure en procédant comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.

1. Dans la liste des laboratoires, sélectionnez le laboratoire souhaité.

1. Dans la page du laboratoire, sélectionnez **Ajouter**.

1. Le volet **Choisir une base** affiche les images de base que vous pouvez utiliser avec les modèles Azure Resource Manager répertoriés en premier. Sélectionnez le modèle Azure Resource Manager souhaité.

   ![Choisir une base](./media/devtest-lab-create-environment-from-arm/choose-a-base.png)

1. Dans le volet **Ajouter**, entrez une valeur **Nom de l’environnement** à présenter aux utilisateurs de l’environnement.

   Le modèle Azure Resource Manager définit les autres champs d’entrée. Si le fichier *azuredeploy.parameter.json* de modèle définit des valeurs par défaut, les champs d’entrée affichent ces valeurs.

   Pour les paramètres de type *chaîne sécurisée*, vous pouvez utiliser des secrets de votre Azure Key Vault. Pour en savoir plus sur le stockage des secrets dans un coffre de clés et leur utilisation lors de la création des ressources de laboratoire, consultez [Stocker les secrets dans Azure Key Vault](devtest-lab-store-secrets-in-key-vault.md).  

   ![Volet Ajouter](./media/devtest-lab-create-environment-from-arm/add.png)

   > [!NOTE]
   > Les valeurs de paramètre suivantes n’apparaissent pas dans les champs d’entrée, même si le modèle les spécifie. Au lieu de cela, le formulaire affiche des champs d’entrée vides dans lesquels les utilisateurs de laboratoire doivent entrer des valeurs lors de la création de l’environnement.
   >
   > - GEN-UNIQUE
   > - GEN-UNIQUE-[N]
   > - GEN-SSH-PUB-KEY
   > - GEN-PASSWORD

1. Sélectionnez **Ajouter** pour créer l’environnement.

   L’environnement lance immédiatement l’approvisionnement avec l’affichage de l’état dans la liste **Mes machines virtuelles**. Le laboratoire crée automatiquement un nouveau groupe de ressources pour approvisionner toutes les ressources définies dans le modèle Azure Resource Manager.

1. Une fois que l’environnement est créé, sélectionnez-le dans la liste **Mes machines virtuelles** pour ouvrir le volet du groupe de ressources et parcourir toutes les ressources approvisionnées par l’environnement.

   ![Ressources d’environnement](./media/devtest-lab-create-environment-from-arm/all-environment-resources.png)

   Vous pouvez également développer l’environnement pour afficher uniquement la liste des machines virtuelles approvisionnées par l’environnement.

   ![Liste Mes machines virtuelles](./media/devtest-lab-create-environment-from-arm/my-vm-list.png)

1. Sélectionnez l’un des environnements pour afficher les actions disponibles comme appliquer des artefacts, attacher des disques de données, modifier le temps d’arrêt automatique, etc.

   ![Actions de l’environnement](./media/devtest-lab-create-environment-from-arm/environment-actions.png)

<a name="automate-deployment-of-environments"></a>
## <a name="automate-environment-creation-with-powershell"></a>Automatiser la création d’environnement avec PowerShell

Il est possible d’utiliser le portail Azure pour ajouter un seul environnement à un laboratoire. Toutefois, lorsqu’un scénario de développement ou de test doit créer plusieurs environnements, le déploiement automatisé est une meilleure option.

Avant de continuer, vérifiez que vous disposez d’un modèle Azure Resource Manager qui définit les ressources à créer. [Ajoutez et configurez le modèle dans un référentiel Git](#configure-your-own-template-repositories) et [ajoutez le référentiel au laboratoire](#add-template-repositories-to-the-lab).

L’exemple de script suivant crée un environnement dans votre laboratoire. Les commentaires vous aident à mieux le comprendre.

1. Enregistrez l’exemple de script PowerShell suivant sur votre disque dur sous *deployenv.ps1*.

   [!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

   ```powershell
   #Requires -Module Az.Resources

   [CmdletBinding()]

   param (
   # ID of the Azure Subscription for the lab
   [string] [Parameter(Mandatory=$true)] $SubscriptionId,

   # Name of the existing lab in which to create the environment
   [string] [Parameter(Mandatory=$true)] $LabName,

   # Name of the connected repository in the lab
   [string] [Parameter(Mandatory=$true)] $RepositoryName,

   # Name of the template (folder name in the Git repository)
   [string] [Parameter(Mandatory=$true)] $TemplateName,

   # Name of the environment to be created in the lab
   [string] [Parameter(Mandatory=$true)] $EnvironmentName,

   # The parameters to be passed to the template. Each parameter is prefixed with "-param_".
   # For example, if the template has a parameter named "TestVMName" with a value of "MyVMName",
   # the string in $Params will have the form: -param_TestVMName MyVMName.
   # This convention allows the script to dynamically handle different templates.
   [Parameter(ValueFromRemainingArguments=$true)]
       $Params
   )

   # Sign in to Azure.
   # Comment out the following statement to completely automate the environment creation.
   Connect-AzAccount

   # Select the subscription that has the lab.  
   Set-AzContext -SubscriptionId $SubscriptionId | Out-Null

   # Get information about the user, specifically the user ID, which is used later in the script.  
   $UserId = $((Get-AzADUser -UserPrincipalName ((Get-AzContext).Account).Id).Id)

   # Get information about the lab, such as lab location.
   $lab = Get-AzResource -ResourceType "Microsoft.DevTestLab/labs" -Name $LabName
   if ($lab -eq $null) { throw "Unable to find lab $LabName in subscription $SubscriptionId." }

   # Get information about the repository in the lab.
   $repository = Get-AzResource -ResourceGroupName $lab.ResourceGroupName `
       -ResourceType 'Microsoft.DevTestLab/labs/artifactsources' `
       -ResourceName $LabName `
       -ApiVersion 2016-05-15 `
       | Where-Object { $RepositoryName -in ($_.Name, $_.Properties.displayName) } `
       | Select-Object -First 1
   if ($repository -eq $null) { throw "Unable to find repository $RepositoryName in lab $LabName." }

   # Get information about the Resource Manager template base for the environment.
   $template = Get-AzResource -ResourceGroupName $lab.ResourceGroupName `
       -ResourceType "Microsoft.DevTestLab/labs/artifactSources/armTemplates" `
       -ResourceName "$LabName/$($repository.Name)" `
       -ApiVersion 2016-05-15 `
       | Where-Object { $TemplateName -in ($_.Name, $_.Properties.displayName) } `
       | Select-Object -First 1
   if ($template -eq $null) { throw "Unable to find template $TemplateName in lab $LabName." }

   # Build the template parameters with parameter name and values.  
   $parameters = Get-Member -InputObject $template.Properties.contents.parameters -MemberType NoteProperty | Select-Object -ExpandProperty Name
   $templateParameters = @()

   # Extract the custom parameters from $Params and format as name/value pairs.
   $Params | ForEach-Object {
       if ($_ -match '^-param_(.*)' -and $Matches[1] -in $parameters) {
           $name = $Matches[1]                
       } elseif ( $name ) {
           $templateParameters += @{ "name" = "$name"; "value" = "$_" }
           $name = $null #reset name variable
       }
   }

   # Once name/value pairs are isolated, create an object to hold the necessary template properties.
   $templateProperties = @{ "deploymentProperties" = @{ "armTemplateId" = "$($template.ResourceId)"; "parameters" = $templateParameters }; }

   # Now, create or deploy the environment in the lab by using the New-AzResource command.
   New-AzResource -Location $Lab.Location `
       -ResourceGroupName $lab.ResourceGroupName `
       -Properties $templateProperties `
       -ResourceType 'Microsoft.DevTestLab/labs/users/environments' `
       -ResourceName "$LabName/$UserId/$EnvironmentName" `
       -ApiVersion '2016-05-15' -Force

   Write-Output "Environment $EnvironmentName completed."
   ```

1. Exécutez le script comme suit, en utilisant vos valeurs spécifiques pour SubscriptionId, LabName, ResourceGroupName, RepositoryName, TemplateName (dossier dans le référentiel Git) et EnvironmentName.

   ```powershell
   ./deployenv.ps1 -SubscriptionId "000000000-0000-0000-0000-0000000000000" -LabName "mydevtestlab" -ResourceGroupName "mydevtestlabRG000000" -RepositoryName "myRepository" -TemplateName "My Environment template name" -EnvironmentName "myGroupEnv"
   ```

Vous pouvez aussi utiliser Azure CLI pour déployer des ressources à l’aide de modèles Resource Manager. Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure CLI](../azure-resource-manager/templates/deploy-cli.md).

> [!NOTE]
> Seul un utilisateur disposant des autorisations de propriétaire de laboratoire peut créer des machines virtuelles à partir d’un modèle Resource Manager avec Azure PowerShell. Si vous souhaitez automatiser la création de machines virtuelles à l’aide d’un modèle Resource Manager et que vous disposez uniquement d’autorisations utilisateur, vous pouvez utiliser la commande CLI [az lab vm create](/cli/azure/lab/vm#az-lab-vm-create).

## <a name="resource-manager-template-limitations-in-devtest-labs"></a>Limitations du modèle Resource Manager dans DevTest Labs

Prenez en compte les limitations suivantes lors de l’utilisation de modèles Resource Manager dans DevTest Labs :

- Vous ne pouvez pas créer de formules ou d’images personnalisées à partir de machines virtuelles de laboratoire qui ont été créées à partir d’un modèle Resource Manager.

- La plupart des stratégies ne sont pas évaluées lorsque vous déployez des modèles Resource Manager.

Par exemple, vous pouvez disposer d’une stratégie de laboratoire telle qu’un utilisateur peut créer cinq machines virtuelles uniquement. Toutefois, un utilisateur peut déployer un modèle Resource Manager qui crée des dizaines de machines virtuelles. Les stratégies qui ne sont pas évaluées incluent :

  - Nombre de machines virtuelles par utilisateur

  - Nombre de machines virtuelles Premium par utilisateur de laboratoire

  - Nombre de disques Premium par utilisateur de laboratoire

## <a name="next-steps"></a>Étapes suivantes
- Lorsque vous créez une machine virtuelle, vous pouvez vous connecter à celle-ci en sélectionnant **Connexion** dans le volet de gestion de la machine virtuelle.
- Vous pouvez afficher et gérer les ressources d’un environnement en sélectionnant celui-ci dans la liste **Mes machines virtuelles** de votre laboratoire.
- Explorez les [modèles Azure Resource Manager de la galerie de modèles Azure QuickStart](https://github.com/Azure/azure-quickstart-templates).

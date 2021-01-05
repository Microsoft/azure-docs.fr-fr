---
title: Intégrer Azure DevTest Labs à Azure Pipelines
description: Découvrez comment intégrer Azure DevTest Labs à votre pipeline de livraison et d’intégration continue d’Azure Pipelines.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 96f99d41d0a7ea07bf3854292f9c3bd6245414b3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87288927"
---
# <a name="integrate-azure-devtest-labs-into-your-azure-pipelines-cicd-pipeline"></a>Intégrer Azure DevTest Labs dans votre pipeline CI/CD Azure Pipelines

Vous pouvez utiliser l’extension *Tâches Azure DevTest Labs* pour intégrer les pipelines de génération et de mise en production Intégration et livraison continues (CI/CD) Azure Pipelines dans Azure DevTest Labs. L’extension installe plusieurs tâches, notamment : 

- Créer une machine virtuelle
- Créer une image personnalisée à partir d’une machine virtuelle
- Supprimer une machine virtuelle 

Ces tâches facilitent, par exemple, le déploiement rapide d’une machine virtuelle avec *image de référence* pour une tâche de test spécifique, puis sa suppression une fois le test terminé.

Cet article explique comment utiliser des tâches Azure DevTest Labs pour créer et déployer une machine virtuelle, créer une image personnalisée, puis supprimer la machine virtuelle, le tout en tant que pipeline de mise en production. Normalement, vous effectuez les tâches individuellement dans vos pipelines personnalisés de génération, test et déploiement.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="prerequisites"></a>Prérequis

- Inscrivez-vous ou connectez-vous à votre organisation [Azure DevOps](https://dev.azure.com), puis [créez un projet](/vsts/organizations/projects/create-project) dans l’organisation. 
  
- Installez l’extension Azure DevTest Labs Tasks à partir de Visual Studio Marketplace.
  
  1. Accédez à [Azure DevTest Labs Tasks](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks).
  1. Sélectionnez **Get it free** (Obtenir gratuitement).
  1. Sélectionnez votre organisation Azure DevOps dans la liste déroulante, puis sélectionnez **Installer**. 
  
## <a name="create-the-template-to-build-an-azure-vm"></a>Créer le modèle pour générer une machine virtuelle Azure 

Cette section décrit comment créer le modèle Azure Resource Manager que vous utilisez pour créer une machine virtuelle Azure à la demande.

1. Pour créer un modèle Resource Manager dans votre abonnement, suivez la procédure indiquée dans [Utiliser un modèle Resource Manager](devtest-lab-use-resource-manager-template.md).
   
1. Avant de générer le modèle Resource Manager, ajoutez [l’artefact WinRM](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts/windows-winrm) dans le cadre de la création de la machine virtuelle. L’accès à WinRM est requis pour les tâches de déploiement telles que *Copie de fichiers Azure* et *PowerShell sur des ordinateurs cibles*. L’artefact WinRM requiert un nom d’hôte en tant que paramètre, qui doit être le nom de domaine complet (FQDN) de la machine virtuelle. 
   
   > [!NOTE]
   > Quand vous utilisez WinRM avec une adresse IP partagée, vous devez ajouter une règle NAT pour mapper un port externe au port WinRM. La règle NAT n’est pas requise si vous créez la machine virtuelle avec une adresse IP publique.
   
   
1. Enregistrez le modèle sur votre ordinateur en tant que fichier nommé *CreateVMTemplate.json*.
   
1. Archivez le modèle dans votre système de contrôle de code source.

## <a name="create-a-script-to-get-vm-properties"></a>Créer un script pour obtenir les propriétés d’une machine virtuelle

Lorsque vous exécutez des étapes de tâche telles que la *copie de fichiers Azure* ou *PowerShell sur des machines cibles* dans le pipeline de mise en production, le script suivant collecte les valeurs dont vous avez besoin pour déployer une application sur une machine virtuelle. Normalement, vous utilisez ces tâches pour déployer votre application sur une machine virtuelle Azure. Les tâches requièrent des valeurs telles que le nom du groupe de ressources de la machine virtuelle, une adresse IP et un nom de domaine complet.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour créer le fichier de script :

1. Ouvrez un éditeur de texte et collez-y le script suivant.
   
   ```powershell
   Param( [string] $labVmId)

   $labVmComputeId = (Get-AzResource -Id $labVmId).Properties.ComputeId

   # Get lab VM resource group name
   $labVmRgName = (Get-AzResource -Id $labVmComputeId).ResourceGroupName

   # Get the lab VM Name
   $labVmName = (Get-AzResource -Id $labVmId).Name

   # Get lab VM public IP address
   $labVMIpAddress = (Get-AzPublicIpAddress -ResourceGroupName $labVmRgName
                   -Name $labVmName).IpAddress

   # Get lab VM FQDN
   $labVMFqdn = (Get-AzPublicIpAddress -ResourceGroupName $labVmRgName
              -Name $labVmName).DnsSettings.Fqdn

   # Set a variable labVmRgName to store the lab VM resource group name
   Write-Host "##vso[task.setvariable variable=labVmRgName;]$labVmRgName"

   # Set a variable labVMIpAddress to store the lab VM Ip address
   Write-Host "##vso[task.setvariable variable=labVMIpAddress;]$labVMIpAddress"

   # Set a variable labVMFqdn to store the lab VM FQDN name
   Write-Host "##vso[task.setvariable variable=labVMFqdn;]$labVMFqdn"
   ```

1. Enregistrez le fichier sous un nom comme *GetLabVMParams.ps1*, puis archivez-le dans votre système de contrôle de code source. 

## <a name="create-a-release-pipeline-in-azure-pipelines"></a>Créer un pipeline de mise en production dans Azure Pipelines

Pour créer un pipeline de mise en production :

1. Dans la page de votre projet Azure DevOps, sélectionnez **Pipelines** > **Mises en production** dans le volet de navigation de gauche.
1. Sélectionnez **Nouveau pipeline**.
1. Sous **Sélectionner un modèle**, faites défiler la liste vers le bas et sélectionnez **Empty job** (Tâche vide), puis sélectionnez **Appliquer**.

### <a name="add-and-set-variables"></a>Ajouter et définir les variables

Les tâches du pipeline utilisent les valeurs que vous avez assignées à la machine virtuelle quand vous avez créé le modèle Resource Manager dans le Portail Azure. 

Pour ajouter des variables pour les valeurs : 

1. Sur la page du pipeline, sélectionnez l’onglet **Variables**.
   
1. Pour chaque variable, sélectionnez **Ajouter**, puis entrez le nom et la valeur :
   
   |Nom|Valeur|
   |---|---|
   |*vmName*|Nom de la machine virtuelle que vous avez assigné dans le modèle Resource Manager|
   |*userName*|Nom d’utilisateur pour accéder à la machine virtuelle|
   |*mot de passe*|Mot de passe du nom d’utilisateur. Sélectionnez l’icône en forme de cadenas pour masquer et sécuriser le mot de passe.

### <a name="create-a-devtest-labs-vm"></a>Créer une machine virtuelle DevTest Labs

L'étape suivante consiste à créer la machine virtuelle avec l'image de référence à utiliser pour les futurs déploiements. Vous créez la machine virtuelle au sein de votre instance Azure DevTest Labs à l’aide de la tâche *Créer une machine virtuelle Azure DevTest Labs*.

1. Dans l’onglet **Pipeline** du pipeline de mise en production, sélectionnez le lien hypertexte à l’**étape 1** d’**Afficher les tâches d’index**, puis sélectionnez le signe plus **+** en regard du **travail Agent**. 
   
1. Sous **Ajouter des tâches**, sélectionnez **Créer une machine virtuelle Azure DevTest Labs**, puis sélectionnez **Ajouter**. 
   
1. Sélectionnez **Créer une machine virtuelle Azure DevTest Labs** dans le volet gauche. 

1. Dans le volet de droite, remplissez le formulaire comme suit :
   
   |Champ|Valeur|
   |---|---|
   |**Abonnement Azure RM**|Sélectionnez une connexion ou un abonnement au service dans la liste déroulante **Connexions au service Azure disponibles** ou **Abonnements Azure disponibles**, puis sélectionnez **Autoriser** si nécessaire.<br /><br />**Remarque :** Pour plus d’informations sur la création d’une connexion d’autorisations plus restreinte à votre abonnement Azure, consultez [Point de terminaison de service Azure Resource Manager](/azure/devops/pipelines/library/service-endpoints#sep-azure-resource-manager).|
   |**Nom du labo**|Sélectionnez le nom d’un labo existant dans lequel la machine virtuelle de labo sera créée.|
   |**Nom du modèle**|Entrez le chemin complet et le nom du fichier de modèle que vous avez enregistré dans votre dépôt de code source. Vous pouvez utiliser les propriétés intégrées pour simplifier le chemin, par exemple :<br /><br />`$(System.DefaultWorkingDirectory)/Templates/CreateVMTemplate.json`|
   |**Paramètres du modèle**|Entrez les paramètres des variables que vous avez définies précédemment :<br /><br />`-newVMName '$(vmName)' -userName '$(userName)' -password (ConvertTo-SecureString -String '$(password)' -AsPlainText -Force)`|
   |**Variables de sortie** > **ID de machine virtuelle de labo**|Entrez la variable pour l’identifiant de la machine virtuelle de labo créée. Si vous utilisez la valeur par défaut **labVMId**, vous pouvez faire référence à la variable en tant que *$(labVMId)* dans les tâches suivantes.<br /><br />Vous pouvez créer un nom autre que celui par défaut, mais veillez à utiliser le nom adéquat dans les tâches suivantes. L’ID de la machine virtuelle du labo est écrit sous la forme suivante :<br /><br />`/subscriptions/{subscription Id}/resourceGroups/{resource group Name}/providers/Microsoft.DevTestLab/labs/{lab name}/virtualMachines/{vmName}`|

### <a name="collect-the-details-of-the-devtest-labs-vm"></a>Collecter les détails de la machine virtuelle DevTest Labs

Exécutez le script que vous avez créé pour collecter les détails de la machine virtuelle DevTest Labs. 

1. Dans l’onglet **Pipeline** du pipeline de mise en production, sélectionnez le lien hypertexte à l’**étape 1** d’**Afficher les tâches d’index**, puis sélectionnez le signe plus **+** en regard du **travail Agent**. 
   
1. Sous **Ajouter des tâches**, sélectionnez **Azure PowerShell**, puis **Ajouter**. 
   
1. Sélectionnez **Script Azure PowerShell : FilePath** dans le volet gauche. 
   
1. Dans le volet de droite, remplissez le formulaire comme suit :
   
   |Champ|Valeur|
   |---|---|
   |**Type de connexion Azure**|Sélectionnez **Azure Resource Manager**.|
   |**Abonnement Azure**|Sélectionnez votre connexion ou abonnement au service.| 
   |**Type de script**|Sélectionnez **Chemin du fichier de script**.|
   |**Chemin du script**|Entrez le chemin complet et le nom du script PowerShell que vous avez enregistré dans votre dépôt de code source. Vous pouvez utiliser les propriétés intégrées pour simplifier le chemin, par exemple :<br /><br />`$(System.DefaultWorkingDirectory/Scripts/GetLabVMParams.ps1`|
   |**Arguments de script**|Entrez le nom de la variable *labVmId* qui a été remplie par la tâche précédente, par exemple :<br /><br />`-labVmId '$(labVMId)'`|

Le script collecte les valeurs requises et les stocke dans des variables d’environnement au sein du pipeline de mise en production afin que vous puissiez facilement y faire référence dans les étapes suivantes.

### <a name="create-a-vm-image-from-the-devtest-labs-vm"></a>Créer une image de machine virtuelle à partir de la machine virtuelle DevTest Labs

La tâche suivante consiste à créer, dans votre instance Azure DevTest Labs, une image de la machine virtuelle qui vient d’être déployée. Vous pouvez ensuite utiliser l’image pour créer des copies de la machine virtuelle à la demande chaque fois que vous souhaitez exécuter une tâche de développement ou exécuter des tests. 

1. Dans l’onglet **Pipeline** du pipeline de mise en production, sélectionnez le lien hypertexte à l’**étape 1** d’**Afficher les tâches d’index**, puis sélectionnez le signe plus **+** en regard du **travail Agent**. 
   
1. Sous **Ajouter des tâches**, sélectionnez **Créer une image personnalisée Azure DevTest Labs**, puis sélectionnez **Ajouter**. 
   
1. Configurez la tâche comme indiqué ci-dessous :
   
   |Champ|Valeur|
   |---|---|
   |**Abonnement Azure RM**|Sélectionnez votre connexion ou abonnement au service.|
   |**Nom du labo**|Sélectionnez le nom d’un labo existant dans lequel l’image sera créée.|
   |**Nom de l’image personnalisée**|Entrez un nom pour l'image personnalisée.|
   |**Description** (facultatif)|Entrez une description afin de faciliter la sélection de l’image appropriée ultérieurement.|
   |**Machine virtuelle de labo source** > **ID de machine virtuelle de labo source**|Si vous avez modifié le nom par défaut de la variable LabVMId, entrez-le ici. La valeur par défaut est **$(labVMId)** .|
   |**Variables de sortie** > **ID de l’image personnalisé**|Vous pouvez modifier le nom par défaut de la variable si nécessaire.|
   
### <a name="deploy-your-app-to-the-devtest-labs-vm-optional"></a>Déployer votre application sur la nouvelle machine virtuelle DevTest Labs (facultatif)

Vous pouvez ajouter des tâches pour déployer votre application sur la nouvelle machine virtuelle DevTest Labs. Les tâches que vous utilisez habituellement pour déployer l’application sont *Copie de fichiers Azure* et *PowerShell sur des ordinateurs cibles*.

Les informations sur la machine virtuelle dont vous avez besoin pour les paramètres de ces tâches sont stockées dans trois variables de configuration nommées **labVmRgName**, **labVMIpAddress** et **labVMFqdn** au sein du pipeline de mise en production. Si vous souhaitez uniquement tester la création d’une machine virtuelle DevTest Labs et d’une image personnalisée, sans y déployer d’application, vous pouvez ignorer cette étape.

### <a name="delete-the-vm"></a>Supprimer la machine virtuelle

La dernière tâche consiste à supprimer la machine virtuelle que vous avez déployée dans votre instance Azure DevTest Labs. Normalement, vous supprimez la machine virtuelle après avoir exécuté les tâches de développement ou exécuté les tests dont vous avez besoin sur la machine virtuelle déployée. 

1. Dans l’onglet **Pipeline** du pipeline de mise en production, sélectionnez le lien hypertexte à l’**étape 1** d’**Afficher les tâches d’index**, puis sélectionnez le signe plus **+** en regard du **travail Agent**. 
   
1. Sous **Ajouter des tâches**, sélectionnez **Supprimer la machine virtuelle Azure DevTest Labs**, puis sélectionnez **Ajouter**. 
   
1. Configurez la tâche comme indiqué ci-dessous :
   
   - Sous **Abonnement Azure RM**, sélectionnez votre connexion ou abonnement au service. 
   - Pour **Lab VM ID** (ID de machine virtuelle de labo), si vous avez modifié le nom par défaut de la variable LabVMId, entrez-le ici. La valeur par défaut est **$(labVMId)** .
   
### <a name="save-the-release-pipeline"></a>Enregistrer le pipeline de mise en production

Pour enregistrer le nouveau pipeline de mise en production :

1. Sélectionnez le nom **Nouveau pipeline de mise en production** dans la page Pipeline de mise en production, puis renommez le pipeline. 
   
1. Sélectionnez l’icône **Enregistrer** en haut à droite. 

## <a name="create-and-run-a-release"></a>Créer et exécuter une mise en production

Pour créer et exécuter une mise en version à l’aide du nouveau pipeline :

1. Sélectionnez **Créer une mise en production** en haut à droite de la page du pipeline de mise en production. 
   
1. Sous **Artefacts**, sélectionnez le dernier build, puis sélectionnez **Créer**.
   
1. À chaque étape de la mise en production, actualisez l’affichage de votre instance DevTest Labs dans le Portail Azure pour afficher la création de la machine virtuelle, la création de l’image et la suppression de la machine virtuelle.

Vous pouvez utiliser l’image personnalisée pour créer des machines virtuelles quand vous en avez besoin.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [Créer des environnements à plusieurs machines virtuelles avec les modèles Resource Manager](devtest-lab-create-environment-from-arm.md).
- Découvrez les autres modèles Resource Manager à démarrage rapide pour l’automatisation de DevTest Labs à partir du [dépôt DevTest Labs GitHub public](https://github.com/Azure/azure-quickstart-templates).
- Si nécessaire, consultez la page [Azure DevOps Troubleshooting](/azure/devops/pipelines/troubleshooting) (Résolution des problèmes liés à Azure DevOps).
 

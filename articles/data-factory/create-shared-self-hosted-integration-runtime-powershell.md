---
title: Créer un runtime d’intégration auto-hébergé partagé à l’aide de PowerShell
description: Apprenez à créer un runtime d’intégration auto-hébergé partagé dans Azure Data Factory afin que plusieurs fabriques de données aient accès au runtime d’intégration.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: abnarain
author: nabhishek
manager: anansub
ms.custom: seo-lt-2019
ms.date: 06/10/2020
ms.openlocfilehash: 8734247a913bdf6a44a9156f6f87705b618f7228
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92632887"
---
# <a name="create-a-shared-self-hosted-integration-runtime-in-azure-data-factory"></a>Créer un runtime d’intégration auto-hébergé partagé dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Ce guide explique comment créer un runtime d’intégration auto-hébergé partagé dans Azure Data Factory. Vous pouvez ensuite utiliser le runtime d’intégration auto-hébergé partagé dans une autre fabrique de données.

## <a name="create-a-shared-self-hosted-ir-using-azure-data-factory-ui"></a>Créer un runtime d’intégration auto-hébergé partagé à l’aide de l’IU Azure Data Factory

Pour créer un runtime d’intégration auto-hébergé partagé à l’aide de l’IU Azure Data Factory, effectuez les étapes suivantes :

1. Dans le runtime d’intégration auto-hébergé à partager, sélectionnez **Octroyer une autorisation à une autre fabrique de données** et dans la page « Configuration du runtime d’intégration », sélectionnez la fabrique de données dans laquelle vous souhaitez créer le runtime d’intégration lié.
      
    ![Bouton pour accorder l’autorisation sur l’onglet Partage](media/create-self-hosted-integration-runtime/grant-permissions-IR-sharing.png)  
    
2. Notez et copiez « l’ID de ressource » ci-dessus du runtime d’intégration auto-hébergé à partager.
         
3. Dans la fabrique de données à laquelle les autorisations ont été accordées, créez un runtime d’intégration auto-hébergé (lié) et entrez l’ID de ressource.
      
    ![Bouton pour créer un runtime d’intégration auto-hébergé](media/create-self-hosted-integration-runtime/create-linkedir-1.png)
   
    ![Bouton pour créer un runtime d’intégration auto-hébergé lié](media/create-self-hosted-integration-runtime/create-linkedir-2.png) 

    ![Champs de saisie du nom et de l’ID de ressource](media/create-self-hosted-integration-runtime/create-linkedir-3.png)

## <a name="create-a-shared-self-hosted-ir-using-azure-powershell"></a>Créer un runtime d’intégration auto-hébergé partagé à l’aide d’Azure PowerShell

Pour créer un runtime d’intégration auto-hébergé partagé à l’aide d’Azure PowerShell, effectuez les étapes suivantes : 
1. Créer une fabrique de données. 
1. Créez un runtime d’intégration auto-hébergé.
1. Partager le runtime d’intégration auto-hébergé avec d’autres fabriques de données.
1. Créer un runtime d’intégration lié.
1. Révoquer le partage.

### <a name="prerequisites"></a>Prérequis 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Abonnement Azure** . Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. 

- **Azure PowerShell** . Suivez les instructions de l'article [Installer Azure PowerShell sur Windows avec PowerShellGet](/powershell/azure/install-az-ps). Vous utilisez PowerShell pour exécuter un script et créer un runtime d’intégration auto-hébergé pouvant être partagé avec d’autres fabriques de données. 

> [!NOTE]  
> Pour obtenir la liste des régions Azure dans lesquelles Data Factory est actuellement disponible, sélectionnez les régions qui vous intéressent sur la page [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=data-factory).

### <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Lancez l’environnement d’écriture de scripts intégré de Windows PowerShell (ISE).

1. Créez des variables. Copiez et collez le script suivant. Remplacez les variables, telles que **SubscriptionName** et **ResourceGroupName** , par des valeurs réelles : 

    ```powershell
    # If input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$". 
    $SubscriptionName = "[Azure subscription name]" 
    $ResourceGroupName = "[Azure resource group name]" 
    $DataFactoryLocation = "EastUS" 

    # Shared Self-hosted integration runtime information. This is a Data Factory compute resource for running any activities 
    # Data factory name. Must be globally unique 
    $SharedDataFactoryName = "[Shared Data factory name]" 
    $SharedIntegrationRuntimeName = "[Shared Integration Runtime Name]" 
    $SharedIntegrationRuntimeDescription = "[Description for Shared Integration Runtime]"

    # Linked integration runtime information. This is a Data Factory compute resource for running any activities
    # Data factory name. Must be globally unique
    $LinkedDataFactoryName = "[Linked Data factory name]"
    $LinkedIntegrationRuntimeName = "[Linked Integration Runtime Name]"
    $LinkedIntegrationRuntimeDescription = "[Description for Linked Integration Runtime]"
    ```

1. Connectez-vous et sélectionnez un abonnement. Ajoutez le code suivant au script pour vous connecter et sélectionner votre abonnement Azure :

    ```powershell
    Connect-AzAccount
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

1. Créez un groupe de ressources et une fabrique de données.

    > [!NOTE]  
    > Cette étape est facultative. Si vous disposez déjà d’une fabrique de données, ignorez cette étape. 

    Créez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe. L’exemple suivant crée un groupe de ressources nommé `myResourceGroup` à l’emplacement Europe Ouest : 

    ```powershell
    New-AzResourceGroup -Location $DataFactoryLocation -Name $ResourceGroupName
    ```

    Exécutez la commande suivante pour créer la fabrique de données : 

    ```powershell
    Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
                             -Location $DataFactoryLocation `
                             -Name $SharedDataFactoryName
    ```

### <a name="create-a-self-hosted-integration-runtime"></a>Créer un runtime d’intégration auto-hébergé

> [!NOTE]  
> Cette étape est facultative. Si vous disposez déjà du runtime d’intégration auto-hébergé que vous souhaitez partager avec d’autres fabriques de données, ignorez cette étape.

Exécutez la commande suivante pour créer un runtime d’intégration auto-hébergé :

```powershell
$SharedIR = Set-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $SharedDataFactoryName `
    -Name $SharedIntegrationRuntimeName `
    -Type SelfHosted `
    -Description $SharedIntegrationRuntimeDescription
```

#### <a name="get-the-integration-runtime-authentication-key-and-register-a-node"></a>Obtenir la clé d’authentification du runtime d’intégration et inscrire un nœud

Exécutez la commande suivante pour obtenir la clé d’authentification du runtime d’intégration auto-hébergé :

```powershell
Get-AzDataFactoryV2IntegrationRuntimeKey `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $SharedDataFactoryName `
    -Name $SharedIntegrationRuntimeName
```

La réponse contient la clé d’authentification de ce runtime d’intégration auto-hébergé. Vous utilisez cette clé lorsque vous inscrivez le nœud du runtime d’intégration.

#### <a name="install-and-register-the-self-hosted-integration-runtime"></a>Installer et inscrire le runtime d’intégration auto-hébergé

1. Téléchargez le programme d’installation du runtime intégration auto-hébergé depuis [Azure Data Factory Integration Runtime](https://aka.ms/dmg).

2. Exécutez le programme d’installation pour installer l’intégration auto-hébergée sur un ordinateur local.

3. Inscrivez la nouvelle intégration auto-hébergée à l’aide de la clé d’authentification que vous avez récupérée à l’étape précédente.

### <a name="share-the-self-hosted-integration-runtime-with-another-data-factory"></a>Partager le runtime d’intégration auto-hébergé avec une autre fabrique de données

#### <a name="create-another-data-factory"></a>Créer une autre fabrique de données

> [!NOTE]  
> Cette étape est facultative. Si vous disposez déjà de la fabrique de données avec laquelle vous souhaitez procéder au partage, ignorez cette étape. Mais pour ajouter ou supprimer des attributions de rôles à d’autres fabriques de données, vous devez disposer des autorisations `Microsoft.Authorization/roleAssignments/write` et `Microsoft.Authorization/roleAssignments/delete`, comme [Administrateur de l’accès utilisateur](../role-based-access-control/built-in-roles.md#user-access-administrator) ou [Propriétaire](../role-based-access-control/built-in-roles.md#owner).

```powershell
$factory = Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
    -Location $DataFactoryLocation `
    -Name $LinkedDataFactoryName
```
#### <a name="grant-permission"></a>Accorder l’autorisation

Accordez l’autorisation à la fabrique de données qui doit accéder au runtime d’intégration auto-hébergé que vous avez créé et inscrit.

> [!IMPORTANT]  
> Ne sautez pas cette étape !

```powershell
New-AzRoleAssignment `
    -ObjectId $factory.Identity.PrincipalId ` #MSI of the Data Factory with which it needs to be shared
    -RoleDefinitionName 'Contributor' `
    -Scope $SharedIR.Id
```

### <a name="create-a-linked-self-hosted-integration-runtime"></a>Créer un runtime d’intégration auto-hébergé lié

Exécutez la commande suivante pour créer un runtime d’intégration auto-hébergé lié :

```powershell
Set-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $LinkedDataFactoryName `
    -Name $LinkedIntegrationRuntimeName `
    -Type SelfHosted `
    -SharedIntegrationRuntimeResourceId $SharedIR.Id `
    -Description $LinkedIntegrationRuntimeDescription
```

À présent, vous pouvez utiliser ce runtime d’intégration lié dans n’importe quel service lié. Le runtime d’intégration lié utilise le runtime d’intégration partagé pour exécuter des activités.

### <a name="revoke-integration-runtime-sharing-from-a-data-factory"></a>Révoquer le partage du runtime d’intégration à partir d’une fabrique de données

Pour révoquer l'accès d'une fabrique de données à partir du runtime d'intégration partagé, exécutez la commande suivante :

```powershell
Remove-AzRoleAssignment `
    -ObjectId $factory.Identity.PrincipalId `
    -RoleDefinitionName 'Contributor' `
    -Scope $SharedIR.Id
```

Pour supprimer le runtime d’intégration lié existant, exécutez la commande suivante sur le runtime d’intégration partagé :

```powershell
Remove-AzDataFactoryV2IntegrationRuntime `
    -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $SharedDataFactoryName `
    -Name $SharedIntegrationRuntimeName `
    -Links `
    -LinkedDataFactoryName $LinkedDataFactoryName
```

### <a name="next-steps"></a>Étapes suivantes

- Étudiez les [concepts de runtime d’intégration dans Azure Data Factory](./concepts-integration-runtime.md).

- Apprenez à [créer un runtime d’intégration auto-hébergé sur le portail Azure](./create-self-hosted-integration-runtime.md).
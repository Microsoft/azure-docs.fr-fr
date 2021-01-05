---
title: 'Démarrage rapide : Créer un espace de travail Synapse avec Azure PowerShell'
description: Créez un espace de travail Azure Synapse avec Azure PowerShell en suivant les étapes décrites dans ce guide.
services: synapse-analytics
author: alehall
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: overview
ms.date: 10/19/2020
ms.author: alehall
ms.reviewer: jrasnick
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 005e3a3b717d4b1b8e5eb02b77a1d228908f8707
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92210565"
---
# <a name="quickstart-create-an-azure-synapse-workspace-with-azure-powershell"></a>Démarrage rapide : Créer un espace de travail Azure Synapse avec Azure PowerShell

Azure PowerShell est un ensemble d’applets de commande pour la gestion des ressources Azure directement à partir de PowerShell. Vous pouvez l’utiliser dans votre navigateur avec Azure Cloud Shell. Vous pouvez également l’installer sur macOS, Linux ou Windows.

Dans ce guide de démarrage rapide, vous allez apprendre à créer un espace de travail Synapse à l’aide d’Azure PowerShell.

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

- [Compte de stockage Azure Data Lake Storage Gen2](../storage/common/storage-account-create.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)

    > [!IMPORTANT]
    > L’espace de travail Azure Synapse doit pouvoir lire et écrire dans le compte ADLS Gen2 sélectionné. Pour tout compte de stockage que vous liez en tant que compte de stockage principal, vous devez activer l’ **espace de noms hiérarchique** lors de la création du compte de stockage, tel que décrit dans [Créer un compte de stockage](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-powershell#create-a-storage-account).

Si vous choisissez d’utiliser Cloud Shell, consultez [Vue d’ensemble d’Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview) pour plus d’informations.

### <a name="install-the-azure-powershell-module-locally"></a>Installer localement le module Azure PowerShell

Si vous choisissez d’utiliser PowerShell localement, cet article vous demande d’installer le module Az PowerShell et de vous connecter à votre compte Azure avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Pour en savoir plus sur l’installation du module Az PowerShell, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps).

Pour plus d’informations sur l’authentification avec Azure PowerShell, consultez [Se connecter avec Azure PowerShell](/powershell/azure/authenticate-azureps).

### <a name="install-the-azure-synapse-powershell-module"></a>Installer le module PowerShell d’Azure Synapse

> [!IMPORTANT]
> Tant que le module PowerShell **Az.Synapse** est en préversion, vous devez l’installer séparément à l’aide de l’applet de commande `Install-Module`. Une fois que ce module PowerShell sera en disponibilité générale, il fera partie intégrante des versions futures du module Az PowerShell et sera disponible par défaut dans Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.Synapse
```

## <a name="create-an-azure-synapse-workspace-using-azure-powershell"></a>Créer un espace de travail Azure Synapse à l’aide d’Azure PowerShell

1. Définissez les variables d’environnement nécessaires qui permettent de créer des ressources pour l’espace de travail Azure Synapse.

   |        Nom de la variable        |                                                 Description                                                 |
   | --------------------------- | ----------------------------------------------------------------------------------------------------------- |
   | StorageAccountName          | Nom de votre compte de stockage ADLS Gen2 existant.                                                           |
   | StorageAccountResourceGroup | Nom du groupe de ressources de votre compte de stockage ADLS Gen2 existant.                                             |
   | FileShareName               | Nom de votre système de fichiers de stockage existant.                                                                  |
   | SynapseResourceGroup        | Choisissez un nouveau nom pour votre groupe de ressources Azure Synapse.                                                    |
   | Région                      | Choisissez une des [régions Azure](https://azure.microsoft.com/global-infrastructure/geographies/#overview). |
   | SynapseWorkspaceName        | Choisissez un nom unique pour votre nouvel espace de travail Azure Synapse.                                                  |
   | SqlUser                     | Choisissez une valeur pour un nouveau nom d’utilisateur.                                                                          |
   | SqlPassword                 | Choisissez un mot de passe sécurisé.                                                                                   |
   | ClientIP                    | Adresse IP publique du système à partir duquel vous exécutez PowerShell.                                             |
   |                             |                                                                                                             |

1. Créez un groupe de ressources en tant que conteneur pour votre espace de travail Azure Synapse :

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $SynapseResourceGroup -Location $Region
   ```

1. Récupérez la clé du compte de stockage ADLS Gen 2 :

   ```azurepowershell-interactive
   $StorageAccountKey = Get-AzStorageAccountKey -ResourceGroupName $StorageAccountResourceGroup -Name $StorageAccountName |
     Select-Object -First 1 -ExpandProperty Value
    ```

1. Récupérez l’URL du point de terminaison du stockage ADLS Gen 2 :

   ```azurepowershell-interactive
   $StorageEndpointUrl = (Get-AzStorageAccount -ResourceGroupName $StorageAccountResourceGroup -Name $StorageAccountName).PrimaryEndpoints.Dfs
   ```

1. (Facultatif) Vous pouvez toujours vérifier quels sont vos clé et point de terminaison du compte de stockage ADLS Gen2 :

   ```azurepowershell-interactive
   Write-Output "Storage Account Key: $StorageAccountKey"
   Write-Output "Storage Endpoint URL: $StorageEndpointUrl"
   ```

1. Créer un espace de travail Azure Synapse :

   ```azurepowershell-interactive
   $Cred = New-Object -TypeName System.Management.Automation.PSCredential ($SqlUser, (ConvertTo-SecureString $SqlPassword -AsPlainText -Force))

   $WorkspaceParams = @{
     Name = $SynapseWorkspaceName
     ResourceGroupName = $SynapseResourceGroup
     DefaultDataLakeStorageAccountName = $StorageAccountName
     DefaultDataLakeStorageFilesystem = $FileShareName
     SqlAdministratorLoginCredential = $Cred
     Location = $Region
   }
   New-AzSynapseWorkspace @WorkspaceParams
   ```

1. Obtenez l’URL dev et web de l’espace de travail Azure Synapse :

   ```azurepowershell-interactive
   $WorkspaceWeb = (Get-AzSynapseWorkspace -Name $SynapseWorkspaceName -ResourceGroupName $StorageAccountResourceGroup).ConnectivityEndpoints.Web
   $WorkspaceDev = (Get-AzSynapseWorkspace -Name $SynapseWorkspaceName -ResourceGroupName $StorageAccountResourceGroup).ConnectivityEndpoints.Dev
   ```

1. Créez une règle de pare-feu pour autoriser votre accès à l’espace de travail Azure Synapse depuis votre machine :

   ```azurepowershell-interactive
   $FirewallParams = @{
     WorkspaceName = $SynapseWorkspaceName
     Name = 'Allow Client IP'
     ResourceGroupName = $StorageAccountResourceGroup
     StartIpAddress = $ClientIP
     EndIpAddress = $ClientIP
   }
   New-AzSynapseFirewallRule @FirewallParams
   ```

1. Ouvrez l’adresse URL web de l’espace de travail Azure Synapse, stockée dans la variable d’environnement `WorkspaceWeb`, pour accéder à votre espace de travail :

   ```azurepowershell-interactive
   Start-Process $WorkspaceWeb
   ```

   ![Espace de travail Azure Synapse sur le web](media/quickstart-create-synapse-workspace-powershell/create-workspace-powershell-1.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Effectuez les étapes ci-dessous pour supprimer l’espace de travail Azure Synapse.

> [!WARNING]
> Le fait de supprimer un espace de travail Azure Synapse entraîne la suppression des moteurs d’analytique et des données stockées dans la base de données des pools SQL et des métadonnées de l’espace de travail. Il n’est plus possible de se connecter aux points de terminaison SQL ni aux points de terminaison Apache Spark. Tous les artefacts de code seront supprimés (requêtes, notebooks, définitions de travaux et pipelines). Le fait de supprimer l’espace de travail **n’affecte pas** les données dans le Data Lake Store Gen2 lié à l’espace de travail.

Si vous n’avez pas besoin de l’espace de travail Azure Synapse que vous avez créé dans cet article, vous pouvez le supprimer en exécutant l’exemple suivant.

```azurepowershell-interactive
Remove-AzSynapseWorkspace -Name $SynapseWorkspaceNam -ResourceGroupName $SynapseResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez ensuite [créer des pools SQL](quickstart-create-sql-pool-studio.md) ou [créer des pools Apache Spark](quickstart-create-apache-spark-pool-studio.md) pour commencer à analyser et à explorer vos données.

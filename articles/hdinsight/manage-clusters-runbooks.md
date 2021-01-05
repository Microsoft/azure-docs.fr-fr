---
title: 'Tutoriel : Utiliser des runbooks Azure Automation pour créer des clusters - Azure HDInsight'
description: Découvrez comment créer et supprimer des clusters Azure HDInsight avec des scripts qui s’exécutent dans le cloud à l’aide de runbooks Azure Automation.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 12/27/2019
ms.openlocfilehash: 05c0aaf6cc33442fa4f36eb38eb0d6d593fc6c1f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75553119"
---
# <a name="tutorial-create-azure-hdinsight-clusters-with-azure-automation"></a>Tutoriel : Créer des clusters Azure HDInsight avec Azure Automation

Azure Automation vous permet de créer des scripts qui s’exécutent dans le cloud et gèrent les ressources Azure à la demande ou selon une planification. Cet article explique comment créer des runbooks PowerShell pour créer et supprimer des clusters Azure HDInsight.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Installer les modules nécessaires à l’interaction avec HDInsight.
> * Créer et stocker les informations d’identification nécessaires lors de la création d’un cluster.
> * Créer un runbook Azure Automation pour créer un cluster HDInsight.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Un [compte Azure Automation](../automation/automation-quickstart-create-account.md) existant.
* Un [compte de stockage Azure](../storage/common/storage-account-create.md) existant, qui sera utilisé comme espace de stockage en cluster.

## <a name="install-hdinsight-modules"></a>Installer les modules HDInsight

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez vos comptes Azure Automation.
1. Sélectionnez **Galerie de modules** sous **Ressources partagées**.
1. Tapez **AzureRM.Profile** dans la zone et appuyez sur Entrée pour lancer la recherche. Sélectionnez le résultat de la recherche disponible.
1. Dans l’écran **AzureRM.Profile**, sélectionnez **Importer**. Cochez la case pour mettre à jour les modules Azure, puis sélectionnez **OK**.

    ![Importer le module AzureRM.Profile](./media/manage-clusters-runbooks/import-azurermprofile-module.png)

1. Revenez à la galerie de modules en sélectionnant **Galerie de modules** sous **Ressources partagées**.
1. Tapez **HDInsight**. Sélectionnez **AzureRM.HDInsight**.

    ![Parcourir les modules HDInsight](./media/manage-clusters-runbooks/browse-modules-hdinsight.png)

1. Dans le panneau **AzureRM.HDInsight**, sélectionnez **Importer**, puis **OK**.

    ![Import le module AzureRM.HDInsight](./media/manage-clusters-runbooks/import-azurermhdinsight-module.png)

## <a name="create-credentials"></a>Créer des informations d’identification

1. Sous **Ressources partagées**, sélectionnez **Informations d’identification**.
1. Sélectionnez **Ajouter des informations d’identification**.
1. Entrez les informations nécessaires dans le panneau **Nouvelles informations d’identification**. Ces informations d’identification servent à stocker le mot de passe du cluster, qui vous permet de vous connecter à Ambari.

    | Propriété | Valeur |
    | --- | --- |
    | Nom | `cluster-password` |
    | Nom d'utilisateur | `admin` |
    | Mot de passe | `SECURE_PASSWORD` |
    | Confirmer le mot de passe | `SECURE_PASSWORD` |

1. Sélectionnez **Create** (Créer).
1. Répétez la même procédure pour de nouvelles informations d’identification `ssh-password` avec le nom d’utilisateur `sshuser` et un mot de passe de votre choix. Sélectionnez **Create** (Créer). Ces informations d’identification servent à stocker le mot de passe SSH de votre cluster.

    ![créer des informations d’identification](./media/manage-clusters-runbooks/create-credentials.png)

## <a name="create-a-runbook-to-create-a-cluster"></a>Créer un runbook pour créer un cluster

1. Sélectionnez **Runbooks** sous **Automatisation des processus**.
1. Sélectionnez **Créer un runbook**.
1. Dans le panneau **Créer un runbook**, entrez un nom pour le runbook, comme `hdinsight-cluster-create`. Sélectionnez **PowerShell** dans la liste déroulante **Type de runbook**.
1. Sélectionnez **Create** (Créer).

    ![Créer un runbook](./media/manage-clusters-runbooks/create-runbook.png)

1. Entrez le code suivant dans l’écran **Modifier le runbook PowerShell** et sélectionnez **Publier** :

    ![Publier un runbook](./media/manage-clusters-runbooks/publish-runbook.png)

    ```powershell
    Param
    (
      [Parameter (Mandatory= $true)]
      [String] $subscriptionID,
    
      [Parameter (Mandatory= $true)]
      [String] $resourceGroup,
    
      [Parameter (Mandatory= $true)]
      [String] $storageAccount,
    
      [Parameter (Mandatory= $true)]
      [String] $containerName,
    
      [Parameter (Mandatory= $true)]
      [String] $clusterName
    )
    ### Authenticate to Azure 
    $Conn = Get-AutomationConnection -Name 'AzureRunAsConnection'
    Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
    
    # Set cluster variables
    $storageAccountKey = (Get-AzureRmStorageAccountKey –Name $storageAccount –ResourceGroupName $resourceGroup)[0].value 
    
    # Setting cluster credentials
    
    #Automation credential for Cluster Admin
    $clusterCreds = Get-AutomationPSCredential –Name 'cluster-password'
    
    #Automation credential for user to SSH into cluster
    $sshCreds = Get-AutomationPSCredential –Name 'ssh-password' 
    
    $clusterType = "Hadoop" #Use any supported cluster type (Hadoop, HBase, Storm, etc.)
    $clusterOS = "Linux"
    $clusterWorkerNodes = 3
    $clusterNodeSize = "Standard_D3_v2"
    $location = Get-AzureRmStorageAccount –StorageAccountName $storageAccount –ResourceGroupName $resourceGroup | %{$_.Location}
    
    ### Provision HDInsight cluster
    New-AzureRmHDInsightCluster –ClusterName $clusterName –ResourceGroupName $resourceGroup –Location $location –DefaultStorageAccountName "$storageAccount.blob.core.windows.net" –DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainer $containerName –ClusterType $clusterType –OSType $clusterOS –Version “3.6” –HttpCredential $clusterCreds –SshCredential $sshCreds –ClusterSizeInNodes $clusterWorkerNodes –HeadNodeSize $clusterNodeSize –WorkerNodeSize $clusterNodeSize
    ```

## <a name="create-a-runbook-to-delete-a-cluster"></a>Créer un runbook pour supprimer un cluster

1. Sélectionnez **Runbooks** sous **Automatisation des processus**.
1. Sélectionnez **Créer un runbook**.
1. Dans le panneau **Créer un runbook**, entrez un nom pour le runbook, comme `hdinsight-cluster-delete`. Sélectionnez **PowerShell** dans la liste déroulante **Type de runbook**.
1. Sélectionnez **Create** (Créer).
1. Entrez le code suivant dans l’écran **Modifier le runbook PowerShell** et sélectionnez **Publier** :

    ```powershell
    Param
    (
      [Parameter (Mandatory= $true)]
      [String] $clusterName
    )
    
    ### Authenticate to Azure 
    $Conn = Get-AutomationConnection -Name 'AzureRunAsConnection'
    Add-AzureRMAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
    
    Remove-AzureRmHDInsightCluster -ClusterName $clusterName
    ```

## <a name="execute-runbooks"></a>Exécuter des runbooks

### <a name="create-a-cluster"></a>Créer un cluster

1. Affichez la liste des runbooks pour votre compte Automation en sélectionnant **Runbooks** sous **Automatisation des processus**.
1. Sélectionnez `hdinsight-cluster-create` ou le nom que vous avez utilisé lors de la création de votre runbook de création de cluster.
1. Sélectionnez **Démarrer** pour exécuter immédiatement le runbook. Vous pouvez également planifier une exécution périodique des runbooks. Consultez [Planification d’un Runbook dans Azure Automation](../automation/shared-resources/schedules.md).
1. Entrez les paramètres nécessaires pour le script et sélectionnez **OK**. Ainsi, vous créez un cluster HDInsight portant le nom que vous avez spécifié dans le paramètre **CLUSTERNAME**.

    ![Exécuter l’option de création d’un runbook de cluster](./media/manage-clusters-runbooks/execute-create-runbook.png)

### <a name="delete-a-cluster"></a>Suppression d'un cluster

Supprimez le cluster en sélectionnant le runbook `hdinsight-cluster-delete` que vous avez créé. Sélectionnez **Démarrer**, entrez le paramètre **CLUSTERNAME** et sélectionnez **OK**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le compte Azure Automation créé pour éviter des frais imprévus. Pour cela, accédez au portail Azure, sélectionnez le groupe de ressources dans lequel vous avez créé le compte Azure Automation, sélectionnez le compte Automation, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Gestion des clusters Apache Hadoop dans HDInsight avec Azure PowerShell](hdinsight-administer-use-powershell.md)
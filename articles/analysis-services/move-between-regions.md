---
title: Déplacer Azure Analysis Services vers une autre région | Microsoft Docs
description: Explique comment déplacer une ressource Azure Analysis Services vers une autre région.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: how-to
ms.date: 12/01/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions
ms.openlocfilehash: 049ff6d14c3967481eb73037814082fa261154e3
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96497926"
---
# <a name="move-analysis-services-to-a-different-region"></a>Déplacer Analysis Services vers une autre région

Cet article explique comment déplacer une ressource de serveur Analysis Services vers une autre région Azure. Vous pouvez déplacer votre serveur vers une autre région pour plusieurs raisons, par exemple, pour tirer parti d’une région Azure plus proche des utilisateurs, pour utiliser des plans de service pris en charge uniquement dans des régions spécifiques, ou pour répondre à des exigences de stratégie et de gouvernance internes. 

Dans cet article et les articles connexes associés, vous allez apprendre à :

> [!div class="checklist"]
> 
> * Sauvegarder un modèle de base de données de serveur source dans un [stockage d’objets blob](../storage/blobs/storage-blobs-introduction.md).
> * Exporter un [modèle de ressource](../azure-resource-manager/templates/overview.md) de serveur source.
> * Obtenir un [jeton de signature d’accès partagé (SAP)](../storage/common/storage-sas-overview.md) de stockage.
> * Modifier le modèle de ressource.
> * Déployer le modèle pour créer un nouveau serveur cible.
> * Restaurer un modèle de base de données sur le nouveau serveur cible.
> * Vérifier le nouveau serveur cible et la nouvelle base de données.
> * Supprimer le serveur source.

Cet article décrit l’utilisation d’un modèle de ressource pour migrer un serveur Analysis Services unique avec une **configuration de base** vers une autre région *et* un autre groupe de ressources dans le même abonnement. L’utilisation d’un modèle conserve les propriétés de serveur configurées pour s’assurer que le serveur cible est configuré avec les mêmes propriétés que le serveur source, à l’exception de la région et du groupe de ressources. Cet article ne décrit pas le déplacement des ressources associées qui peuvent faire partie du même groupe de ressources, par exemple les ressources de source de données, de stockage et de passerelle. 

Avant de déplacer un serveur vers une autre région, il est recommandé de créer un plan détaillé. Tenez compte des ressources supplémentaires, notamment les passerelles et le stockage, qui peuvent également être déplacées. Quel que soit le plan, il est important de tester une ou plusieurs opérations de déplacement à l’aide de serveurs de test avant de déplacer un serveur de production.

> [!IMPORTANT]
> Les applications clientes et les chaînes de connexion se connectent à Analysis Services à l’aide du nom complet du serveur, qui représente un URI incluant la région dans laquelle se trouve le serveur. Par exemple : `asazure://westcentralus.asazure.windows.net/advworks01`. Lors du déplacement d’un serveur vers une autre région, vous créez en fait une nouvelle ressource de serveur dans une autre région, qui aura une région différente dans l’URI du nom de serveur. Les applications clientes et les chaînes de connexion utilisées dans les scripts doivent se connecter au nouveau serveur à l’aide de l’URI du nouveau nom de serveur. L’utilisation d’un [alias de nom de serveur](analysis-services-server-alias.md) peut réduire le nombre d’emplacements où l’URI de nom de serveur doit être modifié, mais cet alias doit être implémenté avant le déplacement d’une région.

> [!IMPORTANT]
> Les régions Azure utilisent des plages d’adresses IP différentes. Si vous avez configuré des exceptions de pare-feu pour la région dans laquelle se trouve votre serveur et/ou votre compte de stockage, il peut être nécessaire de configurer une autre plage d’adresses IP. Pour en savoir plus, voir [Forum aux questions sur la connectivité réseau Analysis Services](analysis-services-network-faq.md).

> [!NOTE]
> Cet article décrit la restauration d’une sauvegarde de base de données sur un serveur cible à partir d’un conteneur de stockage dans la région du serveur source. Dans certains cas, la restauration de sauvegardes à partir d’une autre région peut entraîner des performances médiocres, en particulier pour les bases de données volumineuses. Pour des performances optimales lors de la restauration de la base de données, migrez ou créez un nouveau conteneur de stockage dans la région du serveur cible. Copiez les fichiers de sauvegarde .abf du conteneur de stockage de la région source vers le conteneur de stockage de la région cible avant de restaurer la base de données sur le serveur cible. Même si cela dépasse le cadre de cet article, dans certains cas, en particulier avec les bases de données très volumineuses, le script d’une base de données à partir de votre serveur source, la recréation, puis le traitement sur le serveur cible pour charger les données de la base de données peuvent être plus rentables que l’utilisation d’une sauvegarde/restauration.

> [!NOTE]
> Si vous utilisez une passerelle de données locale pour vous connecter à des sources de données, vous devez également déplacer la ressource de passerelle vers la région du serveur cible. Pour en savoir plus, voir [Installer et configurer une passerelle de données locale](analysis-services-gateway-install.md).

## <a name="prerequisites"></a>Prérequis

- **Compte de stockage Azure** : Requis pour stocker un fichier de sauvegarde .abf.
- **SQL Server Management Studio (SSMS)**  : Requis pour sauvegarder et restaurer des modèles de bases de données.
- **Azure PowerShell**. Requis uniquement si vous choisissez d’effectuer cette tâche à l’aide de PowerShell.

## <a name="prepare"></a>Préparation

### <a name="backup-model-databases"></a>Sauvegarder des modèles de bases de données

Si les **paramètres de stockage** ne sont pas déjà configurés pour le serveur source, suivez les étapes décrites dans [Configurer les paramètres de stockage](analysis-services-backup.md#configure-storage-settings).

Si les paramètres de stockage sont configurés, suivez les étapes de la section [Sauvegarde](analysis-services-backup.md#backup) pour créer un modèle de sauvegarde de base de données .abf dans votre conteneur de stockage. Vous restaurez par la suite la sauvegarde .abf sur votre nouveau serveur cible.


### <a name="export-template"></a>Exporter un modèle

Le modèle contient les propriétés de configuration du serveur source.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour exporter un modèle à l’aide du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Sélectionnez **Toutes les ressources**, puis choisissez votre serveur Analysis Services.

3. Sélectionnez **Paramètres** > **Exporter le modèle**.

4. Choisissez **Télécharger** dans le panneau **Exporter le modèle**.

5. Localisez le fichier .zip que vous avez téléchargé à partir du portail, puis décompressez-le dans un dossier.

   Le fichier zip contient les fichiers .json incluant le modèle et les paramètres nécessaires au déploiement d’un nouveau serveur.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour exporter un modèle à l’aide de PowerShell :

1. Connectez-vous à votre abonnement Azure avec la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) et suivez les instructions qui s'affichent à l’écran :

   ```azurepowershell-interactive
   Connect-AzAccount
   ```
2. Si votre identité est associée à plusieurs abonnements, définissez l’abonnement de la ressource de serveur à déplacer comme abonnement actif.

   ```azurepowershell-interactive
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

3. Exportez le modèle de votre serveur source. Ces commandes enregistrent un modèle json en utilisant ResourceGroupName comme nom de fichier dans votre répertoire actif.

   ```azurepowershell-interactive
   $resource = Get-AzResource `
     -ResourceGroupName <resource-group-name> `
     -ResourceName <server-account-name> `
     -ResourceType Microsoft.AnalysisServices/servers
   Export-AzResourceGroup `
     -ResourceGroupName <resource-group-name> `
     -Resource $resource.ResourceId
   ```
---

### <a name="get-storage-shared-access-signature-sas"></a>Obtenir une signature d’accès partagé (SAP) de stockage

Lors du déploiement d’un serveur cible à partir d’un modèle, un jeton SAP de délégation d’utilisateur (en tant qu’URI) est requis pour spécifier le conteneur de stockage incluant la sauvegarde de base de données.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour obtenir une signature d’accès partagé à l’aide du portail :

1. Dans le portail, sélectionnez le compte de stockage utilisé pour sauvegarder la base de données de votre serveur.

2. Sélectionnez **Explorateur Stockage**, puis développez **Conteneurs d’objets blob**. 

3. Cliquez avec le bouton droit sur votre conteneur de stockage, puis sélectionnez **Obtenir une signature d’accès partagé**.

    :::image type="content" source="media/move-between-regions/get-sas.png" alt-text="Obtenir une SAP":::

4. Dans **Signature d’accès partagé (SAP)** , sélectionnez **Créer**. Par défaut, la signature d’accès partagé expire dans 24 heures.

5. Copiez et enregistrez l’**URI**. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour obtenir une signature d’accès partagé à l’aide de PowerShell, suivez les étapes décrites dans [Créer une SAP de délégation d’utilisateur pour un conteneur ou un objet blob avec PowerShell](../storage/blobs/storage-blob-user-delegation-sas-create-powershell.md#create-a-user-delegation-sas-for-a-blob).

---

### <a name="modify-the-template"></a>Modifier le modèle

Utilisez un éditeur de texte pour modifier le fichier template.json que vous avez exporté, en modifiant les propriétés de la région et du conteneur d’objets blob. 

Pour modifier le modèle :

1. Dans un éditeur de texte, dans la propriété **location**, spécifiez la nouvelle région cible. Dans la propriété **backupBlobContainerUri**, collez l’URI du conteneur de stockage avec la clé SAP. 

    L’exemple suivant définit la région cible pour le serveur advworks1 sur `South Central US` et spécifie l’URI du conteneur de stockage avec une signature d’accès partagé : 

    ```json
    "resources": [
        {
            "type": "Microsoft.AnalysisServices/servers",
            "apiVersion": "2017-08-01",
            "name": "[parameters('servers_advworks1_name')]",
            "location": "South Central US",
            "sku": {
                "name": "S1",
                "tier": "Standard",
                "capacity": 1
            },
            "properties": {
                "asAdministrators": {
                    "members": [
                        "asadmins@adventure-works.com"
                    ]
                },
                "backupBlobContainerUri": "https://storagenorthcentralus.blob.core.windows.net/backup?sp=rl&st=2020-06-01T19:30:42Z&se=2020-06-02T19:30:42Z&sv=2019-10-10&sr=c&sig=PCQ4s9RujJkxu89gO4tiDTbE3%2BFECx6zAdcv8x0cVUQ%3D",
                "querypoolConnectionMode": "All"
            }
        }
    ]         
    ```
2. Enregistrez le modèle.

#### <a name="regions"></a>Régions

Pour obtenir les régions Azure, voir [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Pour récupérer des régions à l’aide de PowerShell, exécutez la commande [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

```azurepowershell-interactive
   Get-AzLocation | format-table 
```

## <a name="move"></a>Déplacer

Pour déployer une nouvelle ressource de serveur dans une autre région, vous utiliserez le fichier **template.json** que vous avez exporté et modifié dans les sections précédentes.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le portail, sélectionnez **Créer une ressource**.

2. Dans **Rechercher sur la Place de marché**, tapez **déploiement de modèle**, puis appuyez sur **Entrée**.

3. Sélectionnez **Déploiement de modèle**.

4. Sélectionnez **Create** (Créer).

5. Sélectionnez **Générer votre propre modèle dans l’éditeur**.

6. Sélectionnez **Charger le fichier**, puis suivez les instructions pour charger le fichier **template.json** que vous avez exporté et modifié.

7. Vérifiez que l’éditeur de modèle affiche les propriétés appropriées pour votre nouveau serveur cible.

8. Sélectionnez **Enregistrer**.

9. Entrez ou sélectionnez les valeurs de propriété :

    - **Abonnement**: Sélectionnez l’abonnement Azure.
    
    - **Groupe de ressources** : Sélectionnez **Créer un nouveau**, puis entrez un nom de groupe de ressources. Vous pouvez sélectionner un groupe de ressources existant, à condition qu’il ne contienne pas déjà un serveur Analysis Services portant le même nom.
    
    - **Emplacement** : Sélectionnez la même région que celle que vous avez spécifiée dans le modèle.

10. Sélectionnez **Examiner et créer**.

11. Passez en revue les termes et les principes de base, puis sélectionnez **Créer**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Utilisez ces commandes pour déployer votre modèle :

```azurepowershell-interactive
   $resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
   $location = Read-Host -Prompt "Enter the location (i.e. South Central US)"

   New-AzResourceGroup -Name $resourceGroupName -Location "$location"
   New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "<local template file>"  
   ```
---

### <a name="get-target-server-uri"></a>Get target server Uri

Pour vous connecter au nouveau serveur cible à partir de SSMS afin de restaurer le modèle de base de données, vous devez obtenir l’URI du nouveau serveur cible.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour obtenir l’URI du serveur dans le portail :

1. Dans le portail, accédez à la nouvelle ressource du serveur cible.

2. Dans la page **Vue d’ensemble**, copiez l’URI **Nom du serveur**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour obtenir l’URI du serveur à l’aide de PowerShell, utilisez les commandes suivantes :

```azurepowershell-interactive
   $resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
   $serverName = Read-Host -Prompt "Enter the server name (i.e. advworks2)"

   Get-AzAnalysisServicesServer -ResourceGroupName $resourceGroupName -Name "$serverName"
```
Copiez **ServerFullName** à partir de la sortie.

---

### <a name="restore-model-database"></a>Restaurer le modèle de base de données

Suivez les étapes décrites dans [Restaurer](analysis-services-backup.md#restore) pour restaurer le modèle de base de données .abf sur le nouveau serveur cible.

Facultatif : Après la restauration du modèle de base de données, traitez le modèle et les tables pour actualiser les données à partir des sources de données. Pour traiter le modèle et la table à l’aide de SSMS :

1. Dans SSMS, cliquez avec le bouton droit sur le modèle de base de données > **Traiter la base de données**.

2. Développez **Tables**, puis cliquez avec le bouton droit sur une table. Dans **Traiter la ou les tables**, sélectionnez toutes les tables, puis sélectionnez **OK**.

## <a name="verify"></a>Vérifier

1. Dans le portail, accédez au nouveau serveur cible.

2. Dans la page Vue d’ensemble, dans **Modèles sur le serveur Analysis Services**, vérifiez que les modèles restaurés s’affichent.

3. Utilisez une application cliente comme Power BI ou Excel pour vous connecter au modèle sur le nouveau serveur. Vérifiez que les objets du modèle tels que les tables, les mesures, les hiérarchies apparaissent. 

4. Exécutez les scripts d’automatisation. Vérifiez que les scripts ont été exécutés avec succès.

Facultatif : [ALM Toolkit](http://alm-toolkit.com/) est un outil *open source* permettant de comparer et de gérer des jeux de données Power BI *et* des modèles de bases de données tabulaires Analysis Services. Utilisez la boîte à outils pour vous connecter aux bases de données du serveur source et cible et les comparer. Si la migration de votre base de données réussit, les objets du modèle auront la même définition. 

:::image type="content" source="media/move-between-regions/alm-toolkit.png" alt-text="ALM Toolkit":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir vérifié que les applications clientes peuvent se connecter au nouveau serveur et que tous les scripts d’automatisation s’exécutent correctement, supprimez votre serveur source. 

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour supprimer le serveur source du portail :

Dans la page **Vue d’ensemble** de votre serveur source, sélectionnez **Supprimer**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour supprimer le serveur source à l’aide de PowerShell, utilisez la commande Remove-AzAnalysisServicesServer.

```azurepowershell-interactive
Remove-AzAnalysisServicesServer -Name "myserver" -ResourceGroupName "myResourceGroup"
```

---

> [!NOTE]
> Après avoir déplacé une région, il est recommandé que votre nouveau serveur cible utilise un conteneur de stockage dans la même région pour les sauvegardes, plutôt que le conteneur de stockage de la région du serveur source.
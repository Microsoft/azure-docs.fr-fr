---
title: Déplacer un compte Stockage Azure vers une autre région | Microsoft Docs
description: Montre comment déplacer un compte Stockage Azure vers une autre région.
services: storage
author: normesta
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.date: 05/11/2020
ms.author: normesta
ms.reviewer: dineshm
ms.openlocfilehash: b70beb90fae794eb5512cb8b466524169c4c7b53
ms.sourcegitcommit: ac7029597b54419ca13238f36f48c053a4492cb6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/29/2020
ms.locfileid: "92792988"
---
# <a name="move-an-azure-storage-account-to-another-region"></a>Déplacer un compte Stockage Azure vers une autre région

Pour déplacer un compte de stockage, créez une copie de votre compte de stockage dans une autre région. Utilisez ensuite AzCopy ou un autre outil de votre choix pour déplacer vos données vers ce compte.

Dans cet article, vous allez apprendre à :

> [!div class="checklist"]
> 
> * Exporter un modèle.
> * Modifier le modèle en ajoutant la région cible et le nom du compte de stockage.
> * Déployer le modèle pour créer le compte de stockage.
> * Configurer le nouveau compte de stockage.
> * Déplacer des données vers le nouveau compte de stockage.
> * Supprimer les ressources dans la région source.

## <a name="prerequisites"></a>Prérequis

- Vérifiez que les services et fonctionnalités utilisés par votre compte sont pris en charge dans la région cible.

- Pour les fonctionnalités en préversion, vérifiez que votre abonnement figure dans la liste verte de la région cible.

<a id="prepare"></a>

## <a name="prepare"></a>Préparation

Pour commencer, exportez un modèle Resource Manager et modifiez-le. 

### <a name="export-a-template"></a>Exporter un modèle

Ce modèle contient des paramètres qui décrivent votre compte de stockage. 

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour exporter un modèle à l’aide du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Choisissez **Toutes les ressources**, puis sélectionnez votre compte de stockage.

3. Sélectionnez **Paramètres** > **Exporter le modèle**.

4. Choisissez **Télécharger** dans le panneau **Exporter le modèle**.

5. Localisez le fichier .zip que vous avez téléchargé à partir du portail, puis décompressez-le dans le dossier de votre choix.

   Ce fichier zip contient les fichiers .json qui composent le modèle et des scripts pour le déployer.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour exporter un modèle à l’aide de PowerShell :

1. Connectez-vous à votre abonnement Azure avec la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) et suivez les instructions qui s'affichent à l’écran :

   ```azurepowershell-interactive
   Connect-AzAccount
   ```
2. Si votre identité est associée à plusieurs abonnements, définissez l’abonnement du compte de stockage à déplacer comme abonnement actif.

   ```azurepowershell-interactive
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

3. Exportez le modèle de votre compte de stockage source. Ces commandes enregistrent un modèle json dans votre répertoire actif.

   ```azurepowershell-interactive
   $resource = Get-AzResource `
     -ResourceGroupName <resource-group-name> `
     -ResourceName <storage-account-name> `
     -ResourceType Microsoft.Storage/storageAccounts
   Export-AzResourceGroup `
     -ResourceGroupName <resource-group-name> `
     -Resource $resource.ResourceId
   ```

---

### <a name="modify-the-template"></a>Modifier le modèle 

Modifiez le modèle en changeant le nom et la région du compte de stockage.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour déployer le modèle à l’aide du portail Azure :

1. Dans le portail Azure, sélectionnez **Créer une ressource**.

2. Dans **Rechercher sur la Place de marché**, tapez **déploiement de modèle**, puis appuyez sur **Entrée**.

3. Sélectionnez **Déploiement de modèle**.

    ![Bibliothèque des modèles Azure Resource Manager](./media/storage-account-move/azure-resource-manager-template-library.png)

4. Sélectionnez **Create** (Créer).

5. Sélectionnez **Générer votre propre modèle dans l’éditeur**.

6. Sélectionnez **Charger le fichier**, puis suivez les instructions pour charger le fichier **template.json** que vous avez téléchargé dans la section précédente.

7. Dans le fichier **template.json**, définissez la valeur par défaut du nom du compte de stockage pour nommer le compte de stockage cible. Cet exemple définit `mytargetaccount` comme valeur par défaut du nom du compte de stockage.
    
    ```json
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccounts_mysourceaccount_name": {
            "defaultValue": "mytargetaccount",
            "type": "String"
        }
    },
 
8. Edit the **location** property in the **template.json** file to the target region. This example sets the target region to `centralus`.

    ```json
    "resources": [{
         "type": "Microsoft.Storage/storageAccounts",
         "apiVersion": "2019-04-01",
         "name": "[parameters('storageAccounts_mysourceaccount_name')]",
         "location": "centralus"
         }]          
    ```
    Pour obtenir les codes d’emplacement des régions, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/).  Le code d’une région est le nom de la région sans espace, **USA Centre** = **centralus**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour déployer le modèle à l’aide de PowerShell :

1. Dans le fichier **template.json**, définissez la valeur par défaut du nom du compte de stockage pour nommer le compte de stockage cible. Cet exemple définit `mytargetaccount` comme valeur par défaut du nom du compte de stockage.
    
    ```json
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccounts_mysourceaccount_name": {
            "defaultValue": "mytargetaccount",
            "type": "String"
        }
    },
    ``` 

2. Remplacez la propriété **location** dans le fichier **template.json** par la région cible. Cet exemple définit `eastus` comme région cible.

    ```json
    "resources": [{
         "type": "Microsoft.Storage/storageAccounts",
         "apiVersion": "2019-04-01",
         "name": "[parameters('storageAccounts_mysourceaccount_name')]",
         "location": "eastus"
         }]          
    ```

    Vous pouvez obtenir des codes de région en exécutant la commande [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

    ```azurepowershell-interactive
    Get-AzLocation | format-table 
    ```
---

<a id="move"></a>

## <a name="move"></a>Déplacer

Déployez le modèle pour créer un compte de stockage dans la région cible. 

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Enregistrez le fichier **template.json**.

2. Entrez ou sélectionnez les valeurs de propriété :

- **Abonnement**: Sélectionnez un abonnement Azure.

- **Groupe de ressources** : Sélectionnez **Créer** et donnez un nom au groupe de ressources.

- **Emplacement** : Sélectionnez un emplacement Azure.

3. Cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**, puis cliquez sur le bouton **Sélectionner un achat**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Obtenez l’ID de l’abonnement dans lequel vous souhaitez déployer l’adresse IP publique cible avec [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription) :

   ```azurepowershell-interactive
   Get-AzSubscription
   ```

2. Utilisez ces commandes pour déployer votre modèle :

   ```azurepowershell-interactive
   $resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
   $location = Read-Host -Prompt "Enter the location (i.e. centralus)"

   New-AzResourceGroup -Name $resourceGroupName -Location "$location"
   New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "<name of your local template file>"  
   ```
---

### <a name="configure-the-new-storage-account"></a>Configurer le nouveau compte de stockage

Certaines fonctionnalités n’étant pas exportées vers un modèle, vous devez les ajouter au nouveau compte de stockage. 

Le tableau suivant liste ces fonctionnalités ainsi que des conseils pour les ajouter à votre nouveau compte de stockage.

| Fonctionnalité    | Assistance    |
|--------|-----------|
| **Stratégies de gestion du cycle de vie** | [Gérer le cycle de vie du stockage Blob Azure](../blobs/storage-lifecycle-management-concepts.md) |
| **Sites web statiques** | [Héberger un site web statique dans le stockage Azure](../blobs/storage-blob-static-website-how-to.md) |
| **Abonnements à des événements** | [Réaction aux événements de stockage Blob](../blobs/storage-blob-event-overview.md) |
| **Alertes** | [Créer, afficher et gérer des alertes de journal d’activité à l’aide d’Azure Monitor](../../azure-monitor/platform/alerts-activity-log.md) |
| **Réseau de diffusion de contenu (CDN)** | [Utiliser Azure CDN pour accéder aux objets blob avec des domaines personnalisés sur HTTPS](../blobs/storage-https-custom-domain-cdn.md) |

> [!NOTE] 
> Si vous configurez un CDN pour le compte de stockage source, il vous suffit de remplacer l’origine de votre CDN existant par le point de terminaison de service blob principal (ou le point de terminaison de site web statique principal) de votre nouveau compte. 

### <a name="move-data-to-the-new-storage-account"></a>Déplacer des données vers le nouveau compte de stockage

AzCopy est l’outil recommandé pour déplacer vos données. Elle est optimisée pour les performances.  Cette méthode est plus rapide car les données sont copiées directement entre les serveurs de stockage. AzCopy n’utilise donc pas la bande passante réseau de votre ordinateur. Utilisez AzCopy à la ligne de commande ou dans le cadre d’un script personnalisé. Consultez [Bien démarrer avec AzCopy](/azure/storage/common/storage-use-azcopy-v10?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

Vous pouvez également utiliser Azure Data Factory pour déplacer vos données. Il propose une interface utilisateur intuitive. Pour utiliser Azure Data Factory, consultez l’un des liens suivants : 

  - [Copier des données vers ou depuis le stockage Blob Azure à l’aide d’Azure Data Factory](/azure/data-factory/connector-azure-blob-storage)
  - [Copier des données vers ou depuis Azure Data Lake Storage Gen2 à l’aide d’Azure Data Factory](/azure/data-factory/connector-azure-data-lake-storage)
  - [Copier des données depuis ou vers Stockage Fichier Azure à l’aide d’Azure Data Factory](/azure/data-factory/connector-azure-file-storage)
  - [Copier des données depuis et vers le stockage Table Azure à l’aide d’Azure Data Factory](/azure/data-factory/connector-azure-table-storage)

---

## <a name="discard-or-clean-up"></a>Ignorer ou nettoyer

Après le déploiement, si vous souhaitez recommencer, vous pouvez supprimer le compte de stockage cible et répéter les étapes décrites dans les sections [Préparer](#prepare) et [Déplacer](#move) de cet article.

Pour valider les changements et terminer le déplacement d’un compte de stockage, supprimez le compte de stockage source.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour supprimer un compte de stockage à l’aide du Portail Azure :

1. Dans le portail Azure, développez le menu de gauche pour ouvrir le menu des services, puis choisissez **Comptes de stockage** pour afficher la liste de vos comptes de stockage.

2. Recherchez le compte de stockage cible à supprimer, puis faites un clic droit sur le bouton **Plus** ( **...** ) se trouvant à droite de la liste.

3. Sélectionnez **Supprimer**, puis confirmez.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour supprimer le groupe de ressources et les ressources associées, y compris le nouveau compte de stockage, utilisez la commande [Remove-AzStorageAccount](/powershell/module/az.storage/remove-azstorageaccount) :

```powershell
Remove-AzStorageAccount -ResourceGroupName  $resourceGroup -AccountName $storageAccount
```
---

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un compte de stockage Azure d’une région vers une autre et nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Déplacer des machines virtuelles Azure vers une autre région](../../site-recovery/azure-to-azure-tutorial-migrate.md)
---
title: Supprimer et exporter des données personnelles à partir d’Azure DevTest Labs
description: Découvrez comment supprimer et exporter des données personnelles à partir du service Azure DevLast Labs pour prendre en charge vos obligations dans le cadre du Règlement général sur la protection des données (RGPD).
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 2c44b2f3aa6f2dfad18ed53804842a5dad8bd94a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85483514"
---
# <a name="export-or-delete-personal-data-from-azure-devtest-labs"></a>Exporter ou supprimer des données personnelles à partir d’Azure DevTest Labs
Cet article fournit des étapes pour supprimer et exporter des données personnelles à partir du service Azure DevTest Labs. 

## <a name="what-personal-data-does-devtest-labs-collect"></a>Quelles sont les données personnelles collectées par DevTest Labs ?
DevTest Labs collecte deux types principaux de données personnelles auprès de l’utilisateur. Ce sont : l’adresse e-mail de l’utilisateur et l’ID de l’objet utilisateur. Ces informations sont essentielles pour permettre au service de fournir des fonctionnalités intégrées pour les administrateurs de laboratoire et les utilisateurs de laboratoire.

### <a name="user-email-address"></a>Adresse e-mail de l’utilisateur
DevTest Labs utilise l’adresse e-mail de l’utilisateur pour envoyer des notifications par e-mail d’arrêt automatique aux utilisateurs de laboratoire. L’e-mail notifie les utilisateurs de l’arrêt de leur ordinateur. Les utilisateurs peuvent retarder ou ignorer l’arrêt s’ils le souhaitent. Vous configurez l’adresse e-mail au niveau du laboratoire ou au niveau de la machine virtuelle.

**Définition de l’e-mail au niveau du laboratoire :**

![Définition de l’e-mail au niveau du laboratoire](./media/personal-data-delete-export/lab-user-email.png)

**Définition de l’e-mail au niveau de la machine virtuelle :**

![Définition de l’e-mail au niveau de la machine virtuelle](./media/personal-data-delete-export/vm-user-email.png)

### <a name="user-object-id"></a>ID de l’objet utilisateur
DevTest Labs utilise l’ID de l’objet utilisateur pour afficher les tendances de coût d’un mois à l’autre et des informations sur le coût par ressource pour les administrateurs de laboratoire. Cela leur permet d’effectuer le suivi des coûts et de gérer les seuils pour leur laboratoire. 

**Tendance de coût estimé pour le mois calendaire actuel :** 
![Tendance de coût estimé pour le mois calendaire actuel](./media/personal-data-delete-export/estimated-cost-trend-per-month.png)

**Coût en cumul mensuel à ce jour estimé par ressource :** 
![Coût en cumul mensuel à ce jour estimé par ressource](./media/personal-data-delete-export/estimated-month-to-date-cost-by-resource.png)


## <a name="why-do-we-need-this-personal-data"></a>Pourquoi avons-nous besoin de ces données personnelles ?
Le service DevTest Labs utilise les données personnelles à des fins opérationnelles. Ces données sont essentielle pour permettre au service de fournir des fonctionnalités clés. Si vous définissez une stratégie de rétention sur l’adresse e-mail de l’utilisateur, les utilisateurs de laboratoire ne reçoivent pas les notifications par e-mail d’arrêt automatique en temps voulu après que leur adresse e-mail a été supprimée de notre système. De même, l’administrateur de laboratoire ne peut pas afficher les tendances de coût d’un mois sur l’autre et le coût par ressource pour les machines dans leurs laboratoires si les ID de l’objet utilisateur sont supprimés selon une stratégie de rétention. Par conséquent, ces données doivent être conservées tant que la ressource de l’utilisateur est active dans le laboratoire.

## <a name="how-can-i-have-the-system-to-forget-my-personal-data"></a>Comment faire pour que le système oublie mes données personnelles ?
En tant qu’utilisateur de laboratoire, si vous voulez que ces données personnelles soient supprimées, vous devez supprimer la ressource correspondante dans le laboratoire. Le service DevTest Labs rend anonymes les données personnelles supprimées 30 jours après leur suppression par l’utilisateur.

Par exemple, si vous supprimez votre machine virtuelle, ou votre e-mail, le service DevTest Labs rend ces données anonymes 30 jours après la suppression de la ressource. La stratégie de rétention de 30 jours après la suppression permet de s’assurer que nous fournissons une projection précise du coût d’un mois sur l’autre à l’administrateur de laboratoire.

## <a name="how-can-i-request-an-export-on-my-personal-data"></a>Comment puis-je demander une exportation de mes données personnelles ?
Vous pouvez exporter des données d’utilisation personnelle et de laboratoire à l’aide du Portail Azure ou de PowerShell. Les données sont exportées sous la forme de deux fichiers CSV différents :

- **disks.csv** : contient des informations sur les disques utilisés par les différentes machines virtuelles.
- **virtualmachines.csv** : contient des informations sur les machines virtuelles du laboratoire.

### <a name="azure-portal"></a>Portail Azure
En tant qu’utilisateur de laboratoire, vous pouvez demander une exportation des données personnelles stockées par le service DevTest Labs. Pour demander une exportation, accédez à l’option **Données personnelles** sur la page **Vue d’ensemble** de votre laboratoire. Sélectionnez le bouton **Demander l’exportation** pour lancer la création d’un fichier Excel téléchargeable dans le compte de stockage de l’administrateur de laboratoire. Vous pouvez ensuite contacter votre administrateur de laboratoire pour consulter ces données.

1. Sélectionnez **Données personnelles** dans le menu de gauche. 

    ![Page Données personnelles](./media/personal-data-delete-export/personal-data-page.png)
2. Sélectionnez le **groupe de ressources** qui contient le laboratoire.

    ![Sélection du groupe de ressources](./media/personal-data-delete-export/select-resource-group.png)
3. Sélectionnez le **compte de stockage** dans le groupe de ressources.
4. Sur la page du **compte de stockage**, sélectionnez **Blobs**.

    ![Sélectionner la vignette Blobs](./media/personal-data-delete-export/select-blobs-tile.png)
5. Sélectionnez le conteneur nommé **labresourceusage** dans la liste des conteneurs.

    ![Sélectionner le conteneur blob](./media/personal-data-delete-export/select-blob-container.png)
6. Sélectionnez le **dossier** nommé d’après votre laboratoire. Dans ce dossier, vous trouverez des fichiers **csv** pour les **disques** et les **machines virtuelles** dans votre laboratoire. Vous pouvez télécharger ces fichiers csv, filtrer le contenu pour l’utilisateur de laboratoire demandant l’accès et le partager avec lui.

    ![Télécharger le fichier CSV](./media/personal-data-delete-export/download-csv-file.png)

### <a name="azure-powershell"></a>Azure PowerShell

```powershell
Param (
    [Parameter (Mandatory=$true, HelpMessage="The storage account name where to store usage data")]
    [string] $storageAccountName,

    [Parameter (Mandatory=$true, HelpMessage="The storage account key")]
    [string] $storageKey,

    [Parameter (Mandatory=$true, HelpMessage="The DevTest Lab name to get usage data from")]
    [string] $labName,

    [Parameter (Mandatory=$true, HelpMessage="The DevTest Lab subscription")]
    [string] $labSubscription
    )

#Login
Login-AzureRmAccount

# Set the subscription for the lab
Get-AzureRmSubscription -SubscriptionId $labSubscription  | Select-AzureRmSubscription

# DTL will create this container in the storage when invoking the action, cannot be changed currently
$containerName = "labresourceusage"

# Get the storage context
$Ctx = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageKey 
$SasToken = New-AzureStorageAccountSASToken -Service Blob, File -ResourceType Container, Service, Object -Permission rwdlacup -Protocol HttpsOnly -Context $Ctx

# Generate the storage blob uri
$blobUri = $Ctx.BlobEndPoint + $SasToken

# blobStorageAbsoluteSasUri and usageStartDate are required
$actionParameters = @{
    'blobStorageAbsoluteSasUri' = $blobUri    
}

$startdate = (Get-Date).AddDays(-7)

$actionParameters.Add('usageStartDate', $startdate.Date.ToString())

# Get the lab resource group
$resourceGroupName = (Find-AzureRmResource -ResourceType 'Microsoft.DevTestLab/labs' | Where-Object { $_.Name -eq $labName}).ResourceGroupName
    
# Create the lab resource id
$resourceId = "/subscriptions/" + $labSubscription + "/resourceGroups/" + $resourceGroupName + "/providers/Microsoft.DevTestLab/labs/" + $labName + "/"

# !!!!!!! this is the new resource action to get the usage data.
$result = Invoke-AzureRmResourceAction -Action 'exportLabResourceUsage' -ResourceId $resourceId -Parameters $actionParameters -Force
 
# Finish up cleanly
if ($result.Status -eq "Succeeded") {
    Write-Output "Telemetry successfully downloaded for " $labName
    return 0
}
else
{
    Write-Output "Failed to download lab: " + $labName
    Write-Error $result.toString()
    return -1
}
```

Les principaux composants de l’exemple ci-dessus sont les suivants :

- La commande Invoke-AzureRmResourceAction.
   
    ```
    Invoke-AzureRmResourceAction -Action 'exportLabResourceUsage' -ResourceId $resourceId -Parameters $actionParameters -Force
    ```
- Deux paramètres d’action
    - **blobStorageAbsoluteSasUri** : URI du compte de stockage avec le jeton de signature d’accès partagé (SAS). Dans le script PowerShell, cette valeur peut être utilisée au lieu de la clé de stockage.
    - **usageStartDate** : date de début d’extraction des données, la date de fin étant la date actuelle à laquelle l’action est exécutée. La granularité est au niveau du jour. Par conséquent, même si vous ajoutez des informations d’heure, elles seront ignorées.

### <a name="exported-data---a-closer-look"></a>Données exportées – Examen approfondi
Examinons à présent de plus près les données exportées. Comme mentionné précédemment, une fois que les données ont été exportées avec succès, il y aura deux fichiers CSV. 

Le fichier **virtualmachines.csv** contient les colonnes de données suivantes :

| Nom de la colonne | Description |
| ----------- | ----------- | 
| SubscriptionId | L’identificateur d’abonnement dans lequel se trouve le laboratoire. |
| LabUId | Identificateur GUID unique pour le laboratoire. |
| LabName | Nom du lab. |
| LabResourceId | ID de ressource lab complet. |
| ResourceGroupName | Nom du groupe de ressources qui contient la machine virtuelle | 
| ResourceId | ID de ressource complet pour la machine virtuelle. |
| ResourceUId | GUID pour la machine virtuelle |
| Nom | Nom de la machine virtuelle. |
| CreatedTime | Date-heure à laquelle la machine virtuelle a été créée. |
| DeletedDate | Date-heure à laquelle la machine virtuelle a été supprimée. Si la valeur est vide, la suppression n’a pas encore eu lieu. |
| ResourceOwner | Propriétaire de la machine virtuelle. Si la valeur est vide, il s’agit d’une machine virtuelle qui peut être revendiquée ou créée par un principal de service. |
| PricingTier | Niveau tarifaire de la machine virtuelle |
| ResourceStatus | État de disponibilité de la machine virtuelle. Actif, s’il existe toujours ou est inactif, si la machine virtuelle a été supprimée. |
| ComputeResourceId | Identificateur complet de la ressource de calcul de l’ordinateur virtuel. |
| Revendicable | Défini sur true si la machine virtuelle est une machine virtuelle revendiquable | 
| EnvironmentId | Identificateur de ressource d’environnement dans lequel l’ordinateur virtuel a été créé. Cette valeur est vide lorsque la machine virtuelle n’a pas été créée dans le cadre d’une ressource d’environnement. |
| ExpirationDate | Date d’expiration de la machine virtuelle. La valeur est vide si une date d’expiration n’a pas été définie.
| GalleryImageReferenceVersion |  Version de l’image de base de la machine virtuelle. |
| GalleryImageReferenceOffer | Offre de l’image de base de la machine virtuelle. |
| GalleryImageReferencePublisher | Éditeur de l’image de base de machine virtuelle. |
| GalleryImageReferenceSku | Référence (SKU) de l’image de base de la machine virtuelle |
| GalleryImageReferenceOsType | Type d’OS de l’image de base de la machine virtuelle |
| CustomImageId | ID complet de l’image personnalisée de base de la machine virtuelle. |

Les colonnes de données contenues dans **Disks. csv** sont répertoriées ci-dessous :

| Nom de la colonne | Description | 
| ----------- | ----------- | 
| SubscriptionId | ID de l’abonnement contenant le laboratoire |
| LabUId | GUID pour le laboratoire |
| LabName | Nom du laboratoire | 
| LabResourceId | ID de ressource complet pour le laboratoire | 
| ResourceGroupName | Nom du groupe de ressources qui contient le laboratoire | 
| ResourceId | ID de ressource complet pour la machine virtuelle. |
| ResourceUId | GUID pour la machine virtuelle |
 |Nom | Nom du disque attaché |
| CreatedTime |La date et l'heure de création du disque de données. |
| DeletedDate | La date et l’heure de suppression du disque de données. |
| ResourceStatus | État de la ressource. Actif, si la ressource existe. Inactif, en cas de suppression. |
| DiskBlobName | Nom de l’objet blob du disque de données. |
| DiskSizeGB | La taille du disque de données. |
| DiskType | Le type du disque de données. 0 pour Standard, 1 pour Premium. |
| LeasedByVmId | ID de ressource de la machine virtuelle à laquelle le disque de données a été attaché. |


> [!NOTE]
> Si vous travaillez avec plusieurs laboratoires et souhaitez obtenir des informations générales, les deux colonnes clés sont **LabUID** et **ResourceUId**, qui sont les ID uniques des abonnements.

Les données exportées peuvent être manipulées et visualisées à l’aide d’outils, comme SQL Server, Power BI, etc. Cette fonctionnalité est particulièrement utile lorsque vous souhaitez signaler l’utilisation de votre laboratoire à votre équipe de gestion qui n’utilise peut-être pas le même abonnement Azure que vous.

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants : 

- [Définir des stratégies pour un laboratoire](devtest-lab-set-lab-policy.md)
- [Forum Aux Questions (FAQ)](devtest-lab-faq.md)

---
title: Azure Data Lake Storage Gen2 PowerShell pour les fichiers et les listes de contrôle d’accès
description: Utilisez les applets de commande PowerShell pour gérer les répertoires et les listes de contrôle d’accès de fichiers et de répertoires dans des comptes de stockage dotés d’un espace de noms hiérarchique (HNS) activé.
services: storage
author: normesta
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.topic: how-to
ms.date: 08/26/2020
ms.author: normesta
ms.reviewer: prishet
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: eaa34da7dbdf8d315cf60d84cf15ef428f4c7900
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95913349"
---
# <a name="use-powershell-to-manage-directories-files-and-acls-in-azure-data-lake-storage-gen2"></a>Utiliser PowerShell pour gérer les répertoires, les fichiers et les listes de contrôle d’accès dans Azure Data Lake Storage Gen2

Cet article explique comment utiliser PowerShell pour créer et gérer des répertoires, des fichiers et des autorisations dans des comptes de stockage dotés d’un espace de noms hiérarchique (HNS) activé. 

[Référence](/powershell/module/Az.Storage/) | [Mappage Gen1 à Gen2](#gen1-gen2-map) | [Envoyer des commentaires](https://github.com/Azure/azure-powershell/issues)

## <a name="prerequisites"></a>Prérequis

> [!div class="checklist"]
> * Un abonnement Azure. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
> * Un compte de stockage doté d’un espace de noms hiérarchique (HNS) activé. Pour créer un test, suivez [ces](../common/storage-account-create.md) instructions.
> * .NET Framework version 4.7.2 ou ultérieure installé. Consultez [Télécharger .NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).
> * PowerShell version `5.1` ou ultérieure.

## <a name="install-the-powershell-module"></a>Installer le module PowerShell

1. Vérifiez que la version de PowerShell installée est `5.1` ou une version ultérieure à l’aide de la commande suivante.    

   ```powershell
   echo $PSVersionTable.PSVersion.ToString() 
   ```
    
   Pour mettre à niveau votre version de PowerShell, consultez [Mise à niveau des instances Windows PowerShell existantes](/powershell/scripting/install/installing-windows-powershell#upgrading-existing-windows-powershell).
    
2. Installez le module **Az.Storage**.

   ```powershell
   Install-Module Az.Storage -Repository PSGallery -Force  
   ```

   Pour plus d’informations sur l’installation des modules PowerShell, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="connect-to-the-account"></a>Se connecter au compte

Ouvrez une fenêtre de commande Windows PowerShell, puis connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount` et suivez les instructions à l’écran.

```powershell
Connect-AzAccount
```

Si votre identité est associée à plusieurs abonnements, définissez comme abonnement actif l’abonnement du compte de stockage dans lequel vous souhaitez créer et gérer des répertoires. Dans cet exemple, remplacez la valeur d’espace réservé `<subscription-id>` par l’ID de votre abonnement.

```powershell
Select-AzSubscription -SubscriptionId <subscription-id>
```

Ensuite, choisissez la façon dont vous souhaitez que vos commandes obtiennent l’autorisation pour le compte de stockage. 

### <a name="option-1-obtain-authorization-by-using-azure-active-directory-ad"></a>Option 1 : Obtenir l’autorisation à l’aide d’Azure Active Directory (AD)

Avec cette approche, le système garantit que votre compte d’utilisateur dispose des autorisations de contrôle d’accès en fonction du rôle Azure (Azure RBAC) appropriées et des autorisations de liste de contrôle d’accès (ACL). 

```powershell
$ctx = New-AzStorageContext -StorageAccountName '<storage-account-name>' -UseConnectedAccount
```

### <a name="option-2-obtain-authorization-by-using-the-storage-account-key"></a>Option n°2 : Obtenir l’autorisation à l’aide de la clé de compte de stockage

Avec cette approche, le système ne vérifie pas les autorisations Azure RBAC ou ACL.

```powershell
$storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
$ctx = $storageAccount.Context
```

## <a name="create-a-container"></a>Créez un conteneur.

Un conteneur agit comme un système de fichiers pour vos fichiers. Vous pouvez en créer un à l’aide de l’applet de commande `New-AzStorageContainer`. 

Cet exemple permet de créer un conteneur nommé `my-file-system`.

```powershell
$filesystemName = "my-file-system"
New-AzStorageContainer -Context $ctx -Name $filesystemName
```

## <a name="create-a-directory"></a>Créer un répertoire

Créez une référence de répertoire à l’aide de l’applet de commande `New-AzDataLakeGen2Item`. 

Cet exemple ajoute un répertoire nommé `my-directory` à un conteneur.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Directory
```

Cet exemple ajoute le même répertoire, mais définit également les autorisations, le masque umask, les valeurs de propriété et les valeurs de métadonnées. 

```powershell
$dir = New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Directory -Permission rwxrwxrwx -Umask ---rwx---  -Property @{"ContentEncoding" = "UDF8"; "CacheControl" = "READ"} -Metadata  @{"tag1" = "value1"; "tag2" = "value2" }
```

## <a name="show-directory-properties"></a>Afficher les propriétés du répertoire

Cet exemple obtient un répertoire à l’aide de l’applet de commande `Get-AzDataLakeGen2Item`, puis affiche les valeurs des propriétés dans la console.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dir =  Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname
$dir.ACL
$dir.Permissions
$dir.Group
$dir.Owner
$dir.Properties
$dir.Properties.Metadata
```
> [!NOTE]
> Pour obtenir le répertoire racine du conteneur, omettez le paramètre `-Path`.

## <a name="rename-or-move-a-directory"></a>Renommer ou déplacer un répertoire

Renommez ou déplacez un répertoire à l’aide de l’applet de commande `Move-AzDataLakeGen2Item`.

Cet exemple renomme un répertoire `my-directory` en `my-new-directory`.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dirname2 = "my-new-directory/"
Move-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -DestFileSystem $filesystemName -DestPath $dirname2
```

> [!NOTE]
> Utilisez le paramètre `-Force` si vous souhaitez remplacer sans invite.

Cet exemple déplace un répertoire nommé `my-directory` vers un sous-répertoire de `my-directory-2`, nommé `my-subdirectory`. 

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dirname2 = "my-directory-2/my-subdirectory/"
Move-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname1 -DestFileSystem $filesystemName -DestPath $dirname2
```

## <a name="delete-a-directory"></a>Supprimer un répertoire

Supprimez un répertoire à l’aide de l’applet de commande `Remove-AzDataLakeGen2Item`.

Cet exemple supprime un répertoire nommé `my-directory`. 

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
Remove-AzDataLakeGen2Item  -Context $ctx -FileSystem $filesystemName -Path $dirname 
```

Vous pouvez utiliser le paramètre `-Force` pour supprimer le fichier sans invite.

## <a name="download-from-a-directory"></a>Télécharger à partir d’un répertoire

Téléchargez un fichier à partir d’un répertoire à l’aide de l’applet de commande `Get-AzDataLakeGen2ItemContent`.

Cet exemple télécharge un fichier nommé `upload.txt` à partir d’un répertoire nommé `my-directory`. 

```powershell
$filesystemName = "my-file-system"
$filePath = "my-directory/upload.txt"
$downloadFilePath = "download.txt"
Get-AzDataLakeGen2ItemContent -Context $ctx -FileSystem $filesystemName -Path $filePath -Destination $downloadFilePath
```

## <a name="list-directory-contents"></a>Afficher le contenu du répertoire

Listez le contenu d’un répertoire à l’aide de l’applet de commande `Get-AzDataLakeGen2ChildItem`. Vous pouvez utiliser le paramètre facultatif `-OutputUserPrincipalName` pour récupérer le nom (au lieu de l’ID d’objet) des utilisateurs.

Cet exemple liste le contenu d’un répertoire nommé `my-directory`.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
Get-AzDataLakeGen2ChildItem -Context $ctx -FileSystem $filesystemName -Path $dirname -OutputUserPrincipalName
```

L’exemple suivant répertorie les propriétés `ACL`, `Permissions`, `Group`et `Owner` de chaque élément figurant dans le répertoire. Le paramètre `-FetchProperty` est requis pour obtenir les valeurs de la propriété `ACL`. 

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$properties = Get-AzDataLakeGen2ChildItem -Context $ctx -FileSystem $filesystemName -Path $dirname -Recurse -FetchProperty
$properties.ACL
$properties.Permissions
$properties.Group
$properties.Owner
```

> [!NOTE]
> Pour répertorier les contenus du répertoire racine du conteneur, omettez le paramètre `-Path`.

## <a name="upload-a-file-to-a-directory"></a>Charger un fichier dans un répertoire

Chargez un fichier dans un répertoire à l’aide de l’applet de commande `New-AzDataLakeGen2Item`.

Cet exemple charge un fichier nommé `upload.txt` dans un répertoire nommé `my-directory`. 

```powershell
$localSrcFile =  "upload.txt"
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$destPath = $dirname + (Get-Item $localSrcFile).Name
New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $destPath -Source $localSrcFile -Force 
```

Cet exemple charge le même fichier, mais définit ensuite les autorisations, le masque umask, les valeurs de propriété et les valeurs de métadonnées du fichier de destination. Cet exemple affiche également ces valeurs dans la console.

```powershell
$file = New-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $destPath -Source $localSrcFile -Permission rwxrwxrwx -Umask ---rwx--- -Property @{"ContentEncoding" = "UDF8"; "CacheControl" = "READ"} -Metadata  @{"tag1" = "value1"; "tag2" = "value2" }
$file1
$file1.Properties
$file1.Properties.Metadata

```

> [!NOTE]
> Pour charger un fichier dans le répertoire racine du conteneur, omettez le paramètre `-Path`.

## <a name="show-file-properties"></a>Afficher les propriétés d’un fichier

Cet exemple obtient un fichier à l’aide de l’applet de commande `Get-AzDataLakeGen2Item`, puis affiche les valeurs des propriétés dans la console.

```powershell
$filepath =  "my-directory/upload.txt"
$filesystemName = "my-file-system"
$file = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filepath
$file
$file.ACL
$file.Permissions
$file.Group
$file.Owner
$file.Properties
$file.Properties.Metadata
```

## <a name="delete-a-file"></a>Supprimer un fichier

Supprimez un fichier à l’aide de l’applet de commande `Remove-AzDataLakeGen2Item`.

Cet exemple supprime un fichier nommé `upload.txt`. 

```powershell
$filesystemName = "my-file-system"
$filepath = "upload.txt"
Remove-AzDataLakeGen2Item  -Context $ctx -FileSystem $filesystemName -Path $filepath 
```

Vous pouvez utiliser le paramètre `-Force` pour supprimer le fichier sans invite.

## <a name="manage-access-control-lists-acls"></a>Gérer les listes de contrôle d’accès (ACL, access control list)

Vous pouvez obtenir, définir et mettre à jour les autorisations d’accès des répertoires et des fichiers.

> [!NOTE]
> Si vous utilisez Azure Active Directory (Azure AD) pour autoriser des commandes, assurez-vous que le [rôle Propriétaire des données blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) est attribué à votre principal de sécurité. Pour en savoir plus sur l’application des autorisations ACL et les conséquences de leur modification, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen2](./data-lake-storage-access-control.md).

### <a name="get-an-acl"></a>Obtenir une liste de contrôle d’accès

Obtenez la liste ACL d’un répertoire ou d’un fichier à l’aide de l’applet de commande `Get-AzDataLakeGen2Item`.

Cet exemple obtient la liste ACL du répertoire racine d’un **conteneur**, puis l’affiche sur la console.

```powershell
$filesystemName = "my-file-system"
$filesystem = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName
$filesystem.ACL
```

Cet exemple obtient la liste ACL d’un **répertoire**, puis affiche cette liste ACL dans la console.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dir = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname
$dir.ACL
```

Cet exemple obtient la liste ACL d’un **fichier**, puis affiche cette liste ACL dans la console.

```powershell
$filePath = "my-directory/upload.txt"
$file = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filePath
$file.ACL
```

L’image suivante montre la sortie après l’obtention de la liste ACL d’un répertoire.

![Obtenir une sortie de liste ACL pour un répertoire](./media/data-lake-storage-directory-file-acl-powershell/get-acl.png)

Dans cet exemple, l’utilisateur propriétaire dispose d’autorisations de lecture, d’écriture et d’exécution. Le groupe propriétaire dispose uniquement d’autorisations de lecture et d’exécution. Pour plus d’informations sur les listes de contrôle d’accès, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

### <a name="set-an-acl"></a>Définir une liste de contrôle d’accès

Utilisez l’applet de commande `set-AzDataLakeGen2ItemAclObject` pour créer une liste ACL pour l’utilisateur propriétaire, le groupe propriétaire ou d’autres utilisateurs. Ensuite, utilisez l’applet de commande `Update-AzDataLakeGen2Item` pour valider la liste ACL.

Cet exemple définit la liste ACL du répertoire racine d’un **conteneur** pour l’utilisateur propriétaire, le groupe propriétaire ou d’autres utilisateurs, puis affiche cette liste sur la console.

```powershell
$filesystemName = "my-file-system"
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rw- 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission rw- -InputObject $acl 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission -wx -InputObject $acl
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Acl $acl
$filesystem = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName
$filesystem.ACL
```

Cet exemple définit la liste ACL d’un **répertoire** pour l’utilisateur propriétaire, le groupe propriétaire ou d’autres utilisateurs, puis affiche la liste ACL dans la console.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rw- 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission rw- -InputObject $acl 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission -wx -InputObject $acl
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl
$dir = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname
$dir.ACL
```
Cet exemple définit la liste ACL d’un **fichier** pour l’utilisateur propriétaire, le groupe propriétaire ou d’autres utilisateurs, puis affiche la liste ACL dans la console.

```powershell
$filesystemName = "my-file-system"
$filePath = "my-directory/upload.txt"
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rw- 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission rw- -InputObject $acl 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission "-wx" -InputObject $acl
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filePath -Acl $acl
$file = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filePath
$file.ACL
```

L’image suivante montre la sortie après la définition de la liste ACL d’un fichier.

![Obtenir une sortie de liste ACL pour un fichier](./media/data-lake-storage-directory-file-acl-powershell/set-acl.png)

Dans cet exemple, l’utilisateur propriétaire et le groupe propriétaire disposent uniquement des autorisations de lecture et d’écriture. Tous les autres utilisateurs disposent des autorisations d’écriture et d’exécution. Pour plus d’informations sur les listes de contrôle d’accès, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

### <a name="add-or-update-an-acl-entry"></a>Ajouter ou mettre à jour une entrée de liste de contrôle d’accès

Tout d’abord, obtenez la liste de contrôle d’accès. Ensuite, utilisez l’applet de commande `set-AzDataLakeGen2ItemAclObject` pour ajouter ou mettre à jour une entrée de liste de contrôle d’accès. Utilisez l’applet de commande `Update-AzDataLakeGen2Item` pour valider la liste de contrôle d’accès.

Cet exemple crée ou met à jour la liste de contrôle d’accès sur un **répertoire** pour un utilisateur.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$acl = (Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname).ACL
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityID xxxxxxxx-xxxx-xxxxxxxxxxx -Permission r-x -InputObject $acl 
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl
```

### <a name="remove-an-acl-entry"></a>Supprimer une entrée de liste de contrôle d’accès

Cet exemple supprime une entrée d’une liste de contrôle d’accès existante.

```powershell
$id = "xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# Create the new ACL object.
[Collections.Generic.List[System.Object]]$aclnew =$acl

foreach ($a in $aclnew)
{
    if ($a.AccessControlType -eq "User"-and $a.DefaultScope -eq $false -and $a.EntityId -eq $id)
    {
        $aclnew.Remove($a);
        break;
    }
}
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $aclnew
```

### <a name="set-an-acl-recursively"></a>Définir une liste de contrôle d’accès (ACL) de manière récursive

Vous pouvez ajouter, mettre à jour et supprimer des listes ACL de manière récursive au niveau des éléments enfants existants d’un répertoire parent sans avoir à apporter ces modifications individuellement à chaque élément enfant. Pour plus d’informations, consultez [Définir des listes de contrôle d’accès (ACL) de manière récursive pour Azure Data Lake Storage Gen2](recursive-access-control-lists.md).

<a id="gen1-gen2-map"></a>

## <a name="gen1-to-gen2-mapping"></a>Mappage de Gen1 à Gen2

Le tableau suivant montre comment les applets de commande utilisées pour Data Lake Storage Gen1 sont mappées aux applets de commande pour Data Lake Storage Gen2.

|Applet de commande Data Lake Storage Gen1| Applet de commande Data Lake Storage Gen2| Notes |
|--------|---------|-----|
|Get-AzDataLakeStoreChildItem|Get-AzDataLakeGen2ChildItem|Par défaut, la cmdlet Get-AzDataLakeGen2ChildItem répertorie uniquement les éléments enfants de premier niveau. Le paramètre -Recurse répertorie les éléments enfants de manière récursive. |
|Get-AzDataLakeStoreItem<br>Get-AzDataLakeStoreItemAclEntry<br>Get-AzDataLakeStoreItemOwner<br>Get-AzDataLakeStoreItemPermission|Get-AzDataLakeGen2Item|Les éléments de sortie de la cmdlet Get-AzDataLakeGen2Item ont les propriétés suivantes : Acl, Owner, Group, Permission.|
|Get-AzDataLakeStoreItemContent|Get-AzDataLakeGen2FileContent|La cmdlet Get-AzDataLakeGen2FileContent télécharge le contenu du fichier dans un fichier local.|
|Move-AzDataLakeStoreItem|Move-AzDataLakeGen2Item||
|New-AzDataLakeStoreItem|New-AzDataLakeGen2Item|Cette cmdlet télécharge le contenu du nouveau fichier à partir d’un fichier local.|
|Remove-AzDataLakeStoreItem|Remove-AzDataLakeGen2Item||
|Set-AzDataLakeStoreItemOwner<br>Set-AzDataLakeStoreItemPermission<br>Set-AzDataLakeStoreItemAcl|Update-AzDataLakeGen2Item|La cmdlet Update-AzDataLakeGen2Item met à jour un seul élément, et non de manière récursive. Si vous souhaitez mettre à jour de manière récursive, répertoriez les éléments à l’aide de la cmdlet Get-AzDataLakeStoreChildItem, puis acheminez-les vers la cmdlet Update-AzDataLakeGen2Item.|
|Test-AzDataLakeStoreItem|Get-AzDataLakeGen2Item|La cmdlet Get-AzDataLakeGen2Item signale une erreur si l’élément n’existe pas.|

## <a name="see-also"></a>Voir aussi

* [Problèmes connus](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
* [Applets de commande PowerShell - Stockage](/powershell/module/az.storage)
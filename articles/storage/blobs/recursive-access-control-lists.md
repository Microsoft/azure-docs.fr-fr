---
title: Définir des listes ACL de manière récursive pour Azure Data Lake Storage Gen2 | Microsoft Docs
description: Vous pouvez ajouter, mettre à jour et supprimer des listes de contrôle d’accès (ACL) de manière récursive pour les éléments enfants existants d’un répertoire parent.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 11/17/2020
ms.author: normesta
ms.reviewer: prishet
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 2544711054678ec1bb5c43d40e4497eec9af2941
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96600334"
---
# <a name="set-access-control-lists-acls-recursively-for-azure-data-lake-storage-gen2"></a>Définir des listes de contrôle d’accès (ACL) pour Azure Data Lake Storage Gen2

L’héritage des listes ACL est déjà disponible pour les nouveaux éléments enfants créés sous un répertoire parent. Maintenant, vous pouvez également ajouter, mettre à jour et supprimer des listes ACL de manière récursive pour les éléments enfants existants d’un répertoire parent sans avoir à apporter ces modifications individuellement à chaque élément enfant.

[Bibliothèques](#libraries) | [Exemples](#code-samples) | [Meilleures pratiques](#best-practice-guidelines)

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

- Un compte de stockage doté d’un espace de noms hiérarchique (HNS) activé. Pour créer un test, suivez [ces](create-data-lake-storage-account.md) instructions.

- Autorisations appropriées pour exécuter le processus ACL récursif. L’autorisation correcte comprend l’un des éléments suivants : 

  - Un [principal de sécurité](../../role-based-access-control/overview.md#security-principal) Azure Active Directory (AD) provisionné qui a reçu le rôle [Propriétaire des données Blob de stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) dans l’étendue du conteneur cible, du groupe de ressources parent ou de l’abonnement.   

  - Utilisateur propriétaire du conteneur ou du répertoire cible auquel vous envisagez d’appliquer le processus ACL récursif. Cela comprend tous les éléments enfants du conteneur ou du répertoire cible. 

- Comprendre comment les listes ACL sont appliquées aux répertoires et aux fichiers. Consultez [Contrôle d’accès dans Azure Data Lake Storage Gen2](./data-lake-storage-access-control.md). 

Pour afficher les instructions d’installation pour PowerShell, le SDK .NET et le SDK Python, consultez la section **Configurer votre projet** de cet article.

## <a name="set-up-your-project"></a>Configuration de votre projet

Installez les bibliothèques nécessaires.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Vérifiez que la version de PowerShell installée est `5.1` ou une version ultérieure à l’aide de la commande suivante.    

   ```powershell
   echo $PSVersionTable.PSVersion.ToString() 
   ```
    
   Pour mettre à niveau votre version de PowerShell, consultez [Mise à niveau des instances Windows PowerShell existantes](/powershell/scripting/install/installing-windows-powershell).
    
2. Installez le module **Az.Storage**.

   ```powershell
   Install-Module Az.Storage -Repository PSGallery -Force  
   ```

   Pour plus d’informations sur l’installation des modules PowerShell, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Ouvrez [Azure Cloud Shell](../../cloud-shell/overview.md) ou, si vous avez [installé](/cli/azure/install-azure-cli) Azure CLI localement, ouvrez une application console de commandes telle que Windows PowerShell.

2. Vérifiez que la version `2.14.0` d’Azure CLI ou une version supérieure est installée à l’aide de la commande suivante.

   ```azurecli
    az --version
   ```
   Si la version d’Azure CLI est inférieure à `2.14.0`, installez une version plus récente. Voir [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

### <a name="net"></a>[.NET](#tab/dotnet)

1. Ouvrez une fenêtre de commande (par exemple : Windows PowerShell).

2. À partir de votre répertoire de projet, installez le package de préversion Azure.Storage.Files.DataLake à l’aide de la commande `dotnet add package`.

   ```console
   dotnet add package Azure.Storage.Files.DataLake -v 12.5.0-preview.1 -s https://pkgs.dev.azure.com/azure-sdk/public/_packaging/azure-sdk-for-net/nuget/v3/index.json
   ```

3. Ajoutez ces instructions using au début de votre fichier de code.

   ```csharp
   using Azure;
   using Azure.Core;
   using Azure.Storage;
   using Azure.Storage.Files.DataLake;
   using Azure.Storage.Files.DataLake.Models;
   using System.Collections.Generic;
   using System.Threading.Tasks;
    ```

### <a name="java"></a>[Java](#tab/java)

Pour commencer, ouvrez [cette page](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake) et recherchez la dernière version de la bibliothèque Java. Ensuite, ouvrez le fichier *pom.xml* dans votre éditeur de texte. Ajoutez un élément de dépendance qui référence cette version.

Si vous envisagez d’authentifier votre application cliente avec Azure Active Directory (AD), ajoutez une dépendance à la bibliothèque de client de secrets Azure. Consultez [Adding the Secret Client Library package to your project](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity#adding-the-package-to-your-project).

Ajoutez ensuite ces instructions Imports à votre fichier de code.

```java
import com.azure.identity.ClientSecretCredential;
import com.azure.identity.ClientSecretCredentialBuilder;
import com.azure.core.http.rest.PagedIterable;
import com.azure.core.http.rest.Response;
import com.azure.storage.common.StorageSharedKeyCredential;
import com.azure.storage.file.datalake.DataLakeDirectoryClient;
import com.azure.storage.file.datalake.DataLakeFileClient;
import com.azure.storage.file.datalake.DataLakeFileSystemClient;
import com.azure.storage.file.datalake.DataLakeServiceClient;
import com.azure.storage.file.datalake.DataLakeServiceClientBuilder;
import com.azure.storage.file.datalake.models.AccessControlChangeResult;
import com.azure.storage.file.datalake.models.AccessControlType;
import com.azure.storage.file.datalake.models.ListPathsOptions;
import com.azure.storage.file.datalake.models.PathAccessControl;
import com.azure.storage.file.datalake.models.PathAccessControlEntry;
import com.azure.storage.file.datalake.models.PathItem;
import com.azure.storage.file.datalake.models.PathPermissions;
import com.azure.storage.file.datalake.models.PathRemoveAccessControlEntry;
import com.azure.storage.file.datalake.models.RolePermissions;
import com.azure.storage.file.datalake.options.PathSetAccessControlRecursiveOptions;
```

### <a name="python"></a>[Python](#tab/python)

1. Téléchargez la [bibliothèque cliente Azure Data Lake Storage pour Python](https://recursiveaclpr.blob.core.windows.net/privatedrop/azure_storage_file_datalake-12.1.0b99-py2.py3-none-any.whl?sv=2019-02-02&st=2020-08-24T07%3A47%3A01Z&se=2021-08-25T07%3A47%3A00Z&sr=b&sp=r&sig=H1XYw4FTLJse%2BYQ%2BfamVL21UPVIKRnnh2mfudA%2BfI0I%3D).

2. Installez la bibliothèque que vous avez téléchargée à l’aide de [pip](https://pypi.org/project/pip/).

   ```
   pip install azure_storage_file_datalake-12.1.0b99-py2.py3-none-any.whl
   ```

3. Ajoutez les instructions d’importation suivantes au début de votre fichier de code.

   ```python
   import os, uuid, sys
   from azure.storage.filedatalake import DataLakeServiceClient
   from azure.core._match_conditions import MatchConditions
   from azure.storage.filedatalake._models import ContentSettings
   ```

---

## <a name="connect-to-your-account"></a>Se connecter à votre compte

Vous pouvez vous connecter à l’aide d’Azure Active Directory (AD) ou à l’aide d’une clé de compte. 

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

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

Le tableau suivant présente chacun des rôles pris en charge et leurs capacités en termes de paramétrage ACL.

|Role|Capacité de paramétrage ACL|
|--|--|
|[Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)|Tous les répertoires et fichiers du compte.|
|[Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)|Seuls les répertoires et les fichiers appartenant au principal de sécurité.|

### <a name="option-2-obtain-authorization-by-using-the-storage-account-key"></a>Option n°2 : Obtenir l’autorisation à l’aide de la clé de compte de stockage

Avec cette approche, le système ne vérifie pas les autorisations Azure RBAC ou ACL.

```powershell
$storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
$ctx = $storageAccount.Context
```

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Si vous utilisez Azure CLI localement, exécutez la commande login.

   ```azurecli
   az login
   ```

   Si l’interface CLI peut ouvrir votre navigateur par défaut, elle le fait et charge une page de connexion Azure par la même occasion.

   Sinon, ouvrez une page de navigateur à l’adresse [https://aka.ms/devicelogin](https://aka.ms/devicelogin) et entrez le code d’autorisation affiché dans votre terminal. Connectez-vous ensuite avec les informations d’identification de votre compte dans le navigateur.

   Pour en savoir plus sur les différentes méthodes d’authentification, consultez [Autoriser l’accès à des données d’objet blob ou de file d’attente avec Azure CLI](./authorize-data-operations-cli.md).

2. Si votre identité est associée à plusieurs abonnements, définissez comme abonnement actif l’abonnement du compte de stockage qui doit héberger votre site web statique.

   ```azurecli
   az account set --subscription <subscription-id>
   ```

   Remplacez la valeur d’espace réservé `<subscription-id>` par l’ID de votre abonnement.

> [!NOTE]
> L’exemple présenté dans cet article illustre l’autorisation Azure Active Directory (AD). Pour en savoir plus sur les méthodes d’autorisation, consultez [Autoriser l’accès à des données d’objet blob ou de file d’attente avec Azure CLI](./authorize-data-operations-cli.md).

### <a name="net"></a>[.NET](#tab/dotnet)

Pour utiliser les extraits de code de cet article, vous devez créer une instance [DataLakeServiceClient](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) qui représente le compte de stockage.

#### <a name="connect-by-using-azure-active-directory-ad"></a>Connexion avec Azure Active Directory (AD)

Vous pouvez utiliser la [bibliothèque de client Azure Identity pour .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity) pour authentifier votre application auprès d’Azure AD.

Après avoir installé le package, ajoutez cette instruction using au début de votre fichier de code.

```csharp
using Azure.Identity;
```

Obtenez un ID client, un secret client et un ID de locataire. Pour ce faire, consultez [Obtenir un jeton à partir d’Azure AD pour autoriser les requêtes à partir d’une application cliente](../common/storage-auth-aad-app.md). Dans le cadre de ce processus, vous devrez attribuer l’un des rôles [Azure RBAC (contrôle d’accès en fonction du rôle Azure)](../../role-based-access-control/overview.md) suivants à votre principal de sécurité. 

|Role|Capacité de paramétrage ACL|
|--|--|
|[Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)|Tous les répertoires et fichiers du compte.|
|[Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)|Seuls les répertoires et les fichiers appartenant au principal de sécurité.|

Cet exemple crée une instance de [DataLakeServiceClient](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) à l’aide d’un ID client, d’une clé secrète client et d’un ID locataire.  

```cs
public void GetDataLakeServiceClient(ref DataLakeServiceClient dataLakeServiceClient, 
    String accountName, String clientID, string clientSecret, string tenantID)
{

    TokenCredential credential = new ClientSecretCredential(
        tenantID, clientID, clientSecret, new TokenCredentialOptions());

    string dfsUri = "https://" + accountName + ".dfs.core.windows.net";

    dataLakeServiceClient = new DataLakeServiceClient(new Uri(dfsUri), credential);
}

``` 

#### <a name="connect-by-using-an-account-key"></a>Connexion avec une clé de compte

Il s’agit du moyen le plus simple de se connecter à un compte. 

Cet exemple crée une instance de [DataLakeServiceClient](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) à l’aide d’une clé de compte.

```cs
public void GetDataLakeServiceClient(ref DataLakeServiceClient dataLakeServiceClient,
    string accountName, string accountKey)
{
    StorageSharedKeyCredential sharedKeyCredential =
        new StorageSharedKeyCredential(accountName, accountKey);

    string dfsUri = "https://" + accountName + ".dfs.core.windows.net";

    dataLakeServiceClient = new DataLakeServiceClient
        (new Uri(dfsUri), sharedKeyCredential);
}
```

> [!NOTE]
> Pour plus d’exemples, consultez la documentation [Bibliothèque cliente Azure Identity pour .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity).

### <a name="java"></a>[Java](#tab/java)

Pour utiliser les extraits de code de cet article, vous devez créer une instance **DataLakeServiceClient** qui représente le compte de stockage. 

#### <a name="connect-by-using-an-account-key"></a>Connexion avec une clé de compte

Il s’agit du moyen le plus simple de se connecter à un compte. 

Cet exemple crée une instance de **DataLakeServiceClient** à l’aide d’une clé de compte.

```java

static public DataLakeServiceClient GetDataLakeServiceClient
(String accountName, String accountKey){

    StorageSharedKeyCredential sharedKeyCredential =
        new StorageSharedKeyCredential(accountName, accountKey);

    DataLakeServiceClientBuilder builder = new DataLakeServiceClientBuilder();

    builder.credential(sharedKeyCredential);
    builder.endpoint("https://" + accountName + ".dfs.core.windows.net");

    return builder.buildClient();
}      
```

#### <a name="connect-by-using-azure-active-directory-azure-ad"></a>Se connecter avec Azure Active Directory (Azure AD)

Vous pouvez utiliser la [bibliothèque de client Azure Identity pour Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity) afin d’authentifier votre application auprès d’Azure AD.

Cet exemple crée une instance **DataLakeServiceClient** à l’aide d’un ID client, d’une clé secrète client et d’un ID locataire.  Pour obtenir ces valeurs, consultez [Obtenir un jeton à partir d’Azure AD pour autoriser les requêtes à partir d’une application cliente](../common/storage-auth-aad-app.md).

```java
static public DataLakeServiceClient GetDataLakeServiceClient
    (String accountName, String clientId, String ClientSecret, String tenantID){

    String endpoint = "https://" + accountName + ".dfs.core.windows.net";
        
    ClientSecretCredential clientSecretCredential = new ClientSecretCredentialBuilder()
    .clientId(clientId)
    .clientSecret(ClientSecret)
    .tenantId(tenantID)
    .build();
           
    DataLakeServiceClientBuilder builder = new DataLakeServiceClientBuilder();
    return builder.credential(clientSecretCredential).endpoint(endpoint).buildClient();
 } 
```

> [!NOTE]
> Pour plus d’exemples, consultez la documentation [Bibliothèque de client Azure Identity pour Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity).

### <a name="python"></a>[Python](#tab/python)

Pour utiliser les extraits de code de cet article, vous devez créer une instance **DataLakeServiceClient** qui représente le compte de stockage. 

### <a name="connect-by-using-azure-active-directory-ad"></a>Se connecter avec Azure Active Directory (AD)

Vous pouvez utiliser la [bibliothèque de client Azure Identity pour Python](https://pypi.org/project/azure-identity/) pour authentifier votre application auprès d’Azure AD.

Cet exemple crée une instance de **DataLakeServiceClient** à l’aide d’un ID client, d’une clé secrète client et d’un ID locataire.  Pour obtenir ces valeurs, consultez [Obtenir un jeton à partir d’Azure AD pour autoriser les requêtes à partir d’une application cliente](../common/storage-auth-aad-app.md). Dans le cadre de ce processus, vous devrez attribuer l’un des rôles [Azure RBAC (contrôle d’accès en fonction du rôle Azure)](../../role-based-access-control/overview.md) suivants à votre principal de sécurité. 

|Role|Capacité de paramétrage ACL|
|--|--|
|[Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)|Tous les répertoires et fichiers du compte.|
|[Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)|Seuls les répertoires et les fichiers appartenant au principal de sécurité.|

```python
def initialize_storage_account_ad(storage_account_name, client_id, client_secret, tenant_id):
    
    try:  
        global service_client

        credential = ClientSecretCredential(tenant_id, client_id, client_secret)

        service_client = DataLakeServiceClient(account_url="{}://{}.dfs.core.windows.net".format(
            "https", storage_account_name), credential=credential)
    
    except Exception as e:
        print(e)
```

> [!NOTE]
> Pour obtenir plus d’exemples, consultez la documentation [Bibliothèque de client Azure Identity pour Python](https://pypi.org/project/azure-identity/).

### <a name="connect-by-using-an-account-key"></a>Connexion avec une clé de compte

Il s’agit du moyen le plus simple de se connecter à un compte. 

Cet exemple crée une instance de **DataLakeServiceClient** à l’aide d’une clé de compte.

```python
try:  
    global service_client
        
    service_client = DataLakeServiceClient(account_url="{}://{}.dfs.core.windows.net".format(
        "https", storage_account_name), credential=storage_account_key)
    
except Exception as e:
    print(e)
```
 
- Remplacez la valeur d’espace réservé `storage_account_name` par le nom de votre compte de stockage.

- Remplacez la valeur d’espace réservé `storage_account_key` par la clé d’accès de votre compte de stockage.

---

## <a name="set-an-acl-recursively"></a>Définir une liste de contrôle d’accès (ACL) de manière récursive

Lorsque vous *définissez* une liste de contrôle d’accès (ACL), vous **remplacez** l’ensemble de l’ACL, y compris toutes ses entrées. Si vous souhaitez modifier le niveau d’autorisation d’un principal de sécurité ou ajouter un nouveau principal de sécurité à la liste de contrôle d’accès sans affecter d’autres entrées existantes, vous devez *mettre à jour* l’ACL à la place. Pour mettre à jour une liste de contrôle d’accès au lieu de la remplacer, consultez la section [Mettre à jour une liste de contrôle d’accès (ACL) de manière récursive](#update-an-acl-recursively) de cet article.  

Si vous choisissez de *définir* la liste de contrôle d’accès, vous devez ajouter une entrée pour l’utilisateur propriétaire, une entrée pour le groupe propriétaire et une entrée pour tous les autres utilisateurs. Pour en savoir plus sur l’utilisateur propriétaire, le groupe propriétaire et tous les autres utilisateurs, consultez [Utilisateurs et identités](data-lake-storage-access-control.md#users-and-identities). 

Cette section contient des exemples de définition d’une liste de contrôle d’accès (ACL).

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Définissez une liste de contrôle d’accès de manière récursive à l’aide de la cmdlet **Set-AzDataLakeGen2AclRecursive**.

Cet exemple définit la liste ACL d’un répertoire nommé `my-parent-directory`. Ces entrées donnent à l’utilisateur propriétaire des autorisations de lecture, d’écriture et d’exécution, donnent au groupe propriétaire uniquement des autorisations de lecture et d’exécution, et ne donnent à tous les autres aucun accès. La dernière entrée de la liste de contrôle d’accès dans cet exemple donne à un utilisateur spécifique avec l’ID d’objet « xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx » des autorisations de lecture et d’exécution.

```powershell
$filesystemName = "my-container"
$dirname = "my-parent-directory/"
$userID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rwx 
$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission r-x -InputObject $acl 
$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission "---" -InputObject $acl
$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission r-x -InputObject $acl 

Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl

```

> [!NOTE]
> Si vous souhaitez définir une entrée de liste de contrôle d’accès **par défaut**, utilisez le paramètre **-DefaultScope** quand vous exécutez la commande **Set-AzDataLakeGen2ItemAclObject**. Par exemple : `$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rwx -DefaultScope`.

Pour voir un exemple qui définit des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’article de référence [Set-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/set-azdatalakegen2aclrecursive).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Définissez une liste de contrôle d’accès de manière récursive à l’aide de la commande [az storage fs access set-recursive](/cli/azure/storage/fs/access#az_storage_fs_access_set_recursive).

Cet exemple définit la liste ACL d’un répertoire nommé `my-parent-directory`. Ces entrées donnent à l’utilisateur propriétaire des autorisations de lecture, d’écriture et d’exécution, donnent au groupe propriétaire uniquement des autorisations de lecture et d’exécution, et ne donnent à tous les autres aucun accès. La dernière entrée de la liste de contrôle d’accès dans cet exemple donne à un utilisateur spécifique avec l’ID d’objet « xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx » des autorisations de lecture et d’exécution.

```azurecli
az storage fs access set-recursive --acl "user::rwx,group::r-x,other::---,user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x" -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Si vous souhaitez définir une entre ACL **par défaut**, ajoutez le préfixe `default:` à chaque entrée. Par exemple, `default:user::rwx` ou `default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x`. 

### <a name="net"></a>[.NET](#tab/dotnet)

Définissez une liste de contrôle d’accès de manière récursive en appelant la méthode **DataLakeDirectoryClient.SetAccessControlRecursiveAsync**. Transmettez à cette méthode une [Liste](/dotnet/api/system.collections.generic.list-1) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Chaque [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) définit une entrée de liste de contrôle d’accès. 

Si vous souhaitez définir une entrée de liste de contrôle d’accès **par défaut**, vous pouvez définir la propriété [PathAccessControlItem.DefaultScope](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem.defaultscope#Azure_Storage_Files_DataLake_Models_PathAccessControlItem_DefaultScope) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) sur **true**. 

Cet exemple définit la liste ACL d’un répertoire nommé `my-parent-directory`. Cette méthode accepte un paramètre booléen nommé `isDefaultScope` qui spécifie s’il faut définir la liste de contrôle d’accès par défaut. Ce paramètre est utilisé dans le constructeur de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Ces entrées de la liste de contrôle d'accès donnent à l’utilisateur propriétaire des autorisations de lecture, d’écriture et d’exécution et donnent uniquement au groupe propriétaire des autorisations de lecture et d’exécution. Les autres groupes ne reçoivent aucun accès. La dernière entrée de la liste de contrôle d’accès dans cet exemple donne à un utilisateur spécifique avec l’ID d’objet « xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx » des autorisations de lecture et d’exécution.

```cs
public async void SetACLRecursively(DataLakeServiceClient serviceClient, bool isDefaultScope)
{
    DataLakeDirectoryClient directoryClient =
        serviceClient.GetFileSystemClient("my-container").
            GetDirectoryClient("my-parent-directory");

    List<PathAccessControlItem> accessControlList = 
        new List<PathAccessControlItem>() 
    {
        new PathAccessControlItem(AccessControlType.User, 
            RolePermissions.Read | 
            RolePermissions.Write | 
            RolePermissions.Execute, isDefaultScope),
                    
        new PathAccessControlItem(AccessControlType.Group, 
            RolePermissions.Read | 
            RolePermissions.Execute, isDefaultScope),
                    
        new PathAccessControlItem(AccessControlType.Other, 
            RolePermissions.None, isDefaultScope),

        new PathAccessControlItem(AccessControlType.User, 
            RolePermissions.Read | 
            RolePermissions.Execute, isDefaultScope,
            entityId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"),
    };

    await directoryClient.SetAccessControlRecursiveAsync
        (accessControlList, null);
}

```

Pour voir un exemple qui définit des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) .NET.

### <a name="java"></a>[Java](#tab/java)

Définissez une liste de contrôle d’accès de manière récursive en appelant la méthode **DataLakeDirectoryClient.setAccessControlRecursive**. Transmettez à cette méthode une [Liste](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) d’objets [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). Chaque [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) définit une entrée de liste de contrôle d’accès. 

Si vous souhaitez définir une entrée de contrôle d’accès **par défaut**, vous pouvez appeler la méthode **setDefaultScope** de [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) et y transmettre la valeur **true**. 

Cet exemple définit la liste ACL d’un répertoire nommé `my-parent-directory`. Cette méthode accepte un paramètre booléen nommé `isDefaultScope` qui spécifie s’il faut définir la liste de contrôle d’accès par défaut. Ce paramètre est utilisé chaque fois que la méthode **setDefaultScope** de [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) est appelée. Ces entrées de la liste de contrôle d'accès donnent à l’utilisateur propriétaire des autorisations de lecture, d’écriture et d’exécution et donnent uniquement au groupe propriétaire des autorisations de lecture et d’exécution. Les autres groupes ne reçoivent aucun accès. La dernière entrée de la liste de contrôle d’accès dans cet exemple donne à un utilisateur spécifique avec l’ID d’objet « xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx » des autorisations de lecture et d’exécution.

```java
static public void SetACLRecursively(DataLakeFileSystemClient fileSystemClient, Boolean isDefaultScope){
    
    DataLakeDirectoryClient directoryClient =
        fileSystemClient.getDirectoryClient("my-parent-directory");

    List<PathAccessControlEntry> pathAccessControlEntries = 
        new ArrayList<PathAccessControlEntry>();

    // Create owner entry.
    PathAccessControlEntry ownerEntry = new PathAccessControlEntry();

    RolePermissions ownerPermission = new RolePermissions();
    ownerPermission.setExecutePermission(true).setReadPermission(true).setWritePermission(true);

    ownerEntry.setDefaultScope(isDefaultScope);
    ownerEntry.setAccessControlType(AccessControlType.USER);
    ownerEntry.setPermissions(ownerPermission);

    pathAccessControlEntries.add(ownerEntry);

    // Create group entry.
    PathAccessControlEntry groupEntry = new PathAccessControlEntry();

    RolePermissions groupPermission = new RolePermissions();
    groupPermission.setExecutePermission(true).setReadPermission(true).setWritePermission(false);

    groupEntry.setDefaultScope(isDefaultScope);
    groupEntry.setAccessControlType(AccessControlType.GROUP);
    groupEntry.setPermissions(groupPermission);

    pathAccessControlEntries.add(groupEntry);

    // Create other entry.
    PathAccessControlEntry otherEntry = new PathAccessControlEntry();

    RolePermissions otherPermission = new RolePermissions();
    otherPermission.setExecutePermission(false).setReadPermission(false).setWritePermission(false);

    otherEntry.setDefaultScope(isDefaultScope);
    otherEntry.setAccessControlType(AccessControlType.OTHER);
    otherEntry.setPermissions(otherPermission);

    pathAccessControlEntries.add(otherEntry);

    // Create named user entry.
    PathAccessControlEntry userEntry = new PathAccessControlEntry();

    RolePermissions userPermission = new RolePermissions();
    userPermission.setExecutePermission(true).setReadPermission(true).setWritePermission(false);

    userEntry.setDefaultScope(isDefaultScope);
    userEntry.setAccessControlType(AccessControlType.USER);
    userEntry.setEntityId("xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx");
    userEntry.setPermissions(userPermission);    
    
    pathAccessControlEntries.add(userEntry);
    
    directoryClient.setAccessControlRecursive(pathAccessControlEntries);        

}
```

### <a name="python"></a>[Python](#tab/python)

Définissez une liste de contrôle d’accès de manière récursive en appelant la méthode **DataLakeDirectoryClient.set_access_control_recursive**.

Si vous souhaitez définir une entrée de liste de contrôle d’accès **par défaut**, ajoutez la chaîne `default:` au début de chaque chaîne d’entrée de liste de contrôle d’accès. 

Cet exemple définit la liste ACL d’un répertoire nommé `my-parent-directory`. 

Cette méthode accepte un paramètre booléen nommé `is_default_scope` qui spécifie s’il faut définir la liste de contrôle d’accès par défaut. Si la valeur de ce paramètre est `True`, les entrées de la liste de contrôle d'accès sont précédées de la chaîne `default:`. 

Ces entrées de la liste de contrôle d'accès donnent à l’utilisateur propriétaire des autorisations de lecture, d’écriture et d’exécution et donnent uniquement au groupe propriétaire des autorisations de lecture et d’exécution. Les autres groupes ne reçoivent aucun accès. La dernière entrée de la liste de contrôle d’accès de cet exemple donne à un utilisateur spécifique les autorisations de lecture et d’exécution de l’ID d’objet « xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx ». Ces entrées accordent aux utilisateurs propriétaires les autorisations de lecture, d’écriture et d’exécution, ce qui donne au groupe propriétaire uniquement des autorisations de lecture et d’exécution. Les autres groupes ne reçoivent aucun accès. La dernière entrée de la liste de contrôle d’accès dans cet exemple donne à un utilisateur spécifique avec l’ID d’objet « xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx » des autorisations de lecture et d’exécution.

```python
def set_permission_recursively(is_default_scope):
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")
              
        acl = 'user::rwx,group::rwx,other::rwx,user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r--'   

        if is_default_scope:
           acl = 'default:user::rwx,default:group::rwx,default:other::rwx,default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r--'

        directory_client.set_access_control_recursive(acl=acl)
        
    except Exception as e:
     print(e)
```

Pour voir un exemple qui traite des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py) Python .

---

## <a name="update-an-acl-recursively"></a>Mettre à jour une liste de contrôle d’accès (ACL) de manière récursive

Lorsque vous *mettez à jour* une liste de contrôle d’accès, vous modifiez l’ACL au lieu de la remplacer. Par exemple, vous pouvez ajouter un nouveau principal de sécurité à la liste de contrôle d’accès sans affecter les autres principaux de sécurité listés dans l’ACL.  Pour remplacez la liste de contrôle d’accès au lieu de la mettre à jour, consultez la section [Mettre à jour une liste de contrôle d’accès (ACL) de manière récursive](#set-an-acl-recursively) de cet article. 

Pour mettre à jour une liste de contrôle d’accès, créez un nouvel objet ACL avec l’entrée de l’ACL à mettre à jour, puis utilisez cet objet dans l’opération de mise à jour de l’ACL. Ne récupérez pas la liste de contrôle d’accès existante, il vous suffit de fournir les entrées ACL à mettre à jour.

Cette section contient des exemples de mise à jour d’une liste de contrôle d’accès (ACL)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Mettez à jour une liste de contrôle d’accès de manière récursive à l’aide de la cmdlet **Update-AzDataLakeGen2AclRecursive**. 

Cet exemple met à jour une entrée de liste de contrôle d’accès avec l’autorisation d’écriture. 

```powershell
$filesystemName = "my-container"
$dirname = "my-parent-directory/"
$userID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission rwx

Update-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl

```

> [!NOTE]
> Si vous souhaitez mettre à jour une entrée de liste de contrôle d’accès **par défaut**, utilisez le paramètre **-DefaultScope** lors de l’exécution de la commande **Set-AzDataLakeGen2ItemAclObject**. Par exemple : `$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission rwx -DefaultScope`.

Pour voir un exemple qui met à jour des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’article de référence [Update-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/update-azdatalakegen2aclrecursive).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Mettez à jour une liste de contrôle d’accès de manière récursive à l’aide de la commande [az storage fs access update-recursive](/cli/azure/storage/fs/access#az_storage_fs_access_update_recursive). 

Cet exemple met à jour une entrée de liste de contrôle d’accès avec l’autorisation d’écriture. 

```azurecli
az storage fs access update-recursive --acl "user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:rwx" -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Si vous souhaitez mettre à jour une entrée ACL **par défaut**, ajoutez le préfixe `default:` à chaque entrée. Par exemple, `default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x`.

### <a name="net"></a>[.NET](#tab/dotnet)

Mettez à jour une liste de contrôle d’accès de manière récursive en appelant la méthode **DataLakeDirectoryClient.UpdateAccessControlRecursiveAsync**.  Transmettez à cette méthode une [Liste](/dotnet/api/system.collections.generic.list-1) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Chaque [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) définit une entrée de liste de contrôle d’accès. 

Si vous souhaitez mettre à jour une entrée de liste de contrôle d’accès **par défaut**, vous pouvez définir la propriété [PathAccessControlItem.DefaultScope](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem.defaultscope#Azure_Storage_Files_DataLake_Models_PathAccessControlItem_DefaultScope) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) sur **true**. 

Cet exemple met à jour une entrée de liste de contrôle d’accès avec l’autorisation d’écriture. Cette méthode accepte un paramètre booléen nommé `isDefaultScope` qui spécifie s’il faut mettre à jour la liste de contrôle d’accès par défaut. Ce paramètre est utilisé dans le constructeur de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem).

```cs
public async void UpdateACLsRecursively(DataLakeServiceClient serviceClient, bool isDefaultScope)
{
    DataLakeDirectoryClient directoryClient =
        serviceClient.GetFileSystemClient("my-container").
        GetDirectoryClient("my-parent-directory");

    List<PathAccessControlItem> accessControlListUpdate = 
        new List<PathAccessControlItem>()
    {
        new PathAccessControlItem(AccessControlType.User, 
            RolePermissions.Read |
            RolePermissions.Write | 
            RolePermissions.Execute, isDefaultScope, 
            entityId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"),
    };

    await directoryClient.UpdateAccessControlRecursiveAsync
        (accessControlListUpdate, null);

}
```

Pour voir un exemple qui met à jour des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) .NET.

### <a name="java"></a>[Java](#tab/java)

Mettez à jour une liste de contrôle d’accès de manière récursive en appelant la méthode **DataLakeDirectoryClient.updateAccessControlRecursive**.  Transmettez à cette méthode une [Liste](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) d’objets [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). Chaque [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) définit une entrée de liste de contrôle d’accès. 

Si vous souhaitez mettre à jour une entrée de contrôle d’accès **par défaut**, vous pouvez appeler la méthode **setDefaultScope** de [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) et y transmettre la valeur **true**. 

Cet exemple met à jour une entrée de liste de contrôle d’accès avec l’autorisation d’écriture. Cette méthode accepte un paramètre booléen nommé `isDefaultScope` qui spécifie s’il faut mettre à jour la liste de contrôle d’accès par défaut. Ce paramètre est utilisé à chaque appel de la méthode **setDefaultScope** de [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). 

```java
static public void UpdateACLRecursively(DataLakeFileSystemClient fileSystemClient, Boolean isDefaultScope){

    DataLakeDirectoryClient directoryClient =
    fileSystemClient.getDirectoryClient("my-parent-directory");

    List<PathAccessControlEntry> pathAccessControlEntries = 
        new ArrayList<PathAccessControlEntry>();

    // Create named user entry.
    PathAccessControlEntry userEntry = new PathAccessControlEntry();

    RolePermissions userPermission = new RolePermissions();
    userPermission.setExecutePermission(true).setReadPermission(true).setWritePermission(true);

    userEntry.setDefaultScope(isDefaultScope);
    userEntry.setAccessControlType(AccessControlType.USER);
    userEntry.setEntityId("xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx");
    userEntry.setPermissions(userPermission);    
    
    pathAccessControlEntries.add(userEntry);
    
    directoryClient.updateAccessControlRecursive(pathAccessControlEntries);          
}
```

### <a name="python"></a>[Python](#tab/python)

Mettez à jour une liste de contrôle d’accès de manière récursive en appelant la méthode **DataLakeDirectoryClient.update_access_control_recursive**. Si vous souhaitez mettre à jour une entrée de liste de contrôle d’accès **par défaut**, ajoutez la chaîne `default:` au début de chaque chaîne d’entrée de liste de contrôle d’accès. 

Cet exemple met à jour une entrée de liste de contrôle d’accès avec l’autorisation d’écriture.

Cet exemple définit la liste ACL d’un répertoire nommé `my-parent-directory`. Cette méthode accepte un paramètre booléen nommé `is_default_scope` qui spécifie s’il faut mettre à jour la liste de contrôle d’accès par défaut. Si la valeur de ce paramètre est `True`, l’entrée de la liste de contrôle d'accès mise à jour est précédée de la chaîne `default:`.  

```python
def update_permission_recursively(is_default_scope):
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")

        acl = 'user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:rwx'   

        if is_default_scope:
           acl = 'default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:rwx'

        directory_client.update_access_control_recursive(acl=acl)

        acl_props = directory_client.get_access_control()
        
        print(acl_props['permissions'])

    except Exception as e:
     print(e)
```

Pour voir un exemple qui traite des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py) Python .

---

## <a name="remove-acl-entries-recursively"></a>Supprimer des entrées de liste de contrôle d’accès (ACL) de manière récursive

Vous pouvez supprimer une ou plusieurs entrées de liste de contrôle d’accès (ACL) de manière récursive. Pour supprimer une entrée de liste de contrôle d’accès, créez un nouvel objet ACL pour l’entrée ACL à supprimer, puis utilisez cet objet dans l’opération de suppression de l’ACL. Ne récupérez pas la liste de contrôle d’accès existante, il vous suffit de fournir les entrées ACL à supprimer. 

Cette section contient des exemples de suppression d’une liste de contrôle d’accès (ACL). 

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Supprimez des entrées de liste de contrôle d’accès à l’aide de la cmdlet **Remove-AzDataLakeGen2AclRecursive**. 

Cet exemple supprime une entrée de liste de contrôle d’accès dans le répertoire racine du conteneur.  

```powershell
$filesystemName = "my-container"
$userID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission "---" 

Remove-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName  -Acl $acl
```

> [!NOTE]
> Si vous souhaitez supprimer une entrée de liste de contrôle d’accès **par défaut**, utilisez le paramètre **-DefaultScope** lors de l’exécution de la commande **Set-AzDataLakeGen2ItemAclObject**. Par exemple : `$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission "---" -DefaultScope`.

Pour voir un exemple qui supprime des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’article de référence [Remove-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/remove-azdatalakegen2aclrecursive).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Supprimez les entrées de liste de contrôle d’accès à l’aide de la commande [az storage fs access remove-recursive](/cli/azure/storage/fs/access#az_storage_fs_access_remove_recursive). 

Cet exemple supprime une entrée de liste de contrôle d’accès dans le répertoire racine du conteneur.  

```azurecli
az storage fs access remove-recursive --acl "user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Si vous souhaitez supprimer une entre ACL **par défaut**, ajoutez le préfixe `default:` à chaque entrée. Par exemple, `default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

### <a name="net"></a>[.NET](#tab/dotnet)

Supprimez des entrées de liste de contrôle d’accès en appelant la méthode **DataLakeDirectoryClient.RemoveAccessControlRecursiveAsync**. Transmettez à cette méthode une [Liste](/dotnet/api/system.collections.generic.list-1) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Chaque [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) définit une entrée de liste de contrôle d’accès. 

Si vous souhaitez supprimer une entrée de liste de contrôle d’accès **par défaut**, vous pouvez définir la propriété [PathAccessControlItem.DefaultScope](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem.defaultscope#Azure_Storage_Files_DataLake_Models_PathAccessControlItem_DefaultScope) de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) sur **true**. 

Cet exemple supprimer une entrée de liste de contrôle d’accès dans la liste ACL du répertoire nommé `my-parent-directory`. Cette méthode accepte un paramètre booléen nommé `isDefaultScope` qui spécifie s’il faut supprimer l’entrée de la liste de contrôle d’accès par défaut. Ce paramètre est utilisé dans le constructeur de [PathAccessControlItem](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem).

```cs
public async void RemoveACLsRecursively(DataLakeServiceClient serviceClient, isDefaultScope)
{
    DataLakeDirectoryClient directoryClient =
        serviceClient.GetFileSystemClient("my-container").
            GetDirectoryClient("my-parent-directory");

    List<RemovePathAccessControlItem> accessControlListForRemoval = 
        new List<RemovePathAccessControlItem>()
        {
            new RemovePathAccessControlItem(AccessControlType.User, isDefaultScope,
            entityId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"),
        };

    await directoryClient.RemoveAccessControlRecursiveAsync
        (accessControlListForRemoval, null);

}
```

Pour voir un exemple qui supprime des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) .NET.

### <a name="java"></a>[Java](#tab/java)

Supprimez des entrées de liste de contrôle d’accès en appelant la méthode **DataLakeDirectoryClient.removeAccessControlRecursive**. Transmettez à cette méthode une [Liste](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) d’objets [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). Chaque [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) définit une entrée de liste de contrôle d’accès. 

Si vous souhaitez supprimer une entrée de contrôle d’accès **par défaut**, vous pouvez appeler la méthode **setDefaultScope** de [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) et y transmettre la valeur **true**.  

Cet exemple supprimer une entrée de liste de contrôle d’accès dans la liste ACL du répertoire nommé `my-parent-directory`. Cette méthode accepte un paramètre booléen nommé `isDefaultScope` qui spécifie s’il faut supprimer l’entrée de la liste de contrôle d’accès par défaut. Ce paramètre est utilisé à chaque appel de la méthode **setDefaultScope** de [PathAccessControlEntry](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html).


```java
static public void RemoveACLRecursively(DataLakeFileSystemClient fileSystemClient, Boolean isDefaultScope){

    DataLakeDirectoryClient directoryClient =
    fileSystemClient.getDirectoryClient("my-parent-directory");

    List<PathRemoveAccessControlEntry> pathRemoveAccessControlEntries = 
        new ArrayList<PathRemoveAccessControlEntry>();

    // Create named user entry.
    PathRemoveAccessControlEntry userEntry = new PathRemoveAccessControlEntry();

    RolePermissions userPermission = new RolePermissions();
    userPermission.setExecutePermission(true).setReadPermission(true).setWritePermission(true);

    userEntry.setDefaultScope(isDefaultScope);
    userEntry.setAccessControlType(AccessControlType.USER);
    userEntry.setEntityId("xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"); 
    
    pathRemoveAccessControlEntries.add(userEntry);
    
    directoryClient.removeAccessControlRecursive(pathRemoveAccessControlEntries);      

}
```

### <a name="python"></a>[Python](#tab/python)

Supprimez des entrées de liste de contrôle d’accès en appelant la méthode **DataLakeDirectoryClient.remove_access_control_recursive**. Si vous souhaitez supprimer une entrée de liste de contrôle d’accès **par défaut**, ajoutez la chaîne `default:` au début de la chaîne d’entrée de liste de contrôle d’accès. 

Cet exemple supprimer une entrée de liste de contrôle d’accès dans la liste ACL du répertoire nommé `my-parent-directory`. Cette méthode accepte un paramètre booléen nommé `is_default_scope` qui spécifie s’il faut supprimer l’entrée de la liste de contrôle d’accès par défaut. Si la valeur de ce paramètre est `True`, l’entrée de la liste de contrôle d'accès mise à jour est précédée de la chaîne `default:`. 

```python
def remove_permission_recursively(is_default_scope):
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")
              
        acl = 'user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'

        if is_default_scope:
           acl = 'default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'

        directory_client.remove_access_control_recursive(acl=acl)

    except Exception as e:
     print(e)
```

Pour voir un exemple qui traite des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py) Python .

---

## <a name="recover-from-failures"></a>Récupérer suite à des échecs

Vous pouvez rencontrer des erreurs d’exécution ou d’autorisation. Pour les erreurs d’exécution, redémarrez le processus à partir du début. Des erreurs d’autorisation peuvent se produire si le principal de sécurité ne dispose pas des autorisations suffisantes pour modifier la liste de contrôle d’accès d’un répertoire ou d’un fichier qui se trouve dans la hiérarchie de répertoires en cours de modification. Résolvez le problème d’autorisation, puis choisissez de reprendre le processus à partir du point de défaillance à l’aide d’un jeton de continuation ou de redémarrer le processus à partir du début. Vous n’êtes pas obligé d’utiliser le jeton de continuation si vous préférez redémarrer à partir du début. Vous pouvez réappliquer les entrées de liste de contrôle d’accès sans incidence négative.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cet exemple retourne des résultats à la variable, puis canalise les entrées ayant échoué vers une table mise en forme.

```powershell
$result = Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl
$result
$result.FailedEntries | ft 
```

En fonction de la sortie du tableau, vous pouvez corriger les erreurs d’autorisation, puis reprendre l’exécution à l’aide du jeton de continuation.

```powershell
$result = Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl -ContinuationToken $result.ContinuationToken
$result

```

Pour voir un exemple qui définit des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’article de référence [Set-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/set-azdatalakegen2aclrecursive).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

En cas d’échec, vous pouvez retourner un jeton de continuation en affectant au paramètre `--continue-on-failure` la valeur `false`. Une fois que vous avez corrigé les erreurs, vous pouvez reprendre le processus à partir du point de défaillance en exécutant à nouveau la commande, puis en affectant le paramètre `--continuation` au jeton de continuation. 

```azurecli
az storage fs access set-recursive --acl "user::rw-,group::r-x,other::---" --continue-on-failure false --continuation xxxxxxx -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login  
```

## <a name="net"></a>[.NET](#tab/dotnet)

Cet exemple retourne un jeton de continuation en cas d’échec. L’application peut appeler à nouveau cet exemple de méthode une fois que l’erreur a été traitée et transmettre le jeton de continuation. Si cet exemple de méthode est appelé pour la première fois, l’application peut transmettre une valeur de `null` pour le paramètre de jeton de continuation. 

```cs
public async Task<string> ResumeAsync(DataLakeServiceClient serviceClient,
    DataLakeDirectoryClient directoryClient,
    List<PathAccessControlItem> accessControlList, 
    string continuationToken)
{
    try
    {
        var accessControlChangeResult =
            await directoryClient.SetAccessControlRecursiveAsync(
                accessControlList, continuationToken: continuationToken, null);

        if (accessControlChangeResult.Value.Counters.FailedChangesCount > 0)
        {
            continuationToken =
                accessControlChangeResult.Value.ContinuationToken;
        }

        return continuationToken;
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
        return continuationToken;
    }

}
```

Pour voir un exemple qui définit des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) .NET.

### <a name="java"></a>[Java](#tab/java)

Cet exemple retourne un jeton de continuation en cas d’échec. L’application peut appeler à nouveau cet exemple de méthode une fois que l’erreur a été traitée et transmettre le jeton de continuation. Si cet exemple de méthode est appelé pour la première fois, l’application peut transmettre une valeur de `null` pour le paramètre de jeton de continuation. 

```java
static public String ResumeSetACLRecursively(DataLakeFileSystemClient fileSystemClient,
DataLakeDirectoryClient directoryClient,
List<PathAccessControlEntry> accessControlList, 
String continuationToken){

    try{
        PathSetAccessControlRecursiveOptions options = new PathSetAccessControlRecursiveOptions(accessControlList);
        
        options.setContinuationToken(continuationToken);
    
        Response<AccessControlChangeResult> accessControlChangeResult =  
            directoryClient.setAccessControlRecursiveWithResponse(options, null, null);

        if (accessControlChangeResult.getValue().getCounters().getFailedChangesCount() > 0)
        {
            continuationToken =
                accessControlChangeResult.getValue().getContinuationToken();
        }
    
        return continuationToken;

    }
    catch(Exception ex){
    
        System.out.println(ex.toString());
        return continuationToken;
    }
}
```

### <a name="python"></a>[Python](#tab/python)

Cet exemple retourne un jeton de continuation en cas d’échec. L’application peut appeler à nouveau cet exemple de méthode une fois que l’erreur a été traitée et transmettre le jeton de continuation. Si cet exemple de méthode est appelé pour la première fois, l’application peut transmettre une valeur de ``None`` pour le paramètre de jeton de continuation. 

```python
def resume_set_acl_recursive(continuation_token):
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")
              
        acl = 'user::rwx,group::rwx,other::rwx,user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x'

        acl_change_result = directory_client.set_access_control_recursive(acl=acl, continuation=continuation_token)

        continuation_token = acl_change_result.continuation

        return continuation_token
        
    except Exception as e:
     print(e) 
     return continuation_token
```

Pour voir un exemple qui traite des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py) Python .

---

Si vous souhaitez que le processus se termine sans être interrompu par des erreurs d’autorisation, vous pouvez le spécifier.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cet exemple utilise le paramètre `ContinueOnFailure` pour que l’exécution continue même si l’opération rencontre une erreur d’autorisation. 

```powershell
$result = Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl -ContinueOnFailure

echo "[Result Summary]"
echo "TotalDirectoriesSuccessfulCount: `t$($result.TotalFilesSuccessfulCount)"
echo "TotalFilesSuccessfulCount: `t`t`t$($result.TotalDirectoriesSuccessfulCount)"
echo "TotalFailureCount: `t`t`t`t`t$($result.TotalFailureCount)"
echo "FailedEntries:"$($result.FailedEntries | ft) 
```

Pour voir un exemple qui définit des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’article de référence [Set-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/set-azdatalakegen2aclrecursive).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vous assurer que le processus se termine sans interruption, définissez le paramètre `--continue-on-failure` sur `true`. 

```azurecli
az storage fs access set-recursive --acl "user::rw-,group::r-x,other::---" --continue-on-failure true --continuation xxxxxxx -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login  
```

### <a name="net"></a>[.NET](#tab/dotnet)

Pour vous assurer que le processus se termine sans interruption, passez un objet **AccessControlChangedOptions** et affectez à la propriété **ContinueOnFailure** de cet objet la valeur ``true``.

Cet exemple définit les entrées ACL de manière récursive. Si ce code rencontre une erreur d’autorisation, il enregistre cet échec et poursuit l’exécution. Cet exemple inscrit le nombre d’échecs sur la console. 

```cs
public async Task ContinueOnFailureAsync(DataLakeServiceClient serviceClient,
    DataLakeDirectoryClient directoryClient, 
    List<PathAccessControlItem> accessControlList)
{
    var accessControlChangeResult = 
        await directoryClient.SetAccessControlRecursiveAsync(
            accessControlList, null, new AccessControlChangeOptions() 
            { ContinueOnFailure = true });

    var counters = accessControlChangeResult.Value.Counters;

    Console.WriteLine("Number of directories changed: " +
        counters.ChangedDirectoriesCount.ToString());

    Console.WriteLine("Number of files changed: " +
        counters.ChangedFilesCount.ToString());

    Console.WriteLine("Number of failures: " +
        counters.FailedChangesCount.ToString());
}
```

Pour voir un exemple qui définit des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0) .NET.

### <a name="java"></a>[Java](#tab/java)

Pour vous assurer que le processus se termine sans interruption, appelez la méthode **setContinueOnFailure** d’un objet [PathSetAccessControlRecursiveOptions](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) et passez la valeur **true**.

Cet exemple définit les entrées ACL de manière récursive. Si ce code rencontre une erreur d’autorisation, il enregistre cet échec et poursuit l’exécution. Cet exemple inscrit le nombre d’échecs sur la console. 

```java
static public void ContinueOnFailure(DataLakeFileSystemClient fileSystemClient,
DataLakeDirectoryClient directoryClient,
List<PathAccessControlEntry> accessControlList){
    
    PathSetAccessControlRecursiveOptions options = 
        new PathSetAccessControlRecursiveOptions(accessControlList);
        
    options.setContinueOnFailure(true);
    
    Response<AccessControlChangeResult> accessControlChangeResult =  
        directoryClient.setAccessControlRecursiveWithResponse(options, null, null);

    AccessControlChangeCounters counters = accessControlChangeResult.getValue().getCounters();

    System.out.println("Number of directories changes: " + 
        counters.getChangedDirectoriesCount());

    System.out.println("Number of files changed: " + 
        counters.getChangedDirectoriesCount());

    System.out.println("Number of failures: " + 
        counters.getChangedDirectoriesCount());
}
```

### <a name="python"></a>[Python](#tab/python)

Pour vous assurer que le processus se termine sans interruption, ne passez pas de jeton de continuation dans la méthode **DataLakeDirectoryClient.set_access_control_recursive**.

Cet exemple définit les entrées ACL de manière récursive. Si ce code rencontre une erreur d’autorisation, il enregistre cet échec et poursuit l’exécution. Cet exemple inscrit le nombre d’échecs sur la console. 

```python
def continue_on_failure():
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")
              
        acl = 'user::rwx,group::rwx,other::rwx,user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r--'

        acl_change_result = directory_client.set_access_control_recursive(acl=acl)

        print("Summary: {} directories and {} files were updated successfully, {} failures were counted."
          .format(acl_change_result.counters.directories_successful, acl_change_result.counters.files_successful,
                  acl_change_result.counters.failure_count))
        
    except Exception as e:
     print(e)
```

Pour voir un exemple qui traite des ACL de manière récursive dans des lots en spécifiant une taille de lot, consultez l’[exemple](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py) Python .

---

## <a name="resources"></a>Ressources

Cette section contient des liens vers des bibliothèques et des exemples de code.

#### <a name="libraries"></a>Bibliothèques

- [PowerShell](https://www.powershellgallery.com/packages/Az.Storage/3.0.0)
- [Azure CLI](/cli/azure/storage/fs/access)
- [.NET](https://pkgs.dev.azure.com/azure-sdk/public/_packaging/azure-sdk-for-net/nuget/v3/index.json)
- [Java](/java/api/overview/azure/storage-file-datalake-readme)
- [Python](https://recursiveaclpr.blob.core.windows.net/privatedrop/azure_storage_file_datalake-12.1.0b99-py2.py3-none-any.whl?sv=2019-02-02&st=2020-08-24T07%3A47%3A01Z&se=2021-08-25T07%3A47%3A00Z&sr=b&sp=r&sig=H1XYw4FTLJse%2BYQ%2BfamVL21UPVIKRnnh2mfudA%2BfI0I%3D)
- [REST](/rest/api/storageservices/datalakestoragegen2/path/update)

#### <a name="code-samples"></a>Exemples de code

- PowerShell : [Fichier Lisezmoi](https://recursiveaclpr.blob.core.windows.net/privatedrop/README.txt?sv=2019-02-02&st=2020-08-24T17%3A03%3A18Z&se=2021-08-25T17%3A03%3A00Z&sr=b&sp=r&sig=sPdKiCSXWExV62sByeOYqBTqpGmV2h9o8BLij3iPkNQ%3D) | [Exemple](https://recursiveaclpr.blob.core.windows.net/privatedrop/samplePS.ps1?sv=2019-02-02&st=2020-08-24T17%3A04%3A44Z&se=2021-08-25T17%3A04%3A00Z&sr=b&sp=r&sig=dNNKS%2BZcp%2F1gl6yOx6QLZ6OpmXkN88ZjBeBtym1Mejo%3D)

- Azure CLI : [Exemple](https://github.com/Azure/azure-cli/blob/2a55a5350696a3a93a13f364f2104ec8bc82cdd3/src/azure-cli/azure/cli/command_modules/storage/docs/ADLS%20Gen2.md)

- NET : [Fichier Lisezmoi](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FREADME%2520for%2520net%3Fsv%3D2019-02-02%26st%3D2020-08-25T23%253A20%253A42Z%26se%3D2021-08-26T23%253A20%253A00Z%26sr%3Db%26sp%3Dr%26sig%3DKrnHvasHoSoVeUyr2g%2fSc2aDVW3De4A%2fvx0lFWZs494%253D&data=02%7C01%7Cnormesta%40microsoft.com%7Cda902e4fe6c24e6a07d908d8494fd4bd%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637339954503767961&sdata=gd%2B2LphTtDFVb7pZko9rkGO9OG%2FVvmeXprHB9IOEYXE%3D&reserved=0) | [Exemple](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0)

- Python : [Fichier Lisezmoi](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FREADME%2520for%2520python%3Fsv%3D2019-02-02%26st%3D2020-08-25T23%253A21%253A47Z%26se%3D2021-08-26T23%253A21%253A00Z%26sr%3Db%26sp%3Dr%26sig%3DRq6Bl5lXrtYk79thy8wX7UTbjyd2f%252B6xzVBFFVYbdYg%253D&data=02%7C01%7Cnormesta%40microsoft.com%7Cda902e4fe6c24e6a07d908d8494fd4bd%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637339954503777915&sdata=3e46Lp2miOHj755Gh0odH3M0%2BdTF3loGCCBENrulVTM%3D&reserved=0) | [Exemple](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py)

## <a name="best-practice-guidelines"></a>Conseils de bonnes pratiques

Cette section décrit certaines des meilleures pratiques pour définir des listes de contrôle d’accès (ACL) de manière récursive. 

#### <a name="handling-runtime-errors"></a>Traitement des erreurs d’exécution

Une erreur d’exécution peut se produire pour de nombreuses raisons (par exemple, une panne ou un problème de connectivité client). Si vous rencontrez une erreur d’exécution, redémarrez le processus ACL récursif. Les listes de contrôle d’accès peuvent être réappliquées à des éléments sans impact négatif. 

#### <a name="handling-permission-errors-403"></a>Traitement des erreurs d’autorisation (403)

Si vous rencontrez une exception de contrôle d’accès lors de l’exécution d’un processus ACL récursif, votre [principal de sécurité](../../role-based-access-control/overview.md#security-principal) AD peut ne pas disposer des autorisations suffisantes pour appliquer une liste de contrôle d’accès à un ou plusieurs des éléments enfants dans la hiérarchie de répertoires. Lorsqu’une erreur d’autorisation se produit, le processus s’arrête et un jeton de continuation est fourni. Corrigez le problème d’autorisation, puis utilisez le jeton de continuation pour traiter le jeu de données restant. Les répertoires et les fichiers qui ont déjà été traités correctement n’ont pas besoin d’être retraités. Vous pouvez également choisir de redémarrer le processus ACL récursif. Les listes de contrôle d’accès peuvent être réappliquées à des éléments sans impact négatif. 

#### <a name="credentials"></a>Informations d'identification 

Nous vous recommandons de provisionner un principal de sécurité Azure AD auquel a été attribué le rôle [Propriétaire des données Blob de stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) dans l’étendue du conteneur ou du compte de stockage cible. 

#### <a name="performance"></a>Performances 

Pour réduire la latence, nous vous recommandons d’exécuter le processus ACL récursif dans une machine virtuelle Azure située dans la même région que votre compte de stockage. 

#### <a name="acl-limits"></a>Limites des listes de contrôle d’accès (ACL)

Le nombre maximal de listes ACL que vous pouvez appliquer à un répertoire ou à un fichier est de 32 ACL d’accès et de 32 ACL par défaut. Pour plus d’informations, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen2](./data-lake-storage-access-control.md).

## <a name="see-also"></a>Voir aussi

- [Contrôle d’accès dans Azure Data Lake Storage Gen2](./data-lake-storage-access-control.md)
- [Problèmes connus](data-lake-storage-known-issues.md)
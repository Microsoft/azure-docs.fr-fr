---
title: 'Tutoriel : Accès d’une application web au stockage avec des identités managées | Azure'
description: Dans ce tutoriel, vous allez apprendre à accéder au stockage Azure pour le compte d’une application à l’aide d’identités managées.
services: storage, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 11/30/2020
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: 72b1d4fe864c23c0ac065e47d96ab0c78866defa
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96435839"
---
# <a name="tutorial-access-azure-storage-from-a-web-app"></a>Tutoriel : Accéder au stockage Azure à partir d’une application web

Découvrez comment accéder au stockage Azure pour le compte d’une application web (et non d’un utilisateur connecté) s’exécutant sur Azure App Service à l’aide d’identités managées.

:::image type="content" alt-text="Diagramme illustrant comment accéder au stockage." source="./media/scenario-secure-app-access-storage/web-app-access-storage.svg" border="false":::

Vous souhaitez ajouter l’accès au plan de données Azure (stockage Azure, Azure SQL Database, Azure Key Vault ou d’autres services) à partir de votre application web. Vous pourriez utiliser une clé partagée, mais vous devriez alors vous occuper de la sécurité opérationnelle afin d’identifier qui pourrait créer, déployer et gérer le secret. La clé pourrait également être archivée dans GitHub, que les pirates savent analyser. Un moyen plus sûr d’accorder à votre application web l’accès aux données consiste à utiliser des [identités managées](../active-directory/managed-identities-azure-resources/overview.md).

Une identité managée d’Azure Active Directory (Azure AD) permet à App Service d’accéder aux ressources par le biais du contrôle d’accès en fonction du rôle (RBAC) sans demander d’informations d’identification d’application. Après avoir affecté une identité managée à votre application web, Azure s’occupe de la création et de la distribution d’un certificat. Vous n’avez pas à vous soucier de la gestion des secrets ou des informations d’identification d’application.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer une identité managée affectée par le système sur une application web.
> * Créer un compte de stockage et un conteneur de stockage d’objets blob Azure.
> * Accéder au stockage à partir d’une application web avec des identités managées.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Une application web s’exécutant sur Azure App Service et pour laquelle le [module d’authentification/autorisation App Service est activé](scenario-secure-app-authentication-app-service.md)

## <a name="enable-managed-identity-on-an-app"></a>Activer une identité managée sur une application

Si vous créez et publiez votre application web à l’aide de Visual Studio, l’identité managée a été activée pour vous sur votre application. Dans votre service d’application, sélectionnez **Identité** dans le volet gauche, puis **Affectée par le système**. Vérifiez qu’**État** a la valeur **Activé**. Si ce n’est pas le cas, sélectionnez **Enregistrer**, puis **Oui** pour activer l’identité managée affectée par le système. Lorsque l’identité managée est activée, l’état prend la valeur **Activé** et l’ID d’objet est disponible.

:::image type="content" alt-text="Capture d’écran montrant l’option d’identité affectée par le système." source="./media/scenario-secure-app-access-storage/create-system-assigned-identity.png":::

Cette étape crée un ID d’objet différent de l’ID d’application créé dans le volet **Authentification/Autorisation**. Copiez l’ID d’objet de l’identité managée affectée par le système. Vous en aurez besoin ultérieurement.

## <a name="create-a-storage-account-and-blob-storage-container"></a>Créer un compte de stockage et un conteneur de stockage d’objets blob

Vous êtes maintenant prêt à créer un compte de stockage et un conteneur de stockage d’objets blob.

Chaque compte de stockage doit appartenir à un groupe de ressources Azure. Un groupe de ressources est un conteneur logique servant à grouper vos services Azure. Quand vous créez un compte de stockage, vous avez le choix entre créer un groupe de ressources ou utiliser un groupe de ressources existant. Cet article montre comment créer un groupe de ressources.

Un compte de stockage v2 à usage général fournit un accès à tous les services de Stockage Azure : objets blob, fichiers, files d’attente, tables et disques. Les étapes décrites ici créent un compte de stockage v2 universel, mais les étapes de création d’un autre type de compte de stockage sont similaires.

Les objets BLOB dans le stockage Azure sont organisés en conteneurs. Avant de pouvoir télécharger un objet blob plus loin dans ce tutoriel, vous devez d’abord créer un conteneur.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour créer un compte de stockage v2 universel dans le portail Azure, procédez comme suit.

1. Dans le menu du portail Azure, sélectionnez **Tous les services**. Dans la liste des ressources, entrez **Comptes de stockage**. Au fur et à mesure de la saisie, la liste est filtrée. Sélectionnez **Comptes de stockage**.

1. Dans la fenêtre **Comptes de stockage** qui apparaît, sélectionnez **Ajouter**.

1. Sélectionnez l’abonnement dans lequel créer le compte de stockage.

1. Sous le champ **Groupe de ressources**, sélectionnez le groupe de ressources qui contient votre application web dans le menu déroulant.

1. Ensuite, entrez un nom pour votre compte de stockage. Le nom choisi doit être unique dans tout Azure. Le nom doit aussi contenir entre 3 et 24 caractères, et uniquement des lettres minuscules et des chiffres.

1. Sélectionnez l’emplacement de votre compte de stockage ou utilisez l’emplacement par défaut.

1. Laissez ces champs définis sur leur valeur par défaut :

    |Champ|Valeur|
    |--|--|
    |Modèle de déploiement|Gestionnaire de ressources|
    |Performances|Standard|
    |Type de compte|StorageV2 (v2 universel)|
    |Réplication|Stockage géoredondant avec accès en lecture (RA-GRS)|
    |Niveau d’accès|À chaud|

1. Cliquez sur **Vérifier + créer** pour passer en revue vos paramètres de compte de stockage et créer le compte.

1. Sélectionnez **Create** (Créer).

Pour créer un conteneur de stockage d’objets blob dans le stockage Azure, effectuez les étapes suivantes.

1. Accédez à votre nouveau compte de stockage dans le portail Azure.

1. Dans le menu de gauche du compte de stockage, faites défiler jusqu’à la section **Service Blob**, puis sélectionnez **Conteneurs**.

1. Sélectionnez le bouton **+ Conteneur**.

1. Entrez un nom pour votre nouveau conteneur. Le nom du conteneur doit être en minuscules, commencer par une lettre ou un chiffre, et peut comporter uniquement des lettres, des chiffres et des tirets (-).

1. Définissez le niveau d’accès public sur le conteneur. Le niveau par défaut est **Private (no anonymous access)** (Privé (aucun accès anonyme)).

1. Sélectionnez **OK** pour créer le conteneur.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour créer un compte de stockage v2 universel et un conteneur de stockage d’objets blob, exécutez le script suivant. Spécifiez le nom du groupe de ressources qui contient votre application web. Entrez un nom pour votre compte de stockage. Le nom choisi doit être unique dans tout Azure. Le nom doit aussi contenir entre 3 et 24 caractères, et uniquement des lettres minuscules et des chiffres.

Spécifiez l’emplacement de votre compte de stockage. Pour voir la liste des emplacements valides pour votre abonnement, exécutez ```Get-AzLocation | select Location```. Le nom du conteneur doit être en minuscules, commencer par une lettre ou un chiffre, et peut comporter uniquement des lettres, des chiffres et des tirets (-).

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```powershell
Connect-AzAccount

$resourceGroup = "securewebappresourcegroup"
$location = "<location>"
$storageName="securewebappstorage"
$containerName = "securewebappblobcontainer"

$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -Location $location `
  -SkuName Standard_RAGRS `
  -Kind StorageV2

$ctx = $storageAccount.Context

New-AzStorageContainer -Name $containerName -Context $ctx -Permission blob
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour créer un compte de stockage v2 universel et un conteneur de stockage d’objets blob, exécutez le script suivant. Spécifiez le nom du groupe de ressources qui contient votre application web. Entrez un nom pour votre compte de stockage. Le nom choisi doit être unique dans tout Azure. Le nom doit aussi contenir entre 3 et 24 caractères, et uniquement des lettres minuscules et des chiffres. 

Spécifiez l’emplacement de votre compte de stockage. Le nom du conteneur doit être en minuscules, commencer par une lettre ou un chiffre, et peut comporter uniquement des lettres, des chiffres et des tirets (-).

L’exemple suivant utilise votre compte Azure AD pour autoriser l’opération à créer le conteneur. Avant de créer le conteneur, attribuez-vous le rôle Contributeur aux données Blob du stockage. Même si vous êtes le propriétaire du compte, vous avez besoin d’autorisations explicites pour effectuer des opérations de données sur le compte de stockage.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
az login

az storage account create \
    --name securewebappstorage \
    --resource-group securewebappresourcegroup \
    --location <location> \
    --sku Standard_ZRS \
    --encryption-services blob

storageId=$(az storage account show -n securewebappstorage -g securewebappresourcegroup --query id --out tsv)

az ad signed-in-user show --query objectId -o tsv | az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee @- \
    --scope $storageId

az storage container create \
    --account-name securewebappstorage \
    --name securewebappblobcontainer \
    --auth-mode login
```

---

## <a name="grant-access-to-the-storage-account"></a>Accorder l’accès au compte de stockage

Vous devez accorder à votre application web un accès au compte de stockage avant de pouvoir créer, lire ou supprimer des objets blob. Lors d’une étape précédente, vous avez configuré l’application web s’exécutant sur App Service avec une identité managée. À l’aide de RBAC Azure, vous pouvez accorder à l’identité managée l’accès à une autre ressource, comme n’importe quel principal de sécurité. Le rôle Contributeur aux données Blob du stockage accorde à l’application web (représentée par l’identité managée affectée par le système) un accès en lecture, écriture et suppression au conteneur d’objets blob et aux données.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Dans le [portail Azure](https://portal.azure.com), accédez à votre compte de stockage pour accorder l’accès à votre application web. Sélectionnez **Contrôle d’accès (IAM)** dans le volet gauche, puis **Attributions de rôle**. La liste des utilisateurs ayant accès au compte de stockage s’affiche. À présent, vous souhaitez ajouter une attribution de rôle à un bot, le service d’application qui a besoin d’accéder au compte de stockage. Sélectionnez **Ajouter** > **Ajouter une attribution de rôle**.

Dans **Rôle**, sélectionnez **Contributeur aux données Blob du stockage** pour permettre à votre application web d’accéder en lecture aux objets blob de stockage. Dans **Attribuer l’accès à**, sélectionnez **App Service**. Dans **Abonnement**, sélectionnez votre abonnement. Sélectionnez ensuite le service d’application auquel vous souhaitez accorder l’accès. Sélectionnez **Enregistrer**.

:::image type="content" alt-text="Capture d’écran montrant l’écran Ajouter une attribution de rôle." source="./media/scenario-secure-app-access-storage/add-role-assignment.png":::

Votre application web a maintenant accès à votre compte de stockage.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Exécutez le script suivant pour affecter à votre application web (représentée par une identité managée affectée par le système) le rôle Contributeur aux données Blob du stockage sur votre compte de stockage.

```powershell
$resourceGroup = "securewebappresourcegroup"
$webAppName="SecureWebApp20201102125811"
$storageName="securewebappstorage"

$spID = (Get-AzWebApp -ResourceGroupName $resourceGroup -Name $webAppName).identity.principalid
$storageId= (Get-AzStorageAccount -ResourceGroupName $resourceGroup -Name $storageName).Id
New-AzRoleAssignment -ObjectId $spID -RoleDefinitionName "Storage Blob Data Contributor" -Scope $storageId
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Exécutez le script suivant pour affecter à votre application web (représentée par une identité managée affectée par le système) le rôle Contributeur aux données Blob du stockage sur votre compte de stockage.

```azurecli-interactive
spID=$(az resource list -n SecureWebApp20201102125811 --query [*].identity.principalId --out tsv)

storageId=$(az storage account show -n securewebappstorage -g securewebappresourcegroup --query id --out tsv)

az role assignment create --assignee $spID --role 'Storage Blob Data Contributor' --scope $storageId
```

---

## <a name="access-blob-storage-net"></a>Accès au stockage d’objets blob (.NET)

La classe [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) est utilisée pour obtenir les informations d’identification d’un jeton pour votre code afin d’autoriser les requêtes d’accès à Stockage Azure. Créez une instance de la classe [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential), qui utilise l’identité managée pour extraire des jetons et les attacher au client de service. L’exemple de code suivant obtient les informations d’identification du jeton authentifiées, et les utilise pour créer un objet de client de service, qui charge un nouvel objet blob.

Pour voir ce code dans un exemple d’application, consultez l’[exemple sur GitHub](https://github.com/Azure-Samples/ms-identity-easyauth-dotnet-storage-graphapi/tree/main/1-WebApp-storage-managed-identity).

### <a name="install-client-library-packages"></a>Installer des packages de bibliothèque de client

Installez le [package NuGet de stockage d’objets blob](https://www.nuget.org/packages/Azure.Storage.Blobs/) pour utiliser le stockage d’objets blob et le [package NuGet de la bibliothèque cliente Azure Identity pour .NET](https://www.nuget.org/packages/Azure.Identity/) afin de vous authentifier avec des informations d’identification Azure AD. Installez les bibliothèques clientes à l’aide de l’interface de ligne de commande .NET Core ou de la console du gestionnaire de package dans Visual Studio.

# <a name="command-line"></a>[Ligne de commande](#tab/command-line)

Ouvrez une ligne de commande et basculez vers le répertoire qui contient votre fichier projet.

Exécutez les commandes d’installation.

```dotnetcli
dotnet add package Azure.Storage.Blobs

dotnet add package Azure.Identity
```

# <a name="package-manager"></a>[Gestionnaire de package](#tab/package-manager)

Ouvrez le projet ou la solution dans Visual Studio, puis ouvrez la console à l’aide de la commande **Outils** > **Gestionnaire de package NuGet** > **Console du gestionnaire de package**.

Exécutez les commandes d’installation.
```powershell
Install-Package Azure.Storage.Blobs

Install-Package Azure.Identity
```

---

### <a name="example"></a>Exemple

```csharp
using System;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Text;
using System.IO;
using Azure.Identity;

// Some code omitted for brevity.

static public async Task UploadBlob(string accountName, string containerName, string blobName, string blobContents)
{
    // Construct the blob container endpoint from the arguments.
    string containerEndpoint = string.Format("https://{0}.blob.core.windows.net/{1}",
                                                accountName,
                                                containerName);

    // Get a credential and create a client object for the blob container.
    BlobContainerClient containerClient = new BlobContainerClient(new Uri(containerEndpoint),
                                                                    new DefaultAzureCredential());

    try
    {
        // Create the container if it does not exist.
        await containerClient.CreateIfNotExistsAsync();

        // Upload text to a new block blob.
        byte[] byteArray = Encoding.ASCII.GetBytes(blobContents);

        using (MemoryStream stream = new MemoryStream(byteArray))
        {
            await containerClient.UploadBlobAsync(blobName, stream);
        }
    }
    catch (Exception e)
    {
        throw e;
    }
}
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez terminé ce tutoriel et que vous n’avez plus besoin de l’application web ni des ressources associées, [nettoyez les ressources que vous avez créées](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> * Créer une identité managée affectée par le système.
> * Créer un compte de stockage et un conteneur de stockage d’objets blob.
> * Accéder au stockage à partir d’une application web avec des identités managées.

> [!div class="nextstepaction"]
> [Accès d’un service d’application à Microsoft Graph pour le compte de l’utilisateur](scenario-secure-app-access-microsoft-graph-as-user.md)
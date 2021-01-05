---
title: Configurer l’accès en lecture public anonyme pour les conteneurs et les objets blob
titleSuffix: Azure Storage
description: Découvrez comment autoriser ou interdire l’accès anonyme aux données d’objet blob pour le compte de stockage. Définissez le paramètre accès public du conteneur pour rendre les conteneurs et les objets blob disponibles pour l’accès anonyme.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/03/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: blobs
ms.openlocfilehash: feac7b890c973b1541c5362f860432687082953f
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533874"
---
# <a name="configure-anonymous-public-read-access-for-containers-and-blobs"></a>Configurer l’accès en lecture public anonyme pour les conteneurs et les objets blob

Le Stockage Azure prend en charge l’accès en lecture public anonyme facultatif pour les conteneurs et les objets blob. Par défaut, l’accès anonyme à vos données n’est jamais autorisé. À moins que vous n’activiez explicitement l’accès anonyme, toutes les demandes adressées à un conteneur et à ses objets blob doivent être autorisées. Quand vous configurez le paramètre de niveau d’accès public d’un conteneur pour autoriser l’accès anonyme, les clients peuvent lire les données dans ce conteneur sans autoriser la requête.

> [!WARNING]
> Lorsqu’un conteneur est configuré pour un accès public, n’importe quel client peut lire les données de ce conteneur. L’accès public présente un risque de sécurité potentiel. Ainsi, si votre scénario ne l’exige pas, Microsoft vous recommande de l’interdire pour le compte de stockage. Pour en savoir plus, consultez la section [Configure anonymous public read access for containers and blobs](anonymous-read-access-prevent.md) (Empêcher l’accès en lecture publique anonyme aux conteneurs et aux objets blob).

Cet article explique comment configurer l’accès en lecture public anonyme pour un conteneur et ses objets blob. Pour plus d’informations sur l’accès anonyme aux données d’objet blob à partir d’une application cliente, consultez [Accéder à des conteneurs publics et à des objets blob anonymement avec .NET](anonymous-read-access-client.md).

## <a name="about-anonymous-public-read-access"></a>À propos de l’accès en lecture public anonyme

L’accès public à vos données est toujours interdit par défaut. Il existe deux paramètres distincts qui affectent l’accès public :

1. **Activer l’accès public pour le compte de stockage** Par défaut, un compte de stockage permet à un utilisateur disposant des autorisations appropriées d’activer un accès public à un conteneur. Les données blob ne sont pas disponibles pour un accès public, à moins que l’utilisateur n’effectue l’étape supplémentaire consistant à configurer explicitement le paramètre d’accès public du conteneur.
1. **Configurer le paramètre d’accès public du conteneur** Par défaut, le paramètre d’accès public d’un conteneur est désactivé, ce qui signifie que l’autorisation est nécessaire pour chaque demande adressée au conteneur ou à ses données. Un utilisateur disposant des autorisations appropriées peut modifier le paramètre d’accès public d’un conteneur afin d’activer l’accès anonyme uniquement si ce dernier est autorisé pour le compte de stockage.

Le tableau suivant résume l’incidence des deux paramètres sur l’accès public pour un conteneur.

| Paramètre d’accès public | L’accès public est désactivé pour un conteneur (paramètre par défaut) | L’accès public pour un conteneur est défini sur Conteneur | L’accès public pour un conteneur est défini sur Blob |
|--|--|--|--|
| L’accès public n’est pas autorisé pour le compte de stockage | Aucun accès public à un conteneur dans le compte de stockage. | Aucun accès public à un conteneur dans le compte de stockage. Le paramètre de compte de stockage remplace le paramètre de conteneur. | Aucun accès public à un conteneur dans le compte de stockage. Le paramètre de compte de stockage remplace le paramètre de conteneur. |
| L’accès public est autorisé pour le compte de stockage (paramètre par défaut) | Aucun accès public à ce conteneur (configuration par défaut). | L’accès public est autorisé à ce conteneur et à ses objets blob. | L’accès public est autorisé aux objets blob dans ce conteneur, mais pas au conteneur lui-même. |

## <a name="allow-or-disallow-public-read-access-for-a-storage-account"></a>Autoriser ou interdire l’accès en lecture public pour un compte de stockage

Par défaut, un compte de stockage est configuré pour permettre à un utilisateur disposant des autorisations appropriées d’activer un accès public à un conteneur. Quand l’accès public est autorisé, un utilisateur disposant des autorisations appropriées peut modifier le paramètre d’accès public d’un conteneur afin d’activer l’accès public anonyme aux données de ce dernier. Les données blob ne sont jamais disponibles pour un accès public, à moins que l’utilisateur n’effectue l’étape supplémentaire consistant à configurer explicitement le paramètre d’accès public du conteneur.

Gardez à l’esprit que l’accès public à un conteneur est toujours désactivé par défaut et doit être explicitement configuré pour autoriser les demandes anonymes. Quel que soit le paramétrage du compte de stockage, vos données ne sont jamais disponibles pour un accès public, sauf si un utilisateur disposant des autorisations appropriées effectue l’étape supplémentaire consistant à activer l’accès public sur le conteneur.

Le fait d’interdire l’accès public pour le compte de stockage empêche l’accès anonyme à tous les conteneurs et objets blob de ce compte. Quand l’accès public est interdit pour le compte, il est impossible de configurer le paramètre d’accès public pour un conteneur afin d’autoriser l’accès anonyme. Pour améliorer la sécurité, Microsoft vous recommande d’interdire l’accès public à vos comptes de stockage, sauf si votre scénario nécessite que les utilisateurs accèdent aux ressources d’objets blob de façon anonyme.

> [!IMPORTANT]
> L’interdiction de l’accès public pour un compte de stockage remplace les paramètres d’accès public pour tous les conteneurs appartenant à ce compte de stockage. Quand l’accès public est interdit pour le compte de stockage, toute requête anonyme ultérieure adressée à ce compte échoue. Avant de changer ce paramètre, assurez-vous de bien comprendre l’impact sur les applications clientes qui peuvent accéder aux données de votre compte de stockage de façon anonyme. Pour en savoir plus, consultez la section [Configure anonymous public read access for containers and blobs](anonymous-read-access-prevent.md) (Empêcher l’accès en lecture publique anonyme aux conteneurs et aux objets blob).

Pour autoriser ou interdire l’accès public pour un compte de stockage, configurez la propriété **AllowBlobPublicAccess** du compte. Cette propriété est disponible pour tous les comptes de stockage créés avec le modèle de déploiement Azure Resource Manager. Pour plus d’informations, consultez [Vue d’ensemble des comptes de stockage](../common/storage-account-overview.md).

La propriété **AllowBlobPublicAccess** n’est pas définie par défaut pour un compte de stockage et ne retourne pas de valeur tant que vous ne la définissez pas explicitement. Le compte de stockage autorise l’accès public quand la valeur de la propriété est **Null** ou **True**.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour autoriser ou interdire l’accès public à un compte de stockage dans le portail Azure, procédez comme suit :

1. Accédez à votre compte de stockage dans le portail Azure.
1. Localisez le paramètre **Configuration** sous **Paramètres**.
1. Définissez **Accès public aux objets blob** sur **Activé** ou **Désactivé**.

    :::image type="content" source="media/anonymous-read-access-configure/blob-public-access-portal.png" alt-text="Capture d’écran montrant comment autoriser ou interdire l’accès public aux objets blob pour le compte":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour autoriser ou interdire l’accès public pour un compte de stockage à l’aide de PowerShell, installez [Azure PowerShell version 4.4.0](https://www.powershellgallery.com/packages/Az/4.4.0) ou ultérieure. Ensuite, configurez la propriété **AllowBlobPublicAccess** pour un compte de stockage nouveau ou existant.

L’exemple suivant crée un compte de stockage et définit explicitement la propriété **AllowBlobPublicAccess** sur la valeur **true**. Il met ensuite à jour le compte de stockage pour définir la propriété **AllowBlobPublicAccess** sur **false**. L’exemple récupère également la valeur de la propriété dans chaque cas. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$location = "<location>"

# Create a storage account with AllowBlobPublicAccess set to true (or null).
New-AzStorageAccount -ResourceGroupName $rgName `
    -AccountName $accountName `
    -Location $location `
    -SkuName Standard_GRS
    -AllowBlobPublicAccess $false

# Read the AllowBlobPublicAccess property for the newly created storage account.
(Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName).AllowBlobPublicAccess

# Set AllowBlobPublicAccess set to false
Set-AzStorageAccount -ResourceGroupName $rgName `
    -AccountName $accountName `
    -AllowBlobPublicAccess $false

# Read the AllowBlobPublicAccess property.
(Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName).AllowBlobPublicAccess
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour autoriser ou interdire l’accès public pour un compte de stockage à l’aide d’Azure CLI, installez Azure CLI version 2.9.0 ou ultérieure. Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](/cli/azure/install-azure-cli). Ensuite, configurez la propriété **allowBlobPublicAccess** pour un compte de stockage nouveau ou existant.

L’exemple suivant crée un compte de stockage et définit explicitement la propriété **allowBlobPublicAccess** sur la valeur **true**. Il met ensuite à jour le compte de stockage pour définir la propriété **allowBlobPublicAccess** sur **false**. L’exemple récupère également la valeur de la propriété dans chaque cas. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --kind StorageV2 \
    --location <location> \
    --allow-blob-public-access true

az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query allowBlobPublicAccess \
    --output tsv

az storage account update \
    --name <storage-account> \
    --resource-group <resource-group> \
    --allow-blob-public-access false

az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query allowBlobPublicAccess \
    --output tsv
```

# <a name="template"></a>[Modèle](#tab/template)

Pour autoriser ou interdire l’accès public pour un compte de stockage à l’aide d’un modèle, créez un modèle avec la propriété **AllowBlobPublicAccess** définie sur **true** ou **false**. Les étapes suivantes montrent comment créer un modèle dans le Portail Azure.

1. Dans le Portail Azure, choisissez **Créer une ressource**.
1. Dans **Rechercher sur la Place de marché**, tapez **déploiement de modèle**, puis appuyez sur **Entrée**.
1. Choisissez **Template deployment (déployer avec des modèles personnalisés) [préversion]** , **Créer**, puis **Créer votre propre modèle dans l’éditeur**.
1. Dans l’éditeur de modèle, collez le code JSON suivant pour créer un nouveau compte et définissez la propriété **AllowBlobPublicAccess** sur **true** ou **false**. N’oubliez pas de remplacer les espaces réservés entre crochets par vos propres valeurs.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {
            "storageAccountName": "[concat(uniqueString(subscription().subscriptionId), 'template')]"
        },
        "resources": [
            {
            "name": "[variables('storageAccountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "location": "<location>",
            "properties": {
                "allowBlobPublicAccess": false
            },
            "dependsOn": [],
            "sku": {
              "name": "Standard_GRS"
            },
            "kind": "StorageV2",
            "tags": {}
            }
        ]
    }
    ```

1. Enregistrez le modèle.
1. Spécifiez un paramètre de groupe de ressources, puis choisissez le bouton **Vérifier + créer** pour déployer le modèle et créer un compte de stockage avec la propriété **allowBlobPublicAccess** configurée.

---

> [!NOTE]
> L’interdiction de l’accès public à un compte de stockage n’affecte aucun site web statique hébergé dans ce compte de stockage. Le conteneur **$web** est toujours accessible publiquement.
>
> Après la mise à jour du paramètre d’accès public pour le compte de stockage, la propagation de la modification peut prendre jusqu’à 30 secondes.

L’autorisation ou l’interdiction de l’accès public aux blobs nécessite la version 2019-04-01 ou ultérieure du fournisseur de ressources Stockage Azure. Pour plus d’informations, consultez [API REST du fournisseur de ressources Stockage Azure](/rest/api/storagerp/).

Les exemples de cette section ont montré comment lire la propriété **AllowBlobPublicAccess** pour le compte de stockage afin de déterminer si l’accès public est actuellement autorisé ou non. Pour en savoir plus sur la façon de vérifier que le paramètre d’accès public d’un compte est configuré de manière à empêcher l’accès anonyme, consultez [Corriger l’accès public anonyme](anonymous-read-access-prevent.md#remediate-anonymous-public-access).

## <a name="set-the-public-access-level-for-a-container"></a>Définir le niveau d’accès public pour un conteneur

Pour octroyer aux utilisateurs anonymes un accès en lecture à un conteneur et ses objets blob, commencez par autoriser l’accès public au compte de stockage, puis définissez le niveau d’accès public du conteneur. Si l’accès public est refusé pour le compte de stockage, vous ne pouvez pas créer un accès public pour un conteneur.

Quand l’accès public est autorisé pour un compte de stockage, vous pouvez configurer un conteneur avec les autorisations suivantes :

- **Aucun accès en lecture public :** le conteneur et ses objets blob ne sont accessibles qu’avec une requête autorisée. Cette option est la configuration par défaut de tous les nouveaux conteneurs.
- **Accès en lecture public pour les objets blob uniquement :** les objets blob à l’intérieur du conteneur peuvent être lus via une requête anonyme, mais les données du conteneur ne sont pas disponibles anonymement. Les clients anonymes ne peuvent pas énumérer les objets blob présents dans le conteneur.
- **Public read access for container and its blobs (Accès en lecture public pour le conteneur et ses objets blob) :** Les données de conteneur et d’objet blob peuvent être lues par une requête anonyme, à l’exception des paramètres d’autorisation et des métadonnées du conteneur. Les clients peuvent énumérer les objets blob présents dans le conteneur par une demande anonyme, mais ne peuvent pas énumérer les conteneurs présents dans le compte de stockage.

Vous ne pouvez pas modifier le niveau d’accès public pour un objet blob individuel. Le niveau d’accès public est défini uniquement au niveau du conteneur. Vous pouvez définir le niveau d’accès public du conteneur lors de la création du conteneur, ou vous pouvez mettre à jour le paramètre d’un conteneur existant.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour mettre à jour le niveau d’accès public pour un ou plusieurs conteneurs du portail Azure, procédez comme suit :

1. Accédez à la vue d’ensemble de votre compte de stockage dans le portail Azure.
1. Sous l’élément **service blob** du panneau de menu, sélectionnez **Conteneurs**.
1. Sélectionnez les conteneurs pour lesquels vous souhaitez définir le niveau d’accès public.
1. Utilisez le bouton **Modifier le niveau d’accès** pour afficher les paramètres d’accès public.
1. Sélectionnez le niveau d’accès public souhaité à partir du menu déroulant **Niveau d’accès public** et cliquez sur le bouton OK pour appliquer la modification aux conteneurs sélectionnés.

    ![Capture d’écran illustrant la façon de définir le niveau d’accès public dans le portail](./media/anonymous-read-access-configure/configure-public-access-container.png)

Quand l’accès public est interdit pour le compte de stockage, le niveau d’accès public d’un conteneur ne peut pas être défini. Si vous essayez de définir le niveau d’accès public du conteneur, vous verrez que le paramètre est désactivé, car l’accès public est interdit pour le compte.

:::image type="content" source="media/anonymous-read-access-configure/container-public-access-blocked.png" alt-text="Capture d’écran montrant que la définition du niveau d’accès public du conteneur est bloquée quand l’accès public est interdit":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour mettre à jour le niveau d’accès public d’un ou de plusieurs conteneurs à l’aide de PowerShell, appelez la commande [Set-AzStorageContainerAcl](/powershell/module/az.storage/set-azstoragecontaineracl). Autorisez cette opération en transmettant votre clé de compte, une chaîne de connexion ou une signature d’accès partagé (SAP). L’opération [Définir l’ACL du conteneur](/rest/api/storageservices/set-container-acl) qui définit le niveau d’accès public du conteneur ne prend pas en charge l’autorisation avec Azure AD. Pour plus d’informations, consultez [Autorisations pour l’appel d’opérations de données d’objet BLOB et de file d’attente](/rest/api/storageservices/authorize-with-azure-active-directory#permissions-for-calling-blob-and-queue-data-operations).

L’exemple suivant crée un conteneur dont l’accès public est désactivé, puis met à jour le paramètre d’accès public du conteneur pour autoriser l’accès anonyme au conteneur et à ses blobs. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
# Set variables.
$rgName = "<resource-group>"
$accountName = "<storage-account>"

# Get context object.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

# Create a new container with public access setting set to Off.
$containerName = "<container>"
New-AzStorageContainer -Name $containerName -Permission Off -Context $ctx

# Read the container's public access setting.
Get-AzStorageContainerAcl -Container $containerName -Context $ctx

# Update the container's public access setting to Container.
Set-AzStorageContainerAcl -Container $containerName -Permission Container -Context $ctx

# Read the container's public access setting.
Get-AzStorageContainerAcl -Container $containerName -Context $ctx
```

Quand l’accès public est interdit pour le compte de stockage, le niveau d’accès public d’un conteneur ne peut pas être défini. Si vous tentez de définir le niveau d’accès public du conteneur, Stockage Azure retourne une erreur indiquant que l’accès public n’est pas autorisé sur le compte de stockage.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour mettre à jour le niveau d’accès public d’un ou de plusieurs conteneurs avec Azure CLI, appelez la commande [az storage container set permission](/cli/azure/storage/container#az-storage-container-set-permission). Autorisez cette opération en transmettant votre clé de compte, une chaîne de connexion ou une signature d’accès partagé (SAP). L’opération [Définir l’ACL du conteneur](/rest/api/storageservices/set-container-acl) qui définit le niveau d’accès public du conteneur ne prend pas en charge l’autorisation avec Azure AD. Pour plus d’informations, consultez [Autorisations pour l’appel d’opérations de données d’objet BLOB et de file d’attente](/rest/api/storageservices/authorize-with-azure-active-directory#permissions-for-calling-blob-and-queue-data-operations).

L’exemple suivant crée un conteneur dont l’accès public est désactivé, puis met à jour le paramètre d’accès public du conteneur pour autoriser l’accès anonyme au conteneur et à ses blobs. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az storage container create \
    --name <container-name> \
    --account-name <account-name> \
    --resource-group <resource-group>
    --public-access off \
    --account-key <account-key> \
    --auth-mode key

az storage container show-permission \
    --name <container-name> \
    --account-name <account-name> \
    --account-key <account-key> \
    --auth-mode key

az storage container set-permission \
    --name <container-name> \
    --account-name <account-name> \
    --public-access container \
    --account-key <account-key> \
    --auth-mode key

az storage container show-permission \
    --name <container-name> \
    --account-name <account-name> \
    --account-key <account-key> \
    --auth-mode key
```

Quand l’accès public est interdit pour le compte de stockage, le niveau d’accès public d’un conteneur ne peut pas être défini. Si vous tentez de définir le niveau d’accès public du conteneur, Stockage Azure retourne une erreur indiquant que l’accès public n’est pas autorisé sur le compte de stockage.

# <a name="template"></a>[Modèle](#tab/template)

N/A.

---

## <a name="check-the-public-access-setting-for-a-set-of-containers"></a>Vérifier le paramètre d’accès public d’un ensemble de conteneurs

Il est possible de vérifier quels conteneurs d’un ou de plusieurs comptes de stockage sont configurés pour l’accès public en répertoriant les conteneurs et en vérifiant le paramètre d’accès public. Cette approche est pratique lorsqu’un compte de stockage ne contient pas un grand nombre de conteneurs, ou lorsque vous vérifiez le paramètre sur un petit nombre de comptes de stockage. Toutefois, les performances peuvent être dégradées si vous tentez d’énumérer un grand nombre de conteneurs.

L’exemple suivant utilise PowerShell pour obtenir le paramètre d’accès public de tous les conteneurs d’un compte de stockage. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"

$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

Get-AzStorageContainer -Context $ctx | Select Name, PublicAccess
```

## <a name="next-steps"></a>Étapes suivantes

- [Empêcher l’accès en lecture public anonyme aux conteneurs et aux objets blob](anonymous-read-access-prevent.md)
- [Accéder anonymement aux conteneurs et objets blob publics avec .NET](anonymous-read-access-client.md)
- [Autorisation d’accès au Stockage Azure](../common/storage-auth.md)

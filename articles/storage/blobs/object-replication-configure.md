---
title: Configurer la réplication d’objets
titleSuffix: Azure Storage
description: Découvrez comment configurer la réplication d’objets pour copier de façon asynchrone des objets blob de blocs du conteneur d’un compte de stockage vers un autre.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/09/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: e3503a9eef5c11db35684ca61fb1ee39525a465d
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94427596"
---
# <a name="configure-object-replication-for-block-blobs"></a>Configurer la réplication d’objets pour des objets blob de blocs

La réplication d’objets copie de façon asynchrone des objets blob de blocs entre un compte de stockage source et un compte de destination. Pour en savoir plus sur le sujet, consultez [Réplication d’objets](object-replication-overview.md).

Lorsque vous configurez la réplication d’objets, vous créez une stratégie de réplication qui spécifie le compte de stockage source et le compte de destination. Une stratégie de réplication comprend une ou plusieurs règles qui spécifient un conteneur source et un conteneur de destination et indiquent quels objets blob de blocs du conteneur source seront répliqués.

Cet article explique comment configurer la réplication d’objets pour votre compte de stockage à l’aide de Portail Azure, de PowerShell ou d’Azure CLI. Vous pouvez également utiliser l’une des bibliothèques clientes du fournisseur de ressources Stockage Azure pour configurer la réplication d’objets.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="create-a-replication-policy-and-rules"></a>Créer une stratégie et des règles de réplication

Avant de configurer la réplication d’objets, créez les comptes de stockage source et de destination, s’ils n’existent pas déjà. Les deux comptes doivent être des comptes de stockage v2 universels. Pour plus d’informations, consultez la rubrique [Créer un compte Stockage Azure](../common/storage-account-create.md).

La réplication d’objets requiert que le contrôle de version des blobs soit activé pour le compte source et le compte de destination, et que le flux de modification de blob soit activé pour le compte source. Pour en savoir plus sur le contrôle de version des blobs, consultez [Contrôle de version des objets blob](versioning-overview.md). Pour en savoir plus sur le flux de modification, consultez [Prise en charge du flux de modification dans Stockage Blob Azure](storage-blob-change-feed.md). Gardez à l’esprit que l’activation de ces fonctionnalités peut occasionner des coûts supplémentaires.

Un compte de stockage peut servir de compte source pour un maximum de deux comptes de destination. Les comptes source et de destination peuvent se trouver dans la même région ou dans des régions différentes. Ils peuvent également résider dans différents abonnements et locataires Azure Active Directory (Azure AD). Vous pouvez créer une seule stratégie de réplication pour chaque paire de comptes.

Lorsque vous configurez une réplication d’objet, vous créez une stratégie de réplication sur le compte de destination via le fournisseur de ressources de Stockage Azure. Une fois la stratégie de réplication créée, le service Stockage Azure lui attribue un ID de stratégie. Vous devez ensuite associer cette stratégie de réplication au compte source à l’aide de l’ID de stratégie. Pour que la réplication ait lieu, l’ID de stratégie doit être le même sur les comptes source et de destination.

Pour configurer une stratégie de réplication d’objet pour un compte de stockage, vous devez être titulaire du rôle **Contributeur** Azure Resource Manager, étendu au niveau du compte de stockage ou à un niveau supérieur. Pour plus d’informations, consultez [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md) dans la documentation Contrôle d’accès en fonction du rôle Azure (Azure RBAC).

### <a name="configure-object-replication-when-you-have-access-to-both-storage-accounts"></a>Configurer la réplication d’objets lorsque vous avez accès aux deux comptes de stockage

Si vous avez accès aux comptes de stockage source et de destination, vous pouvez configurer la stratégie de réplication d’objet sur les deux comptes.

Avant de configurer la réplication d’objets dans le portail Azure, créez les conteneurs source et de destination dans leurs comptes de stockage respectifs, s’ils n’existent pas déjà. Activez également le contrôle de version des blobs et le flux de modification sur le compte source, et le contrôle de version des blobs sur le compte de destination.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Le portail Azure crée automatiquement la stratégie sur le compte source une fois que vous l’avez configurée pour le compte de destination.

Pour créer une stratégie de réplication dans le portail Azure, procédez comme suit :

1. Accédez au compte de stockage source dans le portail Azure.
1. Sous **Service BLOB**, sélectionnez **Réplication d’objet**.
1. Sélectionnez **Configurer des règles de réplication**.
1. Sélectionnez le compte de stockage et l’abonnement de destination.
1. Dans la section **Paires de conteneurs**, sélectionnez un conteneur source dans le compte source et un conteneur de destination dans le compte de destination. Vous pouvez créer jusqu’à 10 paires de conteneurs par stratégie de réplication.

    L’illustration suivante montre un ensemble de règles de réplication.

    :::image type="content" source="media/object-replication-configure/configure-replication-policy.png" alt-text="Capture d’écran montrant des règles de réplication dans Portail Azure":::

1. Si vous le souhaitez, spécifiez un ou plusieurs filtres pour copier uniquement les blobs qui correspondent à un modèle de préfixe. Par exemple, si vous spécifiez le préfixe `b`, seuls les blobs dont le nom commence par cette lettre sont répliqués. Vous pouvez spécifier un répertoire virtuel comme élément du préfixe. La chaîne de préfixe ne prend pas en charge les caractères génériques.

    L’illustration suivante montre des filtres qui limitent les blobs qui sont copiés dans le cadre d’une règle de réplication.

    :::image type="content" source="media/object-replication-configure/configure-replication-copy-prefix.png" alt-text="Capture d’écran montrant des filtres pour une règle de réplication":::

1. Par défaut, l’étendue de copie est définie pour copier uniquement les nouveaux objets. Pour copier tous les objets dans le conteneur ou pour copier les objets à partir d’une date et d’une heure personnalisées, sélectionnez le lien **Modifier** et configurez l’étendue de copie pour la paire de conteneurs.

    L’illustration suivante montre une étendue de copie personnalisée qui copie les objets à partir d’une date et d’une heure spécifiées.

    :::image type="content" source="media/object-replication-configure/configure-replication-copy-scope.png" alt-text="Capture d’écran montrant l’étendue de copie personnalisée pour la réplication d’objets":::

1. Sélectionnez **Enregistrer et appliquer** pour créer la stratégie de réplication et démarrer la réplication des données.

Une fois que vous avez configuré la réplication d’objet, le portail Azure affiche la stratégie et les règles de réplication, comme illustré dans l’image suivante.

:::image type="content" source="media/object-replication-configure/object-replication-policies-portal.png" alt-text="Capture d’écran montrant la stratégie de réplication d’objet dans le portail Azure":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour créer une stratégie de réplication à l’aide de PowerShell, commencez par installer la version [2.5.0](https://www.powershellgallery.com/packages/Az.Storage/2.5.0) ou une version ultérieure du module PowerShell Az.Storage. Pour en savoir plus sur l’installation d’Azure PowerShell, voir [Installer Azure PowerShell avec PowerShellGet](/powershell/azure/install-az-ps).

L’exemple suivant montre comment créer une stratégie de réplication sur les comptes source et de destination. N’oubliez pas de remplacer les valeurs entre crochets par vos propres valeurs :

```powershell
# Sign in to your Azure account.
Connect-AzAccount

# Set variables.
$rgName = "<resource-group>"
$srcAccountName = "<source-storage-account>"
$destAccountName = "<destination-storage-account>"
$srcContainerName1 = "source-container1"
$destContainerName1 = "dest-container1"
$srcContainerName2 = "source-container2"
$destContainerName2 = "dest-container2"

# Enable blob versioning and change feed on the source account.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -EnableChangeFeed $true `
    -IsVersioningEnabled $true

# Enable blob versioning on the destination account.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName `
    -IsVersioningEnabled $true

# List the service properties for both accounts.
Get-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName
Get-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName

# Create containers in the source and destination accounts.
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $srcAccountName |
    New-AzStorageContainer $srcContainerName1
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $destAccountName |
    New-AzStorageContainer $destContainerName1
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $srcAccountName |
    New-AzStorageContainer $srcContainerName2
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $destAccountName |
    New-AzStorageContainer $destContainerName2

# Define replication rules for each container.
$rule1 = New-AzStorageObjectReplicationPolicyRule -SourceContainer $srcContainerName1 `
    -DestinationContainer $destContainerName1 `
    -PrefixMatch b
$rule2 = New-AzStorageObjectReplicationPolicyRule -SourceContainer $srcContainerName2 `
    -DestinationContainer $destContainerName2  `
    -MinCreationTime 2020-05-10T00:00:00Z

# Create the replication policy on the destination account.
$destPolicy = Set-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName `
    -PolicyId default `
    -SourceAccount $srcAccountName `
    -Rule $rule1,$rule2

# Create the same policy on the source account.
Set-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -InputObject $destPolicy
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour créer une stratégie de réplication avec Azure CLI, commencez par installer Azure CLI version 2.11.1 ou ultérieure. Pour plus d’informations, consultez [Prise en main d’Azure CLI](/cli/azure/get-started-with-azure-cli).

Ensuite, activez le contrôle de version des blobs sur les comptes de stockage source et de destination, puis activez le flux de modification sur le compte source en appelant la commande [az storage account blob-service-properties update](/cli/azure/storage/account/blob-service-properties#az_storage_account_blob_service_properties_update). N’oubliez pas de remplacer les valeurs entre crochets par vos propres valeurs :

```azurecli
az login

az storage account blob-service-properties update \
    --resource-group <resource-group> \
    --account-name <source-storage-account> \
    --enable-versioning

az storage account blob-service-properties update \
    --resource-group <resource-group> \
    --account-name <source-storage-account> \
    --enable-change-feed

az storage account blob-service-properties update \
    --resource-group <resource-group> \
    --account-name <dest-storage-account> \
    --enable-versioning
```

Créez les conteneurs source et de destination dans leurs comptes de stockage respectifs.

```azurecli
az storage container create \
    --account-name <source-storage-account> \
    --name source-container-1 \
    --auth-mode login
az storage container create \
    --account-name <source-storage-account> \
    --name source-container-2 \
    --auth-mode login

az storage container create \
    --account-name <dest-storage-account> \
    --name dest-container-1 \
    --auth-mode login
az storage container create \
    --account-name <dest-storage-account> \
    --name dest-container-1 \
    --auth-mode login
```

Créez une stratégie de réplication et une règle associée sur le compte de destination en appelant la commande [az storage account or-policy create](/cli/azure/storage/account/or-policy#az_storage_account_or_policy_create).

```azurecli
az storage account or-policy create \
    --account-name <dest-storage-account> \
    --resource-group <resource-group> \
    --source-account <source-storage-account> \
    --destination-account <dest-storage-account> \
    --source-container source-container-1 \
    --destination-container dest-container-1 \
    --min-creation-time '2020-09-10T00:00:00Z' \
    --prefix-match a

```

Le service Stockage Azure définit l’ID de stratégie de la nouvelle stratégie lors de sa création. Pour ajouter des règles à la stratégie, appelez la commande [az storage account or-policy rule add](/cli/azure/storage/account/or-policy/rule#az_storage_account_or_policy_rule_add) et fournissez l’ID de stratégie.

```azurecli
az storage account or-policy rule add \
    --account-name <dest-storage-account> \
    --resource-group <resource-group> \
    --source-container source-container-2 \
    --destination-container dest-container-2 \
    --policy-id <policy-id> \
    --prefix-match b
```

Ensuite, créez la stratégie sur le compte source à l’aide de l’ID de stratégie.

```azurecli
az storage account or-policy show \
    --resource-group <resource-group> \
    --account-name <dest-storage-account> \
    --policy-id <policy-id> |
    az storage account or-policy create --resource-group <resource-group> \
    --account-name <source-storage-account> \
    --policy "@-"
```

---

### <a name="configure-object-replication-when-you-have-access-only-to-the-destination-account"></a>Configurer la réplication d’objets lorsque vous avez accès uniquement au compte de destination

Si vous n’avez pas d’autorisation sur le compte de stockage source, vous pouvez configurer la réplication d’objets sur le compte de destination, et fournir un fichier JSON contenant la définition de stratégie à un autre utilisateur pour créer la même stratégie sur le compte source. Par exemple, si le compte source se trouve dans un locataire Azure AD différent de celui du compte de destination, vous pouvez adopter cette approche pour configurer la réplication d’objets.

Gardez à l’esprit que, pour créer la stratégie, vous devez être titulaire du rôle **Contributeur** Azure Resource Manager étendu au niveau du compte de stockage de destination ou à un niveau supérieur. Pour plus d’informations, consultez [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md) dans la documentation Contrôle d’accès en fonction du rôle Azure (Azure RBAC).

Le tableau suivant récapitule les valeurs à utiliser pour l’ID de stratégie et les ID de règle dans le fichier JSON de chaque scénario.

| Lorsque vous créez le fichier JSON pour ce compte... | Définissez l’ID de stratégie et les ID de règle sur cette valeur... |
|-|-|
| Compte de destination | Valeur de chaîne *par défaut*. Le service Stockage Azure créera l’ID de stratégie et les ID de règle pour vous. |
| Compte source | Valeurs de l’ID de stratégie et des ID de règle renvoyés lorsque vous téléchargez la stratégie définie sur le compte de destination en tant que fichier JSON. |

L’exemple suivant définit une stratégie de réplication sur le compte de destination avec une règle unique correspondant au préfixe *b*, et définit l’heure de création minimale des objets blob à répliquer. N’oubliez pas de remplacer les valeurs entre crochets par vos propres valeurs :

```json
{
  "properties": {
    "policyId": "default",
    "sourceAccount": "<source-account>",
    "destinationAccount": "<dest-account>",
    "rules": [
      {
        "ruleId": "default",
        "sourceContainer": "<source-container>",
        "destinationContainer": "<destination-container>",
        "filters": {
          "prefixMatch": [
            "b"
          ],
          "minCreationTime": "2020-08-028T00:00:00Z"
        }
      }
    ]
  }
}
```

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour configurer la réplication d’objets sur le compte de destination avec un fichier JSON dans le portail Azure, procédez comme suit :

1. Créez un fichier JSON local définissant la stratégie de réplication sur le compte de destination. Définissez le champ **policyId** sur **par défaut** afin que le service Stockage Azure définisse l’ID de stratégie.

    Un moyen simple de créer un fichier JSON qui définit une stratégie de réplication consiste à commencer par créer une stratégie de réplication test entre deux comptes de stockage dans le portail Azure. Vous pouvez ensuite télécharger les règles de réplication et modifier le fichier JSON en fonction des besoins.

1. Accédez aux paramètres de **Réplication d’objets** pour le compte de destination dans le portail Azure.
1. Sélectionnez **Charger les règles de réplication**.
1. Téléchargez le fichier JSON. Le portail Azure affiche la stratégie et les règles qui seront créées, comme illustré dans l’image suivante.

    :::image type="content" source="media/object-replication-configure/replication-rules-upload-portal.png" alt-text="Capture d’écran montrant comment charger un fichier JSON pour définir une stratégie de réplication":::

1. Sélectionnez **Télécharger** pour créer la stratégie de réplication sur le compte de destination.

Vous pouvez ensuite télécharger un fichier JSON contenant la définition de stratégie que vous pouvez fournir à un autre utilisateur afin de configurer le compte source. Pour télécharger ce fichier JSON, procédez comme suit :

1. Accédez aux paramètres de **Réplication d’objets** pour le compte de destination dans le portail Azure.
1. Sélectionnez le bouton **Plus** en regard de la stratégie que vous souhaitez télécharger, puis sélectionnez **Télécharger les règles**, comme illustré dans l’image suivante.

    :::image type="content" source="media/object-replication-configure/replication-rules-download-portal.png" alt-text="Capture d’écran montrant comment télécharger des règles de réplication dans un fichier JSON":::

1. Enregistrez le fichier JSON sur votre ordinateur local pour le partager avec un autre utilisateur afin de configurer la stratégie sur le compte source.

Le fichier JSON téléchargé contient l’ID de stratégie que le service Stockage Azure a créé pour la stratégie sur le compte de destination. Vous devez utiliser le même ID de stratégie pour configurer la réplication d’objets sur le compte source.

Gardez à l’esprit que le chargement d’un fichier JSON pour créer une stratégie de réplication pour le compte de destination via le portail Azure n’a pas pour effet de créer automatiquement la même stratégie dans le compte source. Un autre utilisateur doit créer la stratégie sur le compte source avant que le service Stockage Azure commence à répliquer des objets.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour télécharger un fichier JSON contenant la définition de stratégie de réplication pour le compte de destination à partir de PowerShell, appelez la commande [Get-AzStorageObjectReplicationPolicy](/powershell/module/az.storage/get-azstorageobjectreplicationpolicy) pour retourner la stratégie. Convertissez ensuite la stratégie en JSON, et enregistrez-la en tant que fichier local, comme illustré dans l’exemple suivant. N’oubliez pas de remplacer les valeurs entre crochets dans le chemin d’accès du fichier par vos propres valeurs :

```powershell
$rgName = "<resource-group>"
$destAccountName = "<destination-storage-account>"

$destPolicy = Get-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName
$destPolicy | ConvertTo-Json -Depth 5 > c:\temp\json.txt
```

Pour utiliser le fichier JSON afin de configurer la stratégie de réplication sur le compte source avec PowerShell, récupérez le fichier local et convertissez le fichier JSON en objet. Appelez ensuite la commande [Set-AzStorageObjectReplicationPolicy](/powershell/module/az.storage/set-azstorageobjectreplicationpolicy) pour configurer la stratégie sur le compte source, comme indiqué dans l’exemple suivant. N’oubliez pas de remplacer les valeurs entre crochets dans le chemin d’accès du fichier par vos propres valeurs :

```powershell
$object = Get-Content -Path C:\temp\json.txt | ConvertFrom-Json
Set-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -PolicyId $object.PolicyId `
    -SourceAccount $object.SourceAccount `
    -DestinationAccount $object.DestinationAccount `
    -Rule $object.Rules
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour écrire la définition de stratégie de réplication pour le compte de destination dans un fichier JSON à partir d’Azure CLI, appelez la commande [az storage account or-policy show](/cli/azure/storage/account/or-policy#az_storage_account_or_policy_show) et canalisez la sortie vers un fichier.

L’exemple suivant écrit la définition de stratégie dans un fichier JSON nommé *policy.json*. N’oubliez pas de remplacer les valeurs entre crochets dans le chemin d’accès du fichier par vos propres valeurs :

```azurecli
az storage account or-policy show \
    --account-name <dest-account-name> \
    --policy-id  <policy-id> > policy.json
```

Pour utiliser le fichier JSON afin de configurer la stratégie de réplication sur le compte source avec Azure CLI, appelez la commande [az storage account or-policy create](/cli/azure/storage/account/or-policy#az_storage_account_or_policy_create) et référencez le fichier *policy.json*. N’oubliez pas de remplacer les valeurs entre crochets dans le chemin d’accès du fichier par vos propres valeurs :

```azurecli
az storage account or-policy create \
    -resource-group <resource-group> \
    --source-account <source-account-name> \
    --policy @policy.json
```

---

## <a name="check-the-replication-status-of-a-blob"></a>Vérifier l’état de réplication d’un objet blob

Vous pouvez vérifier l’état de réplication d’un objet blob dans le compte source à l’aide du portail Azure, de PowerShell ou d’Azure CLI. Les propriétés de la réplication d’objet ne sont remplies qu’une fois la réplication terminée, qu’elle réussisse ou échoue.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour vérifier l’état de réplication d’un objet blob dans le compte source à l’aide du portail Azure, procédez comme suit :

1. Accédez au compte source dans le portail Azure.
1. Localisez le conteneur qui contient l’objet blob source.
1. Sélectionnez l’objet blob pour afficher ses propriétés. Si l’objet blob a été correctement répliqué, vous verrez dans la section **Réplication d’objet** que l’état est défini sur *Terminé*. L’ID de la stratégie de réplication et l’ID de la règle régissant la réplication d’objets pour ce conteneur sont également répertoriés.

:::image type="content" source="media/object-replication-configure/check-replication-status-source.png" alt-text="Capture d’écran illustrant l’état de réplication d’un objet blob dans le compte source":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vérifier l’état de réplication d’un objet blob dans le compte source à l’aide de PowerShell, récupérez la valeur de la propriété **ReplicationStatus** de la réplication d’objets, comme indiqué dans l’exemple suivant. N’oubliez pas de remplacer les valeurs entre crochets par vos propres valeurs :

```powershell
$ctxSrc = (Get-AzStorageAccount -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName).Context
$blobSrc = Get-AzStorageBlob -Container $srcContainerName1 `
    -Context $ctxSrc `
    -Blob <blob-name>
$blobSrc.BlobProperties.ObjectReplicationSourceProperties[0].Rules[0].ReplicationStatus
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vérifier l’état de réplication d’un objet blob dans le compte source à l’aide d’Azure CLI, récupérez la valeur de la propriété **status** de la réplication d’objets, comme indiqué dans l’exemple suivant :

```azurecli
az storage blob show \
    --account-name <source-account-name> \
    --container-name <source-container-name> \
    --name <source-blob-name> \
    --query 'objectReplicationSourceProperties[].rules[].status' \
    --output tsv \
    --auth-mode login
```

---

Si l’état de réplication d’un blob dans le compte source indique un échec, examinez les causes possibles suivantes :

- Assurez-vous que la stratégie de réplication d’objet est configurée sur le compte de destination.
- Vérifiez que le conteneur de destination existe toujours.
- Si le blob source a été chiffré à l’aide d’une clé fournie par le client dans le cadre d’une opération d’écriture, la réplication d’objet échoue. Pour plus d’informations sur les clés fournies par le client, consultez [Fournir une clé de chiffrement lors d’une requête au stockage d’objets blob](encryption-customer-provided-keys.md).

## <a name="remove-a-replication-policy"></a>Supprimer une stratégie de réplication

Pour supprimer une stratégie de réplication et ses règles associées, utilisez Portail Azure, PowerShell ou CLI.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour supprimer une stratégie de réplication dans le portail Azure, procédez comme suit :

1. Accédez au compte de stockage source dans le portail Azure.
1. Sous **Paramètres**, sélectionnez **Réplication d’objet**.
1. Cliquez sur le bouton **Plus** à côté du nom de la stratégie.
1. Sélectionnez **Supprimer les règles**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour supprimer une stratégie de réplication, supprimez la stratégie du compte source et du compte de destination. La suppression de la stratégie supprime également toutes les règles qui lui sont associées.

```powershell
# Remove the policy from the destination account.
Remove-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName `
    -PolicyId $destPolicy.PolicyId

# Remove the policy from the source account.
Remove-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -PolicyId $destPolicy.PolicyId
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour supprimer une stratégie de réplication, supprimez la stratégie du compte source et du compte de destination. La suppression de la stratégie supprime également toutes les règles qui lui sont associées.

```azurecli
az storage account or-policy delete \
    --policy-id <policy-id> \
    --account-name <source-storage-account> \
    --resource-group <resource-group>

az storage account or-policy delete \
    --policy-id <policy-id> \
    --account-name <dest-storage-account> \
    --resource-group <resource-group>
```

---

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble de la réplication d’objets](object-replication-overview.md)
- [Activer et gérer le contrôle de version des objets blob](versioning-enable.md)
- [Flux de modification dans Stockage Blob Azure](storage-blob-change-feed-how-to.md)

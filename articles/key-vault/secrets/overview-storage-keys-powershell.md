---
title: Compte de stockage managé Azure Key Vault - version PowerShell
description: La fonctionnalité de compte de stockage managé assure une intégration fluide entre Azure Key Vault et un compte de stockage Azure.
ms.topic: tutorial
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
manager: rkarlin
ms.date: 09/10/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3bced101516e91259ea9018fe3c4aa44f867cbe6
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96023106"
---
# <a name="manage-storage-account-keys-with-key-vault-and-azure-powershell"></a>Gérer les clés de compte de stockage avec Key Vault et Azure PowerShell
> [!IMPORTANT]
> Nous vous recommandons d’utiliser l’intégration de Stockage Azure avec Azure Active Directory (Azure AD), le service Microsoft basé sur le cloud qui gère les identités et les accès. L’intégration d’Azure AD est disponible pour les [objets blob et les files d’attente Azure](../../storage/common/storage-auth-aad.md) et fournit un accès basé sur les jetons OAuth2 au Stockage Azure (comme Azure Key Vault).
> Azure AD vous permet d’authentifier votre application cliente en utilisant une identité d’application ou d’utilisateur plutôt que les informations d’identification du compte de stockage. Vous pouvez utiliser une [identité Azure AD managée](../../active-directory/managed-identities-azure-resources/index.yml) lors de l’exécution sur Azure. Les identités managées suppriment l’authentification du client ainsi que le stockage des informations d’identification dans ou avec votre application. Utilisez la solution ci-dessous uniquement lorsque l’authentification Azure AD n’est pas possible.

Un compte de stockage Azure utilise des informations d’identification comprenant un nom de compte et une clé. La clé qui est générée automatiquement sert de mot de passe et non pas de clé de chiffrement. Key Vault gère les clés de compte de stockage en les regénérant régulièrement dans le compte de stockage. De plus, il fournit des jetons de signature d’accès partagé pour permettre un accès délégué aux ressources de votre compte de stockage.

Vous pouvez utiliser la fonctionnalité de clé de compte de stockage gérée de Key Vault pour lister (synchroniser) les clés avec un compte de stockage Azure et regénérer (faire tourner) régulièrement les clés. Vous pouvez gérer les clés des comptes de stockage et des comptes de stockage classiques.

Lorsque vous utilisez la fonctionnalité de clé de compte de stockage managé, tenez compte des points suivants :

- Les valeurs de clés ne sont jamais retournées en réponse à l’appelant.
- Seul Key Vault doit gérer vos clés de compte de stockage. Ne gérez pas les clés vous-même et évitez d’interférer avec les processus de Key Vault.
- Un seul objet Key Vault doit gérer les clés de compte de stockage. Vous ne devez pas autoriser la gestion des clients par des objets multiples.
- Régénérez les clés à l’aide de Key Vault uniquement. Ne régénérez pas manuellement vos clés de compte de stockage.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="service-principal-application-id"></a>ID d’application du principal de service

Un locataire Azure AD fournit à chaque application inscrite un [principal de service](../../active-directory/develop/developer-glossary.md#service-principal-object), Le principal de service fait office d’ID d’application, qui est utilisé lors de la configuration de l’autorisation pour l’accès aux autres ressources Azure à l’aide du contrôle d’accès en fonction du rôle (RBAC) Azure.

Key Vault est une application Microsoft préinscrite dans tous les locataires Azure AD. Key Vault est inscrit sous le même ID d’application dans chaque cloud Azure.

| Locataires | Cloud | ID de l'application |
| --- | --- | --- |
| Azure AD | Azure Government | `7e7c393b-45d0-48b1-a35e-2905ddf8183c` |
| Azure AD | Azure public | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |
| Autres  | Quelconque | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide, vous devez d’abord effectuer ce qui suit :

- [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).
- [Création d’un coffre de clés](quick-create-powershell.md)
- [Création d’un compte de stockage Azure](../../storage/common/storage-account-create.md?tabs=azure-powershell). Le nom du compte de stockage doit contenir uniquement des lettres minuscules et des chiffres. Le nom doit compter 3 à 24 caractères.


## <a name="manage-storage-account-keys"></a>Gérer les clés de compte de stockage

### <a name="connect-to-your-azure-account"></a>Se connecter au compte Azure

Authentifiez votre session PowerShell à l’aide de l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

```azurepowershell-interactive
Connect-AzAccount
```
Si vous avez plusieurs abonnements Azure, vous pouvez les lister à l’aide de l’applet de commande [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription), puis spécifier l’abonnement que vous souhaitez utiliser avec l’applet de commande [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId <subscriptionId>
```

### <a name="set-variables"></a>Définition des variables

Commencez par définir les variables utilisées par les cmdlets PowerShell dans les étapes suivantes. Veillez à mettre à jour les espaces réservés « YourResourceGroupName », « YourStorageAccountName » et « YourKeyVaultName », puis définissez $keyVaultSpAppId avec `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` (comme indiqué dans [ID d’application du principal de service](#service-principal-application-id) ci-dessus).

Nous utiliserons également les cmdlets Azure PowerShell [Get-AzContext](/powershell/module/az.accounts/get-azcontext) et [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount) pour récupérer votre ID d’utilisateur et le contexte de votre compte de stockage Azure.

```azurepowershell-interactive
$resourceGroupName = <YourResourceGroupName>
$storageAccountName = <YourStorageAccountName>
$keyVaultName = <YourKeyVaultName>
$keyVaultSpAppId = "cfa8b339-82a2-471a-a3c9-0fc0be7a4093"
$storageAccountKey = "key1" #(key1 or key2 are allowed)

# Get your User Id
$userId = (Get-AzContext).Account.Id

# Get a reference to your Azure storage account
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName

```
>[!Note]
> Pour le compte de stockage classique, utilisez « primary » et « secondary » pour $storageAccountKey <br>
> Utilisez 'Get-AzResource -Name "ClassicStorageAccountName" -ResourceGroupName $resourceGroupName' au lieu de 'Get-AzStorageAccount' pour le compte de stockage classique

### <a name="give-key-vault-access-to-your-storage-account"></a>Octroyer un accès Key Vault à votre compte de stockage

Pour permettre à Key Vault d’accéder et de gérer vos clés de compte de stockage, vous devez l’autoriser à accéder à votre compte de stockage. L’application Key Vault a besoin d’autorisations afin de *répertorier* et de *regénérer* des clés pour votre compte de stockage. Ces autorisations sont activées par le biais du rôle intégré [Rôle de service d’opérateur de clé de compte de stockage](../../role-based-access-control/built-in-roles.md#storage-account-key-operator-service-role).

Attribuez ce rôle au principal de service Key Vault, en limitant la portée à votre compte de stockage, à l’aide de la cmdlet [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) Azure PowerShell.

```azurepowershell-interactive
# Assign Azure role "Storage Account Key Operator Service Role" to Key Vault, limiting the access scope to your storage account. For a classic storage account, use "Classic Storage Account Key Operator Service Role."
New-AzRoleAssignment -ApplicationId $keyVaultSpAppId -RoleDefinitionName 'Storage Account Key Operator Service Role' -Scope $storageAccount.Id
```

Si l’attribution du rôle a réussi, le résultat devrait ressembler à l’exemple suivant :

```console
RoleAssignmentId   : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso/providers/Microsoft.Authorization/roleAssignments/189cblll-12fb-406e-8699-4eef8b2b9ecz
Scope              : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
DisplayName        : Azure Key Vault
SignInName         :
RoleDefinitionName : storage account Key Operator Service Role
RoleDefinitionId   : 81a9662b-bebf-436f-a333-f67b29880f12
ObjectId           : 93c27d83-f79b-4cb2-8dd4-4aa716542e74
ObjectType         : ServicePrincipal
CanDelegate        : False
```

Si Key Vault a déjà été ajouté au rôle sur votre compte de stockage, vous recevrez un message d’erreur *« L'attribution de rôle existe déjà »* . erreur. Vous pouvez également vérifier l’attribution du rôle à la page « Contrôle d’accès (IAM) » du compte de stockage sur le portail Azure.

### <a name="give-your-user-account-permission-to-managed-storage-accounts"></a>Autoriser votre compte d’utilisateur à accéder à des comptes de stockage managés

Utilisez la cmdlet Azure PowerShell [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) pour mettre à jour la stratégie d’accès au coffre de clés et accorder des autorisations de compte de stockage à votre compte d’utilisateur.

```azurepowershell-interactive
# Give your user principal access to all storage account permissions, on your Key Vault instance

Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -UserPrincipalName $userId -PermissionsToStorage get, list, delete, set, update, regeneratekey, getsas, listsas, deletesas, setsas, recover, backup, restore, purge
```

Notez que les autorisations pour les comptes de stockage ne sont pas disponibles sur la page « Stratégies d’accès » du compte de stockage dans le portail Azure.

### <a name="add-a-managed-storage-account-to-your-key-vault-instance"></a>Ajouter un compte de stockage managé à votre instance Key Vault

Utilisez la cmdlet Azure PowerShell [Add-AzKeyVaultManagedStorageAccount](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount) pour créer un compte de stockage géré dans votre instance Key Vault. Le commutateur `-DisableAutoRegenerateKey` spécifie de ne PAS régénérer les clés de compte de stockage.

```azurepowershell-interactive
# Add your storage account to your Key Vault's managed storage accounts

Add-AzKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storageAccountName -AccountResourceId $storageAccount.Id -ActiveKeyName $storageAccountKey -DisableAutoRegenerateKey
```

Si le compte de stockage a été correctement créé sans régénération des clés, le résultat devrait ressembler à l’exemple suivant :

```console
Id                  : https://kvcontoso.vault.azure.net:443/storage/sacontoso
Vault Name          : kvcontoso
AccountName         : sacontoso
Account Resource Id : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
Active Key Name     : key1
Auto Regenerate Key : False
Regeneration Period : 90.00:00:00
Enabled             : True
Created             : 11/19/2018 11:54:47 PM
Updated             : 11/19/2018 11:54:47 PM
Tags                :
```

### <a name="enable-key-regeneration"></a>Activer la régénération des clés

Pour permettre à Key Vault de régénérer régulièrement vos clés de compte de stockage, utilisez la cmdlet Azure PowerShell [Add-AzKeyVaultManagedStorageAccount](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount) afin de définir une période de régénération. Dans cet exemple, nous définissons une période de régénération de trois jours. Le moment venu d’opérer une rotation, Key Vault régénère la clé qui n’est pas active, puis définit la clé nouvellement créée comme active. Une seule des clés est utilisée pour émettre des jetons SAS à tout moment. Il s’agit du nœud actif.

```azurepowershell-interactive
$regenPeriod = [System.Timespan]::FromDays(3)

Add-AzKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storageAccountName -AccountResourceId $storageAccount.Id -ActiveKeyName $storageAccountKey -RegenerationPeriod $regenPeriod
```

Si le compte de stockage a été correctement créé avec régénération des clés, le résultat devrait ressembler à l’exemple suivant :

```console
Id                  : https://kvcontoso.vault.azure.net:443/storage/sacontoso
Vault Name          : kvcontoso
AccountName         : sacontoso
Account Resource Id : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
Active Key Name     : key1
Auto Regenerate Key : True
Regeneration Period : 3.00:00:00
Enabled             : True
Created             : 11/19/2018 11:54:47 PM
Updated             : 11/19/2018 11:54:47 PM
Tags                :
```

## <a name="shared-access-signature-tokens"></a>Jetons de signature d’accès partagé

Vous pouvez également demander à Key Vault de générer des jetons de signature d’accès partagé. Une signature d'accès partagé fournit un accès délégué aux ressources de votre compte de stockage. Vous pouvez accorder aux clients l’accès aux ressources dans votre compte de stockage sans partager les clés de votre compte. Une signature d’accès partagé constitue un moyen sécurisé de partager vos ressources de stockage sans compromettre vos clés de compte.

Les commandes figurant dans cette section effectuent les actions suivantes :

- Définissez une définition de signature d’accès partagé.
- Créez un jeton de signature d’accès partagé pour les services Blob, Fichier, Table et File d’attente. Le jeton est créé pour les types de ressource Service, Conteneur et Objet. Le jeton est créé avec toutes les autorisations, via le protocole https et avec les dates de début et de fin spécifiées.
- Définissez une définition de signature d’accès partagé au stockage géré par Key Vault dans le coffre. La définition comporte l’URI du modèle du jeton de signature d’accès partagé créé. La définition possède le type de signature d’accès partagé `account` et est valide pendant N jours.
- Vérifiez que la signature d’accès partagé a été enregistrée dans votre coffre de clés en tant que secret.
-
### <a name="set-variables"></a>Définition des variables

Commencez par définir les variables utilisées par les cmdlets PowerShell dans les étapes suivantes. Veillez à mettre à jour les espaces réservés <YourStorageAccountName> et <YourKeyVaultName>.

Nous utiliserons également les cmdlets Azure PowerShell [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext) pour obtenir le contexte de votre compte de stockage Azure.

```azurepowershell-interactive
$storageAccountName = <YourStorageAccountName>
$keyVaultName = <YourKeyVaultName>

$storageContext = New-AzStorageContext -StorageAccountName $storageAccountName -Protocol Https -StorageAccountKey Key1 #(or "Primary" for Classic Storage Account)
```

### <a name="create-a-shared-access-signature-token"></a>Créer un jeton de signature d’accès partagé

Créez une définition de signature d’accès partagé à l’aide des cmdlets Azure PowerShell [New-AzStorageAccountSASToken](/powershell/module/az.storage/new-azstorageaccountsastoken).

```azurepowershell-interactive
$start = [System.DateTime]::Now.AddDays(-1)
$end = [System.DateTime]::Now.AddMonths(1)

$sasToken = New-AzStorageAccountSasToken -Service blob,file,Table,Queue -ResourceType Service,Container,Object -Permission "racwdlup" -Protocol HttpsOnly -StartTime $start -ExpiryTime $end -Context $storageContext
```
La valeur de $sasToken se présente comme suit.

```console
?sv=2018-11-09&sig=5GWqHFkEOtM7W9alOgoXSCOJO%2B55qJr4J7tHQjCId9S%3D&spr=https&st=2019-09-18T18%3A25%3A00Z&se=2019-10-19T18%3A25%3A00Z&srt=sco&ss=bfqt&sp=racupwdl
```

### <a name="generate-a-shared-access-signature-definition"></a>Générer une définition de signature d’accès partagé

Utilisez la cmdlet Azure PowerShell [Set-AzKeyVaultManagedStorageSasDefinition](/powershell/module/az.keyvault/set-azkeyvaultmanagedstoragesasdefinition) pour créer une définition de signature d’accès partagé.  Vous pouvez fournir le nom de votre choix au paramètre `-Name`.

```azurepowershell-interactive
Set-AzKeyVaultManagedStorageSasDefinition -AccountName $storageAccountName -VaultName $keyVaultName -Name <YourSASDefinitionName> -TemplateUri $sasToken -SasType 'account' -ValidityPeriod ([System.Timespan]::FromDays(30))
```

### <a name="verify-the-shared-access-signature-definition"></a>Vérifier la définition de la signature d’accès partagé

Vous pouvez vérifier que la définition de la signature d’accès partagé a été stockée dans votre coffre de clés à l’aide de la cmdlet [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret) Azure PowerShell.

Commencez par rechercher la définition de signature d’accès partagé dans votre coffre de clés.

```azurepowershell-interactive
Get-AzKeyVaultSecret -VaultName <YourKeyVaultName>
```

Le secret correspondant à votre définition SAS a les propriétés suivantes :

```console
Vault Name   : <YourKeyVaultName>
Name         : <SecretName>
...
Content Type : application/vnd.ms-sastoken-storage
Tags         :
```

Vous pouvez maintenant utiliser l’applet de commande [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret) avec les paramètres `VaultName` et `Name` pour afficher le contenu de ce secret.

```azurepowershell-interactive
$secret = Get-AzKeyVaultSecret -VaultName <YourKeyVaultName> -Name <SecretName>
$ssPtr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secret.SecretValue)
try {
   $secretValueText = [System.Runtime.InteropServices.Marshal]::PtrToStringBSTR($ssPtr)
} finally {
   [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ssPtr)
}
Write-Output $secretValueText
```

La sortie de cette commande affiche la chaîne de votre définition SAP.

## <a name="next-steps"></a>Étapes suivantes

- [Exemples de clés de compte de stockage managé](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=)
- [Informations de référence PowerShell sur Key Vault](/powershell/module/az.keyvault/#key_vault)

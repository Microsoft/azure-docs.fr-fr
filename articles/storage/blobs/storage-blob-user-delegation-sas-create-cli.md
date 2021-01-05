---
title: Utiliser Azure CLI pour créer une SAS de délégation d’utilisateur pour un conteneur ou un objet blob
titleSuffix: Azure Storage
description: Découvrez comment créer une SAP de délégation d’utilisateur avec des informations d’identification Azure Active Directory à l’aide d’Azure CLI.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/18/2019
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: blobs
ms.custom: devx-track-azurecli
ms.openlocfilehash: 453eaa816ad48626b476fa392999f44e3c1a10cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91714554"
---
# <a name="create-a-user-delegation-sas-for-a-container-or-blob-with-the-azure-cli"></a>Créer une SAP de délégation d’utilisateur pour un conteneur ou un objet blob avec l’interface Azure CLI

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

Cet article explique comment utiliser les informations d’identification Azure Active Directory (Azure AD) pour créer une SAP de délégation d’utilisateur pour un conteneur ou un objet BLOB avec l’interface de ligne de commande Azure.

[!INCLUDE [storage-auth-user-delegation-include](../../../includes/storage-auth-user-delegation-include.md)]

## <a name="install-the-latest-version-of-the-azure-cli"></a>Installez la dernière version de l’interface de ligne de commande Azure

Pour utiliser l’interface de ligne de commande Azure afin de sécuriser une SAP avec des informations d’identification Azure AD, commencez par vérifier que vous avez installé la dernière version de l’interface de ligne de commande Azure. Pour plus d'informations sur l'installation de l’interface de ligne de commande Azure, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

Pour créer une SAP de délégation d’utilisateur à l’aide d’Azure CLI, assurez-vous d’avoir installé la version 2.0.78 ou ultérieure. Utilisez la commande `az --version` pour vérifier la version installée.

## <a name="sign-in-with-azure-ad-credentials"></a>Se connecter avec les informations d'identification Azure AD

Connectez-vous à l’interface de ligne de commande Azure avec vos informations d’identification Azure AD. Pour plus d’informations, consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli).

## <a name="assign-permissions-with-azure-rbac"></a>Attribuer des autorisations avec le contrôle d’accès en fonction du rôle (RBAC) Azure

Pour créer une SAP de délégation d’utilisateur à partir d’Azure PowerShell, le compte Azure AD utilisé pour se connecter à l’interface de ligne de commande Azure doit se voir attribuer un rôle incluant l’action **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey**. Cette autorisation permet au compte Azure AD de demander la *clé de délégation d’utilisateur*. La clé de délégation d’utilisateur est utilisée pour signer les SAP de délégation d’utilisateur. Le rôle qui fournit l’action **Microsoft. Storage/storageAccounts/blobServices/generateUserDelegationKey** doit être attribué au niveau du compte de stockage, du groupe de ressources ou de l’abonnement.

Si vous ne disposez pas des autorisations suffisantes pour attribuer des rôles Azure au principal de sécurité Azure AD, vous devrez peut-être demander au propriétaire du compte ou à l’administrateur d’octroyer les autorisations nécessaires.

L’exemple suivant attribue le rôle de **Contributeur de données d’objet blob de stockage**, qui inclut l’action **Microsoft. Storage/storageAccounts/blobServices/generateUserDelegationKey**. Le rôle est étendu au niveau du compte de stockage.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

Pour plus d’informations sur les rôles intégrés qui incluent l’action **Microsoft. Storage/storageAccounts/blobServices/generateUserDelegationKey**, consultez [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md).

## <a name="use-azure-ad-credentials-to-secure-a-sas"></a>Utiliser des informations d’identification Azure AD pour sécuriser une SAP

Lorsque vous créez une SAP de délégation d’utilisateur avec l’interface de ligne de commande Azure, la clé de délégation d’utilisateur utilisée pour signer la SAP est créée de manière implicite. L’heure de début et l’heure d’expiration que vous spécifiez pour la SAP sont également utilisées comme heure de début et heure d’expiration pour la clé de délégation d’utilisateur.

Étant donné que la clé de délégation d’utilisateur est valide au maximum pendant 7 jours à partir de la date de début, vous devez spécifier pour la SAP un délai d’expiration se situant dans les 7 jours après l’heure de début. La SAP n’est pas valide après l’expiration de la clé de délégation d’utilisateur. Par conséquent, une SAP dont le délai d’expiration est supérieur à 7 jours ne sera toujours valide que pendant 7 jours.

Lors de la création d’une SAP de délégation d’utilisateur, `--auth-mode login` et `--as-user parameters` sont requis. Spécifiez une *connexion* pour le paramètre `--auth-mode` afin que les demandes effectuées auprès du stockage Azure soient autorisées avec vos informations d’identification Azure AD. Spécifiez le paramètre `--as-user` pour indiquer que la SAP doit être une SAP de délégation d’utilisateur.

### <a name="create-a-user-delegation-sas-for-a-container"></a>Créer une SAP de délégation d’utilisateur pour un conteneur

Si vous souhaitez créer une SAP de délégation utilisateur pour un conteneur avec Azure CLI, appelez la commande [az storage container generate-sas](/cli/azure/storage/container#az-storage-container-generate-sas).

Les autorisations prises en charge pour une SAP sur un conteneur incluent Ajouter, Créer, Supprimer, Répertorier, Lire et Écrire. Les autorisations peuvent être spécifiées séparément ou combinées. Pour plus d’informations sur ces permissions, consultez [Créer une SAP de délégation d’utilisateur](/rest/api/storageservices/create-user-delegation-sas).

L’exemple suivant retourne un jeton SAS de délégation d’utilisateur pour un conteneur. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az storage container generate-sas \
    --account-name <storage-account> \
    --name <container> \
    --permissions acdlrw \
    --expiry <date-time> \
    --auth-mode login \
    --as-user
```

Le jeton SAS de délégation d’utilisateur retourné sera semblable à ce qui suit :

```
se=2019-07-27&sp=r&sv=2018-11-09&sr=c&skoid=<skoid>&sktid=<sktid>&skt=2019-07-26T18%3A01%3A22Z&ske=2019-07-27T00%3A00%3A00Z&sks=b&skv=2018-11-09&sig=<signature>
```

### <a name="create-a-user-delegation-sas-for-a-blob"></a>Créer une SAP de délégation d’utilisateur pour un objet blob

Si vous souhaitez créer une SAP de délégation utilisateur pour un objet blob avec Azure CLI, appelez la commande [az storage blob generate-sas](/cli/azure/storage/blob#az-storage-blob-generate-sas).

Les autorisations prises en charge pour une SAP sur un objet blob incluent Ajouter, Créer, Supprimer, Lire et Écrire. Les autorisations peuvent être spécifiées séparément ou combinées. Pour plus d’informations sur ces permissions, consultez [Créer une SAP de délégation d’utilisateur](/rest/api/storageservices/create-user-delegation-sas).

La syntaxe suivante retourne une SAP de délégation d’utilisateur pour un objet blob. L’exemple spécifie le paramètre `--full-uri`, qui retourne l’URI de l’objet blob auquel le jeton SAS est ajouté. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az storage blob generate-sas \
    --account-name <storage-account> \
    --container-name <container> \
    --name <blob> \
    --permissions acdrw \
    --expiry <date-time> \
    --auth-mode login \
    --as-user
    --full-uri
```

L’URI de SAP de délégation d’utilisateur retournée sera semblable à ce qui suit :

```
https://storagesamples.blob.core.windows.net/sample-container/blob1.txt?se=2019-08-03&sp=rw&sv=2018-11-09&sr=b&skoid=<skoid>&sktid=<sktid>&skt=2019-08-02T2
2%3A32%3A01Z&ske=2019-08-03T00%3A00%3A00Z&sks=b&skv=2018-11-09&sig=<signature>
```

> [!NOTE]
> Une SAP de délégation d’utilisateur ne prend pas en charge la définition d’autorisations avec une stratégie d’accès stockée.

## <a name="revoke-a-user-delegation-sas"></a>Révoquer une SAP de délégation d’utilisateur

Pour révoquer une SAP de délégation utilisateur dans Azure CLI, appelez la commande [az storage account revoke-delegation-keys](/cli/azure/storage/account#az-storage-account-revoke-delegation-keys). Cette commande révoque toutes les clés de délégation d’utilisateur associées au compte de stockage spécifié. Toutes les signatures d’accès partagé associées à ces clés sont invalidées.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az storage account revoke-delegation-keys \
    --name <storage-account> \
    --resource-group <resource-group>
```

> [!IMPORTANT]
> Les attributions de rôles Azure et de clés de délégation utilisateur sont mises en cache par le stockage Azure. Il peut donc y avoir un certain délai entre le moment où vous lancez le processus de révocation et celui où la SAP de délégation utilisateur devient non valide.

## <a name="next-steps"></a>Étapes suivantes

- [Créer une SAP de délégation d’utilisateur (API REST)](/rest/api/storageservices/create-user-delegation-sas)
- [Obtenir une opération de clé de délégation d’utilisateur](/rest/api/storageservices/get-user-delegation-key)

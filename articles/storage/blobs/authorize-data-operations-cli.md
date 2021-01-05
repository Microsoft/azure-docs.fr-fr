---
title: Choisir comment autoriser l’accès à des données blob avec Azure CLI
titleSuffix: Azure Storage
description: Spécifiez comment autoriser des opérations de données sur des données blob avec Azure CLI. Vous pouvez autoriser des opérations de données utilisant des informations d’identification Azure AD, avec la clé d’accès au compte ou un jeton de signature d’accès partagé (SAP).
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 11/13/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: 060bfb6c88bbed8ba12c5b5ebfd2e9617f5abfb2
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94637204"
---
# <a name="choose-how-to-authorize-access-to-blob-data-with-azure-cli"></a>Choisir comment autoriser l’accès à des données blob avec Azure CLI

Le service Stockage Azure fournit des extensions pour Azure CLI qui vous permettent de spécifier comment autoriser des opérations sur des données blob. Vous pouvez autoriser des opérations de données des manières suivantes :

- Avec un principal de service Azure Active Directory (Azure AD). Microsoft recommande d’utiliser des informations d’identification Azure AD pour une sécurité et une facilité d’utilisation supérieures.
- Avec la clé d’accès au compte ou un jeton de signature d’accès partagé (SAP).

## <a name="specify-how-data-operations-are-authorized"></a>Spécifier la façon dont les opérations de données sont autorisées

Les commandes Azure CLI pour la lecture et l’écriture de données blob incluent le paramètre facultatif `--auth-mode`. Spécifiez-le pour indiquer comment une opération de données doit être autorisée :

- Définissez le paramètre `--auth-mode` sur `login` pour vous connecter à l’aide d’un principal de sécurité Azure AD (recommandé).
- Définissez le paramètre `--auth-mode` sur la valeur `key` héritée pour tenter de récupérer la clé d’accès au compte à utiliser pour l’autorisation. Si vous omettez le paramètre `--auth-mode`, Azure CLI tente également de récupérer la clé d’accès.

Pour utiliser le paramètre `--auth-mode`, assurez-vous qu’Azure CLI version 2.0.46 ou ultérieure est installé. Exécutez `az --version` pour vérifier la version installée.

> [!IMPORTANT]
> Si vous omettez le paramètre `--auth-mode` ou si vous le définissez sur `key`, Azure CLI tente d’utiliser la clé d’accès au compte pour l’autorisation. Dans ce cas, Microsoft recommande de fournir la clé d’accès sur la commande ou dans la variable d’environnement **AZURE_STORAGE_KEY**. Pour plus d’informations sur les variables d’environnement, voir la section intitulée [Définir des variables d’environnement pour les paramètres d’autorisation](#set-environment-variables-for-authorization-parameters).
>
> Si vous ne fournissez pas la clé d’accès, Azure CLI tente d’appeler le fournisseur de ressources de Stockage Azure afin de le récupérer pour chaque opération. L’exécution de nombreuses opérations de données nécessitant un appel au fournisseur de ressources peut entraîner une limitation. Pour plus d’informations sur les limites du fournisseur de ressources, voir [Objectifs de scalabilité et de performances pour le fournisseur de ressources de Stockage Azure](../common/scalability-targets-resource-provider.md).

## <a name="authorize-with-azure-ad-credentials"></a>Autoriser avec des informations d’identification Azure AD

Quand vous vous connectez à Azure CLI avec des informations d’identification Azure AD, un jeton d’accès OAuth 2.0 est retourné. Ce jeton est utilisé automatiquement par Azure CLI pour autoriser les opérations de données suivantes sur le Stockage Blob ou File d’attente. Pour les opérations prises en charge, vous n’avez plus besoin de fournir une clé de compte ou le jeton SAP avec la commande.

Vous pouvez attribuer des autorisations sur les données Blob à un principal de sécurité Azure AD via le contrôle d’accès Azure en fonction du rôle (RBAC). Pour plus d’informations sur les rôles Azure dans Stockage Azure, consultez [Gérer les droits d’accès aux données Stockage Azure avec Azure RBAC](../common/storage-auth-aad-rbac-portal.md).

### <a name="permissions-for-calling-data-operations"></a>Autorisations pour appeler des opérations de données

Les extensions Stockage Azure sont prises en charge pour les opérations sur les données Blob. Les opérations que vous pouvez appeler dépendent des autorisations accordées au principal de sécurité Azure AD avec lequel vous vous connectez à Azure CLI. Les autorisations sur les conteneurs Stockage Azure sont attribuées via Azure RBAC. Par exemple, si le rôle **Lecteur des données blob du stockage** vous est attribué, vous pouvez exécuter des commandes de script qui lisent les données d’un conteneur. Si le rôle **Contributeur aux données blob du stockage** vous est attribué, vous pouvez exécuter des commandes de script qui lisent, écrivent ou suppriment un conteneur ou les données qu’il contient.

Pour plus d’informations sur les autorisations requises pour chaque opération Stockage Azure sur un conteneur, consultez [Appeler des opérations de stockage avec des jetons OAuth](/rest/api/storageservices/authorize-with-azure-active-directory#call-storage-operations-with-oauth-tokens).  

### <a name="example-authorize-an-operation-to-create-a-container-with-azure-ad-credentials"></a>Exemple : Autoriser une opération à créer un conteneur avec des informations d’identification Azure AD

L’exemple suivant montre comment créer un conteneur à partir d’Azure CLI à l’aide de vos informations d’identification Azure AD. Pour créer le conteneur, vous devez vous connecter à Azure CLI et aurez besoin d’un groupe de ressources et d’un compte de stockage. Pour savoir comment créer ces ressources, voir [Démarrage rapide : Créer, télécharger et lister des objets blob avec Azure CLI](../blobs/storage-quickstart-blobs-cli.md).

1. Avant de créer le conteneur, attribuez-vous le rôle [Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor). Même si vous êtes le propriétaire du compte, vous avez besoin d’autorisations explicites pour effectuer des opérations de données sur le compte de stockage. Pour plus d’informations sur l’attribution de rôles Azure, consultez [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](../common/storage-auth-aad-rbac-portal.md).

    > [!IMPORTANT]
    > La propagation des attributions de rôles Azure peut prendre plusieurs minutes.

1. Appelez la commande [az storage container create](/cli/azure/storage/container#az-storage-container-create) avec le paramètre `--auth-mode` défini sur `login` pour créer le conteneur à l’aide de vos informations d’identification Azure AD. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

    ```azurecli
    az storage container create \
        --account-name <storage-account> \
        --name sample-container \
        --auth-mode login
    ```

## <a name="authorize-with-the-account-access-key"></a>Autoriser avec la clé d’accès au compte

Si vous possédez la clé de compte, vous pouvez appeler toute opération de données Azure Storage. En général, l’utilisation de la clé de compte est moins sécurisée. Si la clé de compte est compromise, toutes les données de votre compte peuvent l’être également.

L’exemple suivant montre comment créer un conteneur à l’aide de la clé d’accès au compte. Spécifiez la clé de compte et fournissez le paramètre `--auth-mode` avec la valeur `key` :

```azurecli
az storage container create \
    --account-name <storage-account> \
    --name sample-container \
    --account-key <key>
    --auth-mode key
```

## <a name="authorize-with-a-sas-token"></a>Autoriser avec un jeton SAP

Si vous possédez un jeton SAP, vous pouvez appeler des opérations de données autorisées par la signature d’accès partagé. L’exemple suivant montre comment créer un conteneur à l’aide d’un jeton SAP :

```azurecli
az storage container create \
    --account-name <storage-account> \
    --name sample-container \
    --sas-token <token>
```

## <a name="set-environment-variables-for-authorization-parameters"></a>Définir des variables d’environnement pour les paramètres d’autorisation

Vous pouvez spécifier des paramètres d’autorisation dans des variables d’environnement afin d’éviter de les inclure à chaque appel à une opération de données Azure Storage. Le tableau suivant décrit les variables d’environnement disponibles.

| Variable d’environnement                  | Description                                                                                                                                                                                                                                                                                                                                                                     |
|---------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    AZURE_STORAGE_ACCOUNT              |    nom du compte de stockage. Cette variable doit être utilisée conjointement avec la clé de compte de stockage ou un jeton SAS. Si ce n’est pas le cas, Azure CLI tente de récupérer la clé d’accès au compte de stockage à l’aide du compte Azure AD authentifié. Si un grand nombre de commandes sont exécutées en même temps, la limitation du fournisseur de ressources de Stockage Azure peut être atteinte. Pour plus d’informations sur les limites du fournisseur de ressources, voir [Objectifs de scalabilité et de performances pour le fournisseur de ressources de Stockage Azure](../common/scalability-targets-resource-provider.md).             |
|    AZURE_STORAGE_KEY                  |    La clé du compte de stockage. Cette variable doit être utilisée conjointement avec le nom du compte de stockage.                                                                                                                                                                                                                                                                          |
|    AZURE_STORAGE_CONNECTION_STRING    |    Chaîne de connexion contenant la clé de compte de stockage ou un jeton SAP. Cette variable doit être utilisée conjointement avec le nom du compte de stockage.                                                                                                                                                                                                                       |
|    AZURE_STORAGE_SAS_TOKEN            |    Utilisez un jeton de signature d’accès partagé (SAP). Cette variable doit être utilisée conjointement avec le nom du compte de stockage.                                                                                                                                                                                                                                                            |
|    AZURE_STORAGE_AUTH_MODE            |    Mode d’autorisation avec lequel exécuter la commande. Les valeurs autorisées sont `login` (recommandée) ou `key`. Si vous spécifiez `login`, Azure CLI utilise vos informations d’identification Azure AD pour autoriser l’opération de données. Si vous spécifiez le mode `key` hérité, Azure CLI tente d’interroger la clé d’accès au compte et d’autoriser la commande avec cette clé.    |

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser Azure CLI pour affecter un rôle Azure gérant l’accès à des données d’objet blob et de file d’attente](../common/storage-auth-aad-rbac-cli.md)
- [Autoriser l’accès à des données blob et de files d’attente avec des identités managées pour les ressources Azure](../common/storage-auth-aad-msi.md)
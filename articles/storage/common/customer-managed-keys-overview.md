---
title: Clés gérées par le client pour le chiffrement de compte
titleSuffix: Azure Storage
description: Vous pouvez utiliser votre propre clé de chiffrement pour protéger les données de votre compte de stockage. Quand vous spécifiez une clé gérée par le client, cette clé est utilisée pour protéger et contrôler l’accès à la clé qui chiffre vos données. Les clés gérées par le client offrent davantage de flexibilité pour gérer les contrôles d’accès.
services: storage
author: tamram
ms.service: storage
ms.date: 09/15/2020
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 8eaadc031039b22e209db1023c65da39e1e096b1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483281"
---
# <a name="customer-managed-keys-for-azure-storage-encryption"></a>Clés gérées par le client pour le chiffrement du service Stockage Azure

Vous pouvez utiliser votre propre clé de chiffrement pour protéger les données de votre compte de stockage. Quand vous spécifiez une clé gérée par le client, cette clé est utilisée pour protéger et contrôler l’accès à la clé qui chiffre vos données. Les clés gérées par le client offrent davantage de flexibilité pour gérer les contrôles d’accès.

Vous devez utiliser Azure Key Vault ou un module de sécurité matériel (HSM) géré par Azure Key Vault Azure (préversion) pour stocker vos clés gérées par le client. Vous pouvez créer vos propres clés et les stocker dans le coffre de clés ou un HSM managé, ou utiliser les API d’Azure Key Vault pour générer des clés. Le compte de stockage et le coffre de clés ou le HSM managé doivent résider dans la même région et dans le même locataire Azure Active Directory, mais ils peuvent se trouver dans des abonnements différents.

Pour plus d’informations sur Azure Key Vault, consultez [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md).

> [!NOTE]
> Azure Key Vault et le module HSM managé par Azure Key Vault prennent en charge les mêmes API et interfaces de gestion pour la configuration.

## <a name="about-customer-managed-keys"></a>À propos des clés gérées par le client

Le diagramme suivant montre comment le service Stockage Azure utilise Azure Active Directory et Azure Key Vault ou un HSM managé pour faire des demandes à l’aide de la clé gérée par le client :

![Diagramme montrant le fonctionnement des clés managées par le client dans Stockage Azure](media/customer-managed-keys-overview/encryption-customer-managed-keys-diagram.png)

La liste suivante décrit les étapes numérotées dans le diagramme :

1. Un administrateur Azure Key Vault accorde des autorisations d’accès à des clés de chiffrement à l’identité managée associée au compte de stockage.
2. Un administrateur Stockage Azure configure le chiffrement avec une clé managée par le client pour le compte de stockage.
3. Stockage Azure utilise l’identité managée associée au compte de stockage pour authentifier l’accès à Azure Key Vault par le biais d’Azure Active Directory.
4. Stockage Azure encapsule la clé de chiffrement du compte avec la clé du client dans Azure Key Vault.
5. Pour les opérations de lecture/écriture, Stockage Azure envoie des requêtes à Azure Key Vault pour désencapsuler la clé de chiffrement du compte afin d’effectuer des opérations de chiffrement et de déchiffrement.

## <a name="customer-managed-keys-for-queues-and-tables"></a>Clés gérées par le client pour les files d’attente et les tables

Les données stockées dans les services Stockage File d’attente et Stockage Table ne sont pas automatiquement protégées par une clé gérée par le client quand les clés gérées par le client sont activées pour le compte de stockage. Vous pouvez éventuellement configurer ces services à inclure dans cette protection au moment où vous créez le compte de stockage.

Pour plus d’informations sur la façon de créer un compte de stockage qui prend en charge les clés gérées par le client pour les files d’attente et les tables, consultez [Créer un compte qui prend en charge les clés gérées par le client pour les tables et les files d’attente](account-encryption-key-create.md).

Les données se trouvant dans Stockage Blob et Azure Files sont toujours protégées par des clés gérées par le client quand les clés gérées par le client sont configurées pour le compte de stockage.

## <a name="enable-customer-managed-keys-for-a-storage-account"></a>Activer des clés gérées par le client pour un compte de stockage

Quand vous configurez une clé gérée par le client, le service Stockage Azure encapsule la clé de chiffrement des données racine pour le compte avec la clé gérée par le client dans le coffre de clés associé ou le HSM managé. L’activation des clés gérées par le client n’a aucun impact sur les performances et prend effet immédiatement.

Quand vous activez ou désactivez les clés gérées par le client, ou quand vous modifiez la clé ou la version de clé, la protection de la clé de chiffrement racine change, mais les données de votre compte Stockage Azure n’ont pas besoin d’être rechiffrées.

Les clés gérées par le client ne peuvent être activées que sur des comptes de stockage existants. Le coffre de clés ou le HSM managé doivent être configurés pour accorder des autorisations à l’identité managée associée au compte de stockage. L’identité managée est disponible uniquement après la création du compte de stockage.

Vous pouvez basculer entre les clés gérées par le client et les clés managées par Microsoft à tout moment. Pour plus d’informations sur les clés managées par Microsoft, consultez [À propos de la gestion des clés de chiffrement](storage-service-encryption.md#about-encryption-key-management).

Pour savoir comment configurer le chiffrement du service Stockage Azure avec des clés gérées par le client dans un coffre de clés, consultez [Configurer le chiffrement avec des clés gérées par le client stockées dans Azure Key Vault](customer-managed-keys-configure-key-vault.md). Pour configurer des clés gérées par le client stockées dans un HSM managé, consultez [Configurer le chiffrement avec des clés gérées par le client stockées dans un HSM managé par Azure Key Vault (préversion)](customer-managed-keys-configure-key-vault-hsm.md).

> [!IMPORTANT]
> Les clés gérées par le client s’appuient sur des identités managées pour les ressources Azure, une fonctionnalité d’Azure AD. Les identités managées ne prennent actuellement pas en charge les scénarios entre répertoires. Quand vous configurez des clés managées par le client dans le portail Azure, une identité managée est affectée automatiquement à votre compte de stockage dans les coulisses. Si, par la suite, vous déplacez l’abonnement, le groupe de ressources ou le compte de stockage d’un annuaire Azure AD vers un autre, l’identité managée associée au compte de stockage n’est pas transférée vers le nouveau locataire ; les clés managées par le client risquent donc de ne plus fonctionner. Pour plus d’informations, consultez **Transfert d’un abonnement entre des répertoires Azure AD** dans [FAQ et problèmes connus en lien avec les identités managées pour ressources Azure](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).  

Le chiffrement du stockage Azure prend en charge les clés RSA et RSA-HSM dans les tailles 2048, 3072 et 4096. Pour plus d’informations sur les clés, consultez [À propos des clés](../../key-vault/keys/about-keys.md).

L’utilisation d’un coffre de clés ou d’un module HSM managé occasionne des coûts. Pour plus d’informations, consultez [Tarification de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).

## <a name="update-the-key-version"></a>Mettre à jour la version de la clé

Lorsque vous configurez un chiffrement avec des clés gérées par le client, vous disposez de deux options pour mettre à jour la version de la clé :

- **Mettre à jour automatiquement la version de la clé :** pour mettre à jour automatiquement une clé gérée par le client quand une nouvelle version est disponible, omettez la version de la clé lorsque vous activez le chiffrement avec des clés gérées par le client pour le compte de stockage. Si la version de la clé est omise, le service Stockage Azure vérifie quotidiennement le coffre de clés ou le HSM managé pour voir s’il existe une nouvelle version d’une clé gérée par le client. Le service Stockage Azure utilise automatiquement la dernière version de la clé.
- **Mettre à jour manuellement la version de la clé :** pour utiliser une version spécifique d’une clé pour le chiffrement du service Stockage Azure, spécifiez la version de la clé lorsque vous activez le chiffrement avec les clés gérées par le client pour le compte de stockage. Si vous spécifiez la version de la clé, Stockage Azure utilise cette version pour le chiffrement jusqu’à ce que vous mettiez manuellement à jour la version de la clé.

    Lorsque la version de la clé est spécifiée explicitement, vous devez mettre à jour manuellement le compte de stockage pour utiliser l’URI de la nouvelle version de la clé lors de la création d’une nouvelle version. Pour savoir comment mettre à jour le compte de stockage afin d’utiliser une nouvelle version de la clé, consultez [Configurer le chiffrement avec des clés gérées par le client stockées dans Azure Key Vault](customer-managed-keys-configure-key-vault.md) ou [Configurer le chiffrement avec des clés gérées par le client stockées dans un module HSM géré par Azure Key Vault Azure (préversion)](customer-managed-keys-configure-key-vault-hsm.md).

La mise à jour de la version d’une clé gérée par le client ne déclenche pas le nouveau chiffrement des données dans le compte de stockage. Aucune autre action n’est requise de la part de l’utilisateur.

> [!NOTE]
> Pour effectuer une rotation d’une clé, créez une nouvelle version de la clé dans le coffre de clés ou le module HSM managé, en fonction de vos stratégies de conformité. Vous pouvez effectuer une rotation de votre clé manuellement ou créer une fonction pour effectuer la rotation selon une planification.

## <a name="revoke-access-to-customer-managed-keys"></a>Révoquer l’accès aux clés gérées par le client

Vous pouvez révoquer l’accès du compte de stockage à la clé gérée par le client à tout moment. Après la révocation de l’accès aux clés gérées par le client, ou après la désactivation ou la suppression de la clé, les clients ne peuvent pas appeler les opérations qui lisent ou écrivent dans un objet blob ou ses métadonnées. Toute tentative d’appel de l’une des opérations suivantes échouent avec le code d’erreur 403 (interdit) pour tous les utilisateurs :

- [List Blobs](/rest/api/storageservices/list-blobs), en cas d’appel avec le paramètre `include=metadata` sur l’URI de requête
- [Get Blob](/rest/api/storageservices/get-blob)
- [Get Blob Properties](/rest/api/storageservices/get-blob-properties)
- [Get Blob Metadata](/rest/api/storageservices/get-blob-metadata)
- [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata)
- [Snapshot Blob](/rest/api/storageservices/snapshot-blob), en cas d’appel avec l’en-tête de requête `x-ms-meta-name`
- [Copy Blob](/rest/api/storageservices/copy-blob)
- [Copy Blob From URL](/rest/api/storageservices/copy-blob-from-url)
- [Set Blob Tier](/rest/api/storageservices/set-blob-tier)
- [Put Block](/rest/api/storageservices/put-block)
- [Put Block From URL](/rest/api/storageservices/put-block-from-url)
- [Append Block](/rest/api/storageservices/append-block)
- [Append Block From URL](/rest/api/storageservices/append-block-from-url)
- [Put Blob](/rest/api/storageservices/put-blob)
- [Put Page](/rest/api/storageservices/put-page)
- [Put Page From URL](/rest/api/storageservices/put-page-from-url)
- [Copie incrémentielle BLOB](/rest/api/storageservices/incremental-copy-blob)

Pour rappeler ces opérations, restaurez l’accès à la clé gérée par le client.

Toutes les opérations de données qui ne sont pas mentionnées dans cette section peuvent se poursuivre après la révocation des clés gérées par le client ou après la suppression ou la désactivation d’une clé.

Pour révoquer l’accès aux clés gérées par le client, utilisez [PowerShell](./customer-managed-keys-configure-key-vault.md#revoke-customer-managed-keys) ou [Azure CLI](./customer-managed-keys-configure-key-vault.md#revoke-customer-managed-keys).

## <a name="customer-managed-keys-for-azure-managed-disks"></a>Clés gérées par le client pour les disques managés Azure

Les clés gérées par le client sont également disponibles pour la gestion du chiffrement des disques managés Azure. Les clés gérées par le client se comportent différemment pour les disques managés que pour les ressources de stockage Azure. Pour plus d’informations, consultez [Chiffrement côté serveur de disques managés Azure](../../virtual-machines/disk-encryption.md) pour Windows ou [Chiffrement côté serveur de disques managés Azure](../../virtual-machines/disk-encryption.md) pour Linux.

## <a name="next-steps"></a>Étapes suivantes

- [Chiffrement du stockage Azure pour les données au repos](storage-service-encryption.md)
- [Configurer le chiffrement avec des clés gérées par le client stockées dans Azure Key Vault](customer-managed-keys-configure-key-vault.md)
- [Configurer le chiffrement avec des clés gérées par le client stockées dans le HSM managé par Azure Key Vault (préversion)](customer-managed-keys-configure-key-vault-hsm.md)
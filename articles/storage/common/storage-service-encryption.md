---
title: Chiffrement du Stockage Azure pour les données au repos
description: Le Stockage Azure protège vos données en les chiffrant automatiquement avant de les rendre persistantes dans le cloud. Vous pouvez vous reposer sur les clés managées par Microsoft pour le chiffrement des données de votre compte de stockage, ou vous pouvez gérer le chiffrement avec vos propres clés.
services: storage
author: tamram
ms.service: storage
ms.date: 09/17/2020
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: da1acc6316d7af87ffe35b9560919c324373591a
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484598"
---
# <a name="azure-storage-encryption-for-data-at-rest"></a>Chiffrement du Stockage Azure pour les données au repos

Stockage Azure chiffre automatiquement vos données lors de leur conservation dans le cloud. Le chiffrement de Stockage Azure protège vos données et vous aide à répondre aux engagements de votre entreprise en matière de sécurité et de conformité.

## <a name="about-azure-storage-encryption"></a>À propos du chiffrement de Stockage Azure

Les données dans Stockage Azure sont chiffrées et déchiffrées en toute transparence à l’aide du [chiffrement AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 256 bits, un des chiffrements par blocs les plus puissants actuellement disponibles, et sont conformes à la norme FIPS 140-2. Le chiffrement du Stockage Azure est similaire au chiffrement BitLocker sur Windows.

Le chiffrement de Stockage Azure est activé pour tous les comptes de stockage, y compris les comptes de stockage classiques et Resource Manager. Le chiffrement de Stockage Azure ne peut pas être désactivé. Étant donné que vos données sont sécurisées par défaut, vous n’avez pas besoin de modifier votre code ou vos applications pour tirer parti du chiffrement du Stockage Azure.

Les données d’un compte de stockage sont chiffrées, quel que soit le niveau de performance (Standard ou Premium), le niveau d’accès (chaud ou froid) ou le modèle de déploiement (Azure Resource Manager ou Classique). Tous les objets blob du niveau Archive sont également chiffrés. Toutes les options de redondance de Stockage Azure prennent en charge le chiffrement, et toutes les données dans les régions primaire et secondaire sont chiffrées quand la géoréplication est activée. Toutes les ressources de Stockage Azure sont chiffrées, y compris les objets blob, disques, fichiers, files d’attente et tables. Toutes les métadonnées d’objet sont également chiffrées. Le chiffrement de Stockage Azure n’implique aucun coût supplémentaire.

Chaque objet blob de blocs, objet blob d’ajout ou objet blob de pages qui a été écrit dans le Stockage Azure après le 20 octobre 2017 est chiffré. Les objets blob créés avant cette date continuent à être chiffrés à l’aide d’un processus en arrière-plan. Pour forcer le chiffrement d’un objet blob qui a été créé avant le 20 octobre 2017, vous pouvez réécrire l’objet. Pour savoir comment vérifier l’état de chiffrement d’un objet blob, consultez [Vérifier l’état de chiffrement d’un objet blob](../blobs/storage-blob-encryption-status.md).

Pour plus d’informations sur les modules cryptographiques de chiffrement de Stockage Azure, consultez [API de chiffrement : nouvelle génération](/windows/desktop/seccng/cng-portal).

Pour plus d’informations sur le chiffrement et la gestion des clés pour les disques managés Azure, consultez [Chiffrement côté serveur de disques managés Azure](../../virtual-machines/disk-encryption.md) pour les machines virtuelles Windows ou [Chiffrement côté serveur de disques managés Azure](../../virtual-machines/disk-encryption.md) pour les machines virtuelles Linux.

## <a name="about-encryption-key-management"></a>À propos de la gestion des clés de chiffrement

Par défaut, les données d'un nouveau compte de stockage sont chiffrées à l'aide de clés managées par Microsoft. Vous pouvez continuer à vous reposer sur les clés managées par Microsoft pour le chiffrement des vos données, ou vous pouvez gérer le chiffrement avec vos propres clés. Si vous choisissez de gérer le chiffrement avec vos propres clés, deux options s’offrent à vous. Vous pouvez utiliser l'un ou l'autre type de gestion des clés, ou les deux :

- Vous pouvez spécifier une *clé gérée par le client* à utiliser pour le chiffrement et le déchiffrement des données dans le stockage d’objets blob et dans Azure Files. <sup>1, 2</sup> clés gérées par le client doivent être stockées dans Azure Key Vault ou le modèle de sécurité matérielle (HSM) Azure Key Vault géré (préversion). Pour plus d’informations sur les clés gérées par le client, consultez [Utiliser des clés gérées par le client pour gérer le chiffrement du stockage Azure](./customer-managed-keys-overview.md).
- Vous pouvez spécifier une *clé fournie par le client* sur les opérations de stockage Blob. Un client qui effectue une requête de lecture ou d’écriture sur le stockage Blob peut inclure une clé de chiffrement dans la requête afin de contrôler la précision avec laquelle les données blob sont chiffrées et déchiffrées. Pour plus d’informations sur les clés fournies par le client, consultez [Fournir une clé de chiffrement lors d’une requête au stockage d’objets blob](../blobs/encryption-customer-provided-keys.md).

Le tableau suivant compare les options de gestion de clés pour le chiffrement de Stockage Azure.

| Paramètre de gestion des clés | Clés managées par Microsoft | Clés gérées par le client | Clés fournies par le client |
|--|--|--|--|
| Opérations de chiffrement/déchiffrement | Azure | Azure | Azure |
| Services de stockage Azure pris en charge | Tous | Stockage Blob, Azure Files<sup>1,2</sup> | Stockage d'objets blob |
| Stockage des clés | Magasin de clés Microsoft | Azure Key Vault ou HSM Key Vault | Magasin de clés du client |
| Responsabilité de la permutation des clés | Microsoft | Customer | Customer |
| Contrôle des clés | Microsoft | Customer | Customer |

<sup>1</sup> Pour plus d’informations sur la création d’un compte qui prend en charge l’utilisation de clés gérées par le client avec Stockage File d’attente, consultez [Créer un compte qui prend en charge les clés gérées par le client pour les files d’attente](account-encryption-key-create.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json).<br />
<sup>2</sup> Pour plus d’informations sur la création d’un compte qui prend en charge l’utilisation de clés gérées par le client avec Stockage Table, consultez [Créer un compte qui prend en charge les clés gérées par le client pour les tables](account-encryption-key-create.md?toc=%2fazure%2fstorage%2ftables%2ftoc.json).

> [!NOTE]
> Les clés gérées par Microsoft font l'objet d'une rotation appropriée en fonction des exigences de conformité. Si vous avez des exigences spécifiques en matière de rotation des clés, Microsoft vous recommande de passer aux clés gérées par le client afin de pouvoir gérer et vérifier vous-même la rotation.

## <a name="doubly-encrypt-data-with-infrastructure-encryption"></a>Chiffrer les données doublement avec le chiffrement de l’infrastructure

Les clients qui doivent s’assurer que leurs données sont sécurisées peuvent également activer le chiffrement AES 256 bits au niveau de l’infrastructure Stockage Azure. Lorsque le chiffrement d’infrastructure est activé, les données d’un compte de stockage sont chiffrées deux fois &mdash; une fois au niveau du service et une fois au niveau de l’infrastructure &mdash; avec deux algorithmes de chiffrement et deux clés différents. Le double chiffrement des données Stockage Azure permet d’éviter un scénario impliquant une possible compromission d’un algorithme ou d’une clé de chiffrement. Dans un tel scénario, la couche de chiffrement supplémentaire continue de protéger vos données.

Le chiffrement au niveau du service prend en charge l’utilisation de clés gérées par Microsoft ou de clés gérées par le client avec Azure Key Vault. Le chiffrement au niveau de l’infrastructure s’appuie sur des clés gérées par Microsoft et utilise systématiquement une clé distincte.

Pour plus d’informations sur la création d’un compte de stockage qui permet le chiffrement de l’infrastructure, consultez [Créer un compte de stockage avec le chiffrement d’infrastructure activé à des fins de double chiffrement des données](infrastructure-encryption-enable.md).

## <a name="encryption-scopes-for-blob-storage-preview"></a>Étendues de chiffrement pour le stockage d’objets blob (version préliminaire)

Par défaut, un compte de stockage est chiffré avec une clé étendue au compte de stockage. Vous pouvez choisir d’utiliser des clés gérées par Microsoft ou des clés gérées par le client stockées dans Azure Key Vault pour protéger et contrôler l’accès à la clé qui chiffre vos données.

Les étendues de chiffrement vous permettent de gérer le chiffrement au niveau d’un objet blob ou d’un conteneur individuel. Vous pouvez utiliser des étendues de chiffrement pour créer des limites sécurisées entre les données qui résident dans le même compte de stockage mais qui appartiennent à des clients différents.

Vous pouvez créer une ou plusieurs étendues de chiffrement pour un compte de stockage à l’aide du fournisseur de ressources de stockage Azure. Lorsque vous créez une étendue de chiffrement, vous spécifiez si l’étendue est protégée par une clé gérée par Microsoft ou par une clé gérée par le client qui est stockée dans Azure Key Vault. Différentes étendues de chiffrement sur le même compte de stockage peuvent utiliser des clés gérées par Microsoft ou par le client.

Une fois que vous avez créé une étendue de chiffrement, vous pouvez spécifier cette étendue de chiffrement sur une requête de création d’un conteneur ou d’un objet blob. Pour plus d’informations sur la création d’une étendue de chiffrement, consultez [Créer et gérer des étendues de chiffrement (version préliminaire)](../blobs/encryption-scope-manage.md).

> [!NOTE]
> Les étendues de chiffrement ne sont pas prises en charge avec les comptes de stockage géographiquement redondant avec accès en lecture (RA-GRS) et de stockage géographiquement redondant interzone avec accès en lecture (RA-GZRS) dans le cadre de la préversion.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

> [!IMPORTANT]
> Cette version préliminaire est destinée uniquement à une utilisation hors production. Les contrats SLA (contrats de niveau de service) de production ne sont actuellement pas disponibles.
>
> Pour éviter les coûts inattendus, veillez à désactiver les étendues de chiffrement dont vous n’avez pas besoin.

### <a name="create-a-container-or-blob-with-an-encryption-scope"></a>Créer un conteneur ou un blob avec une étendue de chiffrement

Les objets blob créés dans le cadre d’une étendue de chiffrement sont chiffrés avec la clé spécifiée pour cette étendue. Vous pouvez spécifier une étendue de chiffrement pour un objet blob individuel lorsque vous le créez l’objet, ou vous pouvez spécifier une étendue de chiffrement par défaut lorsque vous créez un conteneur. Quand une étendue de chiffrement par défaut est spécifiée au niveau d’un conteneur, tous les objets blob de ce conteneur sont chiffrés avec la clé associée à l’étendue par défaut.

Lorsque vous créez un objet blob dans un conteneur qui a une étendue de chiffrement par défaut, vous pouvez spécifier une étendue de chiffrement qui remplace l’étendue de chiffrement par défaut si le conteneur est configuré pour autoriser les remplacements de l’étendue de chiffrement par défaut. Pour empêcher les remplacements de l’étendue de chiffrement par défaut, configurez le conteneur pour refuser les remplacements pour un objet blob individuel.

Les opérations de lecture sur un objet blob qui appartient à une étendue de chiffrement se produisent de façon transparente, tant que l’étendue de chiffrement n’est pas désactivée.

### <a name="disable-an-encryption-scope"></a>Désactiver une étendue de chiffrement

Lorsque vous désactivez une étendue de chiffrement, toutes les opérations de lecture ou d’écriture ultérieures effectuées avec l’étendue de chiffrement échouent avec le code d’erreur HTTP 403 (Interdit). Si vous réactivez l’étendue de chiffrement, les opérations de lecture et d’écriture se poursuivent normalement.

Quand une étendue de chiffrement est désactivée, vous n’êtes plus facturé pour celle-ci. Désactivez les étendues de chiffrement qui ne sont pas nécessaires pour éviter les frais inutiles.

Si votre étendue de chiffrement est protégée par des clés gérées par le client pour Azure Key Vault, vous pouvez également supprimer la clé associée dans le coffre de clés afin de désactiver l’étendue de chiffrement. Gardez à l’esprit que les clés gérées par le client dans Azure Key Vault sont protégées par la suppression réversible et la protection de la suppression, et une clé supprimée est soumise au comportement défini par ces propriétés. Pour plus d'informations, consultez les rubriques suivantes dans la documentation d’Azure Key Vault :

- [Guide pratique pour utiliser la suppression réversible avec Power​Shell](../../key-vault/general/key-vault-recovery.md)
- [Guide pratique pour utiliser la suppression réversible avec Azure CLI](../../key-vault/general/key-vault-recovery.md)

> [!NOTE]
> Il n’est pas possible de supprimer une étendue de chiffrement.

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md)
- [Clés gérées par le client pour le chiffrement du Stockage Azure](customer-managed-keys-overview.md)
- [Étendues de chiffrement pour le stockage d’objets blob (version préliminaire)](../blobs/encryption-scope-overview.md)
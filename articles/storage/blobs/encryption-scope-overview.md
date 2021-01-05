---
title: Étendues de chiffrement pour le stockage d’objets blob (version préliminaire)
description: Les étendues de chiffrement permettent de gérer le chiffrement au niveau d’un objet blob ou d’un conteneur individuel. Vous pouvez utiliser des étendues de chiffrement pour créer des limites sécurisées entre les données qui résident dans le même compte de stockage mais qui appartiennent à des clients différents.
services: storage
author: tamram
ms.service: storage
ms.date: 09/22/2020
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 881a7fc915ab986577599b85f8412fa8107f7902
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017387"
---
# <a name="encryption-scopes-for-blob-storage-preview"></a>Étendues de chiffrement pour le stockage d’objets blob (version préliminaire)

Les étendues de chiffrement permettent de gérer le chiffrement au niveau d’un objet blob ou d’un conteneur individuel. Vous pouvez utiliser des étendues de chiffrement pour créer des limites sécurisées entre les données qui résident dans le même compte de stockage mais qui appartiennent à des clients différents.

Par défaut, un compte de stockage est chiffré avec une clé étendue au compte de stockage entier. Avec une étendue de chiffrement, vous pouvez spécifier qu’un ou plusieurs conteneurs soient chiffrés avec une clé étendue à ces seuls conteneurs.

Vous pouvez choisir d’utiliser des clés gérées par Microsoft ou des clés gérées par le client stockées dans Azure Key Vault pour protéger et contrôler l’accès à la clé qui chiffre vos données. Différentes étendues de chiffrement sur le même compte de stockage peuvent utiliser des clés gérées par Microsoft ou par le client.

Une fois que vous avez créé une étendue de chiffrement, vous pouvez spécifier cette étendue de chiffrement sur une requête de création d’un conteneur ou d’un objet blob. Pour plus d’informations sur la création d’une étendue de chiffrement, consultez [Créer et gérer des étendues de chiffrement (version préliminaire)](encryption-scope-manage.md).

> [!NOTE]
> Les étendues de chiffrement ne sont pas prises en charge avec les comptes de stockage géographiquement redondant avec accès en lecture (RA-GRS) ou de stockage géographiquement redondant interzone avec accès en lecture (RA-GZRS) dans le cadre de la préversion.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

> [!IMPORTANT]
> Cette version préliminaire est destinée uniquement à une utilisation hors production. Les contrats SLA (contrats de niveau de service) de production ne sont actuellement pas disponibles.
>
> Pour éviter les coûts inattendus, veillez à désactiver les étendues de chiffrement dont vous n’avez pas besoin.

## <a name="create-a-container-or-blob-with-an-encryption-scope"></a>Créer un conteneur ou un blob avec une étendue de chiffrement

Les objets blob créés dans le cadre d’une étendue de chiffrement sont chiffrés avec la clé spécifiée pour cette étendue. Vous pouvez spécifier une étendue de chiffrement pour un objet blob individuel lorsque vous le créez l’objet, ou vous pouvez spécifier une étendue de chiffrement par défaut lorsque vous créez un conteneur. Quand une étendue de chiffrement par défaut est spécifiée au niveau d’un conteneur, tous les objets blob de ce conteneur sont chiffrés avec la clé associée à l’étendue par défaut.

Lorsque vous créez un objet blob dans un conteneur qui a une étendue de chiffrement par défaut, vous pouvez spécifier une étendue de chiffrement qui remplace l’étendue de chiffrement par défaut si le conteneur est configuré pour autoriser les remplacements de l’étendue de chiffrement par défaut. Pour empêcher les remplacements de l’étendue de chiffrement par défaut, configurez le conteneur pour refuser les remplacements pour un objet blob individuel.

Les opérations de lecture sur un objet blob qui appartient à une étendue de chiffrement se produisent de façon transparente, tant que l’étendue de chiffrement n’est pas désactivée.

## <a name="disable-an-encryption-scope"></a>Désactiver une étendue de chiffrement

Lorsque vous désactivez une étendue de chiffrement, toutes les opérations de lecture ou d’écriture ultérieures effectuées avec l’étendue de chiffrement échouent avec le code d’erreur HTTP 403 (Interdit). Si vous réactivez l’étendue de chiffrement, les opérations de lecture et d’écriture se poursuivent normalement.

Quand une étendue de chiffrement est désactivée, vous n’êtes plus facturé pour celle-ci. Désactivez les étendues de chiffrement qui ne sont pas nécessaires pour éviter les frais inutiles.

Si votre étendue de chiffrement est protégée par des clés gérées par le client, vous pouvez également supprimer la clé associée dans le coffre de clés afin de désactiver l’étendue de chiffrement. Gardez à l’esprit que les clés gérées par le client sont protégées par la suppression réversible et la protection contre le vidage dans le coffre de clés, et qu’une clé supprimée est soumise au comportement défini par ces propriétés. Pour plus d'informations, consultez les rubriques suivantes dans la documentation d’Azure Key Vault :

- [Guide pratique pour utiliser la suppression réversible avec Power​Shell](../../key-vault/general/key-vault-recovery.md)
- [Guide pratique pour utiliser la suppression réversible avec Azure CLI](../../key-vault/general/key-vault-recovery.md)

> [!NOTE]
> Il n’est pas possible de supprimer une étendue de chiffrement.

## <a name="next-steps"></a>Étapes suivantes

- [Chiffrement du stockage Azure pour les données au repos](../common/storage-service-encryption.md)
- [Créer et gérer des étendues de chiffrement (version préliminaire)](encryption-scope-manage.md)
- [Clés gérées par le client pour le chiffrement du service Stockage Azure](../common/customer-managed-keys-overview.md)
- [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md)
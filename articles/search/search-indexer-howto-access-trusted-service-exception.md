---
title: Accès d’indexeur au service Stockage Azure à l’aide d’une exception de service approuvée
titleSuffix: Azure Cognitive Search
description: Autorisez un accès aux données stockées de manière sécurisée dans le service Stockage Azure via un indexeur dans le service Recherche cognitive Azure.
manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: e139c15ef6de00376a4e1a88000d263c3486994b
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92101373"
---
# <a name="indexer-access-to-azure-storage-using-the-trusted-service-exception-azure-cognitive-search"></a>Accès d’indexeur au service Stockage Azure à l’aide de l’exception de service approuvé (Recherche cognitive Azure)

Les indexeurs d’un service Recherche cognitive Azure qui accèdent aux données dans des comptes Stockage Azure peuvent utiliser la capacité d’[exception de service approuvé](../storage/common/storage-network-security.md#exceptions) pour accéder aux données de manière sécurisée. Ce mécanisme offre aux clients dans l’impossibilité d’accorder un [accès aux indexeurs en utilisant des règles de pare-feu IP](search-indexer-howto-access-ip-restricted.md) une alternative simple, sécurisée et gratuite pour accéder aux données de comptes de stockage.

> [!NOTE]
> La prise en charge de l’accès aux données dans les comptes de stockage par le biais d’une exception de service approuvé est limitée aux solutions Stockage Blob Azure et Azure Data Lake Storage Gen2. Le stockage de tables Azure n’est pas pris en charge.

## <a name="step-1-configure-a-connection-using-a-managed-identity"></a>Étape 1 : Configurer une connexion à l’aide d’une identité managée

Suivez les instructions de la page [Configurer une connexion à un compte Stockage Azure à l’aide d’une identité managée](search-howto-managed-identities-storage.md). Lorsque vous avez terminé, vous aurez inscrit votre service de recherche auprès de Azure Active Directory en tant que service approuvé, et vous aurez accordé des autorisations dans le stockage Azure qui donne à l’identité de recherche des droits spécifiques pour accéder aux données ou aux informations.

## <a name="step-2-allow-trusted-microsoft-services-to-access-the-storage-account"></a>Étape 2 : Autoriser les services Microsoft approuvés à accéder au compte de stockage

Sur le portail Azure, accédez à l’onglet **Pare-feu et réseaux virtuels** du compte de stockage. Vérifiez que l’option **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage** est cochée. Avec cette option, seule l’instance du service de recherche spécifique dotée de l’accès en fonction du rôle approprié au compte de stockage (authentification forte) est autorisée à accéder aux données du compte de stockage, même s’il est sécurisé par des règles de pare-feu IP.

![Exception de service approuvé](media\search-indexer-howto-secure-access\exception.png "Exception de service approuvé")

Les indexeurs pourront désormais accéder aux données du compte de stockage, même si celui-ci est sécurisé par des règles de pare-feu IP.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les indexeurs de Stockage Azure :

- [Indexeur de blobs Azure](search-howto-indexing-azure-blob-storage.md)
- [Indexeur Azure Data Lake Storage Gen2](search-howto-index-azure-data-lake-storage.md)
- [Indexeur de tables Azure](search-howto-indexing-azure-tables.md)
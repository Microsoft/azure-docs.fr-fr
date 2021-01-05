---
title: Effectuer des recherches dans Azure Data Lake Storage Gen2 (version préliminaire)
titleSuffix: Azure Cognitive Search
description: Découvrez comment indexer le contenu et les métadonnées dans Azure Data Lake Storage Gen2. Cette fonctionnalité est actuellement en préversion publique.
manager: nitinme
author: markheff
ms.author: maheff
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/11/2020
ms.openlocfilehash: 4a585fa1c3c56d285859171e6b5673fe57beffb3
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92517319"
---
# <a name="indexing-documents-in-azure-data-lake-storage-gen2"></a>Indexation de documents dans Azure Data Lake Storage Gen2

> [!IMPORTANT] 
> La prise en charge d’Azure Data Lake Storage Gen2 est actuellement en préversion publique. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Vous pouvez demander l’accès aux préversions en remplissant [ce formulaire](https://aka.ms/azure-cognitive-search/indexer-preview). Les [versions d’API REST 2020-06-30-preview](search-api-preview.md) et le portail offrent cette fonctionnalité. Le SDK .NET n’est actuellement pas pris en charge.


Lorsque vous configurez un compte de stockage Azure, vous avez la possibilité d’activer un [espace de noms hiérarchique](../storage/blobs/data-lake-storage-namespace.md). Cela permet d’organiser la collection du contenu d’un compte dans une hiérarchie de répertoires et de sous-répertoires imbriqués. En activant l’espace de noms hiérarchique, vous activez [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md).

Cet article explique comment prendre en main l’indexation des documents qui se trouvent dans Azure Data Lake Storage Gen2.

## <a name="set-up-azure-data-lake-storage-gen2-indexer"></a>Configurer l’indexeur Azure Data Lake Storage Gen2

Vous devez effectuer quelques étapes pour indexer le contenu de Data Lake Storage Gen2.

### <a name="step-1-sign-up-for-the-preview"></a>Étape 1 : S’inscrire à la version préliminaire

Inscrivez-vous à la préversion de l’indexeur Data Lake Storage Gen2 en remplissant [ce formulaire](https://aka.ms/azure-cognitive-search/indexer-preview). Vous recevrez un e-mail de confirmation une fois que vous avez été accepté dans la préversion.

### <a name="step-2-follow-the-azure-blob-storage-indexing-setup-steps"></a>Étape 2 : Suivre les étapes de configuration de l’indexation du stockage d’objets blob Azure

Une fois que vous avez reçu la confirmation de la réussite de votre l’inscription à la préversion, vous êtes prêt à créer le pipeline d’indexation.

Vous pouvez indexer le contenu et les métadonnées de Data Lake Storage Gen2 à l’aide de l’[’API REST version 2020-06-30-preview](search-api-preview.md) ou le portail. Il n’y a pas de prise en charge de .NET SDK pour l’instant.

L’indexation du contenu dans Data Lake Storage Gen2 est identique à l’indexation du contenu dans le stockage d’objets Blob Azure. Pour savoir comment configurer la source de données, l’index et l’indexeur Data Lake Storage Gen2, consultez [Comment indexer des documents dans Stockage Blob Azure avec la Recherche cognitive Azure](search-howto-indexing-azure-blob-storage.md). L’article relatif au stockage d’objets Blob fournit également des informations sur les formats de document pris en charge, les propriétés de métadonnées d’objet Blob extraites, l’indexation incrémentielle, et bien d’autres encore. Ces informations seront identiques pour Data Lake Storage Gen2.

## <a name="access-control"></a>Contrôle d’accès

Azure Data Lake Storage Gen2 implémente un [modèle de contrôle d’accès](../storage/blobs/data-lake-storage-access-control.md) qui prend en charge le contrôle d’accès en fonction du rôle (Azure RBAC) et les listes de contrôle d’accès (ACL) POSIX. Lorsque vous indexez du contenu à partir de Data Lake Storage Gen2, Recherche cognitive Azure n’extraira pas les informations Azure RBAC et ACL du contenu. Par conséquent, ces informations ne seront pas incluses dans votre index Recherche cognitive Azure.

Si la gestion du contrôle d’accès sur chaque document de l’index est importante, le développeur de l’application doit implémenter un [filtrage de sécurité](./search-security-trimming-for-azure-search.md).

## <a name="change-detection"></a>Détection des changements

L’indexeur Data Lake Storage Gen2 prend en charge la détection des modifications. Cela signifie que, lorsque l’indexeur s’exécute, il réindexe uniquement les objets blob modifiés comme déterminé par l’horodateur `LastModified` de l’objet blob.

> [!NOTE] 
> Data Lake Storage Gen2 permet de renommer les répertoires. Quand un répertoire est renommé, les horodateurs des objets blob qu’il contient ne sont pas mis à jour. Par conséquent, l’indexeur ne réindexe pas ces objets blob. Si vous avez besoin que les objets blob d’un répertoire soient réindexés après le changement de nom de celui-ci parce que leurs URL ont changé, vous devez mettre à jour l’horodateur `LastModified` pour tous les objets blob dans l’annuaire afin que l’indexeur sache les réindexer lors d’une exécution ultérieure.
---
title: Problèmes connus avec Azure Data Lake Storage Gen2 | Microsoft Docs
description: Apprenez-en davantage sur les limitations et les problèmes connus d’Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 10/28/2020
ms.author: normesta
ms.reviewer: jamesbak
ms.openlocfilehash: 32f8ab3f7e222108bec4ec81b14c113705370096
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95913060"
---
# <a name="known-issues-with-azure-data-lake-storage-gen2"></a>Problèmes connus avec Azure Data Lake Storage Gen2

Cet article décrit les limitations et les problèmes connus d’Azure Data Lake Storage Gen2.

## <a name="supported-blob-storage-features"></a>Fonctionnalités du stockage Blob prises en charge

Les comptes ayant un espace de noms hiérarchique prennent en charge un nombre croissant de fonctionnalités de stockage Blob. Pour en obtenir la liste complète, consultez [Fonctionnalités de stockage blob disponibles dans Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md).

## <a name="supported-azure-service-integrations"></a>Intégrations de service Azure prises en charge

Azure Data Lake Storage Gen2 prend en charge plusieurs services Azure permettant d’ingérer des données, d’obtenir des données d’analytique et de créer des représentations visuelles. Pour obtenir la liste des services Azure pris en charge, consultez [Services Azure prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md).

Consultez [Services Azure prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md).

## <a name="supported-open-source-platforms"></a>Plateformes open source prises en charge

Plusieurs plateformes open source prennent en charge le stockage Data Lake Gen2. Pour obtenir une liste complète, consultez [Plateformes open source prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md).

Consultez [Plateformes open source prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md).

## <a name="blob-storage-apis"></a>API Stockage Blob

Les API Blob et les API Data Lake Storage Gen2 peuvent fonctionner sur les mêmes données.

Cette section décrit les problèmes et les limitations liés à l’utilisation des API d’objets BLOB et des API Data Lake Storage Gen 2 pour fonctionner sur les mêmes données.

* Il n’est pas possible d’utiliser l’API Blob et les API Data Lake Storage pour écrire dans la même instance d’un fichier. Si vous écrivez dans un fichier à l’aide des API Data Lake Storage Gen 2, les blocs de ce fichier ne seront pas visibles pour les appels à l’API [Obtenir la liste de bloc](/rest/api/storageservices/get-block-list) d’objets BLOB. La seule exception concerne les cas de remplacement. Il est en effet possible de remplacer un fichier ou un objet blob à l’aide de n’importe quelle API.

* Lorsque vous utilisez l’opération [Lister les objets BLOB](/rest/api/storageservices/list-blobs) sans spécifier de délimiteur, les résultats incluront à la fois des répertoires et des objets BLOB. Si vous choisissez d’utiliser un délimiteur, n’utilisez qu’une barre oblique (`/`). Il s’agit du seul délimiteur pris en charge.

* Si vous utilisez l’API [Supprimer un objet BLOB](/rest/api/storageservices/delete-blob) pour supprimer un répertoire, ce répertoire est supprimé uniquement s’il est vide. Cela signifie que vous ne pouvez pas utiliser les répertoires de suppression de l’API d’objet BLOB de manière récursive.

Ces API REST BLOB ne sont pas prises en charge :

* [Placer BLOB (Page)](/rest/api/storageservices/put-blob)
* [Put Page](/rest/api/storageservices/put-page)
* [Obtenir les portées de page](/rest/api/storageservices/get-page-ranges)
* [Copie incrémentielle BLOB](/rest/api/storageservices/incremental-copy-blob)
* [Placer la page à partir de l’URL](/rest/api/storageservices/put-page-from-url)
* [Append Block](/rest/api/storageservices/append-block)
* [Ajouter un bloc à partir d’une URL](/rest/api/storageservices/append-block-from-url)


Les disques de machine virtuelle non gérés ne sont pas pris en charge dans les comptes qui ont un espace de noms hiérarchique. Si vous souhaitez activer un espace de noms hiérarchique sur un compte de stockage, placez les disques de machine virtuelle non gérés dans un compte de stockage pour lequel la fonctionnalité espace de noms hiérarchique n’est pas activée.

<a id="api-scope-data-lake-client-library"></a>

## <a name="support-for-setting-access-control-lists-acls-recursively"></a>Prise en charge de la définition de listes de contrôle d’accès (ACL) de manière récursive

La possibilité d’appliquer les modifications aux listes ACL de manière récursive du répertoire parent vers les éléments enfants est mise à la disposition générale. Dans la version actuelle de cette fonctionnalité, vous pouvez appliquer des modifications aux listes ACL à l’aide de PowerShell, d’Azure CLI, et du Kit de développement logiciel (SDK) .NET, Java et Python. La prise en charge n’est pas encore disponible pour le portail Azure ou l’Explorateur Stockage Azure.

<a id="known-issues-tools"></a>

## <a name="azcopy"></a>AzCopy

Utilisez uniquement la dernière version d’AzCopy ([AzCopy v10](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ftables%2ftoc.json)). Les versions antérieures d’AzCopy, telles qu’AzCopy v8.1, ne sont pas prises en charge.

<a id="storage-explorer"></a>

## <a name="azure-storage-explorer"></a>Explorateur de stockage Azure

Utilisez uniquement les versions `1.6.0` ou ultérieures.

<a id="explorer-in-portal"></a>

## <a name="storage-explorer-in-the-azure-portal"></a>Explorateur Stockage dans le portail Azure

Les ACL ne sont pas encore prises en charge.

<a id="third-party-apps"></a>

## <a name="third-party-applications"></a>Applications tierces

Les applications tierces qui utilisent les API REST continueront à fonctionner si vous les utilisez avec Data Lake Storage Gen2. Les applications qui appellent des API Blob fonctionneront probablement.

## <a name="access-control-lists-acl-and-anonymous-read-access"></a>Listes de contrôle d’accès (ACL) et accès en lecture anonyme

Si l’[accès en lecture anonyme](./anonymous-read-access-configure.md) a été accordé à un conteneur, les listes de contrôle d’accès n’ont aucun effet sur ce conteneur ou les fichiers de ce conteneur.

## <a name="diagnostic-logs"></a>Journaux de diagnostic

Le réglage du nombre de jours de rétention n’est pas encore pris en charge, mais vous pouvez supprimer les journaux manuellement à l’aide de n’importe quel outil pris en charge, comme Explorateur Stockage Azure, REST ou un Kit de développement logiciel (SDK).

## <a name="lifecycle-management-policies-with-premium-tier-for-azure-data-lake-storage"></a>Stratégies de gestion du cycle de vie avec niveau Premium pour Azure Data Lake Storage

Il n’est pas possible de déplacer des données stockées dans le niveau Premium entre les niveaux chaud, froid et archive. Vous pouvez toutefois copier des données du niveau Premium vers le niveau d’accès chaud dans un autre compte.

## <a name="dremio-support-with-premium-performance-blockblobstorage-storage-accounts"></a>Prise en charge Dremio avec des comptes de stockage BlockBlobStorage de niveau de performance Premium

Dremio ne se connecte pas encore à un compte BlockBlobStorage sur lequel la fonctionnalité d’espace de noms hiérarchique est activée. 

## <a name="windows-azure-storage-blob-wasb-driver-unsupported-with-data-lake-storage-gen2"></a>Pilote Windows Azure Storage Blob (WASB) (non pris en charge avec Data Lake Storage Gen2)

Actuellement, le pilote WASB, qui a été conçu pour fonctionner avec l’API Blob uniquement, rencontre des problèmes dans quelques scénarios courants. C’est le cas en particulier quand il s’agit d’un client pour un compte de stockage prenant en charge un espace de noms hiérarchique. L’accès multiprotocole sur Data Lake Storage n’atténue pas ces problèmes. 

Pour le moment (et probablement pour très longtemps), nous ne prenons pas en charge les utilisateurs qui utilisent le pilote WASB en tant que client pour un compte de stockage prenant en charge un espace de noms hiérarchique. Nous vous recommandons plutôt d’utiliser le pilote [Azure Blob File System (ABFS)](data-lake-storage-abfs-driver.md) dans votre environnement Hadoop. Si vous tentez d’effectuer une migration à partir d’un environnement Hadoop local avec une version antérieure à Hadoop Branch-3, ouvrez un ticket de support Azure pour que nous puissions vous contacter et vous indiquer la bonne direction pour vous et votre organisation.
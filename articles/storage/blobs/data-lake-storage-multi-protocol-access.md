---
title: Accès multiprotocole sur Azure Data Lake Storage | Microsoft Docs
description: Utilisez des API d’objets BLOB et des applications qui utilisent des API d’objets BLOB avec Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 02/25/2020
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: e3997fc215637175165402a926bffc6ac8d02771
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77914856"
---
# <a name="multi-protocol-access-on-azure-data-lake-storage"></a>Accès multi-protocole pour Azure Data Lake Storage

Les API d’objets BLOB fonctionnent désormais avec les comptes qui ont un espace de noms hiérarchique. Cela déverrouille de l’écosystème des outils, des applications et des services, ainsi que de nombreuses fonctionnalités de stockage d’objets BLOB pour les comptes qui ont un espace de noms hiérarchique.

Jusqu’à récemment, vous avez peut-être dû maintenir des solutions de stockage distinctes pour le stockage des objets et l’analyse du stockage. Cela est dû au fait que Azure Data Lake Storage Gen2 avait une prise en charge limitée de l’écosystème. Il avait également un accès limité aux fonctionnalités du service BLOB, telles que la journalisation des diagnostics. Une solution de stockage fragmenté est difficile à maintenir, car vous devez déplacer les données entre les comptes pour effectuer différents scénarios. Vous n’avez plus à le faire.

Avec l’accès multi-protocole sur Data Lake Storage, vous pouvez travailler avec vos données à l’aide de l’écosystème d’outils, d’applications et de services. Cela comprend également des outils et des applications tiers. Vous pouvez les rediriger vers des comptes qui ont un espace de noms hiérarchique sans avoir à les modifier. Ces applications fonctionnent *comme telles* même si elles appellent des API d’objets BLOB, car les API d’objets BLOB peuvent désormais fonctionner sur des données de comptes qui ont un espace de noms hiérarchique.

Les fonctionnalités de stockage d’objets BLOB, telles que la [journalisation des diagnostics](../common/storage-analytics-logging.md), les [niveaux d’accès](storage-blob-storage-tiers.md) et les [stratégies de gestion du cycle de vie du stockage BLOB](storage-lifecycle-management-concepts.md), fonctionnent désormais avec les comptes qui ont un espace de noms hiérarchique. Par conséquent, vous pouvez activer des espaces de noms hiérarchiques sur vos comptes de stockage d’objets BLOB sans perdre l’accès à ces fonctionnalités importantes. 

> [!NOTE]
> L’accès multi-protocole sur Data Lake Storage est en disponibilité générale dans toutes les régions. Certains services Azure et certaines fonctionnalités de stockage BLOB que l’accès multiprotocole active restent en version préliminaire.  Ces articles résument la prise en charge actuelle des fonctionnalités de stockage d’objets BLOB et des intégrations de service Azure. 
>
> [Fonctionnalités de stockage d'objets BLOB disponibles dans Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md)
>
>[Services Azure prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md)

## <a name="how-multi-protocol-access-on-data-lake-storage-works"></a>Comment fonctionne l’accès multi-protocole pour Azure Data Lake Storage

Les API d’objets BLOB et les API Data Lake Storage Gen2 peuvent fonctionner sur les mêmes données dans les comptes de stockage qui ont un espace de noms hiérarchique. Data Lake Storage Gen2 achemine les API d’objets BLOB via l’espace de noms hiérarchique afin que vous puissiez bénéficier des avantages des premières opérations de répertoire de classe et des listes de contrôle d’accès (ACL) compatibles avec POSIX. 

![Accès multi-protocole sur Data Lake Storage conceptuelle](./media/data-lake-storage-interop/interop-concept.png) 

Les outils et les applications existants qui utilisent l’API BLOB bénéficient automatiquement de ces avantages. Les développeurs ne sont pas obligés de les modifier. Data Lake Storage Gen2 applique de manière cohérente les listes de contrôle d’accès au niveau des répertoires et des fichiers, quel que soit le protocole utilisé par les outils et les applications pour accéder aux données. 

## <a name="see-also"></a>Voir aussi

- [Fonctionnalités de stockage d'objets BLOB disponibles dans Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md)
- [Services Azure prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-azure-services.md)
- [Plateformes open source prenant en charge Azure Data Lake Storage Gen2](data-lake-storage-supported-open-source-platforms.md)
- [Problèmes connus avec Azure Data Lake Storage Gen2](data-lake-storage-known-issues.md)





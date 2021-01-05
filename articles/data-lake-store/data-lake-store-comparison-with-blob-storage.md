---
title: Comparaison d’Azure Data Lake Storage Gen1 et du Stockage Blob
description: Découvrez les différences entre Azure Data Lake Storage Gen1 et le service Stockage Blob Azure pour certains aspects essentiels du traitement de Big Data.
author: twooley
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: twooley
ms.openlocfilehash: 77ac3c0809c08719d77457c59ef311ad43ef99cd
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92108335"
---
# <a name="comparing-azure-data-lake-storage-gen1-and-azure-blob-storage"></a>Comparaison d’Azure Data Lake Storage Gen1 et d’Azure Storage Blob

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)] 

Le tableau de cet article résume les différences entre Azure Data Lake Storage Gen1 et Azure Storage Blob pour certains aspects essentiels du traitement des données volumineuses. Azure Blob Storage est un magasin d’objets extensible généraliste conçu pour s’adapter à une large gamme de scénarios de stockage. Azure Data Lake Storage Gen1 est un référentiel à très grande échelle qui est optimisé pour les charges de travail d’analyse du Big Data.

| Category | Azure Data Lake Storage Gen1 | Stockage Blob Azure |
| -------- | ---------------------------- | ------------------ |
| Objectif |Stockage optimisé pour les charges de travail d’analyse de données volumineuses |Magasin d’objets polyvalent adapté à un large éventail de scénarios de stockage, y compris l’analyse Big Data |
| Cas d'utilisation |Données par lots, interactives, d’analyse de diffusion en continu et d’apprentissage machine (par exemple, fichiers journaux, données IoT, données sur le parcours de navigation, jeux de données volumineux) |N’importe quel type de données texte ou binaires, par exemple données d’application backend, de sauvegarde, de stockage de médias pour le streaming, et d’usage général Prise en charge complète des charges de travail d’analytique, données par lots, données interactives, analytique de streaming et données d’apprentissage automatique (par exemple, fichiers journaux, données IoT, données sur le parcours de navigation, jeux de données volumineux) |
| Concepts clés |Le compte Data Lake Storage Gen1 contient des dossiers, qui contiennent des données stockées sous forme de fichiers |Le compte de stockage a des conteneurs, qui possèdent des données sous la forme d’objets BLOB |
| Structure |Système de fichiers hiérarchique |Magasin d’objets avec espace de noms plat |
| API |API REST sur HTTPS |API REST sur HTTP/HTTPS |
| API côté serveur |[API REST compatible WebHDFS](/rest/api/datalakestore/) |[API REST d’Azure Blob Storage](/rest/api/storageservices/Blob-Service-REST-API) |
| Client de système de fichiers Hadoop |Oui |Oui |
| Opérations de données - authentification |Basées sur les [Identités Azure Active Directory](../active-directory/develop/authentication-vs-authorization.md) |Basées sur les secrets partagés - [Clés d’accès au compte](../storage/common/storage-account-keys-manage.md) et [Clés de signature d’accès partagé](../storage/common/storage-sas-overview.md). |
| Opérations de données - protocole d’authentification |[OpenID Connect](https://openid.net/connect/). Les appels doivent contenir un JWT (jeton web JSON) valide émis par Azure Active Directory.|Code d’authentification de message basé sur le hachage (HMAC). Les appels doivent contenir un hachage SHA-256 codé en Base64 sur une partie de la requête HTTP. |
| Opérations de données - autorisation |Listes de contrôle d’accès (ACL) POSIX.  Les listes ACL basées sur les identités Azure Active Directory peuvent être définies aux niveaux fichier et dossier. |Pour l’autorisation au niveau des comptes – utilisez des [clés d’accès au compte](../storage/common/storage-account-keys-manage.md)<br>Pour l’autorisation au niveau d'un compte, d'un conteneur ou d'un objet blob - utilisez des [clés de signature d’accès partagé](../storage/common/storage-sas-overview.md) |
| Opérations de données - Audit |Disponible. Pour plus d’informations, voir [ici](data-lake-store-diagnostic-logs.md) . |Disponible |
| Chiffrement des données au repos |<ul><li>Transparent, côté serveur</li> <ul><li>Avec des clés gérées par le service</li><li>Avec des clés gérées par le client dans Azure KeyVault</li></ul></ul> |<ul><li>Transparent, côté serveur</li> <ul><li>Avec des clés gérées par le service</li><li>Avec des clés gérées par le client dans Azure KeyVault (préversion)</li></ul><li>chiffrement côté client</li></ul> |
| Opérations de gestion (par exemple, créer un compte) |[Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) pour la gestion des comptes |[Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) pour la gestion des comptes |
| Kits de développement logiciel pour développeur |.NET, Java, Python, Node.js |.NET, Java, Python, Node.js, C++, Ruby, PHP, Go, Android, iOS |
| Performances des charges de travail d’analyse |Optimisation des performances pour les charges de travail d’analyses parallèles. Débit et nombre d’E/S par seconde élevés. |Optimisation des performances pour les charges de travail d’analyses parallèles. |
| Limites de taille |Aucune limite de taille pour les comptes, les fichiers ou le nombre de fichiers |Pour les limites spécifiques, consultez [Objectifs d’extensibilité des comptes de stockage standard](../storage/common/scalability-targets-standard-account.md) et [Objectifs de performance et d'extensibilité du stockage Blob ](../storage/blobs/scalability-targets.md). Vous pouvez élargir les limites du compte en contactant le [support Azure](https://azure.microsoft.com/support/faq/) |
| Géo-redondance |Localement redondant (plusieurs copies des données dans une région Azure) |Localement redondant (LRS), redondant dans une zone (ZRS), globalement redondant (GRS), accès en lecture redondant globalement (RA-GRS). Pour plus d’informations, voir [ici](../storage/common/storage-redundancy.md) |
| État du service |Mise à la disposition générale |Mise à la disposition générale |
| Disponibilité régionale |Voir [ici](https://azure.microsoft.com/regions/#services) |Disponible dans toutes les régions Azure |
| Price |Consultez la page [Tarification](https://azure.microsoft.com/pricing/details/data-lake-store/) |Consultez la page [Tarification](https://azure.microsoft.com/pricing/details/storage/) |
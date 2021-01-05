---
title: Sécurisation des applications PaaS à l’aide du stockage Azure | Microsoft Docs
description: Découvrez les bonnes pratiques de sécurité du stockage Azure pour protéger vos applications mobiles et web PaaS.
services: security
documentationcenter: na
author: TomShinder
manager: barbkess
editor: ''
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/28/2018
ms.author: tomsh
ms.openlocfilehash: 082408f357e97a2ed2153d43dbea459ff09ba704
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94693179"
---
# <a name="best-practices-for-securing-paas-web-and-mobile-applications-using-azure-storage"></a>Bonnes pratiques pour la sécurisation des applications mobiles et web PaaS avec le Stockage Azure
Dans cet article, nous abordons un ensemble de bonnes pratiques de sécurité du Stockage Azure pour protéger vos applications mobiles et web PaaS (Platform-as-a-Service). Ces bonnes pratiques sont issues de notre expérience d’Azure, mais également de celle des clients, comme vous.

Azure permet de déployer et d’utiliser le stockage plus aisément que dans le cadre d’une configuration locale. Grâce au stockage Azure, vous pouvez atteindre des niveaux élevés de scalabilité et de disponibilité avec relativement peu d’effort. En plus de constituer la base des machines virtuelles Windows et Linux Azure, le Stockage Azure peut prendre en charge les grandes applications distribuées.

Le stockage Azure fournit les quatre services suivants : Stockage Blob, Stockage Table, Stockage Fichier et Stockage File d’attente. Pour en savoir plus, consultez [Présentation du stockage Microsoft Azure](../../storage/common/storage-introduction.md).

Le [Guide de sécurité du Stockage Azure](../../storage/blobs/security-recommendations.md) constitue une source précieuse pour obtenir plus d’informations sur la sécurité et le Stockage Azure. Cet article relatif aux bonnes pratiques traite de façon générale certains concepts abordés dans le guide de sécurité et propose des liens vers ce dernier, ainsi que vers d’autres sources, pour obtenir des informations supplémentaires.

Cet article traite les bonnes pratiques suivantes :

- Signatures d’accès partagé (SAP)
- Contrôle d’accès en fonction du rôle Azure (Azure RBAC)
- Chiffrement côté client pour les données de valeur élevée
- Storage Service Encryption


## <a name="use-a-shared-access-signature-instead-of-a-storage-account-key"></a>Utiliser la signature d’accès partagé au lieu d’une clé de compte de stockage
Le contrôle d’accès est critique. Pour vous aider à contrôler l’accès au Stockage Azure, Azure génère deux clés de compte de stockage (SAK) 512 bits quand vous créez un compte de stockage. Le niveau de redondance des clés vous permet d’éviter des interruptions de service pendant une rotation des clés de routine. 

Les clés d’accès de stockage sont des secrets de priorité élevée et ne doivent être accessibles qu’aux personnes responsables du contrôle d’accès de stockage. Si des personnes non autorisées accèdent à ces clés, elles ont un contrôle complet du stockage et peuvent remplacer, supprimer ou ajouter des fichiers au stockage. Cela inclut les logiciels malveillants et autres types de contenu qui peuvent nuire à vos clients ou à votre organisation.

Vous devez toujours avoir un moyen de fournir l’accès aux objets dans le stockage. Pour fournir un accès plus précis, vous pouvez tirer parti de la signature d’accès partagé (SAP). Vous pouvez utiliser la SAP pour partager des objets spécifiques dans le stockage pendant un intervalle de temps prédéfini et avec des autorisations spécifiques. Une signature d’accès partagé vous permet de définir :

- L’intervalle pendant lequel la SAP est valide, y compris l’heure de début et l’heure d’expiration.
- Les autorisations accordées par la SAP. Par exemple, une SAP sur un objet blob peut accorder à un utilisateur des autorisations en lecture et en écriture sur cet objet blob, mais pas d’autorisations de suppression.
- Une adresse IP ou plage d’adresses IP facultative à partir de laquelle le stockage Azure accepte la signature d’accès partagé. Par exemple, vous pouvez spécifier une plage d’adresses IP appartenant à votre organisation. Cela fournit une autre mesure de sécurité à votre SAP.
- Le protocole sur lequel le stockage Azure accepte la SAP. Vous pouvez utiliser ce paramètre facultatif pour restreindre l’accès aux clients à l’aide de HTTPS.

La SAP vous permet de partager du contenu comme vous le souhaitez sans communiquer vos clés de compte de stockage. Utiliser systématiquement la SAP dans vos applications est un moyen sécurisé de partager vos ressources de stockage sans compromettre vos clés de compte de stockage.

Pour en savoir plus sur la signature d’accès partagé, consultez [Utilisation des signatures d’accès partagé](../../storage/common/storage-sas-overview.md). 

## <a name="use-azure-role-based-access-control"></a>Utiliser le contrôle d’accès en fonction du rôle Azure
Une autre façon de gérer l’accès consiste à utiliser le [contrôle d’accès en fonction du rôle dans Azure (Azure RBAC)](../../role-based-access-control/overview.md). Grâce à Azure RBAC, vous vous concentrez sur l’octroi aux employés des autorisations exactes dont ils ont besoin, selon les principes de sécurité de moindre privilège et du besoin de connaître. Un trop grand nombre d’autorisations peut exposer un compte aux attaquants. Si le nombre d’autorisations est trop faible, les employés ne peuvent pas effectuer leur travail efficacement. Azure RBAC permet de résoudre ce problème en offrant une gestion précise de l’accès pour Azure. Ces principes sont impératifs pour les organisations qui veulent appliquer des stratégies de sécurité portant sur l’accès aux données.

Vous pouvez utiliser des rôles intégrés Azure dans Azure pour attribuer des privilèges aux utilisateurs. Par exemple, utilisez le rôle Contributeur de comptes de stockage pour les opérateurs de cloud qui doivent gérer des comptes de stockage, et le rôle Contributeur de comptes de stockage classiques pour gérer des comptes de stockage classiques. Concernant les opérateurs de cloud qui doivent gérer des machines virtuelles, mais pas le réseau virtuel ou le compte de stockage auquel elles sont connectées, vous pouvez les ajouter au rôle Collaborateur de machine virtuelle.

Les organisations qui n’appliquent aucun contrôle d’accès aux données en utilisant des capacités telles qu’Azure RBAC risquent d’octroyer plus de privilèges que nécessaire à leurs utilisateurs. Cela risque d’entraîner la compromission des données en accordant à certains utilisateurs un accès non justifié à certaines données.

Pour en savoir plus sur Azure RBAC, consultez :

- [Ajouter ou supprimer des attributions de rôles Azure avec le portail Azure](../../role-based-access-control/role-assignments-portal.md)
- [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md)
- [Guide de sécurité du Stockage Azure](../../storage/blobs/security-recommendations.md) 

## <a name="use-client-side-encryption-for-high-value-data"></a>Utiliser le chiffrement côté client pour les données de valeur élevée
Avec le chiffrement côté client, vous pouvez chiffrer par programmation des données en transit avant leur chargement dans le Stockage Azure et déchiffrer des données par programmation quand vous les récupérez. Ce dispositif fournit le chiffrement des données en transit, mais également le chiffrement des données au repos. Le chiffrement côté client est la méthode de chiffrement des données la plus sûre, mais il vous oblige à effectuer des modifications par programmation dans votre application et à mettre en place des processus de gestion des clés.

Le chiffrement côté client vous permet également d’avoir un contrôle exclusif de vos clés de chiffrement. Vous pouvez créer et gérer vos propres clés de chiffrement. Il utilise une technique d’enveloppe où la bibliothèque cliente du Stockage Azure génère une clé de chiffrement de contenu (CEK) qui est ensuite incluse dans un wrapper (chiffrée) à l’aide de la clé de chiffrement de clés (KEK). La clé de chiffrement de clés est identifiée par un identificateur de clé et peut être une paire de clés asymétriques ou une clé symétrique pouvant être gérée localement ou stockée dans [Azure Key Vault](../../key-vault/general/overview.md).

Le chiffrement côté client est intégré aux bibliothèques clientes de stockage Java et .NET. Pour plus d’informations sur le chiffrement de données dans les applications clientes, et sur la génération et la gestion de vos propres clés de chiffrement, consultez [Chiffrement côté client et Azure Key Vault pour le Stockage Microsoft Azure](../../storage/common/storage-client-side-encryption.md).

## <a name="enable-storage-service-encryption-for-data-at-rest"></a>Activer Storage Service Encryption pour les données au repos
Quand le [chiffrement du service de stockage](../../storage/common/storage-service-encryption.md) pour le stockage de fichiers est activé, les données sont chiffrées automatiquement à l’aide du chiffrement AES-256. Microsoft gère en totalité le chiffrement, le déchiffrement et la gestion des clés. Cette fonctionnalité est disponible pour les types de redondance LRS et GRS.

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a présenté un ensemble de bonnes pratiques de sécurité du stockage Azure pour protéger vos applications mobiles et web PaaS. Pour en savoir plus sur la sécurisation de vos déploiements PaaS, consultez :

- [Sécurisation des déploiements PaaS](paas-deployments.md)
- [Sécurisation des applications mobiles et web PaaS à l’aide d’Azure App Services](paas-applications-using-app-services.md)
- [Sécurisation des bases de données PaaS dans Azure](paas-applications-using-sql.md)
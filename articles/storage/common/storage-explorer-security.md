---
title: Guide de sécurité Explorateur Stockage Azure | Microsoft Docs
description: Guide de sécurité pour Explorateur Stockage Azure
services: storage
author: cralvord
ms.service: storage
ms.topic: best-practice
ms.date: 07/30/2020
ms.author: cralvord
ms.openlocfilehash: 283ec9999f9b4362035b6770383984efb0879d49
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92783757"
---
# <a name="azure-storage-explorer-security-guide"></a>Guide de sécurité Explorateur Stockage Azure

Explorateur Stockage Microsoft Azure vous permet d’utiliser facilement les données de Stockage Azure en toute sécurité sur Windows, macOS et Linux. En suivant ces instructions, vous pouvez vous assurer que vos données restent protégées.

## <a name="general"></a>Général

- **Utilisez toujours la version la plus récente d’Explorateur Stockage.** Les versions d’Explorateur Stockage peuvent contenir des correctifs de sécurité. Rester à jour permet de garantir la sécurité générale.
- **Connectez-vous uniquement aux ressources auxquelles vous faites confiance.** Les données que vous téléchargez à partir de sources non approuvées peuvent être malveillantes, et le chargement de données vers une source non approuvée peut entraîner la perte ou le vol de données.
- **Utilisez le protocole HTTPS chaque fois que possible.** Explorateur Stockage utilise le protocole HTTPS par défaut. Certains scénarios vous permettent d’utiliser le protocole HTTP, mais HTTP ne doit être utilisé qu’en dernier recours.
- **Assurez-vous que seules les autorisations nécessaires sont accordées aux personnes qui en ont besoin.** Évitez d’être trop permissif quand vous accordez à quiconque l’accès à vos ressources.
- **Soyez prudent lors de l’exécution d’opérations critiques.** Certaines opérations, telles que la suppression et le remplacement, sont irréversibles et peuvent entraîner une perte de données. Assurez-vous d’utiliser les ressources correctes avant d’exécuter ces opérations.

## <a name="choosing-the-right-authentication-method"></a>Choix de la méthode d’authentification appropriée

Explorateur Stockage offre différentes façons d’accéder à vos ressources Stockage Azure. Quelle que soit la méthode choisie, voici nos recommandations.

### <a name="azure-ad-authentication"></a>Authentification Azure AD

La façon la plus simple et la plus sécurisée d’accéder à vos ressources Stockage Azure consiste à vous connecter avec votre compte Azure. La connexion utilise l’authentification Azure AD, ce qui vous permet d’effectuer les opérations suivantes :

- Accorder l’accès à des utilisateurs et groupes spécifiques.
- Révoquer l’accès à des utilisateurs et groupes spécifiques à tout moment.
- Appliquer des conditions d’accès, telles que l’authentification multifacteur.

Nous vous recommandons d’utiliser l’authentification Azure AD dans la mesure du possible.

Cette section décrit les deux technologies basées sur Azure AD qui peuvent être utilisées pour sécuriser vos ressources de stockage.

#### <a name="azure-role-based-access-control-azure-rbac"></a>Contrôle d’accès en fonction du rôle Azure (Azure RBAC)

Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) vous offre un contrôle d’accès affiné sur vos ressources Azure. Les autorisations et rôles RBAC Azure peuvent être gérés à partir du portail Azure.

L’Explorateur Stockage prend en charge l’accès RBAC Azure aux comptes de stockage, aux objets blobs et aux files d’attente. Si vous avez besoin d’accéder à des partages de fichiers ou à des tables, vous devez attribuer des rôles RBAC Azure qui accordent l’autorisation de lister les clés de compte de stockage.

#### <a name="access-control-lists-acls"></a>Listes ACL

Les [listes de contrôle d’accès (ACL, access-control list)](../blobs/data-lake-storage-access-control.md) vous permettent de contrôler l’accès au niveau des fichiers et des dossiers dans les conteneurs de blobs ADLS Gen2. Vous pouvez gérer vos ACL à l’aide d’Explorateur Stockage.

### <a name="shared-access-signatures-sas"></a>Signatures d’accès partagé (SAP)

Si vous ne pouvez pas utiliser l’authentification Azure AD, nous vous recommandons d’utiliser des signatures d’accès partagé. Grâce aux signatures d’accès partagé, vous pouvez :

- Fournir un accès limité anonyme aux ressources sécurisées.
- Révoquer une SAP immédiatement si elle est générée à partir d’une stratégie d’accès partagé.

Toutefois, avec les signatures d’accès partagé, vous ne pouvez pas :

- Limiter les utilisateurs autorisés à utiliser une SAP. Une SAP valide peut être utilisée par toute personne qui la possède.
- Révoquer une SAP si elle n’est pas générée à partir d’une stratégie d’accès partagé.

Lorsque vous utilisez une SAP dans Explorateur Stockage, nous vous recommandons de suivre les instructions suivantes :

- **Limitez la distribution des jetons et des URI SAP.** Ne distribuez que des jetons et URI SAP à des personnes de confiance. En limitant la distribution des SAP, vous réduisez les risques d’utilisation abusive.
- **Utilisez uniquement les jetons et les URI SAP des entités auxquelles vous faites confiance.**
- **Utilisez des stratégies d’accès partagé lors de la génération de jetons et d’URI SAP, si possible.** Une SAP basée sur une stratégie d’accès partagé est plus sécurisée qu’une SAP simple, car elle peut être révoquée en supprimant la stratégie d’accès partagé.
- **Générez des jetons avec un accès aux ressources et des autorisations minimums.** Des autorisations minimales limitent les dommages potentiels qui pourraient être causés si une signature d’accès partagé est mal utilisée.
- **Générez des jetons qui sont valides uniquement aussi longtemps que nécessaire.** Une brève durée de vie est particulièrement importante pour une SAP simple, car il n’existe aucun moyen de la révoquer une fois qu’elle est générée.

> [!IMPORTANT]
> Lorsque vous partagez des jetons et des URI SAP à des fins de résolution des problèmes, tels que des demandes de service ou des rapports de bogues, biffez toujours au moins la partie signature de la SAP.

### <a name="storage-account-keys"></a>Clés de compte de stockage

Les clés de compte de stockage accordent un accès illimité aux services et aux ressources au sein d’un compte de stockage. Pour cette raison, nous vous recommandons de limiter l’utilisation des clés pour accéder aux ressources dans Explorateur Stockage. Utilisez plutôt des fonctionnalités RBAC ou SAP Azure pour fournir l’accès.

Certains rôles Azure accordent des autorisations pour récupérer des clés de compte de stockage. Les personnes disposant de ces rôles peuvent contourner efficacement les autorisations accordées ou refusées par le contrôle RBAC Azure. Nous vous recommandons de ne pas accorder cette autorisation, sauf si elle est nécessaire.

Explorateur Stockage tentera d’utiliser les clés de compte de stockage, si elles sont disponibles, pour authentifier les demandes. Vous pouvez désactiver cette fonctionnalité dans Paramètres ( **Services > Comptes de stockage > Désactiver l’utilisation des clés** ). Certaines fonctionnalités ne prennent pas en charge le contrôle RBAC Azure, comme l’utilisation de comptes de stockage classiques. De telles fonctionnalités nécessitent toujours des clés et ne sont pas concernées par ce paramètre.

Si vous devez utiliser des clés pour accéder à vos ressources de stockage, nous vous recommandons de suivre les instructions suivantes :

- **Ne partagez vos clés de compte avec personne.**
- **Traitez vos clés de compte de stockage comme des mots de passe.** Si vous devez rendre vos clés accessibles, utilisez des solutions de stockage sécurisées telles qu’[Azure Key Vault](https://azure.microsoft.com/services/key-vault/).

> [!NOTE]
> Si vous pensez qu’une clé de compte de stockage a été partagée ou distribuée par erreur, vous pouvez générer de nouvelles clés pour votre compte de stockage à partir du portail Azure.

### <a name="public-access-to-blob-containers"></a>Accès public aux conteneurs de blobs

Explorateur Stockage vous permet de modifier le niveau d’accès de vos conteneurs Stockage Blob Azure. Les conteneurs de blobs non privés permettent à toute personne anonyme d’accéder en lecture aux données de ces conteneurs.

Lorsque vous autorisez l’accès public à un conteneur de blobs, nous vous recommandons de suivre les instructions suivantes :

- **N’autorisez pas l’accès public à un conteneur de blobs qui peut contenir des données potentiellement sensibles.** Assurez-vous que votre conteneur de blobs ne contient aucune donnée privée.
- **Ne chargez aucune donnée potentiellement sensible dans un conteneur de blobs avec accès au blob ou au conteneur.** 

## <a name="next-steps"></a>Étapes suivantes

- [Recommandations de sécurité](../blobs/security-recommendations.md)
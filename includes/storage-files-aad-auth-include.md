---
title: Fichier include
description: Fichier include
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/30/2019
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 84fa97ec964d490eb9571c7e030704562a4a81d0
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95558532"
---
[Azure Files](../articles/storage/files/storage-files-introduction.md) prend en charge l’authentification basée sur l’identité sur SMB (Server Message Block) via [Active Directory Domain Services (AD DS) en local](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) et [Azure Active Directory Domain Services (Azure AD DS)](../articles/active-directory-domain-services/overview.md). Cet article traite de la façon dont les partages de fichiers Azure peuvent tirer parti des services de domaine, localement ou dans Azure, pour prendre en charge l’accès basé sur l’identité aux partages de fichiers Azure sur SMB. L’activation de l’accès basé sur l’identité pour vos partages de fichiers Azure vous permet de remplacer les serveurs de fichiers existants par des partages de fichiers Azure sans remplacer votre service d’annuaire existant, en conservant l’accès fluide des utilisateurs aux partages. 

Azure Files applique l’autorisation d’accès de l’utilisateur à la fois au niveau du partage de fichiers et au niveau du répertoire/fichier. L’attribution d’autorisations au niveau du partage peut être effectuée sur des utilisateurs ou groupes Azure Active Directory (Azure AD) gérés par le modèle de [contrôle d’accès en fonction du rôle (Azure RBAC)](../articles/role-based-access-control/overview.md). Avec RBAC, les informations d’identification que vous utilisez pour l’accès aux fichiers doivent être disponibles ou synchronisées avec Azure AD. Vous pouvez attribuer des rôles Azure intégrés tels que Lecteur de partage SMB de données de fichier de stockage à des utilisateurs ou des groupes dans Azure AD pour accorder un accès en lecture à un partage de fichiers Azure.

Au niveau du répertoire/fichier, Azure Files prend en charge la préservation, l’héritage et l’application des [DACL Windows](/windows/win32/secauthz/access-control-lists) tout comme les serveurs de fichiers Windows. Vous pouvez choisir de conserver les DACL Windows lors de la copie de données via SMB entre votre partage de fichiers existant et vos partages de fichiers Azure. Que vous ayez l’intention ou non d’appliquer l’autorisation, vous pouvez utiliser les partages de fichiers Azure pour sauvegarder des listes de contrôle d’accès ainsi que vos données.
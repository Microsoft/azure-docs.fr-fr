---
title: Gérer les données utilisateur dans Azure Active Directory B2C | Microsoft Docs
description: Découvrez comment supprimer ou exporter des données utilisateur dans Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/06/2018
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: d9bdf7258296b82e65e03f6b8af8021b9a7be0f0
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94952469"
---
# <a name="manage-user-data-in-azure-active-directory-b2c"></a>Gérer les données utilisateur dans Azure Active Directory B2C

 Cet article explique comment gérer les données utilisateur dans Azure Active Directory B2C (Azure AD B2C) en utilisant les opérations fournies par l’[API Microsoft Graph](/graph/use-the-api). La gestion des données utilisateur inclut la suppression ou l’exportation de données à partir des journaux d’audit.

[!INCLUDE [gdpr-intro-sentence.md](../../includes/gdpr-intro-sentence.md)]

## <a name="delete-user-data"></a>Supprimer des données utilisateur

Les données utilisateur sont stockées dans l’annuaire Azure AD B2C et dans les journaux d’audit. Toutes les données d’audit utilisateur sont conservées pendant 7 jours dans Azure AD B2C. Si vous souhaitez supprimer des données utilisateur avant l’expiration de ces 7 jours, vous pouvez utiliser l’opération [Supprimer un utilisateur](/graph/api/user-delete). Une opération DELETE est requise pour chacun des clients Azure AD B2C susceptibles de contenir des données.

Un ID objet est attribué à chaque utilisateur dans Azure AD B2C. L’ID objet constitue un identificateur non ambigu que vous pouvez utiliser pour supprimer des données utilisateur dans Azure AD B2C. En fonction de votre architecture, l’ID objet peut être un identificateur de corrélation utile pour d’autres services, tels que des bases de données de gestion des finances, du marketing et de la relation client.

La manière la plus précise d’obtenir l’ID objet pour un utilisateur consiste à le faire dans le cadre d’un parcours d’authentification auprès d’Azure AD B2C. Si vous recevez une requête de données valide d’un utilisateur par le biais d’autres méthodes, un processus hors connexion, tel qu’une recherche effectuée par un agent de service clientèle, peut être nécessaire pour trouver l’utilisateur et noter l’ID objet associé.

L’exemple suivant montre un flux de suppression de données possible :

1. L’utilisateur se connecte et sélectionne **Supprimer mes données**.
2. L’application offre une option permettant de supprimer les données dans une Section Administration.
3. L’application force une authentification auprès d’Azure AD B2C. Azure AD B2C retourne un jeton avec l’ID objet de l’utilisateur à l’application.
4. L’application reçoit le jeton, et l’ID objet est utilisé pour supprimer les données utilisateur via un appel à l’API Microsoft Graph. L’API Microsoft Graph supprime les données utilisateur et renvoie un code d’état 200 OK.
5. L’application orchestre la suppression des données utilisateur dans d’autres systèmes organisationnels en fonction des besoins à l’aide de l’ID objet ou d’autres identificateurs.
6. L’application confirme la suppression des données et indique les étapes suivantes à l’utilisateur.

## <a name="export-customer-data"></a>Exporter des données client

Le processus d’exportation de données client à partir d’Azure AD B2C est semblable au processus de suppression.

Les données d’utilisateur d’Azure AD B2C sont limitées à ce qui suit :

- **Données stockées dans Azure Active Directory** : vous pouvez récupérer les données dans un parcours utilisateur d’authentification Azure AD B2C à l’aide de l’ID objet ou de tout nom de connexion, tel qu’une adresse e-mail ou un nom d’utilisateur.
- **Rapport d’événements d’audit spécifiques de l’utilisateur** : vous pouvez indexer les données à l’aide de l’ID objet.

Dans l’exemple suivant de flux de données d’exportation, les étapes décrites comme étant effectuées par l’application peuvent également être réalisées par un processus principal ou par un utilisateur avec un rôle d’administrateur dans l’annuaire :

1. L’utilisateur se connecte à l’application. Azure AD B2C applique l’authentification avec Azure AD Multi-Factor Authentication si nécessaire.
2. L’application utilise les informations d’identification de l’utilisateur pour appeler une opération de l’API Microsoft Graph afin de récupérer les attributs utilisateur. L’API Microsoft Graph fournit les données d’attribut au format JSON. Selon le schéma,vous pouvez définir le contenu du jeton d’ID pour inclure toutes les données personnelles d’un utilisateur.
3. L’application récupère l’activité d’audit de l’utilisateur. L’API Microsoft Graph fournit les données d’événement à l’application.
4. L’application agrège les données et les met à la disposition de l’utilisateur.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment gérer la manière dont les utilisateurs accèdent à votre application, voir [Gérer l’accès utilisateur](manage-user-access.md).
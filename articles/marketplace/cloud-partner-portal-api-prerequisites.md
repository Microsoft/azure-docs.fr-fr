---
title: Prérequis à l’utilisation d’API - Place de marché Azure
description: Prérequis à l’utilisation des API du portail Cloud Partner.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 09/23/2020
ms.openlocfilehash: 374cc3398d1037b19ceddcbdaafd333f643e44ff
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91261156"
---
# <a name="api-prerequisites"></a>Conditions préalables à l'utilisation d'API

> [!NOTE]
> Les API de Portail Cloud Partner sont intégrées à Espace partenaires et continueront d’y fonctionner. La transition introduit de légères modifications. Passez en revue les changements répertoriés dans les [informations de référence relatives aux API de Portail Cloud Partner](cloud-partner-portal-api-overview.md) pour vous assurer que votre code continue de fonctionner après la transition vers Espace partenaires. N’utilisez des API du Portail Cloud Partner que pour des produits existants qui étaient déjà intégrés avant la transition vers l’Espace partenaires. Les nouveaux produits doivent utiliser des API de soumission à l’Espace partenaires.

Vous avez besoin de deux ressources de programmation requises pour utiliser les API du Portail Cloud Partner : un principal de service et un jeton d’accès Azure Active Directory (Azure AD).

## <a name="create-service-principal-in-azure-active-directory-tenant"></a>Créer un principal du service dans un locataire Azure Active Directory

Tout d'abord, vous devez créer un principal de service dans votre locataire Azure AD. Ce locataire se verra attribuer son propre jeu d'autorisations sur le portail Cloud Partner. Votre code appellera les API en utilisant ce locataire plutôt que vos informations d’identification personnelles. Pour obtenir une explication complète de la création d’un principal du service, consultez [Guide pratique pour Utilisez le portail pour créer une application Azure AD et un principal du service pouvant accéder aux ressources](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="add-service-principal-to-your-account"></a>Ajouter un principal de service à votre compte

Maintenant que vous avez créé le principal de service dans votre locataire, vous pouvez l’ajouter en tant qu’utilisateur à votre du Portail Cloud Partner. Comme un utilisateur, sur le portail, le principal de service peut être un propriétaire ou un contributeur. Pour plus d’informations, consultez **Étapes suivantes** ci-dessous.

## <a name="next-steps"></a>Étapes suivantes

Consultez [Gérer les applications Azure AD](partner-center-portal/manage-account.md#manage-azure-ad-applications).

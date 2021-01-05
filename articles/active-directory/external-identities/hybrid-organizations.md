---
title: Collaboration B2B pour les organisations hybrides - Azure AD
description: Donnez accès aux ressources cloud et locales aux partenaires avec Azure AD B2B Collaboration.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 427f7ad4d6a1b9839b1197ef9f7ca15400ea0f59
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87907190"
---
# <a name="azure-active-directory-b2b-collaboration-for-hybrid-organizations"></a>Azure Active Directory B2B Collaboration pour les organisations hybrides

Azure Active Directory (Azure AD) B2B Collaboration vous permet de donner plus facilement accès aux applications et ressources de votre organisation aux partenaires externes. Cela est vrai même dans une configuration hybride où vous avez à la fois des ressources cloud et des ressources locales. Que vous gériez actuellement les comptes des partenaires externes en local sur votre système d’identité local ou dans le cloud en tant qu’utilisateurs Azure AD B2B n’a pas d’importance. Vous pouvez désormais accorder à ces utilisateurs l’accès aux ressources en local et dans le cloud, à l’aide des mêmes identifiants de connexion pour les deux environnements.

## <a name="grant-b2b-users-in-azure-ad-access-to-your-on-premises-apps"></a>Accorder aux utilisateurs B2B dans Azure AD l’accès à vos applications locales

Si votre organisation utilise des fonctionnalités d’Azure AD B2B Collaboration pour inviter des utilisateurs d’organisations partenaires à votre instance Azure AD, vous pouvez désormais fournir à ces utilisateurs B2B un accès aux applications locales.

Pour les applications qui utilisent l’authentification SAML, vous pouvez mettre ces applications à la disposition des utilisateurs B2B via le portail Azure à l’aide du Proxy d’application Azure AD pour l’authentification.

Pour les applications qui utilisent Authentification Windows intégrée (IWA) avec la délégation contrainte Kerberos (KCD), vous utilisez également le proxy Azure AD pour l’authentification. Toutefois, pour pouvoir travailler, un objet utilisateur est nécessaire dans Windows Server Active Directory en local. Deux méthodes vous permettent de créer des objets utilisateur locaux qui représentent vos utilisateurs invités B2B.

- Vous pouvez utiliser Microsoft Identity Manager (MIM) 2016 SP1 et l’agent de gestion MIM pour Microsoft Graph.
- Vous pouvez utiliser un script PowerShell. (Cette solution ne nécessite pas MIM.)

Pour plus d’informations sur l’implémentation de ces solutions, consultez [Grant B2B users in Azure AD access to your on-premises applications](hybrid-cloud-to-on-premises.md) (Accorder aux utilisateurs B2B dans Azure AD l’accès à vos applications locales).

## <a name="grant-locally-managed-partner-accounts-access-to-cloud-resources"></a>Accorder un accès aux comptes de partenaires gérés en local aux ressources cloud

Avant Azure AD, les organisations dotées de systèmes d’identité locaux possédaient généralement des comptes de partenaires gérés dans leur répertoire local. Si vous êtes une organisation de ce type, vous souhaitez vous assurer que vos partenaires continuent à avoir accès à vos applications et autres ressources à mesure qu’elles migrent vers le cloud. Dans l’idéal, vous voulez que ces utilisateurs se servent du même ensemble d’informations d’identification pour accéder aux ressources cloud et locales. 

Nous vous offrons désormais des méthodes vous permettant d’utiliser Azure AD Connect pour synchroniser ces comptes locaux vers le cloud en tant qu’« utilisateurs invités ». Ces comptes sont identiques à ceux des utilisateurs Azure AD B2B.

Pour protéger les données de votre entreprise, vous pouvez contrôler l’accès aux seules ressources autorisées et configurer des stratégies d’autorisation qui traitent ces utilisateurs invités différemment de la façon dont ils traitent vos employés.

Pour connaître les détails de l’implémentation, consultez [Grant locally-managed partner accounts access to cloud resources using Azure AD B2B collaboration](hybrid-on-premises-to-cloud.md) (Octroyer aux comptes de partenaires gérés localement un accès aux ressources cloud à l’aide d’Azure AD B2B Collaboration).
 
## <a name="next-steps"></a>Étapes suivantes

- [Grant B2B users in Azure AD access to your on-premises applications](hybrid-cloud-to-on-premises.md) (Accorder aux utilisateurs B2B dans Azure AD l’accès à vos applications locales)
- [Octroyer aux comptes de partenaires gérés localement un accès aux ressources cloud à l’aide d’Azure AD B2B Collaboration](hybrid-on-premises-to-cloud.md)



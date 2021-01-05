---
title: Comment la plateforme d’identités Microsoft utilise le protocole SAML
description: Cet article fournit une vue d’ensemble des profils SAML d’authentification unique et de déconnexion unique dans Azure Active Directory.
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 10/05/2018
ms.author: kenwith
ms.custom: aaddev
ms.reviewer: paulgarn
ms.openlocfilehash: 06f80f94be25e42c9e8f0270e6cb15aca086ae18
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994398"
---
# <a name="how-microsoft-identity-platform-uses-the-saml-protocol"></a>Comment la plateforme d’identités Microsoft utilise le protocole SAML

La plateforme d’identités Microsoft utilise le protocole SAML 2.0 pour permettre aux applications de fournir une expérience d’authentification unique aux utilisateurs. Les profils SAML [Authentification unique](single-sign-on-saml-protocol.md) et [Déconnexion unique](single-sign-out-saml-protocol.md) d’Azure AD expliquent comment les assertions, les protocoles et les liaisons SAML sont utilisées dans le service de fournisseur d’identité.

Le protocole SAML nécessite que le fournisseur d’identité (la plateforme d’identités Microsoft) et le fournisseur de services (l’application) échangent des informations les concernant.

Lorsqu’une application est enregistrée auprès d’Azure AD, le développeur d’applications enregistre les informations liées à la fédération auprès d’Azure AD. Ces informations englobent notamment **l’URI de redirection** et **l’URI des métadonnées** de l’application.

La plateforme d’identités Microsoft utilise l’**URI de métadonnées** du service cloud pour récupérer la clé de signature et l’URI de déconnexion. Le client peut ouvrir l’application dans **Azure AD -> Inscription de l’application** puis, dans **Paramètres -> Propriétés**, il peut mettre à jour l’URL de déconnexion. De cette façon, la plateforme d’identités Microsoft peut envoyer la réponse à l’URL correcte. 

Azure Active Directory expose les points de terminaison d’authentification unique et de déconnexion unique communs (indépendants du client) et spécifiques au client. Ces URL représentent les emplacements adressables et ne jouent pas simplement le rôle d’identificateurs. Vous pouvez donc accéder au point de terminaison pour lire les métadonnées.

* Le point de terminaison propre au locataire se trouve à l’adresse `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml`. L’espace réservé *\<TenantDomainName>* représente un nom de domaine inscrit ou le GUID TenantID d’un locataire Azure AD. Par exemple, les métadonnées de fédération du client contoso.com sont sur : https://login.microsoftonline.com/contoso.com/FederationMetadata/2007-06/FederationMetadata.xml

* Le point de terminaison indépendant du locataire se trouve à l’adresse `https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml`. Dans cette adresse de point de terminaison, l’élément **common** remplace le nom de domaine ou l’ID du locataire.

Pour plus d’informations sur les documents de métadonnées de fédération publiés par Azure AD, consultez la page [Métadonnées de fédération](../azuread-dev/azure-ad-federation-metadata.md).

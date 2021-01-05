---
title: Authentification OpenID Connect avec Azure Active Directory
description: Guide architectural sur l’implémentation de l’authentification OpenID Connect avec Azure Active Directory.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/10/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0e5bf7e51de38d42e64f6737e687c5946a464160
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96168659"
---
# <a name="openid-connect-authentication-with-azure-active-directory"></a>Authentification OpenID Connect avec Azure Active Directory

OpenID Connect (OIDC) est un protocole d'authentification basé sur le protocole OAuth2 (utilisé pour l'autorisation). OIDC utilise les flux de messages standardisés d'OAuth2 pour fournir des services d'identité. 

L'objectif de conception d'OIDC est le suivant : « simplifier les choses simples et rendre possibles les choses compliquées ». OIDC permet aux développeurs d'authentifier les utilisateurs sur leurs sites web et applications sans avoir à posséder et gérer des fichiers de mots de passe. Le concepteur de l'application dispose ainsi d'un moyen sûr de vérifier l'identité de la personne qui utilise le navigateur ou l'application native connectée à l'application.

L'authentification de l'utilisateur doit s'effectuer auprès d'un fournisseur d'identité qui vérifiera la session ou les informations d'identification de l'utilisateur. Pour cela, il vous faut un agent approuvé. Les applications natives lancent généralement le navigateur système à cette fin. Les vues intégrées ne sont pas considérées comme fiables, car rien n'empêche l'application d'espionner le mot de passe de l'utilisateur. 

En plus de l'authentification, l'utilisateur peut être invité à donner son consentement. Le consentement est l'autorisation explicite d'accès à des ressources protégées que l'utilisateur octroie à une application. Le consentement est différent de l'authentification car il ne doit être octroyé qu'une seule fois pour une ressource. Le consentement reste valide jusqu'à ce que l'utilisateur ou l'administrateur révoque manuellement l'octroi. 

## <a name="use-when"></a>Cas d'utilisation

Le consentement de l'utilisateur et la connexion web sont nécessaires.

![diagramme architectural](./media/authentication-patterns/oidc-auth.png)

## <a name="components-of-system"></a>Composants du système

* **Utilisateur** : sollicite un service à partir de l'application.

* **Agent approuvé** : composant avec lequel l'utilisateur interagit. Cet agent approuvé est généralement un navigateur web.

* **Application** : l'application, ou serveur de ressources, est l'emplacement où résident la ressource ou les données. Elle se fie au fournisseur d'identité pour authentifier et autoriser de manière sécurisée l'agent approuvé. 

* **Azure AD** : Le fournisseur OIDC, également connu sous le nom de fournisseur d'identité, gère de manière sécurisée tout ce qui a trait aux informations de l'utilisateur, à son accès et aux relations d'approbation entre les parties d'un flux. Il authentifie l'identité de l'utilisateur, octroie et révoque l'accès aux ressources, et émet des jetons. 

## <a name="implement-oidc-with-azure-ad"></a>Implémenter OIDC avec Azure AD

* [Intégration des applications dans Azure AD](../saas-apps/tutorial-list.md) 

* [Protocoles OAuth 2.0 et OpenID Connect sur la Plateforme d'identités Microsoft](../develop/active-directory-v2-protocols.md) 

* [Plateforme d’identités Microsoft et protocole OpenID Connect](../develop/v2-protocols-oidc.md) 

* [Connexion web avec OpenID Connect dans Azure Active Directory B2C](../../active-directory-b2c/openid-connect.md) 

* [Sécuriser votre application en utilisant OpenID Connect et Azure AD](/learn/modules/secure-app-with-oidc-and-azure-ad/) 


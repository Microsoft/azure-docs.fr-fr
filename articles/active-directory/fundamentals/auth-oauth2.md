---
title: Authentification OAUTH 2,0 avec Azure Active Directory
description: Guide architectural sur l’implémentation de l’authentification OAUTH 2.0 avec Azure Active Directory.
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
ms.openlocfilehash: c1604d79ce5eb9949028cd677b340bf3d4b09f6c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96172837"
---
# <a name="oauth-20-authentication-with-azure-active-directory"></a>Authentification OAuth 2,0 avec Azure Active Directory

Le protocole OAuth 2,0 est le protocole de l’industrie pour l’autorisation. Il permet à un utilisateur d’accorder un accès limité à ses ressources protégées. Conçu pour fonctionner spécifiquement avec le protocole HTTP (Hypertext Transfer Protocol), OAuth sépare le rôle du client du propriétaire de la ressource. Le client demande l’accès aux ressources contrôlées par le propriétaire de la ressource et hébergées par le serveur de ressources. Le serveur de ressources émet des jetons d’accès avec l’approbation du propriétaire de la ressource. Le client utilise les jetons d’accès pour accéder aux ressources protégées hébergées par le serveur de ressources. 

OAuth 2,0 est directement lié à OpenID Connect (OIDC). Étant donné que OIDC est une couche d’authentification et d’autorisation reposant sur OAuth 2,0, il n’est pas compatible avec OAuth 1,0. Azure Active Directory (Azure AD) prend en charge tous les flux OAuth 2,0. 

## <a name="use-when"></a>Contexte d'utilisation :

Pour les clients riches & les scénarios d’application moderne et l’accès à l’API Web RESTful.

![Diagramme de l’architecture](./media/authentication-patterns/oauth.png)

## <a name="components-of-system"></a>Composants du système

* **Utilisateur** : Demande un service à partir de l’application Web (application). L’utilisateur est généralement le propriétaire de la ressource qui détient les données et qui a le pouvoir d’autoriser les clients à accéder aux données ou à la ressource. 

* **Navigateur Web** : Le navigateur Web avec lequel l’utilisateur interagit est le client OAuth. 

* **Application web** : L’application Web, ou serveur de ressources, est l’emplacement où réside(nt) la ressource ou les données. Il approuve le serveur d’autorisation pour authentifier et autoriser le client OAuth en toute sécurité. 

* **Azure AD** : Azure AD est le serveur d’autorisation, également appelé fournisseur d’identité (IdP). Il gère en toute sécurité tout ce qu’il faut faire avec les informations de l’utilisateur, son accès et la relation d’approbation. Il est responsable de l’émission des jetons qui accordent et révoquent l’accès aux ressources.

## <a name="implement-oauth-20-with-azure-ad"></a>Implémenter OAuth 2,0 avec Azure AD

* [Intégration des applications dans Azure AD](../saas-apps/tutorial-list.md) 

* [Protocoles OAuth 2,0 et OpenID Connect sur la plateforme Microsoft Identity](../develop/active-directory-v2-protocols.md) 

* [Types d’applications et OAuth2](../develop/v2-app-types.md) 


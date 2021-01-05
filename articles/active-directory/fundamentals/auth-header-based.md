---
title: Authentification basée sur l’en-tête avec Azure Active Directory
description: Guide architectural sur l’implémentation de l’authentification basée sur l’en-tête avec Azure Active Directory.
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
ms.openlocfilehash: 69e87fc919893a544f5d0b1b615a110f25486e57
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96168744"
---
# <a name="header-based-authentication-with-azure-active-directory"></a>Authentification basée sur l’en-tête avec Azure Active Directory

Les applications héritées utilisent couramment l’authentification basée sur les en-têtes. Dans ce scénario, un utilisateur (ou un expéditeur de message) s’authentifie auprès d’une solution d’identité intermédiaire. La solution intermédiaire authentifie l’utilisateur et propage les en-têtes HTTP (Hypertext Transfer Protocol) requis au service web de destination. Azure Active Directory (AD) prend en charge ce modèle via son service de proxy d’application et l’intégration à d’autres solutions de contrôleur réseau. 

Dans notre solution, le proxy d’application fournit un accès à distance à l’application, authentifie l’utilisateur et transmet les en-têtes requis par l’application. 

## <a name="use-when"></a>Cas d'utilisation

Les utilisateurs distants doivent utiliser l’authentification unique (SSO) de façon sécurisée dans les applications locales qui requièrent l’authentification basée sur l’en-tête.

![Authentification basée sur l’en-tête d’une image architecturale](./media/authentication-patterns/header-based-auth.png)

## <a name="components-of-system"></a>Composants du système

* **Utilisateur** : Accède aux applications héritées prises en charge par le proxy d’application.

* **Navigateur web**  : Composant avec lequel l’utilisateur interagit pour accéder à l’URL externe de l’application.

* **Azure AD** : Authentifie l’utilisateur. 

* **Service Proxy d’application** : Agit comme un proxy inversé pour envoyer la demande de l’utilisateur à l’application locale. Il se trouve dans Azure AD et peut également appliquer toutes les stratégies d’accès conditionnel.

* **Connecteur de proxy d’application** : Installé localement sur les serveurs Windows pour assurer la connectivité aux applications. Il utilise uniquement des connexions sortantes. Retourne la réponse à Azure AD.

* **Applications héritées** : Applications qui reçoivent les requêtes des utilisateurs du proxy d’application. L’application héritée reçoit les en-têtes HTTP nécessaires pour configurer une session et retourner une réponse. 

## <a name="implement-header-based-authentication-with-azure-ad"></a>Implémenter l’authentification basée sur l’en-tête avec Azure AD

* [Ajouter des applications locales pour un accès à distance via le service Proxy d’application dans Azure AD](../manage-apps/application-proxy-add-on-premises-application.md)  

* [Authentification basée sur l’en-tête pour une authentification unique avec le proxy d’application et PingAccess](../manage-apps/application-proxy-configure-single-sign-on-with-headers.md) 

* [Sécuriser les applications héritées avec les réseaux et les contrôleurs de livraison d’applications](../manage-apps/secure-hybrid-access.md)
---
title: Authentification RADIUS avec Azure Active Directory
description: Guide architectural sur l’implémentation de l’authentification RADIUS avec Azure Active Directory.
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
ms.openlocfilehash: 97c524e1f4c05787f1dd61dea5a463e8fa83511a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96168625"
---
# <a name="radius-authentication-with-azure-active-directory"></a>Authentification RADIUS avec Azure Active Directory

Le Protocole RADIUS (Remote Authentication Dial-In User Service) (RADIUS) est un protocole réseau qui sécurise un réseau en permettant une authentification et une autorisation centralisées des utilisateurs de l’accès à distance. De nombreuses applications s’appuient toujours sur le protocole RADIUS pour authentifier les utilisateurs.

Le Serveur Microsoft Windows possède un rôle appelé serveur NPS (Network Policy Server), qui peut agir en tant que serveur RADIUS et prendre en charge l’authentification RADIUS.

Azure Active Directory (Azure AD) active Multi-Factor Authentication avec les systèmes RADIUS. Si un client souhaite appliquer Azure AD Multi-Factor Authentication à l'une des charges de travail RADIUS mentionnées précédemment, il peut installer l'extension NPS d'Azure AD Multi-Factor Authentication sur son serveur NPS Windows. 

Le serveur NPS Windows authentifie les informations d’identification d’un utilisateur par rapport à Active Directory, puis envoie la demande Multi-Factor Authentication à Azure. L’utilisateur reçoit alors un défi sur son authentificateur mobile. Une fois l’opération réussie, l’application cliente est autorisée à se connecter au service. 

## <a name="use-when"></a>Contexte d'utilisation : 

Vous devez ajouter Multi-Factor Authentication à des applications telles que
* un réseau privé virtuel (VPN)
* un accès WiFi
* une passerelle Bureau à distance (RDG)
* VDI (Virtual Desktop Infrastructure)
* Autres qui dépendent du protocole RADIUS pour authentifier les utilisateurs dans le service. 

> [!NOTE]
> Plutôt que de compter sur RADIUS et sur l'extension NPS d'Azure AD Multi-Factor Authentication pour appliquer Azure AD Multi-Factor Authentication à des charges de travail VPN, nous vous recommandons de mettre à niveau votre VPN vers SAML et de fédérer directement votre VPN avec Azure AD. Cela offre à votre VPN l’intégralité de la protection Azure AD, y compris l’Accès conditionnel, la Multi-Factor Authentication, la conformité des appareils et la Protection des identités.

![diagramme architectural](./media/authentication-patterns/radius-auth.png)


## <a name="components-of-the-system"></a>Composants du système 

* **L’application cliente (client VPN)** : Envoie une demande d’authentification au client RADIUS.

* **client RADIUS**: Convertit les demandes à partir de l’application cliente et les envoie au serveur RADIUS sur lequel l’extension NPS est installée.

* **serveur RADIUS**: Établit une connexion avec Active Directory pour effectuer l’authentification principale pour la demande RADIUS. En cas de réussite, transmet la demande à l'extension NPS d'Azure AD Multi-Factor Authentication.

* **extension NPS**: Déclenche l'envoi d'une demande d'authentification secondaire à Azure AD Multi-Factor Authentication. En cas de réussite, l’extension NPS termine la demande d’authentification en fournissant au serveur RADIUS des jetons de sécurité qui incluent la requête Multi-Factor Authentication, émise par le service d’émission de jeton de sécurité d’Azure.

* **Azure AD Multi-Factor Authentication** : Communique avec Azure AD pour récupérer les informations de l’utilisateur et procède à une authentification secondaire à l’aide d’une méthode de vérification configurée par l’utilisateur.

## <a name="implement-radius-with-azure-ad"></a>Implémenter RADIUS avec Azure AD 

* [Fournir des fonctionnalités Azure AD Multi-Factor Authentication à l'aide de NPS](../authentication/howto-mfa-nps-extension.md) 

* [Configurer l'extension NPS d'Azure AD Multi-Factor Authentication](../authentication/howto-mfa-nps-extension-advanced.md) 

* [VPN avec Azure AD Multi-Factor Authentication à l'aide de l'extension NPS](../authentication/howto-mfa-nps-extension-vpn.md) 

  
‎ 


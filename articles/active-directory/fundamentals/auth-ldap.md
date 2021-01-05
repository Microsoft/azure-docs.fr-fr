---
title: Authentification LDAP avec Azure Active Directory
description: Guide architectural sur l’implémentation de l’authentification LDAP avec Azure Active Directory.
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
ms.openlocfilehash: f209d394e1a0c2c4ddde9cbf8df2704647e2822a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96168710"
---
# <a name="ldap-authentication-with-azure-active-directory"></a>Authentification LDAP avec Azure Active Directory

Le protocole LDAP (Lightweight Directory Access Protocol) est un protocole d’application pour l’utilisation de divers services d’annuaire. Les services d’annuaire, tels que Active Directory, [stocker des informations d’utilisateur et de compte](https://www.dnsstuff.com/active-directory-service-accounts)et des informations de sécurité telles que les mots de passe. Le service autorise ensuite le partage des informations avec d’autres périphériques sur le réseau. Les applications d’entreprise telles que les courriers électroniques, les gestionnaires de relations client (CRM) et les logiciels des ressources humaines (RH) peuvent utiliser le protocole LDAP pour s’authentifier, accéder et trouver des informations. 

Azure Active Directory (Azure AD) prend en charge ce modèle via Azure AD Domain Services (AD DS). Elle permet aux organisations qui adoptent une stratégie Cloud First de moderniser leur environnement en déplaçant leurs ressources LDAP locales vers le Cloud. Les avantages immédiats sont les suivants : 

* Intégration avec Azure AD. Les ajouts d’utilisateurs et de groupes, ou des modifications d’attribut à leurs objets, sont automatiquement synchronisés à partir de votre locataire Azure AD pour AD DS. Les modifications apportées aux objets dans les Active Directory locaux sont synchronisées avec Azure AD, puis avec AD DS.

* Simplifiez les opérations. Réduit la nécessité de conserver et de corriger manuellement les infrastructures locales. 

* Fiable. Vous bénéficiez de services gérés et hautement disponibles 

## <a name="use-when"></a>Cas d'utilisation

Il est nécessaire qu’une application ou un service utilise l’authentification LDAP.

![Diagramme de l’architecture](./media/authentication-patterns/ldap-auth.png)

## <a name="components-of-system"></a>Composants du système

* **Utilisateur** : Accède aux applications LDAP via un navigateur.

* **Navigateur Web** : L’interface avec laquelle l’utilisateur interagit pour accéder à l’URL externe de l’application.

* **Réseau virtuel** : Un réseau privé dans Azure par le biais duquel l’application héritée peut consommer des services LDAP. 

* **Applications héritées** : Applications ou charges de travail de serveur qui nécessitent le protocole LDAP déployé dans un réseau virtuel dans Azure, ou qui ont une visibilité pour AD DS des adresses IP d’instance via des itinéraires réseau. 

* **Azure AD** : Synchronise les informations d’identité à partir de l’annuaire local de l’organisation via Azure AD Connect.

* **Azure AD Domain Services (AD DS)** : Effectue une synchronisation unidirectionnelle à partir de Azure AD pour fournir l’accès à un ensemble central d’utilisateurs, de groupes et d’informations d’identification. L’instance AD DS est assignée à un réseau virtuel. Les applications, les services et les machines virtuelles qui se connectent à ce réseau virtuel peuvent ensuite utiliser des fonctionnalités AD DS courantes telles que la jonction de domaine, la stratégie de groupe, LDAP et l’authentification Kerberos/NTLM.
   > [!NOTE]
   >  Dans les environnements où l’organisation ne peut pas synchroniser les hachages de mot de passe ou les utilisateurs se connectent à l’aide de cartes à puce, nous vous recommandons d’utiliser une forêt de ressources dans AD DS. 

* **Azure AD Connect** : Outil permettant de synchroniser les informations d’identité locales avec Microsoft Azure AD. L’Assistant Déploiement et les expériences guidées vous aident à configurer les composants requis et les composants requis pour la connexion, y compris la synchronisation et l’authentification à partir de Active Directory à Azure AD. 

* **Active Directory** : Service d’annuaire qui stocke les [informations d’identité locales, telles que les informations d’utilisateur et de compte](https://www.dnsstuff.com/active-directory-service-accounts), et les informations de sécurité telles que les mots de passe.

## <a name="implement-ldap-authentication-with-azure-ad"></a>Implémenter l’authentification LDAP avec Azure AD

* [Créer et configurer une instance Azure AD DS](../../active-directory-domain-services/tutorial-create-instance.md) 

* [Configuration de la mise en réseau virtuelle pour une instance Azure AD DS](../../active-directory-domain-services/tutorial-configure-networking.md) 

* [Configurer LDAP sécurisé pour un domaine géré Azure AD DS](../../active-directory-domain-services/tutorial-configure-ldaps.md) 

* [Créer une approbation de forêt sortante vers un domaine local dans Azure AD DS](../../active-directory-domain-services/tutorial-create-forest-trust.md)


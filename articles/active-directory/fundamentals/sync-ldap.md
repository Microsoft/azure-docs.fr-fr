---
title: Synchronisation LDAP avec Azure Active Directory
description: Guide architectural sur la réalisation d’une synchronisation LDAP avec Azure Active Directory.
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
ms.openlocfilehash: 1f34e734b315c7c05ce77f5e168a452fc1c1c547
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96168676"
---
# <a name="ldap-synchronization-with-azure-active-directory"></a>Synchronisation LDAP avec Azure Active Directory

Lightweight Directory Access Protocol (LDAP) : Le protocole de services d’annuaire LDAP s’exécute sur la pile TCP/IP. Il offre un mécanisme permettant de se connecter, de rechercher et de modifier des annuaires Internet. Le service d’annuaire LDAP est basé sur un modèle client-serveur et sa fonction consiste à activer l’accès à un annuaire existant. De nombreuses entreprises dépendent de serveurs LDAP locaux pour stocker les utilisateurs et les groupes pour leurs applications métier critiques. 

Azure Active Directory (Azure AD) peut remplacer la synchronisation LDAP par Azure AD Connect. Le service de synchronisation Azure AD Connect effectue toutes les opérations liées à la synchronisation des données d’identité entre vos environnements locaux et Azure AD. 

## <a name="use-when"></a>Cas d'utilisation

Vous devez synchroniser les données d’identité entre vos répertoires LDAP v3 locaux et Azure AD. 

![Diagramme architectural](./media/authentication-patterns/ldap-sync.png)

## <a name="components-of-system"></a>Composants du système

* **Utilisateur** : Accède à une application qui s’appuie sur l’utilisation d’un annuaire LDAP v3 pour le tri des utilisateurs et des mots de passe.

* **Navigateur web**  : Composant avec lequel l’utilisateur interagit pour accéder à l’URL externe de l’application

* **Application web** : Application avec dépendances sur les annuaires LDAP v3.

* **Azure AD** : Azure AD synchronise les informations d’identité (utilisateurs, groupes, mots de passe) à partir des annuaires LDAP locaux de l’organisation via Azure AD Connect. 

* **Azure AD Connect** : un outil qui permet de se connecter à des infrastructures d’identité locales pour Microsoft Azure AD. L’assistant et les expériences guidées vous aident à déployer et à configurer les composants pré-requis et ceux requis pour la connexion. 

* **Connecteur personnalisé** : Un connecteur LDAP générique vous permet d’intégrer le service de synchronisation Azure AD Connect dans le serveur LDAP v3. Il se trouve sur Azure AD Connect.

* **Active Directory** : Active Directory est un service d’annuaire inclus dans la plupart des systèmes d’exploitation Windows Server. Les serveurs qui exécutent les services d'annuaire Active Directory sont appelés contrôleurs de domaine et s’authentifient et autorisent tous les utilisateurs et ordinateurs d’un domaine Windows.

* **Serveur LDAP v3** : Annuaire compatible avec le protocole LDAP stockant des utilisateurs d’entreprise et des mots de passe utilisés pour l’authentification des services d’annuaire.

## <a name="implement-ldap-synchronization-with-azure-ad"></a>Implémenter la synchronisation LDAP avec Azure AD

* [Identité hybride : outils d’intégration d’annuaire](../hybrid/plan-hybrid-identity-design-considerations-tools-comparison.md) 

* [Feuille de route d’installation d’Azure AD Connect](../hybrid/how-to-connect-install-roadmap.md) 

* [Présentation et création d’un connecteur LDAP](/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericldap) 

   > [!NOTE]
   > Les connecteurs LDAP sont une configuration avancée nécessitant une certaine connaissance de Forefront Identity Manager et/ou de Microsoft Identity Manager. En production, nous vous recommandons de soumettre au [Support Premier](https://support.microsoft.com/premier) ou au Microsoft Partner Network toutes les questions liées à cette configuration.


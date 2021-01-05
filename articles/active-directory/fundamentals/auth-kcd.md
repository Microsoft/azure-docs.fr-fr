---
title: Délégation Kerberos contrainte avec Azure Active Directory
description: Guide architectural sur l’implémentation de la délégation Kerberos contrainte avec Azure Active Directory.
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
ms.openlocfilehash: a1c7aa4d2300a6dee44da067b122fc7af97f7aa9
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96172854"
---
# <a name="windows-authentication---kerberos-constrained-delegation-with-azure-active-directory"></a>Authentification Windows-délégation Kerberos confrontée avec Azure Active Directory

La délégation Kerberos avec restriction (KCD) assure la délégation restreinte entre les ressources et est basée sur les noms principaux de service. Les administrateurs de domaine doivent créer les délégations et se limiter à un seul domaine. Le KCD basé sur les ressources est souvent utilisé pour fournir l’authentification Kerberos pour une application Web qui a des utilisateurs dans plusieurs domaines au sein d’une forêt Active Directory.

Le Proxy d’application Azure Active Directory fournit une authentification unique (SSO) et un accès à distance aux applications basées sur KCD qui requièrent un ticket Kerberos pour l’accès et la délégation Kerberos (KCD).

Vous activez l’authentification unique auprès de vos applications KCD locales qui utilisent l’authentification Windows intégrée (IWA) en accordant aux connecteurs du proxy d’application l’autorisation d’emprunter l’identité des utilisateurs dans Active Directory. Le connecteur de proxy d’application utilise cette autorisation pour envoyer et recevoir des jetons pour le compte de l’utilisateur.

## <a name="use-when"></a>Cas d'utilisation

Il est nécessaire de fournir un accès à distance, de protéger avec l’authentification préalable et de fournir une authentification unique aux applications IWA locales.

![Diagramme de l’architecture](./media/authentication-patterns/kcd-auth.png)

## <a name="components-of-system"></a>Composants du système

* **Utilisateur** : Accède à l’application héritée prise en charge par le proxy d’application.

* **Navigateur Web** : Composant avec lequel l’utilisateur interagit pour accéder à l’URL externe de l’application.

* **Azure AD** : Authentifie l’utilisateur. 

* **Service Proxy d’application** : Agit comme un proxy inversé pour envoyer la demande de l’utilisateur à l’application locale. Il se trouve dans Azure AD. Le proxy d’application peut également appliquer toutes les politiques d’accès conditionnel.

* **Connecteur de proxy d’application** : Installé sur site sur des serveurs Windows pour fournir une connectivité à l’application. Retourne la réponse à Azure AD. Effectue une négociation KCD avec Active Directory, en usurpant l’identité de l’utilisateur pour obtenir un jeton Kerberos pour l’application.

* **Active Directory** : Envoie le jeton Kerberos pour l’application au connecteur de proxy d’application.

* **Applications héritées** : Applications qui reçoivent les requêtes des utilisateurs du proxy d’application. Les applications héritées renvoient la réponse au connecteur de proxy d’application.

## <a name="implement-windows-authentication-kcd-with-azure-ad"></a>Implémenter l’authentification Windows (KCD) avec Azure AD

* [Délégation contrainte Kerberos pour l’authentification unique à vos applications avec le proxy d’application](../manage-apps/application-proxy-configure-single-sign-on-with-kcd.md) 

* [Ajouter une application locale pour l’accès à distance via le proxy d’application dans Azure Active Directory](../manage-apps/application-proxy-add-on-premises-application.md)


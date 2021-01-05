---
title: Accès à distance aux applications locales – Proxy d’application Azure Active Directory
description: Le proxy d’application Azure Active Directory offre un accès à distance sécurisé pour les applications web. Après une authentification unique à Azure AD, les utilisateurs peuvent accéder aux applications cloud et locales par le biais d’une URL externe ou un portail d’applications interne. Par exemple, Application Proxy peut fournir un accès à distance et une authentification unique aux applications Bureau à distance, SharePoint, Teams, Tableau, Qlik et aux applications métier (LOB).
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 9d620342d53bdae0b3a520000f6d240eb0e28d15
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96180590"
---
# <a name="remote-access-to-on-premises-applications-through-azure-ad-application-proxy"></a>Accès à distance aux applications locales par le bais du service Proxy d'application Azure AD

Le proxy d’application Azure Active Directory offre un accès à distance sécurisé pour les applications web. Après une authentification unique à Azure AD, les utilisateurs peuvent accéder aux applications cloud et locales par le biais d’une URL externe ou un portail d’applications interne. Par exemple, Application Proxy peut fournir un accès à distance et une authentification unique aux applications Bureau à distance, SharePoint, Teams, Tableau, Qlik et aux applications métier (LOB).

Le Proxy d’application Azure AD est :

- **Simple à utiliser**. Les utilisateurs peuvent accéder aux applications locales de la même manière qu’ils accèdent aux applications Microsoft 365 et aux autres applications SaaS intégrées à Azure AD. Vous n’avez pas besoin de modifier ou de mettre à jour vos applications pour qu’elles fonctionnent avec le Proxy d’application.

- **Sécurisé**. Les applications locales peuvent utiliser les contrôles d’autorisation et les analyses de sécurité d’Azure. Par exemple, les applications locales peuvent utiliser l’accès conditionnel et la vérification en deux étapes. Proxy d’application ne vous oblige à ouvrir les connexions entrantes par le biais de votre pare-feu.

- **Économique**. Les solutions locales vous obligent généralement à configurer et à gérer des zones démilitarisées (DMZ), des serveurs Edge ou autres infrastructures complexes. Proxy d’application s’exécute dans le cloud, ce qui simplifie son utilisation. Pour utiliser le Proxy d’application, vous n’avez pas besoin de modifier l’infrastructure de réseau ou d’installer des appliances supplémentaires dans votre environnement local.

## <a name="what-is-application-proxy"></a>Présentation de Proxy d’application
Le Proxy d’application est une fonctionnalité d’Azure AD qui permet aux utilisateurs d’accéder à des applications web locales à partir d’un client distant. Le Proxy d’application inclut le service Proxy d’application qui s’exécute dans le cloud et le connecteur Proxy d’Application qui s’exécute sur un serveur local. Azure AD, le service Proxy d'application et le connecteur Proxy d’application fonctionnent ensemble pour transmettre en toute sécurité le jeton de connexion utilisateur Azure AD à l'application Web.

Proxy d’application fonctionne avec les ressources suivantes :

* Applications web qui utilisent [l’authentification Windows intégrée](application-proxy-configure-single-sign-on-with-kcd.md) pour l’authentification
* Applications web qui utilisent l’accès [basé sur un en-tête](./application-proxy-configure-single-sign-on-with-headers.md) ou sur un formulaire
* API web que vous voulez exposer aux applications enrichies sur différents appareils
* Applications hébergées derrière une [passerelle Bureau à distance](application-proxy-integrate-with-remote-desktop-services.md)
* Applications clientes complètes intégrées à la bibliothèque d’authentification Microsoft (MSAL)

Proxy d’application prend en charge l’authentification unique. Pour plus d’informations sur les méthodes prises en charge, consultez [Choix d’une méthode d’authentification unique](sso-options.md#choosing-a-single-sign-on-method).

Le proxy d’application est l’outil recommandé pour permettre aux utilisateurs distants d’accéder aux ressources internes. Proxy d’application remplace la nécessité d’un VPN ou d’un proxy inversé. Il n’est pas destiné aux utilisateurs internes sur le réseau d’entreprise.  Ces utilisateurs qui utilisent inutilement le proxy d’application peuvent ralentir les performances de manière inattendue et indésirable.

## <a name="how-application-proxy-works"></a>Fonctionnement de Proxy d’application

Le diagramme suivant montre comment Azure AD et Application Proxy fonctionnent ensemble pour fournir une authentification unique aux applications locales.

![Diagramme du Proxy d’application Azure AD](./media/application-proxy/azureappproxxy.png)

1. Une fois que l’utilisateur a accédé à l’application par le biais d’un point de terminaison, il est dirigé vers la page de connexion Azure AD.
2. Après la connexion, Azure AD envoie un jeton à l’appareil client de l’utilisateur.
3. Le client envoie le jeton au service Proxy d’application qui récupère le nom d’utilisateur principal (UPN) et le nom de sécurité principal (SPN) du jeton. Proxy d’application envoie la requête au connecteur Proxy d’application.
4. Si vous avez configuré l’authentification unique, le connecteur effectue toute authentification supplémentaire requise pour le compte de l’utilisateur.
5. Le connecteur envoie la requête à l’application locale.
6. La réponse est envoyée à l’utilisateur par le biais du connecteur et du service Proxy d’application.

> [!NOTE]
> Comme la plupart des agents hybrides Azure AD, le connecteur Proxy d’application ne nécessite pas l’ouverture de connexions entrantes via votre pare-feu. Le trafic utilisateur à l’étape 3 se termine au niveau du service de proxy d’application (dans Azure AD). Le connecteur Proxy d’application (local) est responsable du reste de la communication.
>


| Composant | Description |
| --------- | ----------- |
| Point de terminaison  | Le point de terminaison est une URL ou un [portail d’utilisateurs finaux](end-user-experiences.md). Les utilisateurs peuvent accéder à des applications en dehors de votre réseau au moyen d’une URL externe. Les utilisateurs au sein de votre réseau peuvent accéder à l’application par le biais d’une URL ou d’un portail d’utilisateurs finaux. Lorsque les utilisateurs passent par l’un de ces points de terminaison, ils s’authentifient dans Azure AD, puis sont acheminés via le connecteur vers l’application locale.|
| Azure AD | Azure AD effectue l’authentification en utilisant l’annuaire des locataires stocké dans le cloud. |
| Service Proxy d’application | Ce service Proxy d’application s’exécute dans le cloud dans le cadre d’Azure AD. Il transmet le jeton d’authentification de l’utilisateur pour le connecteur Proxy d’application. Proxy d'application transmet tous les en-têtes accessibles de la requête et définit les en-têtes à l’adresse IP du client conformément à son protocole. Si la requête entrante vers le proxy possède déjà cet en-tête, l’adresse IP du client est ajoutée à la fin de la liste délimitée par des virgules qui est, en fait, la valeur de l’en-tête.|
| Connecteur Proxy d'application | Le connecteur est un agent léger qui s’exécute sur un serveur Windows au sein de votre réseau. Le connecteur gère la communication entre le service Application Proxy dans le cloud et l’application locale. Le connecteur n’utilise que des connexions sortantes ; vous n’êtes donc pas obligé d’ouvrir des ports entrants ou de placer quoi que ce soit dans la zone démilitarisée. Les connecteurs sont sans état et extraient les informations dont ils ont besoin sur le cloud. Pour savoir comment les connecteurs équilibrent la charge ou s’authentifient, voir [Présentation des connecteurs de proxy d’application Azure AD](application-proxy-connectors.md).|
| Active Directory (AD) | Active Directory s'exécute localement pour effectuer l’authentification des comptes de domaine. Lorsque l’authentification unique est configurée, le connecteur communique avec AD pour effectuer toute authentification supplémentaire requise.
| Application locale | Enfin, l’utilisateur est en mesure d’accéder à une application locale.

## <a name="next-steps"></a>Étapes suivantes
Pour commencer à utiliser Proxy d’application, consultez [Tutoriel : Ajouter des applications locales pour un accès à distance via le service Proxy d’application](application-proxy-add-on-premises-application.md).
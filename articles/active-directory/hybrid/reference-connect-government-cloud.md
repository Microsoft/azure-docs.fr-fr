---
title: 'Azure AD Connect : Considérations relatives à l’identité hybride pour l’Azure Government Cloud'
description: Considérations spéciales relatives au déploiement d’Azure AD Connect avec l’Azure Government Cloud.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/14/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e163ea34948906060996ee952f45ec0cdb467557
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504353"
---
# <a name="hybrid-identity-considerations-for-the-azure-government-cloud"></a>Considérations relatives à l’identité hybride pour l’Azure Government Cloud

Cet article contient des considérations relatives à l’intégration d’un environnement hybride avec le Microsoft Azure Government Cloud. Ces informations sont fournies à titre de référence pour les administrateurs et architectes qui travaillent avec l’Azure Government Cloud.

> [!NOTE]
> Pour intégrer un environnement Microsoft Active Directory (local ou hébergé dans un IaaS faisant partie de la même instance cloud) au cloud Azure Government, vous devez effectuer une mise à niveau vers la dernière version d’[Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594).

> [!NOTE]
> Cet article contient des références au terme *liste verte*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

Pour obtenir la liste complète des points de terminaison du ministère de la défense du gouvernement des États-Unis, reportez-vous à la [documentation](/office365/enterprise/office-365-u-s-government-dod-endpoints).

## <a name="azure-ad-pass-through-authentication"></a>Authentification directe Azure AD

Les informations suivantes décrivent l’implémentation de l’authentification directe et de l’Azure Government Cloud.

### <a name="allow-access-to-urls"></a>Autoriser l'accès à des URL

Avant de déployer l’agent d’authentification directe, vérifiez s’il existe un pare-feu entre vos serveurs et Azure AD. Si votre pare-feu ou proxy autorise les programmes sécurisés ou avec blocage DNS, ajoutez les connexions suivantes.

> [!NOTE]
> Les instructions suivantes s’appliquent également à l’installation du [connecteur de proxy d’application](../manage-apps/what-is-application-proxy.md) pour les environnements Azure Government.

|URL |Utilisation|
|-----|-----|
|&#42;.msappproxy.us</br>&#42;.servicebus.usgovcloudapi.net|L’agent utilise ces URL pour communiquer avec le service cloud Azure AD. |
|`mscrl.microsoft.us:80` </br>`crl.microsoft.us:80` </br>`ocsp.msocsp.us:80` </br>`www.microsoft.us:80`| Azure utilise ces URL pour vérifier les certificats.|
|login.windows.us </br>secure.aadcdn.microsoftonline-p.com </br>&#42;.microsoftonline.us </br>&#42;.microsoftonline-p.us </br>&#42;.msauth.net </br>&#42;.msauthimages.net </br>&#42;.msecnd.net</br>&#42;.msftauth.net </br>&#42;.msftauthimages.net</br>&#42;.phonefactor.net </br>enterpriseregistration.windows.net</br>management.azure.com </br>policykeyservice.dc.ad.msft.net</br>ctldl.windowsupdate.us:80| L’agent utilise ces URL lors du processus d’inscription.

### <a name="install-the-agent-for-the-azure-government-cloud"></a>Installer l’agent pour le cloud Azure Government

Procédez comme suit pour installer l’agent pour l’Azure Government Cloud :

1. Dans le terminal en ligne de commande, accédez au dossier contenant le fichier exécutable qui installe l’agent.
1. Exécutez les commandes suivantes qui spécifient que l’installation est destinée à Azure Government.

   Pour l’authentification directe :

   ```
   AADConnectAuthAgentSetup.exe ENVIRONMENTNAME="AzureUSGovernment"
   ```

   Pour le proxy d’application :

   ```
   AADApplicationProxyConnectorInstaller.exe ENVIRONMENTNAME="AzureUSGovernment" 
   ```

## <a name="single-sign-on"></a>Authentification unique

### <a name="set-up-your-azure-ad-connect-server"></a>Configurer votre serveur Azure AD Connect

Si vous utilisez l’authentification directe comme méthode de connexion, aucune vérification supplémentaire de conditions préalables n’est nécessaire. Si vous utilisez la synchronisation de hachage du mot de passe comme méthode de connexion, et s’il y a un pare-feu entre Azure AD Connect et Azure AD, vérifiez les points suivants :

- Vous utilisez Azure AD Connect, version 1.1.644.0 ou ultérieure.
- Si votre pare-feu ou proxy autorise les programmes sécurisés ou avec blocage DNS, ajoutez les connexions aux URL &#42;.msappproxy.us sur le port 443.

  Dans le cas contraire, autorisez l’accès aux plages d’adresses IP du centre de données Azure, qui sont mises à jour chaque semaine. Cette condition préalable s’applique uniquement lorsque vous activez la fonctionnalité. Elle n’est pas requise pour les connexions d’utilisateur réelles.

### <a name="roll-out-seamless-single-sign-on"></a>Déployer l’Authentification unique transparente

Vous pouvez déployer progressivement l’Authentification unique transparente Azure AD vers vos utilisateurs en suivant les instructions suivantes. Vous commencez par ajouter l’URL Azure AD `https://autologon.microsoft.us` aux paramètres de zone intranet de tous les utilisateurs ou d’utilisateurs sélectionnés en utilisant une stratégie de groupe dans Active Directory.

Vous devez également activer un paramètre de stratégie de zone intranet **Autoriser les mises à jour de la barre d’état via un script dans la stratégie de groupe**.

## <a name="browser-considerations"></a>Considérations sur le navigateur

### <a name="mozilla-firefox-all-platforms"></a>Mozilla Firefox (toutes les plateformes)

Mozilla Firefox n'utilise pas automatiquement l’authentification Kerberos. Chaque utilisateur doit ajouter manuellement l’URL Azure AD à ses paramètres Firefox en procédant comme suit :

1. Exécuter Firefox et entrer  **about:config** dans la barre d’adresse. Ignorer les notifications éventuelles.
1. Recherchez la préférence **Network. Negotiate-auth. Trusted-URI** . Cette préférence répertorie les sites approuvés par Firefox pour l’authentification Kerberos.
1. Cliquez avec le bouton droit sur le nom de la préférence, puis sélectionnez **Modifier**.
1. Entrez `https://autologon.microsoft.us` dans la zone.
1. Sélectionnez  **OK** , puis rouvrez le navigateur.

### <a name="microsoft-edge-based-on-chromium-all-platforms"></a>Microsoft Edge basé sur Chromium (toutes les plateformes)

Si vous avez remplacé les paramètres de stratégie `AuthNegotiateDelegateAllowlist` ou `AuthServerAllowlist` dans votre environnement, veillez à y ajouter également l’URL d’Azure AD `https://autologon.microsoft.us`.

### <a name="google-chrome-all-platforms"></a>Google Chrome (toutes les plateformes)

Si vous avez remplacé les paramètres de stratégie `AuthNegotiateDelegateWhitelist` ou `AuthServerWhitelist` dans votre environnement, veillez à y ajouter également l’URL d’Azure AD `https://autologon.microsoft.us`.

## <a name="next-steps"></a>Étapes suivantes

- [Authentification directe](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites)
- [Authentification unique](how-to-connect-sso-quick-start.md#step-1-check-the-prerequisites)

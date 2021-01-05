---
title: Ports et protocoles nécessaires pour l’identité hybride - Azure | Microsoft Docs
description: Cette page contient des informations de référence techniques sur les ports qui doivent être ouverts pour Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: de97b225-ae06-4afc-b2ef-a72a3643255b
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 03/04/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: da318840426d1c0b94eab06b89ff3152df9d26fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80331088"
---
# <a name="hybrid-identity-required-ports-and-protocols"></a>Ports et protocoles nécessaires à l’identité hybride
Le document suivant est une référence technique sur les ports et les protocoles nécessaires à l’implémentation d’une solution d’identité hybride. Consultez la figure ci-dessous et reportez-vous au tableau correspondant.

![Qu’est-ce qu’Azure AD Connect ?](./media/reference-connect-ports/required3.png)

## <a name="table-1---azure-ad-connect-and-on-premises-ad"></a>Tableau 1 : Azure AD Connect et AD local
Ce tableau décrit les ports et les protocoles nécessaires à la communication entre le serveur Azure AD Connect et l’AD local.

| Protocol | Ports | Description |
| --- | --- | --- |
| DNS |53 (TCP/UDP) |Recherches DNS dans la forêt de destination. |
| Kerberos |88 (TCP/UDP) |Authentification Kerberos auprès de la forêt AD. |
| MS-RPC |135 (TCP) |Utilisé pendant la configuration initiale de l’Assistant Azure AD au moment d’établir une liaison avec la forêt AD, ainsi que pendant la synchronisation du mot de passe. |
| LDAP |389 (TCP/UDP) |Utilisé pour l’importation de données à partir d’AD. Les données sont chiffrées à l’aide de Kerberos Sign & Seal. |
| SMB | 445 (TCP) |Utilisé par l’authentification unique transparente pour créer un compte d’ordinateur dans la forêt AD. |
| LDAP/SSL |636 (TCP/UDP) |Utilisé pour l’importation de données à partir d’AD. Le transfert de données est signé et chiffré. Utilisé uniquement si vous utilisez TLS. |
| RPC |49152-65535 (port RPC aléatoire élevé)(CP) |Utilisé pendant la configuration initiale d’Azure AD Connect au moment d’établir une liaison avec les forêts AD, ainsi que pendant la synchronisation du mot de passe. Pour plus d’informations, consultez les sections [KB929851](https://support.microsoft.com/kb/929851), [KB832017](https://support.microsoft.com/kb/832017) et [KB224196](https://support.microsoft.com/kb/224196). |
|WinRM  | 5985 (TCP) |Utilisé uniquement si vous installez AD FS avec l’assistant de connexion gMSA par Azure AD Connect|
|Services web AD DS | 9389 (TCP) |Utilisé uniquement si vous installez AD FS avec l’assistant de connexion gMSA par Azure AD Connect |

## <a name="table-2---azure-ad-connect-and-azure-ad"></a>Tableau 2 : Azure AD Connect et Azure AD
Ce tableau décrit les ports et les protocoles nécessaires à la communication entre le serveur Azure AD Connect et Azure AD.

| Protocol | Ports | Description |
| --- | --- | --- |
| HTTP |80 (TCP) |Utilisé pour télécharger des listes de révocation de certificats en vue de vérifier les certificats TLS/SSL. |
| HTTPS |443 (TCP) |Utilisé pour établir une synchronisation avec Azure AD. |

Pour obtenir la liste des URL et adresses IP que vous devez ouvrir dans votre pare-feu, consultez [URL et plages d’adresses IP Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) et [Résolution des problèmes de connectivité Azure AD Connect](tshoot-connect-connectivity.md#troubleshoot-connectivity-issues-in-the-installation-wizard).

## <a name="table-3---azure-ad-connect-and-ad-fs-federation-serverswap"></a>Tableau 3 - Azure AD Connect et serveurs de fédération AD FS/WAP
Ce tableau décrit les ports et les protocoles nécessaires à la communication entre le serveur Azure AD Connect et les serveurs de fédération AD FS/WAP.  

| Protocol | Ports | Description |
| --- | --- | --- |
| HTTP |80 (TCP) |Utilisé pour télécharger des listes de révocation de certificats en vue de vérifier les certificats TLS/SSL. |
| HTTPS |443 (TCP) |Utilisé pour établir une synchronisation avec Azure AD. |
| WinRM |5985 |Écouteur WinRM |

## <a name="table-4---wap-and-federation-servers"></a>Tableau 4 - Serveurs de fédération et WAP
Ce tableau décrit les ports et les protocoles nécessaires à la communication entre les serveurs de fédération et les serveurs WAP.

| Protocol | Ports | Description |
| --- | --- | --- |
| HTTPS |443 (TCP) |Utilisé pour l’authentification. |

## <a name="table-5---wap-and-users"></a>Tableau 5 - WAP et utilisateurs
Ce tableau décrit les ports et les protocoles nécessaires à la communication entre les utilisateurs et les serveurs WAP.

| Protocol | Ports | Description |
| --- | --- | --- |
| HTTPS |443 (TCP) |Utilisé pour l’authentification des appareils. |
| TCP |49443 (TCP) |Utilisé pour l’authentification par certificat. |

## <a name="table-6a--6b---pass-through-authentication-with-single-sign-on-sso-and-password-hash-sync-with-single-sign-on-sso"></a>Tableau 6a et 6b - Authentification directe avec authentification unique (SSO) et synchronisation du hachage de mot de passe avec authentification unique (SSO)
Les tableaux suivants décrivent les ports et les protocoles nécessaires à la communication entre Azure AD Connect et Azure AD.

### <a name="table-6a---pass-through-authentication-with-sso"></a>Tableau 6a - Authentification directe avec authentification unique (SSO)
|Protocol|Numéro de port|Description
| --- | --- | ---
|HTTP|80|Active le trafic HTTP sortant pour valider la sécurité, comme SSL. Également nécessaire pour que la fonctionnalité de mise à jour automatique du connecteur fonctionne correctement.
|HTTPS|443| Active le trafic HTTPS sortant pour les opérations telles que l’activation et la désactivation de la fonctionnalité, l’inscription des connecteurs, le téléchargement des mises à jour du connecteur et la gestion des demandes de connexion de tous les utilisateurs.

En outre, Azure AD Connect doit être en mesure d’établir des connexions IP directes aux [plages d’adresses IP du centre de données Azure](https://www.microsoft.com/download/details.aspx?id=41653).

### <a name="table-6b---password-hash-sync-with-sso"></a>Tableau 6b - synchronisation du hachage de mot de passe avec authentification unique

|Protocol|Numéro de port|Description
| --- | --- | ---
|HTTPS|443| Active l’inscription à l’authentification unique (obligatoire uniquement pour le processus d’inscription SSO).

En outre, Azure AD Connect doit être en mesure d’établir des connexions IP directes aux [plages d’adresses IP du centre de données Azure](https://www.microsoft.com/download/details.aspx?id=41653). Là encore, cette option est uniquement requise pour le processus d’inscription SSO.

## <a name="table-7a--7b---azure-ad-connect-health-agent-for-ad-fssync-and-azure-ad"></a>Tableau 7a & 7b - Agent Azure AD Connect Health pour (AD FS/Sync) et Azure AD
Les tableaux suivants décrivent les points de terminaison, les ports et les protocoles nécessaires à la communication entre les agents Azure AD Connect Health et Azure AD.

### <a name="table-7a---ports-and-protocols-for-azure-ad-connect-health-agent-for-ad-fssync-and-azure-ad"></a>Tableau 7a - Ports et protocoles pour l’agent Azure AD Connect Health pour (AD FS/Sync) et Azure AD
Ce tableau décrit les ports et les protocoles sortants nécessaires à la communication entre les agents Azure AD Connect Health et Azure AD.  

| Protocol | Ports | Description |
| --- | --- | --- |
| HTTPS |443 (TCP) |Règle de trafic sortant |
| Azure Service Bus |5671 (TCP) |Règle de trafic sortant |

Le port 5671 d’Azure Service Bus n'est plus requis pour la dernière version de l'agent. La dernière version de l’agent Azure AD Connect Health ne nécessite que le port 443.

### <a name="7b---endpoints-for-azure-ad-connect-health-agent-for-ad-fssync-and-azure-ad"></a>7b - Points de terminaison pour l’agent Azure AD Connect Health pour (AD FS/Sync) et Azure AD
Pour obtenir la liste des points de terminaison, consultez la section [Configuration requise pour l’agent Azure AD Connect Health](how-to-connect-health-agent-install.md#requirements).


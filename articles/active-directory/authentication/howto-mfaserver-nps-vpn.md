---
title: Serveur Azure MFA et VPN tiers – Azure Active Directory
description: Guides de configuration détaillé pour le serveur Azure MFA à intégrer à Cisco, Citrix et Juniper.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 88fe09199cb50d2a3796c3b638dca1a723016dc4
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742015"
---
# <a name="advanced-scenarios-with-azure-mfa-server-and-third-party-vpn-solutions"></a>Scénarios avancés avec le serveur Azure MFA et les solutions VPN tierces

Le serveur Azure Multi-Factor Authentication (serveur Azure MFA) peut être utilisé pour se connecter en toute transparence à différentes solutions VPN tierces. Cet article se concentre sur l’appliance Cisco&reg; ASA VPN, l’appliance Citrix NetScaler SSL VPN et l’appliance Juniper Networks Secure Access/Pulse Secure Connect Secure SSL VPN. Nous avons créé des guides de configuration pour gérer ces trois appareils courants. Le serveur Azure MFA peut également s’intégrer à la plupart des autres systèmes qui utilisent RADIUS, LDAP, IIS ou l’authentification basée sur les revendications pour AD FS. Pour plus de détails, consultez la rubrique [Configurations de serveur Azure MFA](howto-mfaserver-deploy.md#next-steps).

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisée avec Azure AD Multi-Factor Authentication](tutorial-enable-azure-mfa.md).
>
> Si vous utilisez l’authentification multifacteur basée sur le cloud, consultez [Intégrer votre infrastructure VPN avec Azure MFA](howto-mfa-nps-extension-vpn.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

## <a name="cisco-asa-vpn-appliance-and-azure-mfa-server"></a>Appliance Cisco ASA VPN et serveur Azure MFA
Le serveur Azure MFA s’intègre à votre appliance Cisco&reg; ASA VPN pour fournir une sécurité supplémentaire pour les connexions VPN Cisco AnyConnect&reg; et l’accès au portail.  Vous pouvez utiliser le protocole RADIUS ou LDAP.  Sélectionnez l'une des opérations suivantes pour télécharger les guides de configuration détaillés.

| Guide de configuration | Description |
| --- | --- |
| [Cisco ASA avec Anyconnect VPN et Azure MFA Configuration pour LDAP](https://download.microsoft.com/download/A/2/0/A201567C-C3DE-4227-AF89-4567A470899E/Cisco_ASA_Azure_MFA_LDAP.docx) | Intégrer votre appliance VPN Cisco ASA avec Azure MFA à l’aide de LDAP |
| [Cisco ASA avec Anyconnect VPN et Azure MFA Configuration pour RADIUS](https://download.microsoft.com/download/4/5/7/4579C1CF-35B0-4FBE-8A1A-B49CB2CC0382/Cisco_ASA_Azure_MFA_RADIUS.docx) | Intégrer votre appliance VPN Cisco ASA avec Azure MFA à l’aide de RADIUS |

## <a name="citrix-netscaler-ssl-vpn-and-azure-mfa-server"></a>Citrix NetScaler SSL VPN et serveur Azure MFA
Le serveur Azure MFA s’intègre à votre appliance VPN Citrix NetScaler SSL pour offrir une sécurité supplémentaire pour les connexions VPN et le portail Citrix NetScaler SSL.  Vous pouvez utiliser le protocole RADIUS ou LDAP.  Sélectionnez l'une des opérations suivantes pour télécharger les guides de configuration détaillés.

| Guide de configuration | Description |
| --- | --- |
| [Citrix NetScaler SSL VPN et Azure MFA Configuration pour LDAP](https://download.microsoft.com/download/2/4/E/24E1E722-72DF-471F-A88A-D1338DB1AF83/Citrix_NS_Azure_MFA_LDAP.docx) | Intégrer votre appliance VPN Citrix NetScaler SSL avec Azure MFA à l’aide de LDAP |
| [Citrix NetScaler SSL VPN et Azure MFA Configuration pour RADIUS](https://download.microsoft.com/download/1/A/4/1A482764-4A63-45C2-A5EC-2B673ACCDD12/Citrix_NS_Azure_MFA_RADIUS.docx) | Intégrer votre appliance VPN Citrix NetScaler SSL avec Azure MFA à l’aide de RADIUS |

## <a name="juniperpulse-secure-ssl-vpn-appliance-and-azure-mfa-server"></a>Appliance Juniper/Pulse Secure SSL VPN et serveur Azure MFA
Le serveur Azure MFA s’intègre à votre appliance Juniper/Pulse Secure SSL VPN pour fournir une sécurité supplémentaire pour les connexions VPN et l’accès au portail Juniper/Pulse Secure SSL.  Vous pouvez utiliser le protocole RADIUS ou LDAP.  Sélectionnez l'une des opérations suivantes pour télécharger les guides de configuration détaillés.

| Guide de configuration | Description |
| --- | --- |
| [Juniper/Pulse Secure SSL VPN et Azure MFA Configuration pour LDAP](https://download.microsoft.com/download/6/5/8/6587B418-75B1-4FCB-84D4-984BC479309E/JuniperPulse_Azure_MFA_LDAP.docx) | Intégrer votre appliance VPN Juniper/Pulse Secure SSL avec Azure MFA à l’aide de LDAP |
| [Juniper/Pulse Secure SSL VPN et Azure MFA Configuration pour RADIUS](https://download.microsoft.com/download/7/9/A/79AB3DAD-4799-4379-B1DA-B95ABDF231DC/JuniperPulse_Azure_MFA_RADIUS.docx) | Intégrer votre appliance VPN Juniper/Pulse Secure SSL avec Azure MFA à l’aide de RADIUS |

## <a name="next-steps"></a>Étapes suivantes

- [Augmenter votre infrastructure d’authentification existante avec l’extension NPS pour Azure Multi-Factor Authentication](howto-mfa-nps-extension.md)

- [Configurer les paramètres d’Azure Multi-Factor Authentication](howto-mfa-mfasettings.md)

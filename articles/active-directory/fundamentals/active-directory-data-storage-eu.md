---
title: Stockage de données d’identité pour les clients européens - Azure AD
description: Découvrez où Azure Active Directory stocke les données relatives à l’identité de ses clients européens.
services: active-directory
author: ajburnle
manager: daveba
ms.author: ajburnle
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 09/15/2020
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7a8b013723707c4a3a087a90674227c3d41c5108
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94836935"
---
# <a name="identity-data-storage-for-european-customers-in-azure-active-directory"></a>Stockage de données d’identité pour les clients européens dans Azure Active Directory
Les données d’identité sont stockées par Azure AD dans un emplacement géographique en fonction de l’adresse fournie par votre organisation lors de l’abonnement à un service Microsoft Online tel que Microsoft 365 ou Azure. Pour obtenir des informations sur le lieu de stockage des données d’identité, vous pouvez utiliser la section [Où se trouvent vos données ?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) du Centre de gestion de la confidentialité Microsoft.

Pour les clients qui ont fourni une adresse en Europe, Azure AD conserve la plupart des données d’identité dans des centres de données européens. Ce document apporte des informations sur toutes les données stockées en dehors de l’Europe par les services Azure AD.

## <a name="microsoft-azure-ad-multi-factor-authentication"></a>Microsoft Azure AD Multi-Factor Authentication

Pour Azure AD Multi-Factor Authentication basé sur le cloud, l’authentification s’effectue dans le centre de données le plus proche de l’utilisateur. Des centres de données pour Azure AD Multi-Factor Authentication sont présents en Amérique du Nord, en Europe et en Asie-Pacifique.

* Les authentifications multifacteur s’appuyant sur des appels téléphoniques sont issues de centres de données des États-Unis et sont routées par des fournisseurs globaux.
* L’authentification multifacteur par SMS est acheminée par les fournisseurs globaux.
* Les demandes d’authentification multifacteur utilisant les notifications push d’application Microsoft Authenticator qui proviennent de centres de données européens sont traitées dans les centres de données de l’Union européenne.
    * Les services spécifiques aux fournisseurs d’appareils, comme les notifications push Apple, peuvent se trouver en dehors de l’Europe.
* Les demandes d’authentification multifacteur utilisant des codes OATH qui proviennent de centres de données européens sont validées dans l’Union européenne.

Pour plus d’informations sur les informations utilisateur collectées par le serveur Azure Multi-Factor Authentication (serveur MFA) et Azure AD MFA basé sur le cloud, consultez [Collecte de données utilisateur via Azure Multi-Factor Authentication](../authentication/howto-mfa-reporting-datacollection.md).

## <a name="password-based-single-sign-on-for-enterprise-applications"></a>Authentification unique par mot de passe pour les applications d’entreprise
 
Si un client crée une application d’entreprise (par le biais de la galerie Azure AD Galerie ou non) et active l’authentification unique par mot de passe, l’URL de connexion de l’application et les champs de connexion capturés personnalisés sont stockés aux États-Unis. Pour plus d’informations sur cette fonctionnalité, consultez [Configurer l’authentification unique par mot de passe](../manage-apps/configure-password-single-sign-on-non-gallery-applications.md).

## <a name="microsoft-azure-active-directory-b2c-azure-ad-b2c"></a>Microsoft Azure Active Directory B2C (Azure AD B2C)

Les données de configuration de stratégie Azure AD B2C et les conteneurs de clés sont stockés dans les centres de données des États-Unis. Ces derniers ne contiennent pas les données personnelles utilisateur. Pour plus d’informations sur les configurations de stratégie, consultez l’article [Azure Active Directory B2C : stratégies intégrées](../../active-directory-b2c/user-flow-overview.md).

## <a name="microsoft-azure-active-directory-b2b-azure-ad-b2b"></a>Microsoft Azure Active Directory B2B (Azure AD B2B) 
    
Azure AD B2B stocke les invitations avec lien d’acceptation et URL de redirection dans les centres de données des États-Unis. En outre, les adresses e-mail des utilisateurs qui se désabonnent des invitations B2B sont également stockées dans les centres de données des États-Unis.

## <a name="microsoft-azure-active-directory-domain-services-azure-ad-ds"></a>Microsoft Azure Active Directory Domain Services (Azure AD DS)

Azure AD DS stocke les données utilisateur au même emplacement que le réseau virtuel Azure choisi par le client. Par conséquent, si le réseau se situe en dehors de l’Europe, les données sont répliquées et stockées en dehors de l’Europe.

## <a name="federation-in-microsoft-exchange-server-2013"></a>Fédération dans Microsoft Exchange Server 2013
    
- Identificateur d’application (AppID) : numéro unique généré par le système d’authentification Azure Active Directory pour identifier les organisations Exchange.
- Liste des domaines fédérés approuvés pour l’application
- Clé publique de signature des jetons de l’application 

Pour plus d’informations sur la fédération dans Microsoft Exchange Server, consultez [Fédération : Article d’aide sur Exchange 2013](/exchange/federation-exchange-2013-help).


## <a name="other-considerations"></a>Autres considérations

Les services et les applications s’intégrant avec Azure AD ont accès aux données d’identité. Évaluez chaque service et application que vous utilisez pour déterminer comment ils traitent les données d’identité, et s’ils répondent aux besoins de stockage de données de votre entreprise.

Pour plus d’informations sur la résidence des données des services de Microsoft, consultez la section [Où se trouvent vos données ?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) du Microsoft Trust Center.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les fonctions et fonctionnalités décrites ci-dessus, consultez les articles suivants :
- [Présentation d’authentification multifacteur](../authentication/concept-mfa-howitworks.md)

- [Réinitialisation de mot de passe en libre-service Azure AD](../authentication/concept-sspr-howitworks.md)

- [Qu’est-ce qu’Azure Active Directory B2C ?](../../active-directory-b2c/overview.md)

- [Qu'est-ce que la collaboration B2B d'Azure AD ?](../external-identities/what-is-b2b.md)

- [Services de domaine Azure Active Directory (AD)](../../active-directory-domain-services/overview.md)
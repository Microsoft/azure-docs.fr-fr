---
title: Présentation de l’identité hybride avec Azure Active Directory
description: L’identité hybride a une identité d’utilisateur commune pour l’authentification et l’autorisation, tant en local que dans le cloud.
keywords: introduction à Azure AD Connect, présentation d’Azure AD Connect, qu’est-ce qu’Azure AD Connect, installation d’active directory
services: active-directory
author: billmath
manager: daveba
ms.assetid: 59bd209e-30d7-4a89-ae7a-e415969825ea
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 05/17/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 895355b5acebffe6ad24b15b0c709d21e8f20be4
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/11/2020
ms.locfileid: "90016646"
---
# <a name="what-is-hybrid-identity-with-azure-active-directory"></a>Présentation de l’identité hybride avec Azure Active Directory

Les entreprises et les organisations utilisent aujourd’hui de plus en plus souvent une combinaison d’applications locales et cloud.  Les utilisateurs doivent avoir accès à ces applications en local et dans le cloud, Les scénarios de gestion des utilisateurs en local et dans le cloud constituent un défi. 

Les solutions d'identité de Microsoft regroupent des fonctionnalités, locales et cloud.  Ces solutions créent une identité d'utilisateur unique commune pour l'authentification et l'autorisation d'accès à toutes les ressources, indépendamment de leur l'emplacement. Nous appelons cette identité **identité hybride**.

Avec l’identité hybride pour Azure AD et la gestion de l’identité hybride, ces scénarios deviennent possibles.

Pour obtenir l’identité hybride avec Azure AD, l'une des trois méthodes d’authentification peut être utilisée, selon vos scénarios.   Ces trois méthodes sont les suivantes : 

- **[Synchronisation de hachage de mot de passe (PHS)](whatis-phs.md)**  
- **[Authentification directe (PTA)](how-to-connect-pta.md)**  
- **[Fédération (AD FS)](whatis-fed.md)** 

Ces méthodes d’authentification s'accompagnent également de fonctionnalités d'[authentification unique](how-to-connect-sso.md).  L’authentification unique connecte automatiquement les utilisateurs lorsque leurs appareils d’entreprise sont connectés au réseau de l’entreprise.

Pour plus d'informations, consultez [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure Active Directory](./choose-ad-authn.md). 

## <a name="common-scenarios-and-recommendations"></a>Scénarios et recommandations courants 

Voici quelques scénarios courants de gestion des identités hybrides et des accès avec des recommandations concernant l’option (ou les options) d’identité hybride la ou les mieux appropriées dans chaque cas. 

|J’ai besoin de :|PHS et SSO<sup>1</sup>| PTA et SSO<sup>2</sup> | AD FS<sup>3</sup>| 
|-----|-----|-----|-----| 
|Synchroniser de nouveaux comptes d’utilisateurs, de contacts et de groupes créés automatiquement dans mon Active Directory local vers le cloud.|![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| ![Recommandé](./media/whatis-hybrid-identity/ic195031.png) |![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| 
|Configurer mon locataire pour des scénarios hybrides Microsoft 365.|![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| ![Recommandé](./media/whatis-hybrid-identity/ic195031.png) |![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| 
|Permettre à mes utilisateurs de se connecter et d’accéder aux services cloud à l’aide de leur mot de passe local.|![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| ![Recommandé](./media/whatis-hybrid-identity/ic195031.png) |![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| 
|Implémenter l’authentification unique à l’aide des informations d’identification d’entreprise.|![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| ![Recommandé](./media/whatis-hybrid-identity/ic195031.png) |![Recommandé](./media/whatis-hybrid-identity/ic195031.png)|  
|M’assurer qu’aucun hachage du mot de passe n’est stocké dans le cloud.| |![Recommandé](./media/whatis-hybrid-identity/ic195031.png)|![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| 
|Activer des solutions d’authentification multifacteur cloud.|![Recommandé](./media/whatis-hybrid-identity/ic195031.png)|![Recommandé](./media/whatis-hybrid-identity/ic195031.png)|![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| 
|Activer des solutions d’authentification multifacteur locales.| | |![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| 
|Prendre en charge l’authentification par carte à puce pour mes utilisateurs.<sup>4</sup>| | |![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| 
|Afficher les notifications d’expiration du mot de passe dans le portail Office et sur le bureau Windows 10.| | |![Recommandé](./media/whatis-hybrid-identity/ic195031.png)| 

> <sup>1</sup> Synchronisation de hachage de mot de passe avec authentification unique (SSO). 
> 
> <sup>2</sup> Authentification directe et authentification unique.  
> 
> <sup>3</sup> Authentification unique fédérée avec AD FS.  
>  
> <sup>4</sup> AD FS peut être intégré à l’infrastructure de clé publique (PKI) de votre entreprise pour permettre l’authentification à l’aide de certificats. Ces certificats peuvent être des certificats logiciels déployés via des canaux d’approvisionnement approuvés tels que les certificats de gestion des périphériques mobiles (GPM), d’objet de stratégie de groupe (GPO), de carte à puce (y compris les cartes PIV/CAC) ou Hello for Business (approbation de certificat). Pour plus d’informations sur la prise en charge de l’authentification par carte à puce, consultez [ce  blog](/archive/blogs/samueld/adfs-certauth-aad-o365). 
> 

## <a name="license-requirements-for-using-azure-ad-connect"></a>Licences requises pour Azure AD Connect

[!INCLUDE [active-directory-free-license.md](../../../includes/active-directory-free-license.md)]

## <a name="next-steps"></a>Étapes suivantes 

- [Présentation d’Azure AD Connect et Connect Health](whatis-azure-ad-connect.md) 
- [Qu’est-ce que la synchronisation de hachage de mot de passe (PHS) ?](whatis-phs.md) 
- [Qu’est-ce que l’authentification directe (PTA) ?](how-to-connect-pta.md) 
- [Qu’est-ce que la fédération ?](whatis-fed.md) 
- [Qu’est-ce que l’authentification unique ?](how-to-connect-sso.md)
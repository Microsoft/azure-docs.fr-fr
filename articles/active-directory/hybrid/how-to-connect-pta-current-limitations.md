---
title: 'Azure AD Connect : authentification directe - Limitations actuelles | Microsoft Docs'
description: Cet article décrit les limitations actuelles de l’authentification directe Azure Active Directory (Azure AD)
services: active-directory
keywords: Authentification directe Azure AD Connect, installation d’Active Directory, composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/04/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3e9c4489f59f72e4d0b5c7a0b911da188eb0828c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89280194"
---
# <a name="azure-active-directory-pass-through-authentication-current-limitations"></a>Authentification directe Azure Active Directory : Limites actuelles

>[!IMPORTANT]
>L’authentification directe Azure Active Directory (Azure AD) est une fonctionnalité gratuite que vous pouvez utiliser sans avoir besoin d’aucune édition payante d’Azure AD. L’authentification directe est disponible seulement dans l’instance mondiale d’Azure AD, et pas sur le [cloud Microsoft Azure Allemagne](https://www.microsoft.de/cloud-deutschland) ni sur le [cloud Microsoft Azure Government](https://azure.microsoft.com/features/gov/).

## <a name="supported-scenarios"></a>Scénarios pris en charge

Les scénarios suivants sont pris en charge :

- L’utilisateur se connecte à des applications s’appuyant sur un navigateur web.
- L’utilisateur se connecte aux clients Outlook à l’aide des protocoles hérités, comme Exchange ActiveSync, EAS, SMTP, POP et IMAP.
- L’utilisateur se connecte aux applications clientes Office héritées et aux applications Office prenant en charge [l’authentification moderne](https://www.microsoft.com/en-us/microsoft-365/blog/2015/11/19/updated-office-365-modern-authentication-public-preview) : Versions 2013 et 2016 d'Office.
- L’utilisateur se connecte à des applications de protocole héritées, telles que PowerShell version 1.0, etc.
- Jonctions Azure AD pour les appareils Windows 10.
- Mots de passe d'application pour l’authentification multifacteur.

## <a name="unsupported-scenarios"></a>Scénarios non pris en charge

Les scénarios suivants ne sont _pas_ pris en charge :

- Détection des utilisateurs avec des [informations d’identification volées](../identity-protection/overview-identity-protection.md).
- Azure AD Domain Services a besoin que la synchronisation du hachage de mot de passe soit activée sur le locataire. Ainsi, les locataires qui utilisent l’authentification directe _uniquement_ ne fonctionnent pas pour les scénarios qui ont besoin d’Azure AD Domain Services.
- L’authentification directe n’est pas intégrée à [Azure AD Connect Health](./whatis-azure-ad-connect.md).

> [!IMPORTANT]
> Comme solution de contournement _uniquement_ pour les scénarios non pris en charge (à l’exception de l’intégration d’Azure AD Connect Health), activez la synchronisation de hachage du mot de passe dans la page [Fonctionnalités facultatives](how-to-connect-install-custom.md#optional-features) de l’Assistant Azure AD Connect.
> 
> [!NOTE]
> L’activation de la synchronisation de hachage du mot de passe vous donne la possibilité de basculer l’authentification en cas d’interruption de votre infrastructure locale. Ce basculement de l’authentification directe vers la synchronisation de hachage du mot de passe n’est pas automatique. Vous devrez basculer la méthode de connexion manuellement avec Azure AD Connect. Si le serveur exécutant Azure AD Connect tombe en panne, vous devrez demander de l’aide au Support Microsoft pour désactiver l’authentification directe.

## <a name="next-steps"></a>Étapes suivantes
- [Démarrage rapide](how-to-connect-pta-quick-start.md) : soyez opérationnel avec l’authentification directe Azure AD.
- [Migrer à partir d’AD FS vers l’authentification directe](https://aka.ms/ADFSTOPTADPDownload) : guide détaillé de la migration d’AD FS (ou d’autres technologies de fédération) vers l’authentification directe.
- [Verrouillage intelligent](../authentication/howto-password-smart-lockout.md) : apprenez à configurer la fonctionnalité Verrouillage intelligent sur votre locataire pour protéger les comptes d'utilisateur.
- [Présentation technique approfondie](how-to-connect-pta-how-it-works.md) : découvrez comment fonctionne la fonctionnalité d'authentification directe.
- [Forum Aux Questions](how-to-connect-pta-faq.md) : trouvez des réponses aux questions fréquemment posées sur la fonctionnalité Authentification directe.
- [Résoudre les problèmes](tshoot-connect-pass-through-authentication.md) : apprenez à résoudre les problèmes courants liés à la fonctionnalité d’authentification directe.
- [Présentation approfondie de sécurité](how-to-connect-pta-security-deep-dive.md) : obtenez des informations techniques détaillées sur la fonctionnalité d’authentification directe.
- [Authentification unique fluide Azure AD](how-to-connect-sso.md) : explorez en détail cette fonctionnalité complémentaire.
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) : utilisez le Forum Azure Active Directory pour consigner de nouvelles demandes de fonctionnalités.

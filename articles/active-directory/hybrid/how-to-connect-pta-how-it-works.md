---
title: 'Azure AD Connect : Fonctionnement de l’authentification directe | Microsoft Docs'
description: Cet article décrit le fonctionnement de l’authentification directe Azure Active Directory
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
ms.date: 07/19/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: fe92f761ac0b16da7c3cc3c69c1fa4b00f4e7579
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996591"
---
# <a name="azure-active-directory-pass-through-authentication-technical-deep-dive"></a>Authentification directe Azure Active Directory : Présentation technique approfondie
Cet article propose une vue d’ensemble du fonctionnement de l’authentification directe Azure Active directory (Azure AD). Pour en savoir plus sur la sécurité et accéder à d’autres détails techniques, consultez l’article [Présentation approfondie des fonctions de sécurité](how-to-connect-pta-security-deep-dive.md).

## <a name="how-does-azure-active-directory-pass-through-authentication-work"></a>Comment l’authentification directe Azure Active Directory fonctionne-t-elle ?

>[!NOTE]
>Pour que l’authentification directe fonctionne, les utilisateurs doivent être approvisionnés dans Azure AD à partir d’une instance Active Directory locale à l’aide d’Azure AD Connect. La fonctionnalité Authentification directe ne s’applique pas aux utilisateurs « cloud uniquement ».

Quand un utilisateur tente de se connecter à une application sécurisée par Azure AD et que l’authentification directe est activée sur le client, les étapes suivantes se produisent :

1. L'utilisateur tente d’accéder à une application, par exemple, [Outlook Web App](https://outlook.office365.com/owa/).
2. Si l’utilisateur n’est pas déjà connecté, il est redirigé vers la page **Connexion de l'utilisateur** Azure AD.
3. L’utilisateur entre son nom d’utilisateur dans la page de connexion Azure AD, puis sélectionne le bouton **Suivant**.
4. L’utilisateur entre son mot de passe dans la page de connexion Azure AD, puis sélectionne le bouton **Se connecter**.
5. Lors de la réception de la demande de connexion, Azure AD place le nom d’utilisateur et le mot de passe (chiffré à l’aide de la clé publique des agents d’authentification) dans une file d’attente.
6. Un agent d’authentification local récupère le nom d’utilisateur et le mot de passe chiffré à partir de la file d’attente. Notez que l’agent n’interroge pas fréquemment les requêtes à partir de la file d’attente, mais qu’il les récupère via une connexion permanente établie au préalable.
7. L’agent déchiffre le mot de passe à l’aide de sa clé privée.
8. L’agent valide le nom d’utilisateur et le mot de passe par rapport à votre annuaire Active Directory à l’aide des API Windows standard, un mécanisme similaire à celui utilisé par les services de fédération Active Directory (AD FS). Le nom d’utilisateur peut être soit le nom d’utilisateur sur site par défaut, généralement `userPrincipalName`, soit un autre attribut (appelé `Alternate ID`) configuré dans Azure AD Connect.
9. Le contrôleur de domaine Active Directory sur site évalue la demande et retourne la réponse appropriée (succès, échec, mot de passe expiré ou utilisateur verrouillé) à l’agent.
10. L’agent d’authentification retourne à son tour cette réponse à Azure AD.
11. Azure AD évalue la réponse et répond à l’utilisateur comme il convient. Par exemple, Azure AD connecte immédiatement l’utilisateur ou demande une authentification Azure AD Multi-Factor Authentication.
12. Si l’utilisateur parvient à se connecter, il peut accéder à l’application.

Le schéma suivant illustre tous les composants et les étapes impliquées dans ce processus :

![Authentification directe](./media/how-to-connect-pta-how-it-works/pta2.png)

## <a name="next-steps"></a>Étapes suivantes
- [Limitations actuelles](how-to-connect-pta-current-limitations.md) : Découvrez les scénarios pris en charge et ceux qui ne le sont pas.
- [Démarrage rapide](how-to-connect-pta-quick-start.md) : soyez opérationnel sur l’authentification directe Azure AD.
- [Migrer à partir d’AD FS vers l’authentification directe](https://aka.ms/adfstoPTADP) : guide détaillé de la migration d’AD FS (ou d’autres technologies de fédération) vers l’authentification directe.
- [Verrouillage intelligent](../authentication/howto-password-smart-lockout.md) : configurez la fonctionnalité Verrouillage intelligent sur votre locataire pour protéger les comptes d’utilisateur.
- [Forum aux questions](how-to-connect-pta-faq.md) : obtenez des réponses aux questions fréquemment posées.
- [Résoudre les problèmes](tshoot-connect-pass-through-authentication.md) : apprenez à résoudre les problèmes courants liés à la fonctionnalité d’authentification directe.
- [Présentation approfondie de la sécurité](how-to-connect-pta-security-deep-dive.md) : obtenez des informations techniques détaillées sur la fonctionnalité d’authentification directe.
- [Authentification unique fluide Azure AD](how-to-connect-sso.md) : explorez en détail cette fonctionnalité complémentaire.
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) : utilisez le Forum Azure Active Directory pour consigner de nouvelles demandes de fonctionnalités.


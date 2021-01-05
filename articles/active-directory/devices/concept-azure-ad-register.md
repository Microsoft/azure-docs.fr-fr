---
title: En quoi consiste un appareil inscrit sur Azure AD ?
description: Découvrez comment les appareils inscrits sur Azure AD fournissent à vos utilisateurs la prise en charge des scénarios d’appareil mobile et Bring Your Own Device (BYOD).
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: conceptual
ms.date: 06/27/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 72c40b7962090492f8b4b85e555a947130c3628d
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91256436"
---
# <a name="azure-ad-registered-devices"></a>Appareils inscrits sur Azure AD

L’objectif des appareils inscrits sur Azure AD est de fournir à vos utilisateurs la prise en charge des scénarios d’appareil mobile et Bring Your Own Device (BYOD). Dans ces scénarios, un utilisateur peut accéder aux ressources contrôlées Azure Active Directory de votre organisation à l’aide d’un appareil personnel.

| Appareils inscrits sur Azure AD | Description |
| --- | --- |
| **Définition** | Inscrit à Azure AD sans devoir utiliser un compte professionnel pour se connecter à l’appareil |
| **Public principal** | S’applique à tous les utilisateurs présentant les critères suivants : |
|   | BYOD |
|   | Appareils mobiles |
| **Appartenance de l’appareil** | Utilisateur ou organisation |
| **Systèmes d’exploitation** | Windows 10, iOS, Android et Mac OS |
| **Approvisionnement** | Windows 10 : paramètres |
|   | iOS/Android : application Microsoft Authenticator ou de portail d’entreprise |
|   | MacOS : portail d’entreprise |
| **Options de connexion de l’appareil** | Informations d’identification locales de l’utilisateur final |
|   | Mot de passe |
|   | Windows Hello |
|   | PIN |
|   | Biométrie ou modèle des autres appareils |
| **Gestion des appareils** | Gestion des périphériques mobiles (exemple : Microsoft Intune) |
|   | Gestion des applications mobiles |
| **Fonctionnalités clés** | Ressources de l’authentification unique au cloud |
|   | Accès conditionnel pour les utilisateurs inscrits sur Intune |
|   | Accès conditionnel via la stratégie de protection d’application |
|   | Permet la connexion par téléphone à l’application Microsoft Authenticator |

![Appareils inscrits sur Azure AD](./media/concept-azure-ad-register/azure-ad-registered-device.png)

Les appareils inscrits sur Azure AD sont connectés à l’aide d’un compte local comme un compte Microsoft sur un appareil Windows 10. Toutefois, ils disposent également d’un compte Azure AD associé afin d’accéder à des ressources d’organisation. L’accès aux ressources de l’organisation peut être limité par ce compte Azure AD et les stratégies d’accès conditionnel appliquées à l’identité de l’appareil.

Les administrateurs peuvent sécuriser et mieux contrôler ces appareils inscrits sur Azure AD à l’aide d’outils de gestion des périphériques mobiles (MDM) tels que Microsoft Intune. Ces outils permettent d’appliquer les configurations requises par l’organisation, comme l’exigence du chiffrement du stockage, la complexité des mots de passe et la mise à jour des logiciels de sécurité. 

L’inscription sur Azure AD peut être effectuée en accédant à une application professionnelle pour la première fois ou vous pouvez vous inscrire manuellement à l’aide du menu Paramètres de Windows 10. 

## <a name="scenarios"></a>Scénarios

Un utilisateur de votre organisation souhaite accéder aux outils pour la messagerie électronique et signaler des congés tout en bénéficiant de l’inscription depuis chez lui. Votre organisation dispose de ces outils derrière une stratégie d’accès conditionnel qui nécessite un accès depuis un appareil compatible avec Intune. L’utilisateur ajoute son compte professionnel et inscrit son ordinateur personnel sur Azure AD. Les stratégies Intune requises sont ensuite appliquées en accordant l’accès à leurs ressources à l’utilisateur.

Un autre utilisateur souhaite accéder à sa messagerie professionnelle sur son téléphone Android personnel qui a été rooté. Votre entreprise exige un appareil conforme et a créé une stratégie de conformité à Intune pour bloquer les appareils rootés. L’employé ne peut pas accéder aux ressources de l’organisation sur cet appareil.

## <a name="next-steps"></a>Étapes suivantes

- [Gérer les identités d’appareil à l’aide du portail Azure](device-management-azure-portal.md)
- [Gérer les appareils obsolètes dans Azure AD](manage-stale-devices.md)

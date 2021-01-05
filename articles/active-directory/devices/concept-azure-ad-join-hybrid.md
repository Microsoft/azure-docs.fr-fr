---
title: Qu’est-ce qu’un appareil joint Azure AD hybride ?
description: Découvrez comment la gestion d’identité des appareils peut vous aider à gérer les appareils qui accèdent aux ressources de votre environnement.
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
ms.openlocfilehash: 259a1324c412dad40d32a8b8e026d84e6f5aa066
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85554913"
---
# <a name="hybrid-azure-ad-joined-devices"></a>Appareils joints Azure AD hybrides

Pendant plus de dix ans, de nombreuses organisations ont utilisé la jonction de domaine dans leur répertoire Active Directory local pour permettre :

- Aux services informatiques de gérer les appareils professionnels à partir d’un emplacement central.
- Aux utilisateurs de se connecter à leurs appareils à partir de leurs comptes Active Directory professionnels ou scolaires.

En règle générale, les organisations disposant d’empreintes locales s’appuient sur des méthodes de création d’images pour approvisionner les appareils. Ils utilisent souvent **Configuration Manager** ou les **stratégies de groupe** pour les gérer.

Si votre environnement comporte une empreinte locale AD et vous souhaitez également profiter des fonctionnalités proposées par Azure Active Directory, vous pouvez implémenter les appareils joints Azure AD hybrides. Il s’agit d’appareils joints à votre service Active Directory local et inscrits auprès d’Azure Active Directory.

| Jonction Azure AD Hybride | Description |
| --- | --- |
| **Définition** | Joint aux systèmes AD local et Azure AD nécessitant un compte professionnel pour se connecter à l’appareil |
| **Public principal** | Convient aux organisations hybrides avec une infrastructure AD locale existante |
|   | S’applique à tous les utilisateurs d’une organisation |
| **Appartenance de l’appareil** | Organisation |
| **Systèmes d’exploitation** | Windows 10, 8.1 et 7 |
|   | Windows Server 2008/R2, 2012/R2, 2016 et 2019 |
| **Approvisionnement** | Windows 10, Windows Server 2016/2019 |
|   | Jonction de domaine par informatique et jointure automatique via une configuration Azure AD Connect ou ADFS |
|   | Jonction de domaine par Windows Autopilot et jointure automatique via une configuration Azure AD Connect ou ADFS |
|   | Windows 8.1, Windows 7, Windows Server 2012 R2, Windows Server 2012 et Windows Server 2008 R2 : requièrent une identité MSI |
| **Options de connexion de l’appareil** | Comptes professionnels utilisant : |
|   | Mot de passe |
|   | Windows Hello Entreprise pour Win10 |
| **Gestion des appareils** | Stratégie de groupe |
|   | Configuration Manager autonome ou cogestion avec Microsoft Intune |
| **Fonctionnalités clés** | Authentification unique vers les ressources de cloud et locales |
|   | Accès conditionnel via la jonction de domaine ou via Intune si cogéré |
|   | Réinitialisation de mot de passe en libre-service et réinitialisation du code confidentiel Windows Hello sur l’écran de verrouillage |
|   | Enterprise State Roaming entre appareils |

![Appareils joints Azure AD hybrides](./media/concept-azure-ad-join-hybrid/azure-ad-hybrid-joined-device.png)

## <a name="scenarios"></a>Scénarios

Utilisez des appareils joints Azure AD hybrides si :

- Des applications Win32 sont déployées sur ces appareils qui s’appuient sur l’authentification d’ordinateur Active Directory.
- Vous souhaitez continuer à utiliser la stratégie de groupe pour gérer la configuration de l’appareil.
- Vous souhaitez continuer à utiliser des solutions de création d’images existantes pour déployer et configurer des appareils.
- Vous devez prendre en charge les appareils Windows 7 et 8.1 de bas niveau, en plus des appareils Windows 10

## <a name="next-steps"></a>Étapes suivantes

- [Planifier votre implémentation de la jointure d’Azure AD hybride](hybrid-azuread-join-plan.md)
- [Gérer les identités d’appareil à l’aide du portail Azure](device-management-azure-portal.md)
- [Gérer les appareils obsolètes dans Azure AD](manage-stale-devices.md)

---
title: Qu’est-ce qu’un appareil joint Azure AD ?
description: Découvrez les appareils joints Azure AD et comment la gestion d’identité des appareils peut vous aider à gérer les appareils qui accèdent aux ressources de votre environnement.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2b7cc0535549f2dbcdef4ab043ee506527fdbc5f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90601450"
---
# <a name="azure-ad-joined-devices"></a>Appareils joints Azure AD

La jonction Azure AD est destinée aux organisations axées en priorité ou uniquement sur le cloud. Toute organisation peut déployer des appareils joints Azure AD, quels que soient leur taille et leur secteur d’activité. La jonction Azure AD fonctionne même dans un environnement hybride et permet l’accès aux applications et ressources locales et cloud.

| Azure AD Join | Description |
| --- | --- |
| **Définition** | Joint uniquement à Azure AD et nécessitant un compte professionnel pour se connecter à l’appareil |
| **Public principal** | Approprié pour les organisations utilisant uniquement le cloud et pour les organisations hybrides. |
|   | S’applique à tous les utilisateurs d’une organisation |
| **Appartenance de l’appareil** | Organisation |
| **Systèmes d’exploitation** | Tous les appareils Windows 10 à l’exception de Windows 10 Famille |
|   | [Machines virtuelles Windows Server 2019 s’exécutant dans Azure](howto-vm-sign-in-azure-ad-windows.md) (Server Core n’est pas pris en charge) |
| **Approvisionnement** | Libre-service : Windows OOBE ou Paramètres |
|   | Inscription en bloc |
|   | Windows Autopilot |
| **Options de connexion de l’appareil** | Comptes professionnels utilisant : |
|   | Mot de passe |
|   | Windows Hello Entreprise |
|   | Clés de sécurité FIDO2.0 (préversion) |
| **Gestion des appareils** | Gestion des périphériques mobiles (exemple : Microsoft Intune) |
|   | Cogestion avec Microsoft Intune et Microsoft Endpoint Configuration Manager |
| **Fonctionnalités clés** | Authentification unique vers les ressources de cloud et locales |
|   | Accès conditionnel par le biais de l’inscription GPM et de l’évaluation de la conformité GPM |
|   | Réinitialisation de mot de passe en libre-service et réinitialisation du code confidentiel Windows Hello sur l’écran de verrouillage |
|   | Enterprise State Roaming entre appareils |

Les appareils joints Azure AD sont connectés au moyen d’un compte professionnel Azure AD. L’accès aux ressources de l’organisation peut être limité par ce compte Azure AD et les [stratégies d’accès conditionnel](../conditional-access/howto-conditional-access-policy-compliant-device.md) appliquées à l’identité de l’appareil.

Les administrateurs peuvent sécuriser et mieux contrôler les appareils joints Azure AD à l’aide d’outils de gestion des périphériques mobiles (GPM), tels que Microsoft Intune ou dans des scénarios de cogestion à l’aide de Microsoft Endpoint Configuration Manager. Ces outils permettent d’appliquer les configurations requises par l’organisation, comme l’exigence du chiffrement du stockage, la complexité des mots de passe, les installations de logiciels et les mises à jour logicielles. Les administrateurs peuvent mettre des applications d’organisation à la disposition des appareils joints Azure AD des à l’aide de Configuration Manager pour [gérer des applications à partir des Microsoft Store pour les entreprises et l’enseignement](/configmgr/apps/deploy-use/manage-apps-from-the-windows-store-for-business).

La jonction Azure AD peut se faire à l’aide d’options libre-service telles que le mode OOBE (Out of Box Experience), l’inscription en bloc ou [Windows Autopilot](/intune/enrollment-autopilot).

Les appareils joints AD Azure peuvent conserver un accès avec authentification unique vers les ressources locales lorsqu’ils se trouvent sur le réseau de l’organisation. Les appareils joints Azure AD peuvent toujours s’authentifier sur des serveurs locaux (fichiers, impression et autres applications).

## <a name="scenarios"></a>Scénarios

Bien qu’Azure AD Join soit principalement conçu pour les organisations qui ne disposent pas d’une infrastructure Windows Server Active Directory locale, vous pouvez tout à fait l’utiliser dans les scénarios où :

- Vous souhaitez passer à une infrastructure cloud avec Azure AD et un système de gestion des appareils mobiles comme Intune.
- Vous ne pouvez pas utiliser de jonction de domaine locale, par exemple, si vous avez besoin de contrôler des appareils mobiles, tels que des tablettes et des téléphones.
- Vos utilisateurs ont principalement besoin d’accéder à Microsoft 365 ou d’autres applications SaaS intégrées à Azure AD.
- Vous souhaitez gérer un groupe d’utilisateurs dans Azure AD et non dans un répertoire Active Directory. Par exemple, ce scénario peut concerner les travailleurs saisonniers, les prestataires ou les étudiants.
- Vous souhaitez fournir des fonctionnalités de jonction aux travailleurs dans des succursales distantes avec une infrastructure locale limitée.

Vous pouvez configurer des appareils Azure AD joints pour tous les appareils Windows 10, à l’exception de Windows 10 Famille.

Les appareils joints Azure AD ont pour objectif de simplifier :

- Les déploiements Windows des appareils professionnels
- L’accès aux applications et aux ressources d’organisation à partir de n’importe quel appareil Windows
- Gestion cloud des appareils professionnels
- Aux utilisateurs de se connecter à leurs appareils à partir de leurs comptes Azure AD ou Active Directory synchronisés professionnels ou scolaires.

![Appareils joints Azure AD](./media/concept-azure-ad-join/azure-ad-joined-device.png)

Il existe différentes manières de déployer Azure AD Join :

- [Windows Autopilot](/windows/deployment/windows-autopilot/windows-10-autopilot)
- [Déploiement en bloc](/intune/windows-bulk-enroll)
- [Expérience en libre-service](azuread-joined-devices-frx.md)

## <a name="next-steps"></a>Étapes suivantes

- [Planifier votre implémentation de la jointure d’Azure AD](azureadjoin-plan.md)
- [Guide pratique pour gérer le groupe Administrateurs local sur des appareils joints à Azure AD](assign-local-admin.md)
- [Gérer les identités d’appareil à l’aide du portail Azure](device-management-azure-portal.md)
- [Gérer les appareils obsolètes dans Azure AD](manage-stale-devices.md)

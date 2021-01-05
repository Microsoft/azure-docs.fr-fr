---
title: Accès conditionnel - Bloquer l’authentification héritée - Azure Active Directory
description: Créer une stratégie d’accès conditionnel personnalisée pour bloquer les protocoles d’authentification héritée
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 09/22/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2499a89907eb63f01adbc80a6d206911c09c1259
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91265950"
---
# <a name="conditional-access-block-legacy-authentication"></a>Accès conditionnel : Bloquer l’authentification héritée

En raison des risques accrus associés aux protocoles d’authentification hérités, Microsoft recommande aux organisations de bloquer les demandes d’authentification utilisant ces protocoles et d’exiger une authentification moderne.

## <a name="create-a-conditional-access-policy"></a>Créer une stratégie d’accès conditionnel

Les étapes suivantes vous aideront à créer une stratégie d’accès conditionnel pour bloquer les requêtes d’authentification héritées. Ces stratégies sont mises en [mode rapport seul](howto-conditional-access-insights-reporting.md) pour commencer afin que les administrateurs puissent déterminer l’impact qu’elles auront sur les utilisateurs existants. Lorsque les administrateurs sont sûrs que la stratégie s’applique comme prévu, ils peuvent l’**activer** ou échelonner le déploiement en ajoutant certains groupes et en en excluant d’autres.

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**
   1. Sous **Inclure**, sélectionnez **Tous les utilisateurs**.
   1. Sous **Exclure**, sélectionnez **Utilisateurs et groupes**, puis choisissez les comptes qui doivent conserver la possibilité d’utiliser l’authentification héritée. Vous devez exclure au moins un compte pour ne pas être verrouillé. Si vous n’excluez pas de compte, vous ne pourrez pas créer cette stratégie.
   1. Sélectionnez **Terminé**.
1. Sous **Applications ou actions cloud**, sélectionnez **Toutes les applications cloud**.
   1. Sélectionnez **Terminé**.
1. Sous **Conditions** > **Applications clientes**, définissez **Configurer** sur **Oui**.
   1. Cochez uniquement les cases **Clients Exchange ActiveSync** et **Autres clients**.
   1. Sélectionnez **Terminé**.
1. Sous **Contrôles d'accès** > **Accorder**, sélectionnez **Bloquer l’accès**.
   1. Sélectionnez **Sélectionner**.
1. Confirmez vos paramètres et définissez **Activer la stratégie** sur **Rapport seul**.
1. Sélectionnez **Créer** pour créer votre stratégie.

## <a name="next-steps"></a>Étapes suivantes

[Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)

[Déterminer l'impact à l'aide du mode Rapport seul de l'Accès conditionnel](howto-conditional-access-insights-reporting.md)

[Simuler le comportement de connexion à l’aide de l’outil What If pour l’accès conditionnel](troubleshoot-conditional-access-what-if.md)

[Comment configurer une application ou un appareil multifonction pour envoyer un e-mail à l’aide de Microsoft 365](/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-office-3)

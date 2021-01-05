---
title: Accès conditionnel - Bloquer l’accès - Azure Active Directory
description: Créer une stratégie d’accès conditionnel personnalisé à
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb,
ms.collection: M365-identity-device-management
ms.openlocfilehash: 84e0801daa5bf83889be87987d440e377287b5ea
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92366188"
---
# <a name="conditional-access-block-access"></a>Accès conditionnel : Bloquer l’accès

Pour les organisations utilisant une approche de migration vers le cloud conservatrice, la stratégie Bloquer tout est une option possible. 

> [!CAUTION]
> Une mauvaise configuration de stratégie de blocage peut entraîner l’exclusion des organisations du portail Azure.

Ce type de stratégie peut avoir des effets secondaires imprévus. Des opérations appropriées de test et de validation sont essentielles avant l’activation. Les administrateurs doivent utiliser des outils tels que le [mode rapport seul d’accès conditionnel](concept-conditional-access-report-only.md) et [l’outil What If dans l’accès conditionnel](what-if-tool.md) lorsqu’ils apportent des modifications.

## <a name="user-exclusions"></a>Exclusions d’utilisateurs

Les stratégies d’accès conditionnel sont des outils puissants. Nous vous recommandons donc d’exclure les comptes suivants de votre stratégie :

* Comptes **d’accès d’urgence** ou **de secours** pour empêcher le verrouillage du compte sur l’ensemble du locataire. Dans le scénario improbable où tous les administrateurs seraient verrouillés hors de votre locataire, votre compte administratif d’accès d’urgence peut être utilisé pour vous connecter au locataire et prendre les mesures nécessaires pour récupérer l’accès.
   * Pour plus d’informations, consultez l’article [Gérer des comptes d’accès d’urgence dans Azure AD](../roles/security-emergency-access.md).
* Les **comptes de service** et les **principaux de service** , comme le compte de synchronisation Azure AD Connect. Les comptes de service sont des comptes non interactifs qui ne sont pas liés à un utilisateur particulier. Ils sont généralement utilisés par les services principaux autorisant l’accès par programme aux applications, mais ils sont également utilisés pour se connecter aux systèmes à des fins administratives. Les comptes de service comme ceux-ci doivent être exclus, car l’authentification MFA ne peut pas être effectuée par programme. Les appels effectués par les principaux de service ne sont pas bloqués par l’accès conditionnel.
   * Si votre organisation utilise ces comptes dans des scripts ou du code, envisagez de les remplacer par des [identités managées](../managed-identities-azure-resources/overview.md). Pour contourner provisoirement le problème, vous pouvez exclure ces comptes spécifiques de la stratégie de base.

## <a name="create-a-conditional-access-policy"></a>Créer une stratégie d’accès conditionnel

Les étapes suivantes vont permettre de créer des stratégies d’accès conditionnel pour bloquer l’accès à toutes les applications à l’exception d’[Office 365](concept-conditional-access-cloud-apps.md#office-365) si les utilisateurs ne sont pas sur un réseau approuvé. Ces stratégies sont mises en [mode rapport seul](howto-conditional-access-insights-reporting.md) pour commencer afin que les administrateurs puissent déterminer l’impact qu’elles auront sur les utilisateurs existants. Lorsque les administrateurs sont sûrs que les stratégies s’appliquent comme prévu, ils peuvent les **activer** .

La première stratégie bloque l’accès à toutes les applications à l’exception des applications Microsoft 365 si l’emplacement n’est pas approuvé.

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie** .
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations** , sélectionnez **Utilisateurs et groupes** .
   1. Sous **Inclure** , sélectionnez **Tous les utilisateurs** .
   1. Sous **Exclure** , sélectionnez **Utilisateurs et groupes** , puis choisissez les comptes d’accès d’urgence ou de secours de votre organisation. 
   1. Sélectionnez **Terminé** .
1. Sous **Applications ou actions cloud** , sélectionnez les options suivantes :
   1. Sous **Inclure** , sélectionnez **Toutes les applications cloud** .
   1. Sous **Exclure** , sélectionnez **Office 365** , **Sélectionner** , puis **Terminé** .
1. Sous **Conditions**  :
   1. Sous **Conditions** > **Emplacement** .
      1. Définissez **Configurer** sur **Oui**
      1. Sous **Inclure** , sélectionnez **Tous les emplacements** .
      1. Sous **Exclure** , sélectionnez **Tous les emplacements approuvés** .
      1. Sélectionnez **Terminé** .
   1. Sous **Applications clientes (préversion)** , définissez **Configurer** sur **Oui** , puis sélectionnez **Terminé** , puis **Terminé** .
1. Sous **Contrôles d’accès** > **Accorder** , sélectionnez **Bloquer l’accès** , puis **Sélectionner** .
1. Confirmez vos paramètres et définissez **Activer la stratégie** sur **Rapport seul** .
1. Sélectionnez **Créer** pour créer votre stratégie.

Une deuxième stratégie est créée ci-dessous pour exiger l’authentification multifacteur ou un appareil conforme pour les utilisateurs de Microsoft 365.

1. Sélectionnez **Nouvelle stratégie** .
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations** , sélectionnez **Utilisateurs et groupes** .
   1. Sous **Inclure** , sélectionnez **Tous les utilisateurs** .
   1. Sous **Exclure** , sélectionnez **Utilisateurs et groupes** , puis choisissez les comptes d’accès d’urgence ou de secours de votre organisation. 
   1. Sélectionnez **Terminé** .
1. Sous **Applications ou actions cloud** > **Inclure** , sélectionnez **Sélectionner les applications** , choisissez **Office 365** , puis **Sélectionner** et **Terminé** .
1. Sous **Contrôles d’accès** > **Accorder** , sélectionnez **Accorder l’accès** .
   1. Sélectionnez **Exiger l’authentification multifacteur** et **Exiger que l’appareil soit marqué comme conforme** , puis **Sélectionner** .
   1. Vérifiez que l’option **Exiger tous les contrôles sélectionnés** est sélectionnée.
   1. Sélectionnez **Sélectionner** .
1. Confirmez vos paramètres et définissez **Activer la stratégie** sur **Rapport seul** .
1. Sélectionnez **Créer** pour créer votre stratégie.

## <a name="next-steps"></a>Étapes suivantes

[Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)

[Déterminer l'impact à l'aide du mode Rapport seul de l'Accès conditionnel](howto-conditional-access-insights-reporting.md)

[Simuler le comportement de connexion à l’aide de l’outil What If pour l’accès conditionnel](troubleshoot-conditional-access-what-if.md)

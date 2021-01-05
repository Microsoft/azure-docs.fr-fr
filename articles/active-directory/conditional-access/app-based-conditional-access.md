---
title: Applications clientes approuvées avec l’accès conditionnel - Azure Active Directory
description: Découvrez comment exiger des applications clientes approuvées pour l’accès aux applications cloud avec l’accès conditionnel dans Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 03/04/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: spunukol, rosssmi
ms.collection: M365-identity-device-management
ms.openlocfilehash: 122cc6a2be17cb35e77b638a60fc5fa4f035c0d4
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91266137"
---
# <a name="how-to-require-approved-client-apps-for-cloud-app-access-with-conditional-access"></a>Procédure : Exiger des applications clientes approuvées pour l’accès aux applications cloud avec l’accès conditionnel

Les appareils mobiles sont régulièrement utilisés pour effectuer des tâches aussi bien personnelles que professionnelles. Tout en veillant à ce que le personnel puisse être productif, les organisations veulent également empêcher la perte de données depuis des applications potentiellement non sécurisées. Avec l’accès conditionnel, les organisations peuvent limiter l’accès aux seules applications clientes approuvées (avec une authentification moderne).

Cet article présente deux scénarios permettant de configurer des stratégies d’accès conditionnel pour des ressources comme Microsoft 365, Exchange Online et SharePoint Online.

- [Scénario 1 : Les applications Microsoft 365 demandent une application cliente approuvée](#scenario-1-microsoft-365-apps-require-an-approved-client-app)
- [Scénario 2 : Exchange Online et SharePoint Online demandent une application cliente approuvée](#scenario-2-exchange-online-and-sharepoint-online-require-an-approved-client-app)

Dans l’accès conditionnel, on parle de nécessité d’une application cliente approuvée pour cette fonctionnalité. Pour obtenir la liste des applications clientes approuvées, consultez [Spécification d’application cliente approuvée](concept-conditional-access-grant.md#require-approved-client-app).

> [!NOTE]
> Pour exiger des applications clientes approuvées pour les appareils iOS et Android, ces derniers doivent d’abord s’inscrire auprès d'Azure AD.

## <a name="scenario-1-microsoft-365-apps-require-an-approved-client-app"></a>Scénario 1 : Les applications Microsoft 365 demandent une application cliente approuvée

Dans ce scénario, Contoso a décidé que les utilisateurs se servant d’appareils mobiles peuvent accéder à tous les services Microsoft 365, à condition qu’ils utilisent des applications clientes approuvées, comme Outlook Mobile, OneDrive et Microsoft Teams. Tous les utilisateurs de Contoso se connectent déjà à l’aide d’informations d’identification Azure AD et disposent des licences qui leur sont attribuées, notamment Azure AD Premium P1 ou P2 et Microsoft Intune.

Les organisations doivent effectuer les trois étapes suivantes pour exiger l’utilisation d’une application cliente approuvée sur des appareils mobiles.

**Étape 1 : Stratégie pour les clients à authentification moderne basés sur Android et iOS, nécessitant l’utilisation d’une application cliente approuvée lors de l’accès à Exchange Online**

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**
   1. Sous **Inclure**, sélectionnez **Tous les utilisateurs** ou les **Utilisateurs et groupes** particuliers auxquels vous souhaitez appliquer cette stratégie. 
   1. Sélectionnez **Terminé**.
1. Sous **Applications cloud ou actions** > **Inclure**, sélectionnez **Office 365**.
1. Sous **Conditions**, sélectionnez **Plateformes d’appareils**.
   1. Définissez **Configurer** sur **Oui**.
   1. Incluez **Android** et **iOS**.
1. Sous **Conditions**, sélectionnez **Applications clientes (préversion)** .
   1. Définissez **Configurer** sur **Oui**.
   1. Sélectionnez **Applications mobiles et clients de bureau** et **Clients de l’authentification moderne**.
1. Sous **Contrôles d’accès** > **Octroyer**, sélectionnez **Accorder l’accès**, **Demander une application cliente approuvée** et sélectionnez **Sélectionner**.
1. Confirmez vos paramètres et réglez **Activer la stratégie** sur **Activé**.
1. Sélectionnez **Créer** pour créer et activer votre stratégie.

**Étape 2 : Configurer une stratégie d’accès conditionnel Azure AD pour Exchange Online avec ActiveSync (EAS)**

1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**
   1. Sous **Inclure**, sélectionnez **Tous les utilisateurs** ou les **Utilisateurs et groupes** particuliers auxquels vous souhaitez appliquer cette stratégie. 
   1. Sélectionnez **Terminé**.
1. Sous **Applications cloud ou actions** > **Inclure**, sélectionnez **Office 365 Exchange Online**.
1. Sous **Conditions** :
   1. **Applications clientes (préversion)**  :
      1. Définissez **Configurer** sur **Oui**.
      1. Sélectionnez **Applications mobiles et clients de bureau** et **Clients Exchange ActiveSync**.
1. Sous **Contrôles d’accès** > **Octroyer**, sélectionnez **Accorder l’accès**, **Demander une application cliente approuvée** et sélectionnez **Sélectionner**.
1. Confirmez vos paramètres et réglez **Activer la stratégie** sur **Activé**.
1. Sélectionnez **Créer** pour créer et activer votre stratégie.

**Étape 3 : Configurer la stratégie Intune App Protection pour les applications clientes iOS et Android**

Consultez l’article [Guide pratique pour créer et assigner des stratégies de protection d’application](/intune/apps/app-protection-policies) afin de connaître les étapes de création des stratégies de protection d’application pour Android et iOS. 

## <a name="scenario-2-exchange-online-and-sharepoint-online-require-an-approved-client-app"></a>Scénario 2 : Exchange Online et SharePoint Online demandent une application cliente approuvée

Dans ce scénario, Contoso a décidé que les utilisateurs peuvent uniquement accéder aux e-mails et aux données SharePoint sur les appareils mobiles, à condition qu’ils utilisent une application cliente approuvée comme Outlook Mobile. Tous les utilisateurs de Contoso se connectent déjà à l’aide d’informations d’identification Azure AD et disposent des licences qui leur sont attribuées, notamment Azure AD Premium P1 ou P2 et Microsoft Intune.

Les organisations doivent effectuer les trois étapes suivantes pour exiger l’utilisation d’une application cliente approuvée sur des appareils mobiles et des clients ActiveSync.

**Étape 1 : Stratégie pour les clients à authentification moderne basés sur Android et iOS, nécessitant l’utilisation d’une application cliente approuvée lors de l’accès à Exchange Online et SharePoint Online**

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**
   1. Sous **Inclure**, sélectionnez **Tous les utilisateurs** ou les **Utilisateurs et groupes** particuliers auxquels vous souhaitez appliquer cette stratégie. 
   1. Sélectionnez **Terminé**.
1. Sous **Applications cloud ou actions** > **Inclure**, sélectionnez **Office 365 Exchange Online** et **Office 365 SharePoint Online**.
1. Sous **Conditions**, sélectionnez **Plateformes d’appareils**.
   1. Définissez **Configurer** sur **Oui**.
   1. Incluez **Android** et **iOS**.
1. Sous **Conditions**, sélectionnez **Applications clientes (préversion)** .
   1. Définissez **Configurer** sur **Oui**.
   1. Sélectionnez **Applications mobiles et clients de bureau** et **Clients de l’authentification moderne**.
1. Sous **Contrôles d’accès** > **Octroyer**, sélectionnez **Accorder l’accès**, **Demander une application cliente approuvée** et sélectionnez **Sélectionner**.
1. Confirmez vos paramètres et réglez **Activer la stratégie** sur **Activé**.
1. Sélectionnez **Créer** pour créer et activer votre stratégie.

**Étape 2 : Stratégie pour les clients Exchange ActiveSync nécessitant l’utilisation d’une application cliente approuvée**

1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**
   1. Sous **Inclure**, sélectionnez **Tous les utilisateurs** ou les **Utilisateurs et groupes** particuliers auxquels vous souhaitez appliquer cette stratégie. 
   1. Sélectionnez **Terminé**.
1. Sous **Applications cloud ou actions** > **Inclure**, sélectionnez **Office 365 Exchange Online**.
1. Sous **Conditions** :
   1. **Applications clientes (préversion)**  :
      1. Définissez **Configurer** sur **Oui**.
      1. Sélectionnez **Applications mobiles et clients de bureau** et **Clients Exchange ActiveSync**.
1. Sous **Contrôles d’accès** > **Octroyer**, sélectionnez **Accorder l’accès**, **Demander une application cliente approuvée** et sélectionnez **Sélectionner**.
1. Confirmez vos paramètres et réglez **Activer la stratégie** sur **Activé**.
1. Sélectionnez **Créer** pour créer et activer votre stratégie.

**Étape 3 : Configurer la stratégie Intune App Protection pour les applications clientes iOS et Android**

Consultez l’article [Guide pratique pour créer et assigner des stratégies de protection d’application](/intune/apps/app-protection-policies) afin de connaître les étapes de création des stratégies de protection d’application pour Android et iOS. 

## <a name="next-steps"></a>Étapes suivantes

[Qu’est-ce que l’accès conditionnel ?](overview.md)

[Composants de l’accès conditionnel](concept-conditional-access-policies.md)

[Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)

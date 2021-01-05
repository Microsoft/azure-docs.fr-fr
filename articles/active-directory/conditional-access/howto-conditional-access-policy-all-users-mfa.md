---
title: Accès conditionnel – Exiger l’authentification multifacteur pour tous les utilisateurs – Azure Active Directory
description: Créer une stratégie d’accès conditionnel personnalisée pour demander à tous les administrateurs d’effectuer l’authentification multifacteur
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 05/26/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6e6185c4bde71285fc163cae2af46f64ba052195
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994755"
---
# <a name="conditional-access-require-mfa-for-all-users"></a>Accès conditionnel : Exiger MFA pour tous les utilisateurs

Comme Alex Weinert, directeur de la sécurité des identités chez Microsoft, le mentionne dans son billet de blog [Votre mot de pa$$e n’a pas d’importance](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Your-Pa-word-doesn-t-matter/ba-p/731984) :

> Votre mot de passe n’a pas d’importance, mais l’authentification multifacteur, elle, en a ! Nos études révèlent que votre compte court 99,9 % moins de risque d’être compromis si vous utilisez l’authentification multifacteur.

Les instructions de cet article aideront votre organisation à créer une stratégie d’authentification multifacteur équilibrée pour votre environnement.

## <a name="user-exclusions"></a>Exclusions d’utilisateurs

Les stratégies d’accès conditionnel sont des outils puissants. Nous vous recommandons donc d’exclure les comptes suivants de votre stratégie :

* Comptes **d’accès d’urgence** ou **de secours** pour empêcher le verrouillage du compte sur l’ensemble du locataire. Dans le scénario improbable où tous les administrateurs seraient verrouillés hors de votre locataire, votre compte administratif d’accès d’urgence peut être utilisé pour vous connecter au locataire et prendre les mesures nécessaires pour récupérer l’accès.
   * Pour plus d’informations, consultez l’article [Gérer des comptes d’accès d’urgence dans Azure AD](../roles/security-emergency-access.md).
* Les **comptes de service** et les **principaux de service**, comme le compte de synchronisation Azure AD Connect. Les comptes de service sont des comptes non interactifs qui ne sont pas liés à un utilisateur particulier. Ils sont généralement utilisés par les services principaux autorisant l’accès par programme aux applications, mais ils sont également utilisés pour se connecter aux systèmes à des fins administratives. Les comptes de service comme ceux-ci doivent être exclus, car l’authentification MFA ne peut pas être effectuée par programme. Les appels effectués par les principaux de service ne sont pas bloqués par l’accès conditionnel.
   * Si votre organisation utilise ces comptes dans des scripts ou du code, envisagez de les remplacer par des [identités managées](../managed-identities-azure-resources/overview.md). Pour contourner provisoirement le problème, vous pouvez exclure ces comptes spécifiques de la stratégie de base.

## <a name="application-exclusions"></a>Exclusions d’applications

Des organisations peuvent utiliser de nombreuses applications cloud. Ces applications ne nécessitent pas toutes une sécurité égale. Par exemple, les applications de gestion de la paie et des présences peuvent nécessiter une authentification multifacteur, ce qui n’est probablement pas le cas de l’application de gestion de la cafétéria. Les administrateurs peuvent choisir d’exclure des applications spécifiques de leur stratégie.

## <a name="create-a-conditional-access-policy"></a>Créer une stratégie d’accès conditionnel

Les étapes suivantes vous aideront à créer une stratégie d’accès conditionnel pour demander à tous les utilisateurs d’effectuer l’authentification multifacteur.

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**
   1. Sous **Inclure**, sélectionnez **Tous les utilisateurs**
   1. Sous **Exclure**, sélectionnez **Utilisateurs et groupes**, puis choisissez les comptes d’accès d’urgence ou de secours de votre organisation. 
   1. Sélectionnez **Terminé**.
1. Sous **Applications ou actions cloud** > **Inclure**, sélectionnez **Toutes les applications cloud**.
   1. Sous **Exclure**, sélectionnez toutes les applications ne nécessitant pas d’authentification multifacteur.
1. Sous **Conditions** > **Applications clientes (préversion)** > **Sélectionnez les applications clientes auxquelles cette stratégie s’applique**, laissez toutes les valeurs par défaut sélectionnées et sélectionnez **Terminé**.
1. Sous **Contrôles d’accès** > **Accorder**, sélectionnez **Accorder l'accès**, **Requérir l’authentification multifacteur**, et sélectionnez **Sélectionner**.
1. Confirmez vos paramètres et réglez **Activer la stratégie** sur **Activé**.
1. Sélectionnez **Créer** pour créer votre stratégie.

### <a name="named-locations"></a>Emplacements nommés

Les organisations peuvent choisir d’intégrer des emplacements réseau connus (appelés **emplacements nommés**) à leurs stratégies d’accès conditionnel. Ces emplacements nommés peuvent inclure des réseaux IPv4 approuvés, comme ceux qui sont utilisés dans un siège social. Pour plus d’informations sur la configuration des emplacements nommés, consultez l’article [Qu’est-ce que la condition d’emplacement de l’accès conditionnel Azure Active Directory ?](location-condition.md).

Dans l’exemple de stratégie ci-dessus, une organisation peut choisir de ne pas exiger l’authentification multifacteur en cas d’accès à une application cloud à partir de son réseau d’entreprise. Dans un tel cas, elle peut ajouter la configuration suivante à la stratégie :

1. Sous **Attributions**, sélectionnez **Conditions** > **Emplacements**.
   1. Configurez **Oui**.
   1. Incluez **N’importe quel emplacement**.
   1. Excluez **Tous les emplacements approuvés**.
   1. Sélectionnez **Terminé**.
1. Sélectionnez **Terminé**.
1. **Enregistrez** les modifications apportées à votre stratégie.

## <a name="next-steps"></a>Étapes suivantes

[Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)

[Déterminer l'impact à l'aide du mode Rapport seul de l'Accès conditionnel](howto-conditional-access-insights-reporting.md)

[Simuler le comportement de connexion à l’aide de l’outil What If pour l’accès conditionnel](troubleshoot-conditional-access-what-if.md)

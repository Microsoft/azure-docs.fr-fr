---
title: Surveiller l’identité et l’accès dans Azure Security Center | Microsoft Docs
description: Découvrez comment utiliser les fonctionnalités d’identité et d’accès dans Azure Security Center pour surveiller les problèmes liés à l’activité d’accès et à l’identité de vos utilisateurs.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 9f04e730-4cfa-4078-8eec-905a443133da
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/08/2020
ms.author: memildin
ms.openlocfilehash: b36f52fd97a9f7ff294f14ec394ff9a9f607b5b0
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187151"
---
# <a name="monitor-identity-and-access"></a>Surveiller l’identité et l’accès

Le périmètre de sécurité a évolué d’un périmètre de réseau vers un périmètre d’identité. Avec ce développement, la sécurité se résume moins à la protection de votre réseau et plus à la gestion de la sécurité de vos applications, données et utilisateurs.

Le fait de surveiller les activités et les paramètres de configuration liés à l’identité vous permet de prendre des mesures proactives avant qu’un incident ne survienne, ou des mesures réactives pour contrer des tentatives d’attaques.

## <a name="what-identity-and-access-safeguards-does-security-center-provide"></a>Quelles sont les mesures de protection des identités et des accès proposées par le Centre de sécurité ? 

Le Centre de sécurité Azure dispose de deux contrôles de sécurité dédiés pour vous assurer de respecter les exigences en matière d’identité et de sécurité de votre organisation : 

 - **Gérer l’accès et les autorisations** : nous vous encourageons à adopter le [modèle d’accès Privilège minimum](/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models) et à accorder à vos utilisateurs uniquement l’accès dont ils ont besoin pour effectuer leur travail. Ce contrôle comprend également des recommandations pour implémenter le [contrôle d’accès Azure en fonction du rôle (Azure RBAC)](../role-based-access-control/overview.md) pour contrôler l’accès à vos ressources.
 
 - **Activer la MFA** : lorsque [MFA](https://www.microsoft.com/security/business/identity/mfa) est activée, vos comptes sont plus sûrs et les utilisateurs peuvent toujours s’authentifier auprès de presque n’importe quelle application avec l’authentification unique.

### <a name="example-recommendations-for-identity-and-access"></a>Exemples de recommandations relatives à l’identité et à l’accès

Voici des exemples de recommandations que vous pouvez voir dans ces deux contrôles sur la page **Recommandations** du Centre de sécurité :

- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement
- Trois propriétaires au plus doivent être désignés pour votre abonnement
- Les comptes externes disposant d’autorisations de lecture doivent être supprimés de votre abonnement
- Vous devez supprimer les comptes dépréciés de votre abonnement. (Un compte déprécié est un compte qui n’est plus nécessaire et qui n’est pas autorisé à se connecter par Azure Active Directory.)

> [!TIP]
> Pour plus d’informations sur ces recommandations et sur les autres que vous pouvez voir dans ces contrôles, consultez [Recommandations relatives à l’identité et à l’accès](recommendations-reference.md#recs-identity).

### <a name="limitations"></a>Limites

Il existe certaines limitations à la protection de l’identité et de l’accès du Centre de sécurité :

- Les recommandations d’identité ne sont pas disponibles pour les abonnements avec plus de 600 comptes. Dans ce cas, ces recommandations sont répertoriées sous « évaluations non disponibles ».
- Les recommandations d’identité ne sont pas disponibles pour les agents d’administration du partenaire fournisseur de solutions Cloud (CSP).
- Les recommandations d’identité n’identifient pas les comptes qui sont gérés à l’aide d’un système Privileged Identity Management (PIM). Si vous utilisez un outil PIM, il se peut que des résultats inexacts soient visibles dans le contrôle **Gérer l’accès et les autorisations**.

## <a name="multi-factor-authentication-mfa-and-azure-active-directory"></a>Authentification multifacteur (MFA) et Azure Active Directory 

L’activation de MFA nécessite des [autorisations de locataire Azure Active Directory (AD)](../active-directory/roles/permissions-reference.md).

- Si vous disposez d’une édition Premium d’AD, activez MFA à l’aide de l’[accès conditionnel](../active-directory/conditional-access/concept-conditional-access-policy-common.md).
- Si vous utilisez l’édition gratuite d’AD, activez les **paramètres de sécurité par défaut** comme décrit dans la [documentation d’Azure Active Directory](../active-directory/fundamentals/concept-fundamentals-security-defaults.md).

## <a name="identify-accounts-without-multi-factor-authentication-mfa-enabled"></a>Identifier les comptes sans authentification multifacteur (MFA) activée

Pour connaître les comptes pour lesquels la MFA n’est pas activée, utilisez la requête Azure Resource Graph suivante. La requête retourne toutes les ressources défectueuses (comptes) de la recommandation « la MFA doit être activée sur les comptes avec des autorisations de propriétaire sur votre abonnement ». 

1. Ouvrez l’**Explorateur Azure Resource Graph**.

    :::image type="content" source="./media/security-center-identity-access/opening-resource-graph-explorer.png" alt-text="Lancement de la page de recommandations de l’Explorateur Azure Resource Graph**" :::

1. Entrez la requête suivante et sélectionnez **Exécuter la requête**.

    ```kusto
    securityresources
     | where type == "microsoft.security/assessments"
     | where properties.displayName == "MFA should be enabled on accounts with owner permissions on your subscription"
     | where properties.status.code == "Unhealthy"
    ```

1. La propriété `additionalData` révèle la liste des ID d’objet de compte pour les comptes auxquels la MFA n’est pas appliquée. 

    > [!NOTE]
    > Les comptes sont représentés par un ID d’objet plutôt qu’un nom de compte pour garantir la confidentialité des titulaires de compte.

> [!TIP]
> Vous pouvez également utiliser la méthode [Assessments - Get](/rest/api/securitycenter/assessments/get) de l’API REST du Centre de sécurité.


## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les recommandations qui s’appliquent à d’autres types de ressources Azure, consultez l’article suivant :

- [Protection de votre réseau dans Azure Security Center](security-center-network-recommendations.md)
---
title: Stratégies d’accès conditionnel courantes - Azure Active Directory
description: Stratégies d’accès conditionnel couramment utilisées pour les organisations
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 07/02/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: bdaaed72b85bc3c0196a8897be311efa5fb714e2
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92366341"
---
# <a name="common-conditional-access-policies"></a>Stratégies d’accès conditionnel courantes

Les [paramètres de sécurité par défaut](../fundamentals/concept-fundamentals-security-defaults.md) sont intéressants, mais beaucoup d’organisations ont besoin d’une plus grande flexibilité. Par exemple, de nombreuses organisations ont besoin de pouvoir exclure des comptes spécifiques, tels que leurs comptes d’administration d’accès d’urgence ou d’administration de secours avec des stratégies d’accès conditionnel nécessitant l’authentification multifacteur. Pour ces organisations, les stratégies courantes référencées dans cet article peuvent être utilisées.

![Stratégies d’accès conditionnel sur le portail Azure](./media/concept-conditional-access-policy-common/conditional-access-policies-azure-ad-listing.png)

## <a name="emergency-access-accounts"></a>Comptes d’accès d’urgence

Pour plus d’informations sur les comptes d’accès d’urgence et la raison pour laquelle ils sont importants, consultez les articles suivants : 

* [Gérer des comptes d’accès d’urgence dans Azure AD](../roles/security-emergency-access.md)
* [Créer une stratégie de gestion du contrôle d'accès résiliente avec Azure Active Directory](../authentication/concept-resilient-controls.md)

## <a name="typical-policies-deployed-by-organizations"></a>Stratégies standard déployées par les organisations

* [Bloquer l’authentification héritée](howto-conditional-access-policy-block-legacy.md)\*
* [Exiger MFA pour les administrateurs](howto-conditional-access-policy-admin-mfa.md)\*
* [Exiger MFA pour la gestion Azure](howto-conditional-access-policy-azure-management.md)\*
* [Exiger MFA pour tous les utilisateurs](howto-conditional-access-policy-all-users-mfa.md)\*

\* Ces quatre stratégies, quand elles sont configurées conjointement, simulent les fonctionnalités activées par les [paramètres de sécurité par défaut](../fundamentals/concept-fundamentals-security-defaults.md).

## <a name="additional-policies"></a>Stratégies supplémentaires

* [Accès conditionnel basé sur les risques de connexion (requiert Azure AD Premium P2)](howto-conditional-access-policy-risk.md)
* [Accès conditionnel basé sur les risques des utilisateurs (requiert Azure AD Premium P2)](howto-conditional-access-policy-risk-user.md)
* [Exiger un emplacement approuvé pour l’inscription MFA](howto-conditional-access-policy-registration.md)
* [Bloquer l’accès par emplacement](howto-conditional-access-policy-location.md)
* [Exiger un appareil conforme](howto-conditional-access-policy-compliant-device.md)
* [Bloquer l’accès à l’exception d’applications spécifiques](howto-conditional-access-policy-block-access.md)

## <a name="next-steps"></a>Étapes suivantes

- [Simuler le comportement de connexion à l’aide de l’outil What If pour l’accès conditionnel.](troubleshoot-conditional-access-what-if.md)

- [Utilisez le mode Rapport uniquement pour l’accès conditionnel afin de déterminer l’impact des nouvelles décisions de stratégie.](concept-conditional-access-report-only.md)

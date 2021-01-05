---
title: Connecter les données Azure AD Identity Protection à Azure Sentinel
description: Découvrez comment diffuser des journaux et des alertes en continu depuis Azure AD Identity Protection vers Azure Sentinel pour afficher des tableaux de bord, créer des alertes personnalisées et améliorer les examens.
author: yelevin
manager: rkarlin
ms.assetid: 91c870e5-2669-437f-9896-ee6c7fe1d51d
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 06/24/2020
ms.author: yelevin
ms.openlocfilehash: 69ab76bf213653ea10db8dfd181b615a7e0f47b5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85564472"
---
# <a name="connect-data-from-azure-active-directory-azure-ad-identity-protection"></a>Connecter les données à partir de Azure Active Directory (Azure AD) Identity Protection

Vous pouvez diffuser des journaux en continu depuis [Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md) vers Azure Sentinel pour transmettre en continu des alertes à Azure Sentinel et afficher des tableaux de bord, créer des alertes personnalisées et améliorer les examens. Azure Active Directory Identity Protection fournit une vue consolidée des utilisateurs à risque, des détections de risques et des vulnérabilités, et permet d’éliminer ces risques immédiatement et de définir des stratégies de remédiation automatique pour les événements futurs. Le service repose sur l’expérience de Microsoft en matière de protection des identités des consommateurs et bénéficie d’une précision spectaculaire grâce à plus de 13 milliards d’ouvertures de session par jour. 

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’un [abonnement Azure AD Premium P2](https://azure.microsoft.com/pricing/details/active-directory/).
- Vous devez posséder des autorisations d’administrateur général ou d’administrateur de la sécurité.


## <a name="connect-to-azure-ad-identity-protection"></a>Connexion à Azure AD Identity Protection

Si vous avez un abonnement Azure AD Premium P2, Azure AD Identity Protection est inclus. Si des [stratégies sont activées](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md) et génèrent des alertes, les données d’alerte peuvent facilement être diffusées en continu dans Azure Sentinel.

1. Dans Azure Sentinel, sélectionnez **Connecteurs de données** puis cliquez sur la vignette **Azure AD Identity Protection**.

1. Cliquez sur **Connecter** pour démarrer la diffusion en continu d’événements Azure AD Identity Protection dans Azure Sentinel.

1. Pour utiliser le schéma pertinent dans Log Analytics pour les alertes d’Azure AD Identity Protection, recherchez **SecurityAlert**.

Si vous souhaitez tester le connecteur, vous pouvez [simuler des détections](../active-directory/identity-protection/howto-identity-protection-simulate-risk.md) pour générer des exemples d’alertes qui seront diffusées en continu dans Azure Sentinel.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter Azure AD Identity Protection à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).

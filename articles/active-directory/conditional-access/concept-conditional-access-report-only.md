---
title: Qu’est-ce que le mode rapport seul de l’accès conditionnel ? - Azure Active Directory
description: Comment le mode rapport seul peut-il faciliter le déploiement de la stratégie d’accès conditionnel ?
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 05/01/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: dawoo
ms.collection: M365-identity-device-management
ms.openlocfilehash: df25278d37e518e99bc315a315e017b98c7f4e58
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89049381"
---
# <a name="what-is-conditional-access-report-only-mode"></a>Qu’est-ce que le mode rapport seul de l’accès conditionnel ?

L’accès conditionnel est largement utilisé par nos clients pour maintenir leur niveau de sécurité en appliquant des contrôles d’accès adaptés aux circonstances. Toutefois, l’une des problématiques posées par le déploiement d’une stratégie d’accès conditionnel dans une organisation est de déterminer l’impact sur les utilisateurs finaux. Il peut être difficile d’anticiper le nombre et le nom des utilisateurs affectés par des initiatives de déploiement courantes, comme bloquer l’authentification héritée, imposer l’authentification multifacteur pour une population d’utilisateurs ou implémenter des stratégies de connexion à risque. 

Le mode rapport seul est un nouvel état de la stratégie d’accès conditionnel qui permet aux administrateurs d’évaluer l’impact des stratégies d’accès conditionnel avant de les activer dans leur environnement.  Avec la mise en production du mode rapport seul :

- Il est possible d’activer les stratégies d’accès conditionnel en mode rapport seul.
- Lors de la connexion, les stratégies en mode rapport seul sont évaluées, mais non appliquées.
- Les résultats sont journalisés dans les onglets **Accès conditionnel** et **Rapport seul** des détails du journal de connexion.
- Les clients disposant d’un abonnement Azure Monitor peuvent surveiller l’impact de leurs stratégies d’accès conditionnel dans le classeur Insights sur l’accès conditionnel.

> [!WARNING]
> Les stratégies en mode rapport seul qui exigent des appareils conformes peuvent inviter les utilisateurs Mac, iOS et Android à sélectionner un certificat d’appareil lors de l’évaluation de la stratégie, même si la conformité des appareils n’est pas appliquée. Ces invites sont susceptibles de se répéter jusqu’à ce que l’appareil ait été rendu conforme. Pour éviter aux utilisateurs finaux de recevoir des invites lors de la connexion, excluez les plateformes d’appareils Mac, iOS et Android des stratégies rapport seul qui effectuent des vérifications de conformité des appareils. Notez que le mode Rapport uniquement n’est pas applicable pour les stratégies d’autorité de certification avec l’étendue « Actions de l’utilisateur ».

![Onglet Rapport seul dans le journal de connexion Azure AD](./media/concept-conditional-access-report-only/report-only-detail-in-sign-in-log.png)

## <a name="policy-results"></a>Résultats de la stratégie

Lorsqu’une stratégie en mode rapport seul est évaluée pour une connexion donnée, quatre nouvelles valeurs de résultat sont possibles :

| Résultats | Description |
| --- | --- |
| Rapport seul : Succès | Toutes les conditions de stratégie configurées, tous les contrôles d’autorisation non interactifs requis et tous les contrôles de session sont remplis. Par exemple, une exigence d’authentification multifacteur est satisfaite par une revendication MFA déjà présente dans le jeton ; une stratégie d’appareil conforme est satisfaite en effectuant une vérification de l’appareil sur un appareil conforme. |
| Rapport seul : Échec | Toutes les conditions de stratégie configurées sont remplies, mais pas tous les contrôles d’autorisation non interactifs requis et contrôles de session. Par exemple, une stratégie s’applique à un utilisateur pour lequel un contrôle de bloc est configuré ou un appareil ne respecte pas une stratégie d’appareil conforme. |
| Rapport seul : Action requise de la part de l’utilisateur | Toutes les conditions de stratégie configurées sont remplies, mais une action de l’utilisateur est nécessaire pour satisfaire les contrôles d’autorisation requis ou les contrôles de session. En mode rapport seul, l’utilisateur n’est pas invité à satisfaire les contrôles requis (par exemple, les demandes d’authentification multifacteur ou les conditions d’utilisation).   |
| Rapport seul : Non applicable | Toutes les conditions de stratégie configurées ne sont pas satisfaites. Par exemple, l’utilisateur est exclu de la stratégie ou la stratégie s’applique uniquement à certains emplacements nommés approuvés. |

## <a name="conditional-access-insights-workbook"></a>Classeur Insights sur l’accès conditionnel

Les administrateurs ont la possibilité de créer plusieurs stratégies en mode rapport seul. Il est donc nécessaire de comprendre l’impact individuel de chacune et l’impact combiné de plusieurs stratégies évaluées ensemble. Le nouveau classeur Insights sur l’accès conditionnel permet aux administrateurs de visualiser les requêtes d’accès conditionnel et de surveiller l’impact d’une stratégie pour un intervalle de temps, un ensemble d’applications et des utilisateurs donnés. 
 
## <a name="next-steps"></a>Étapes suivantes

[Configurer le mode rapport seul sur une stratégie d’accès conditionnel](howto-conditional-access-insights-reporting.md)

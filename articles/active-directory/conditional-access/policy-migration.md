---
title: Migrer des stratégies d’accès conditionnel - Azure Active Directory
description: Découvrez ce que vous devez savoir pour migrer les stratégies classiques dans le portail Azure.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 12/04/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: d1811d5b9ae4d3e34b48e1cdc156438f2bad98d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90601652"
---
# <a name="conditional-access-classic-policy-migration"></a>Migration des stratégies classiques d’accès conditionnel

L’accès conditionnel est l’outil utilisé par Azure Active Directory pour réunir des signaux, prendre des décisions et appliquer des stratégies d’organisation. L’accès conditionnel est au cœur du nouveau plan de contrôle basé sur les identités. Si l’objectif est toujours le même, la publication du nouveau portail Microsoft Azure a introduit des améliorations significatives pour le fonctionnement de l’accès conditionnel.

Envisagez d’effectuer la migration des stratégies que vous n’avez pas créées dans le portail Azure, car :

- Vous pouvez maintenant résoudre des scénarios que vous ne pouviez pas gérer auparavant.
- Vous pouvez réduire le nombre de stratégies que vous devez gérer en les consolidant.
- Vous pouvez gérer toutes vos stratégies d’accès conditionnel depuis un emplacement central.
- Le portail Azure Classic va être mis hors service.

Cet article explique ce que vous devez savoir pour migrer vos stratégies d’accès conditionnel existantes vers la nouvelle infrastructure.

## <a name="classic-policies"></a>Stratégies classiques

Dans le [Portail Azure](https://portal.azure.com), les stratégies d’accès conditionnel se trouvent sous **Azure Active Directory** > **Sécurité** > **Accès conditionnel**. Votre organisation peut également avoir d’anciennes politiques d’accès conditionnel qui n’ont pas été créées à l’aide de cette page. Ces stratégies sont appelées *stratégies classiques*. Les stratégies classiques sont des stratégies d’accès conditionnel que vous avez créées aux emplacements suivants :

- Le Portail Azure Classic
- Le Portail classique Intune
- Le portail Intune App Protection

Sur la page **Accès conditionnel**, vous pouvez accéder à vos stratégies classiques en cliquant sur [**Stratégies classiques**](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ConditionalAccessBlade/ClassicPolicies) dans la section **Gérer**. 

![Accès conditionnel dans Azure AD affichant la vue des stratégies classiques](./media/policy-migration/71.png)

L’affichage **Stratégies classiques** vous permet d’effectuer les opérations suivantes :

- Filtrer vos stratégies classiques.
- Désactiver des stratégies classiques.
- Passer en revue les paramètres d’une stratégie classique et désactivez-la.

   ![Détails de la stratégie classique, y compris la configuration de stratégie existante](./media/policy-migration/74.png)

> [!WARNING]
> Une fois désactivée, une stratégie classique ne peut pas être réactivée.

La vue détaillée d’une stratégie classique vous permet de documenter les paramètres, de modifier les groupes inclus ou exclus et de désactiver la stratégie.

![Détails de la stratégie – Groupes à inclure ou à exclure](./media/policy-migration/75.png)

En modifiant les groupes sélectionnés ou en excluant des groupes particuliers, vous pouvez tester l’effet d’une stratégie classique désactivée pour quelques utilisateurs de test avant de désactiver la stratégie pour tous les utilisateurs et groupes inclus.
 
## <a name="migration-considerations"></a>Considérations relatives à la migration

Dans cet article, les stratégies d’accès conditionnel Azure AD sont également appelées *nouvelles stratégies*.
Vos stratégies classiques continuent de fonctionner aux côtés de vos nouvelles stratégies jusqu’à ce que vous les désactiviez ou supprimiez. 

Les aspects suivants sont importants dans le contexte d’une consolidation de stratégies :

- Alors que les stratégies classiques sont liées à une application cloud spécifique, vous pouvez sélectionner autant d’applications cloud que nécessaire dans une nouvelle stratégie.
- Tous les contrôles d’une stratégie classique et d’une nouvelle stratégie pour une application cloud doivent être satisfaits (*AND*). 
- Dans une nouvelle stratégie, vous pouvez :
   - Combiner plusieurs conditions si votre scénario l’exige. 
   - Sélectionner plusieurs exigences d’octroi en tant que contrôle d’accès et les combiner avec un opérateur logique *OR* (nécessitent un des contrôles sélectionnés) ou avec un opérateur logique *AND* (nécessitent tous les contrôles sélectionnés).

### <a name="exchange-online"></a>Exchange Online

Si vous souhaitez migrer des stratégies classiques pour **Exchange Online** qui incluent **Exchange Active Sync** comme condition d’applications clientes, vous ne pourrez peut-être pas les consolider en une nouvelle stratégie. 

C’est, par exemple, le cas si vous souhaitez prendre en charge tous les types d’applications clientes. Dans une nouvelle stratégie qui a **Exchange Active Sync** comme condition d’applications clientes, vous ne pouvez pas sélectionner d’autres applications clientes.

![Accès conditionnel sélectionnant les applications clientes](./media/policy-migration/64.png)

En outre, une consolidation en une nouvelle stratégie n’est pas possible si vos stratégies classiques contiennent plusieurs conditions. Une nouvelle stratégie pour laquelle **Exchange Active Sync** est configuré comme condition d’applications clientes ne prend pas en charge d’autres conditions :   

![Exchange ActiveSync ne prend pas en charge les conditions sélectionnées](./media/policy-migration/08.png)

Si vous avez une nouvelle stratégie pour laquelle **Exchange Active Sync** est configuré comme condition d’applications clientes, vous devez vous assurer que toutes les autres conditions ne sont pas configurées. 

![Conditions d’accès conditionnel](./media/policy-migration/16.png)
 
Les stratégies classiques basées sur une application pour Exchange Online qui incluent **Exchange Active Sync** comme condition d’applications clientes autorisent les plateformes d’appareils **prises en charge** et **non prises en charge**. Bien que vous ne puissiez pas configurer de plateformes d’appareils individuelles dans une nouvelle stratégie connexe, vous pouvez limiter la prise en charge aux [plateformes d’appareils prises en charge](concept-conditional-access-conditions.md#device-platforms). 

![Sélection d’Exchange ActiveSync pour l’accès conditionnel](./media/policy-migration/65.png)

Vous pouvez consolider plusieurs stratégies classiques qui incluent **Exchange Active Sync** comme condition d’applications clientes si elles ont :

- Uniquement **Exchange Active Sync** comme condition 
- Plusieurs exigences d’octroi d’accès configurées

Un scénario courant est la consolidation des éléments suivants :

- Une stratégie classique basée sur un appareil à partir du portail Azure Classic 
- Une stratégie classique basée sur une application dans le portail Intune App Protection 
 
Dans ce cas, vous pouvez consolider vos stratégies classiques en une nouvelle stratégie pour laquelle les deux exigences sont sélectionnées.

![Contrôle de l’octroi de l’accès conditionnel](./media/policy-migration/62.png)

### <a name="device-platforms"></a>Plateformes d’appareils

Les stratégies classiques dotées de contrôles basés sur une application sont préconfigurées avec iOS et Android en tant que condition de plateforme d’appareil. 

Dans une nouvelle stratégie, vous devez sélectionner les [plateformes d’appareils](concept-conditional-access-conditions.md#device-platforms) à prendre en charge individuellement.

![Sélection des plateformes d’appareils à accès conditionnel](./media/policy-migration/41.png)

## <a name="next-steps"></a>Étapes suivantes

- [Utilisez le mode Rapport uniquement pour l’accès conditionnel afin de déterminer l’impact des nouvelles décisions de stratégie.](concept-conditional-access-report-only.md)
- Pour savoir comment configurer une stratégie d’accès conditionnel, consultez [Stratégies communes d’accès conditionnel](concept-conditional-access-policy-common.md).
- Si vous êtes prêt à configurer des stratégies d’accès conditionnel pour votre environnement, consultez l’article [Guide pratique : planifier votre déploiement d’accès conditionnel dans Azure Active Directory](plan-conditional-access.md). 

---
title: 'Conception d’identités hybrides : exigences de protection des données Azure | Microsoft Docs'
description: Lorsque vous planifiez votre solution d’identité hybride, vous devez identifier les exigences de protection des données pour votre entreprise et les options disponibles pour remplir au mieux ces exigences.
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: 40dc4baa-fe82-4ab6-a3e4-f36fa9dcd0df
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 056d0caf2bfc6e99d5d659a2561bc41844feb79a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "64918793"
---
# <a name="plan-for-enhancing-data-security-through-a-strong-identity-solution"></a>Planifier l'amélioration de la sécurité des données grâce à une solution d'identité solide
La première étape de la protection des données est d’identifier qui peut accéder à ces données. En outre, vous devez disposer d’une solution d’identité qui peut s’intégrer à votre système pour fournir des fonctionnalités d’authentification et d’autorisation. Authentification et autorisation sont souvent confondues et leurs rôles mal compris. En réalité elles sont différentes, comme l'illustre la figure ci-dessous :

![cycle de vie des appareils mobiles](./media/plan-hybrid-identity-design-considerations/mobile-devicemgt-lifecycle.png)

**Étapes du cycle de vie de gestion des appareils mobiles**

Lorsque vous planifiez votre solution d'identité hybride, vous devez comprendre les exigences de protection des données pour votre entreprise et les options disponibles pour remplir au mieux ces exigences.

> [!NOTE]
> Une fois que vous avez terminé la planification de la sécurité des données, consultez [Déterminer les exigences d'authentification multifacteur](plan-hybrid-identity-design-considerations-multifactor-auth-requirements.md) pour vous assurer que vos sélections relatives aux exigences d'authentification multifacteur n'ont pas été affectées par les décisions que vous avez prises dans cette section.
> 
> 

## <a name="determine-data-protection-requirements"></a>Déterminer les exigences de protection des données
À l'ère de la mobilité, la plupart des entreprises ont un objectif commun : permettre à leurs utilisateurs d'être productifs sur leurs appareils mobiles, en local ou à distance depuis n'importe où, pour accroître la productivité. Les entreprises ayant cette exigence seront également préoccupées par le nombre de menaces à réduire pour préserver la sécurité des données de l'entreprise et la confidentialité de l'utilisateur. Chaque société peut avoir des exigences différentes à cet égard ; des règles de conformité différentes en fonction du secteur d'activité de l'entreprise entraîneront des décisions de conception différentes. 

Toutefois, certains aspects de la sécurité devraient être examinés et validés, indépendamment du secteur.

## <a name="data-protection-paths"></a>Chemins de protection des données
![chemins de protection des données](./media/plan-hybrid-identity-design-considerations/data-protection-paths.png)

**Chemins de protection des données**

Dans le diagramme ci-dessus, le composant d'identité sera le premier vérifié avant l'accès aux données. Toutefois, ces données peuvent être dans différents états pendant qu'elles sont accessibles. Chaque numéro sur ce diagramme représente un chemin dans lequel les données peuvent être situées à un moment donné dans le temps. Ces numéros sont expliqués ci-dessous :

1. Protection des données au niveau de l'appareil.
2. Protection des données pendant leur transit.
3. Protection des données au repos en local.
4. Protection des données au repos dans le cloud.

Il est nécessaire que la solution d’identité hybride soit capable de tirer parti des ressources de gestion des identités en local et dans le cloud pour identifier l’utilisateur avant qu’elle n’accorde l’accès aux données. Lors de la planification de votre solution d'identité hybride, veillez à ce que les questions suivantes soient traitées en fonction des besoins de votre organisation :

## <a name="data-protection-at-rest"></a>Protection des données au repos
Indépendamment de l'endroit où les données sont au repos (appareil, cloud ou en local), il est important d'effectuer une évaluation pour comprendre les besoins de l'organisation à cet égard. Pour cette zone, assurez-vous que les questions suivantes sont posées :

* Votre entreprise a-t-elle besoin de protéger des données au repos ?
  * Si oui, la solution d'identité hybride peut-elle être intégrée à votre infrastructure locale actuelle ?
  * Si oui, la solution d’identité hybride peut-elle être intégrée à vos charges de travail situées dans le cloud ?
* La gestion d'identité de cloud est-elle en mesure de protéger les informations d'identification de l'utilisateur et d'autres données stockées dans le cloud ?

## <a name="data-protection-in-transit"></a>Protection des données en transit
Les données en transit entre l'appareil et le centre de données ou entre l'appareil et le cloud doivent être protégées. Toutefois, être en transit n'implique pas nécessairement un processus de communication avec un composant en dehors de votre service cloud ; le déplacement se fait en interne, comme entre deux réseaux virtuels. Pour cette zone, assurez-vous que les questions suivantes sont posées :

* Votre entreprise a-t-elle besoin de protéger des données en transit ?
  * Si oui, la solution d’identité hybride peut-elle être intégrée à vos contrôles de sécurité, comme SSL/TLS ?
* La gestion des identités de cloud maintient-elle l'authentification du trafic vers et dans le magasin d'annuaires (dans et entre les centres de données) ?

## <a name="compliance"></a>Conformité
Les règlements, les lois et les exigences de conformité aux réglementations varieront selon le secteur d'activité de votre entreprise. Les entreprises de secteurs très réglementés doivent résoudre les problèmes de gestion des identités liés à des problèmes de conformité. Les réglementations telles que Sarbanes-Oxley (SOX), Health Insurance Portability Accountability Act (HIPAA), la loi Gramm-Leach-Bliley (GLBA) et le paiement carte norme (PCI DSS) sont strictes en ce qui concerne l'identité et l'accès. La solution d'identité hybride adoptée par votre entreprise doit avoir les fonctionnalités essentielles répondant aux exigences d'une ou plusieurs de ces règlements. Pour cette zone, assurez-vous que les questions suivantes sont posées :

* La solution d'identité hybride est-elle conforme aux exigences réglementaires pour votre entreprise ?
* La solution d’identité hybride intègre-t-elle 
* des fonctionnalités qui permettront à votre entreprise d'être conforme aux exigences réglementaires ? 

> [!NOTE]
> Veillez à noter chaque réponse et à comprendre le raisonnement derrière la réponse. [Définir la stratégie de protection des données](plan-hybrid-identity-design-considerations-data-protection-strategy.md) examinera les options disponibles et les avantages/inconvénients de chaque option.  En répondant à chacune de ces questions, vous sélectionnerez l’option correspondant le mieux à vos besoins métier.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
 [Déterminer les exigences de gestion de contenu](plan-hybrid-identity-design-considerations-contentmgt-requirements.md)

## <a name="see-also"></a>Voir aussi
[Présentation des considérations relatives à la conception](plan-hybrid-identity-design-considerations-overview.md)


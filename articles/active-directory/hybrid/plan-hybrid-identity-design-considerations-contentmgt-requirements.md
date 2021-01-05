---
title: 'Conception d’identités hybrides : exigences en matière de gestion de contenu Azure | Microsoft Docs'
description: Fournit un aperçu de la façon de déterminer les besoins de gestion de contenu de votre entreprise. Généralement, lorsqu'un utilisateur a son propre appareil, il a peut-être également plusieurs informations d'identification qu’il utilisera en alternance en fonction de l'application utilisée. Il est important de faire la différence entre le contenu créé avec des informations d'identification personnelles et celui créé avec des informations d'identification d’entreprise. Votre solution d'identité doit être en mesure d'interagir avec les services du cloud pour fournir une expérience transparente à l'utilisateur final, tout en garantissant sa confidentialité et en augmentant la protection contre les fuites de données.
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: dd1ef776-db4d-4ab8-9761-2adaa5a4f004
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 57990fda7475b95bd6582fa5a495ac8d24aa55d5
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94408530"
---
# <a name="determine-content-management-requirements-for-your-hybrid-identity-solution"></a>Déterminer les exigences de gestion de contenu pour votre solution d'identité hybride
La compréhension des exigences de gestion de contenu pour votre entreprise peut affecter directement votre décision quant à la solution identité hybride à utiliser. Avec la prolifération des appareils multiples et la possibilité pour les utilisateurs d'apporter leurs appareils ([BYOD](/mem/intune/fundamentals/byod-technology-decisions)), la société doit protéger ses données, mais elle doit également préserver la confidentialité de l'utilisateur. Généralement, lorsqu'un utilisateur a son propre appareil, il a peut-être également plusieurs informations d'identification qu’il utilisera en alternance en fonction de l'application utilisée. Il est important de faire la différence entre le contenu créé avec des informations d'identification personnelles et celui créé avec des informations d'identification d’entreprise. Votre solution d'identité doit être en mesure d'interagir avec les services du cloud pour fournir une expérience transparente à l'utilisateur final, tout en garantissant sa confidentialité et en augmentant la protection contre les fuites de données. 

Votre solution d'identité sera exploitée par différents contrôles techniques afin de fournir une gestion de contenu comme indiqué dans la figure ci-dessous :

![contrôles de sécurité](./media/plan-hybrid-identity-design-considerations/securitycontrols.png)

**Contrôles de sécurité qui exploiteront votre système de gestion d'identité**

En général, les exigences de gestion de contenu exploitent votre système de gestion d'identité dans les domaines suivants :

* Confidentialité : identification de l'utilisateur qui possède une ressource et application des contrôles appropriés pour maintenir l'intégrité.
* Classification des données : identifier l'utilisateur ou le groupe et le niveau d'accès à un objet en fonction de sa classification. 
* Protection contre les fuites de données : les contrôles de sécurité destinés à protéger les données devront interagir avec le système d'identité pour valider l'identité de l'utilisateur. Ceci est également important pour objectif de piste d'audit.

> [!NOTE]
> Lire [Classification des données pour l'adoption du cloud](https://download.microsoft.com/download/0/A/3/0A3BE969-85C5-4DD2-83B6-366AA71D1FE3/Data-Classification-for-Cloud-Readiness.pdf) pour plus d'informations sur les meilleures pratiques et directives de classification des données.
> 
> 

Lors de la planification de votre solution d'identité hybride, veillez à ce que les questions suivantes soient traitées en fonction des besoins de votre organisation :

* Votre entreprise a-t-elle mis en place des contrôles de sécurité pour la confidentialité des données ?
  * Si oui, les contrôles de sécurité pourront-ils être intégrés dans la solution d'identité hybride que vous vous apprêtez à adopter ?
* Votre entreprise utilise-t-elle la classification des données ?
  * Si oui, la solution actuelle pourra-t-elle être intégrée dans la solution d'identité hybride que vous vous apprêtez à adopter ?
* Votre entreprise a-t-elle actuellement une solution contre la fuite de données ? 
  * Si oui, la solution actuelle pourra-t-elle être intégrée dans la solution d'identité hybride que vous vous apprêtez à adopter ?
* Votre entreprise a-t-elle besoin d'auditer l'accès aux ressources ?
  * Si oui, de quel type de ressources ?
  * Si oui, quel est le niveau d'information nécessaire ?
  * Si oui, où le journal d'audit doit-il résider ? En local ou dans le cloud ?
* Votre société a-t-elle besoin de chiffrer des messages électroniques qui contiennent des données sensibles (SSN, numéros de carte de crédit, etc.) ?
* Votre société a-t-elle besoin de crypter tous les documents/contenus partagés avec des partenaires commerciaux externes ?
* Votre société a-t-elle besoin d'appliquer des stratégies d'entreprise sur certains types de courriers électroniques (ne pas répondre à tous, ne pas transférer) ?

> [!NOTE]
> Veillez à noter chaque réponse et à comprendre le raisonnement derrière la réponse. [Définir la stratégie de protection des données](plan-hybrid-identity-design-considerations-data-protection-strategy.md) examinera les options disponibles et les avantages/inconvénients de chaque option.  En répondant à chacune de ces questions, vous sélectionnerez l’option correspondant le mieux à vos besoins métier.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
[Déterminer les exigences de contrôle d’accès](plan-hybrid-identity-design-considerations-accesscontrol-requirements.md)

## <a name="see-also"></a>Voir aussi
[Présentation des considérations relatives à la conception](plan-hybrid-identity-design-considerations-overview.md)
---
title: Identity Protection et utilisateurs B2B - Azure Active Directory
description: 'Utilisation d’Identity Protection avec les utilisateurs B2B : fonctionnement et limitations connues'
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: conceptual
ms.date: 10/18/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: c9ec9e110c3a476c9096ae3e216c9780da0e0f49
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88949194"
---
# <a name="identity-protection-and-b2b-users"></a>Utilisateurs Identity Protection et B2B

Grâce à la collaboration entre Azure AD et B2B, les organisations peuvent appliquer des stratégies basées sur les risques pour les utilisateurs B2B à l’aide d’Identity Protection. Ces stratégies doivent être configurées de deux manières :

- Les administrateurs peuvent configurer les stratégies intégrées d’Identity Protection basées sur les risques, qui s’appliquent à toutes les applications, y compris les utilisateurs invités.
- Les administrateurs peuvent configurer leurs stratégies d’accès conditionnel, en utilisant le risque de connexion comme condition, qui comprend les utilisateurs invités.

## <a name="how-is-risk-evaluated-for-b2b-collaboration-users"></a>Comment le risque est-il évalué pour les utilisateurs de la collaboration B2B ?

Le risque de l’utilisateur pour les utilisateurs de la collaboration B2B est évalué dans leur répertoire de départ. Le risque de connexion en temps réel pour ces utilisateurs est évalué dans le répertoire de ressources lorsqu’ils essaient d’accéder à la ressource.

## <a name="limitations-of-identity-protection-for-b2b-collaboration-users"></a>Limitations d’Identity Protection pour les utilisateurs de la collaboration B2B

Il existe des limitations dans l’implémentation d’Identity Protection pour les utilisateurs de la collaboration B2B dans un répertoire de ressources, car leur identité existe dans leur répertoire de départ. Les limitations principales sont les suivantes :

- Si un utilisateur invité déclenche la stratégie de risque de l’utilisateur Identity Protection pour forcer la réinitialisation du mot de passe, **il sera bloqué**. Ce bloc est dû à l’impossibilité de réinitialiser les mots de passe dans le répertoire des ressources.
- **Les utilisateurs invités n’apparaissent pas dans le rapport utilisateurs à risque**. Cette perte de visibilité est due à l’évaluation des risques se produisant dans le répertoire de départ de l’utilisateur B2B.
- Les administrateurs **ne peuvent pas ignorer ou corriger un utilisateur à risque de la collaboration B2B** dans leur répertoire de ressources. Cette perte de fonctionnalité est due au fait que les administrateurs du répertoire de ressources n’ont pas accès au répertoire de départ de l’utilisateur B2B.

### <a name="why-cant-i-remediate-risky-b2b-collaboration-users-in-my-directory"></a>Pourquoi ne puis-je pas corriger les utilisateurs à risque de la collaboration B2B dans mon annuaire ?

L’évaluation et la correction des risques pour les utilisateurs B2B se produisent dans leur répertoire de départ. En raison de ce fait, les utilisateurs invités n’apparaissent pas dans le rapport utilisateurs à risque dans le répertoire des ressources, et les administrateurs dans le répertoire de ressources ne peuvent pas forcer la réinitialisation de mot de passe sécurisé pour ces utilisateurs.

### <a name="what-do-i-do-if-a-b2b-collaboration-user-was-blocked-due-to-a-risk-based-policy-in-my-organization"></a>Que dois-je faire si un utilisateur de la collaboration B2B a été bloqué en raison d’une stratégie basée sur les risques dans mon organisation ?

Si un utilisateur B2B risqué dans votre répertoire est bloqué par votre stratégie basée sur des risques, l’utilisateur doit corriger ce risque dans son répertoire de départ. Les utilisateurs peuvent corriger leurs risques en effectuant une réinitialisation de mot de passe sécurisé dans leur répertoire de départ. Si la réinitialisation de mot de passe en libre-service n’est pas activée dans son répertoire de départ, l’utilisateur doit contacter le personnel informatique de son organisation pour qu’un administrateur fasse disparaître manuellement le risque ou réinitialiser son mot de passe.

### <a name="how-do-i-prevent-b2b-collaboration-users-from-being-impacted-by-risk-based-policies"></a>Comment puis-je éviter que les utilisateurs de la collaboration B2B ne soient affectés par les politiques basées sur les risques ?

L’exclusion des utilisateurs B2B des stratégies d’accès conditionnel en fonction des risques de votre organisation empêchera les utilisateurs B2B d’être affectés ou bloqués par leur évaluation des risques. Pour exclure ces utilisateurs B2B, créez un groupe dans Azure AD qui contient tous les utilisateurs invités de votre organisation. Ensuite, ajoutez ce groupe en tant qu’exclusion pour vos stratégies de risque utilisateur et de risque de connexion Identity Protection, ainsi que toutes les stratégies d’accès conditionnel qui utilisent le risque de connexion en tant que condition.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants sur Azure AD B2B Collaboration :

- [Qu'est-ce que la collaboration B2B d'Azure AD ?](../external-identities/what-is-b2b.md)
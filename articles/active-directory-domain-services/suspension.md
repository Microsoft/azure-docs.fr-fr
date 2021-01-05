---
title: Domaines suspendus dans Azure AD Domain Services | Microsoft Docs
description: Découvrez les différents états d’intégrité d’un domaine managé Azure AD DS et comment restaurer un domaine suspendu.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 95e1d8da-60c7-4fc1-987d-f48fde56a8cb
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: b0b48e7ad494386052e6d94c32d7215b3f4f0202
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618822"
---
# <a name="understand-the-health-states-and-resolve-suspended-domains-in-azure-active-directory-domain-services"></a>Comprendre les états d’intégrité et résoudre les domaines suspendus dans Azure Active Directory Domain Services

Quand Azure AD Domain Services (Azure AD DS) ne parvient pas à mettre en service un domaine managé pendant une longue période, ça met ce domaine en état de suspension. Si un domaine managé reste dans un état suspendu, il est automatiquement supprimé. Pour assurer l’intégrité de votre domaine managé Azure AD DS et éviter la suspension, résolvez toutes les alertes aussi rapidement que possible.

Cet article explique pourquoi les domaines managés sont suspendus et comment récupérer un domaine suspendu.

## <a name="overview-of-managed-domain-states"></a>Vue d’ensemble des états des domaines managés

Tout au long du cycle de vie d’un domaine managé, différents états indiquent son intégrité. Si le domaine managé signale un problème, résolvez rapidement la cause sous-jacente pour empêcher l’état de continuer de se dégrader.

![Progression des états affichés par un domaine managé vers la suspension](media/active-directory-domain-services-suspension/suspension-timeline.PNG)

Un domaine managé peut avoir l’un des états suivants :

* [Exécution](#running-state)
* [Doit être surveillé](#needs-attention-state)
* [Suspendu](#suspended-state)
* [Supprimé](#deleted-state)

## <a name="running-state"></a>État En cours d’exécution

Un domaine managé qui est configuré correctement et sans problème se trouve dans l’état *En cours d’exécution*. Il s’agit de l’état souhaité pour un domaine managé.

### <a name="what-to-expect"></a>À quoi s’attendre

* La plateforme Azure peut superviser régulièrement l’intégrité du domaine managé.
* Les contrôleurs du domaine managé sont corrigés et mis à jour régulièrement.
* Les modifications effectuées dans Azure Active Directory sont régulièrement synchronisées avec le domaine managé.
* Des sauvegardes du domaine managé sont effectuées régulièrement.

## <a name="needs-attention-state"></a>État Doit être surveillé

Un domaine managé présentant un ou plusieurs problèmes qui doivent être résolus se trouve dans l’état *Doit être surveillé*. La page d’intégrité du domaine managé liste les alertes et indique où un problème se pose.

Certaines alertes sont temporaires et résolues automatiquement par la plateforme Azure. Pour d’autres alertes, vous pouvez résoudre le problème en suivant les étapes de résolution indiquées. En cas d’alerte critique, [formulez une demande de support Azure][azure-support] pour bénéficier d’une aide supplémentaire.

Un groupe de sécurité réseau restrictif représente un exemple d’alerte. Dans cette configuration, la plateforme Azure peut ne pas être en mesure de mettre à jour et de superviser le domaine managé. Une alerte est générée et l’état passe à *Doit être surveillé*.

Pour plus d’informations, consultez [Résolution des alertes liées aux domaines managés][resolve-alerts].

### <a name="what-to-expect"></a>À quoi s’attendre

Quand un domaine managé présente l’état *Doit être surveillé*, la plateforme Azure peut ne pas être en mesure de superviser, corriger, mettre à jour ou sauvegarder des données régulièrement. Dans certains cas, comme avec une configuration réseau non valide, les contrôleurs du domaine managé peuvent être inaccessibles.

* L’état du domaine managé n’est pas sain et la supervision continue de l’intégrité peut s’arrêter jusqu’à la résolution de l’alerte.
* Les contrôleurs du domaine managé ne peuvent pas être corrigés ou mis à jour.
* Les modifications effectuées dans Azure Active Directory peuvent ne pas être synchronisées avec le domaine managé.
* Des sauvegardes du domaine managé ne sont peut-être pas effectuées.
* Si vous résolvez des alertes non critiques qui ont un impact sur le domaine managé, l’intégrité doit revenir à l’état *En cours d’exécution*.
* Les alertes critiques sont déclenchées en cas de problèmes de configuration où la plateforme Azure est incapable d’accéder aux contrôleurs de domaine. Si ces alertes critiques ne sont pas résolues dans les 15 jours, le domaine managé passe à l’état *Suspendu*.

## <a name="suspended-state"></a>État Suspendu

Un domaine managé passe à l’état **Suspendu** pour l’une des raisons suivantes :

* Une ou plusieurs alertes critiques n’ont pas été résolues au bout de 15 jours.
    * Les alertes critiques peuvent être dues à une configuration incorrecte qui bloque l’accès aux ressources dont a besoin Azure AD DS. Par exemple, l’alerte [AADDS104 : erreur réseau][alert-nsg] n’a pas été résolue pendant plus de 15 jours dans le domaine managé.
* Il y a un problème de facturation avec l’abonnement Azure ou l’abonnement Azure a expiré.

Les domaines managés sont suspendus quand la plateforme Azure ne parvient pas à gérer, superviser, corriger ou sauvegarder le domaine. Un domaine managé reste à l’état *Suspendu* pendant 15 jours. Pour conserver l’accès au domaine managé, résolvez immédiatement les alertes critiques.

### <a name="what-to-expect"></a>À quoi s’attendre

Le comportement suivant se produit quand un domaine managé se trouve dans l’état *Suspendu* :

* Les contrôleurs du domaine managé sont déprovisionnés et ne sont pas accessibles dans le réseau virtuel.
* L’accès LDAP sécurisé au domaine managé via Internet, s’il est activé, cesse de fonctionner.
* Des échecs se produisent lors de l’authentification auprès du domaine managé, de la connexion aux machines virtuelles jointes à un domaine ou des connexions via LDAP/LDAPS.
* Les sauvegardes du domaine managé ne sont plus effectuées.
* La synchronisation avec Azure AD cesse.

### <a name="how-do-you-know-if-your-managed-domain-is-suspended"></a>Comment savoir si votre domaine managé a été suspendu ?

Vous voyez une [alerte][resolve-alerts] dans la page d’intégrité Azure AD DS du portail Azure qui indique que le domaine est suspendu. L’état du domaine indique également *Suspendu*.

### <a name="restore-a-suspended-domain"></a>Restaurer un domaine suspendu

Pour restaurer l’intégrité d’un domaine managé qui est dans l’état *Suspendu*, procédez comme suit :

1. Dans le portail Azure, recherchez et sélectionnez **Services de domaine**.
1. Choisissez votre domaine managé dans la liste, par exemple *aaddscontoso.com*, puis sélectionnez **Intégrité**.
1. Sélectionnez l’alerte, par exemple *AADDS503* ou *AADDS504*, selon la cause de la suspension.
1. Choisissez le lien de résolution fourni dans l’alerte et suivez les étapes indiquées.

Un domaine managé peut uniquement être restauré à la date de la dernière sauvegarde. La date de la dernière sauvegarde est affichée dans la page **Intégrité** du domaine managé. Toutes les modifications apportées après la dernière sauvegarde ne pourront pas être restaurées. Les sauvegardes de domaine managé sont stockées pendant 30 jours. Les sauvegardes datant de plus de 30 jours sont supprimées.

Une fois que vous avez résolu les alertes quand le domaine managé est dans l’état *Suspendu*, [formulez une demande de support Azure][azure-support] pour revenir à un état sain. Si la sauvegarde date de moins de 30 jours, le support Azure peut restaurer le domaine managé.

## <a name="deleted-state"></a>État Supprimé

Si un domaine managé reste à l’état *Suspendu* pendant 15 jours, il est supprimé. Ce processus est irrécupérable.

### <a name="what-to-expect"></a>À quoi s’attendre

Quand un domaine managé passe à l’état *Supprimé*, le comportement suivant se produit :

* Toutes les ressources et sauvegardes du domaine managé sont supprimées.
* Vous ne pouvez pas restaurer le domaine managé. Vous devez créer un domaine managé de remplacement pour réutiliser Azure AD DS.
* Lorsque le domaine managé est supprimé, vous n’êtes plus facturé pour celui-ci.

## <a name="next-steps"></a>Étapes suivantes

Pour assurer l’intégrité de votre domaine managé et réduire le risque de suspension, découvrez comment [résoudre les alertes liés à votre domaine managé][resolve-alerts].

<!-- INTERNAL LINKS -->
[alert-nsg]: alert-nsg.md
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md
[resolve-alerts]: troubleshoot-alerts.md

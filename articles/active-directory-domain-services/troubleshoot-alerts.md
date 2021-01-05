---
title: Alertes et résolutions courantes dans Azure AD Domain Services | Microsoft Docs
description: Découvrez comment résoudre les alertes courantes générées dans le cadre de l’état d’intégrité d’Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 54319292-6aa0-4a08-846b-e3c53ecca483
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 80011b2ae2a70ce4c540ac31897cc8be4a6580c3
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618618"
---
# <a name="known-issues-common-alerts-and-resolutions-in-azure-active-directory-domain-services"></a>Problèmes connus : Alertes courantes et résolutions dans Azure Active Directory Domain Services

En tant qu’élément central de l’identité et de l’authentification des applications, Azure Active Directory Domain Services (Azure AD DS) rencontre parfois des problèmes. Si vous rencontrez des problèmes, les alertes courantes et les étapes de dépannage associées sont là pour vous aider à rétablir le service. De même, vous pouvez à tout moment [formuler une demande de support Azure][azure-support] pour bénéficier d’une aide supplémentaire.

Cet article fournit des solutions aux problèmes fréquemment rencontrés dans Azure AD DS.

## <a name="aadds100-missing-directory"></a>AADDS100 : Annuaire manquant

### <a name="alert-message"></a>Message d’alerte

*L’annuaire Azure AD associé à votre domaine géré a peut-être été supprimé. Le domaine géré n’est plus dans une configuration prise en charge. Microsoft ne peut pas surveiller, gérer, mettre à jour et synchroniser votre domaine géré.*

### <a name="resolution"></a>Résolution

Cette erreur se produit généralement quand un abonnement Azure est déplacé vers un nouvel annuaire Azure AD et que l’ancien annuaire Azure AD associé à Azure AD DS est supprimé.

Cette erreur est irrécupérable. Pour résoudre l’alerte, [supprimez votre domaine managé](delete-aadds.md), puis recréez-le dans votre nouveau répertoire. Si vous rencontrez des problèmes lors de la suppression du domaine managé, [créez une demande de support Azure][azure-support] pour obtenir une assistance supplémentaire.

## <a name="aadds101-azure-ad-b2c-is-running-in-this-directory"></a>AADDS101 : Azure AD B2C est en cours d’exécution dans cet annuaire

### <a name="alert-message"></a>Message d’alerte

*Les services de domaine Azure AD ne peuvent pas être activés dans un annuaire Azure AD B2C.*

### <a name="resolution"></a>Résolution

Azure AD DS se synchronise automatiquement avec un annuaire Azure AD. Si l’annuaire Azure AD est configuré pour B2C, Azure AD DS ne peut pas être déployé et synchronisé.

Pour utiliser Azure AD DS, vous devez recréer votre domaine managé dans un annuaire non Azure AD B2C en effectuant les étapes suivantes :

1. [Supprimez le domaine managé](delete-aadds.md) de votre annuaire Azure AD existant.
1. Créez un annuaire Azure AD qui n’est pas un annuaire Azure AD B2C.
1. [Créez un domaine managé de remplacement](tutorial-create-instance.md).

L’intégrité du domaine managé se met automatiquement à jour dans les deux heures, et l’alerte est supprimée.

## <a name="aadds103-address-is-in-a-public-ip-range"></a>AADDS103 : L’adresse est dans une plage d’adresses IP publiques

### <a name="alert-message"></a>Message d’alerte

*La plage d’adresses IP pour le réseau virtuel dans lequel vous avez activé les services de domaine Azure AD est dans une plage d’adresses IP publiques. Les services de domaine Azure AD doivent être activés dans un réseau virtuel avec une plage d’adresses IP privées. Cette configuration affecte la capacité de Microsoft à surveiller, gérer, mettre à jour et synchroniser votre domaine géré.*

### <a name="resolution"></a>Résolution

Avant de commencer, assurez-vous de bien comprendre les [espaces d’adressage privé IP v4](https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces).

À l’intérieur d’un réseau virtuel, les machines virtuelles peuvent effectuer des requêtes sur les ressources Azure qui se trouvent dans la même plage d’adresses IP que celles configurées pour le sous-réseau. Si vous configurez une plage d’adresses IP publiques pour un sous-réseau, les demandes routées au sein d’un réseau virtuel peuvent ne pas atteindre les ressources web prévues. Cette configuration risque d’entraîner des erreurs imprévisibles avec Azure AD DS.

> [!NOTE]
> Si vous êtes propriétaire de la plage d’adresses IP dans l’Internet configuré sur votre réseau virtuel, vous pouvez ignorer cette alerte. Il est toutefois impossible pour Azure AD Domain Services de respecter le contrat [SLA](https://azure.microsoft.com/support/legal/sla/active-directory-ds/v1_0/) avec cette configuration, car elle peut entraîner des erreurs imprévisibles.

Pour résoudre cette alerte, supprimez votre domaine managé existant et recréez-le sur un réseau virtuel avec une plage d’adresses IP privées. Ce processus entraîne des perturbations, car le domaine managé est indisponible, et toutes les ressources personnalisées que vous avez créées, comme les unités d’organisation ou les comptes de service, sont perdues.

1. [Supprimez le domaine managé](delete-aadds.md) de votre annuaire.
1. Pour mettre à jour la plage d’adresses IP du réseau virtuel, recherchez et sélectionnez *Réseau virtuel* dans le portail Azure. Sélectionnez le réseau virtuel pour Azure AD DS qui a une plage d’adresses IP publiques définie alors qu’il ne le devrait pas.
1. Sous **Paramètres**, sélectionnez *Espace d’adressage*.
1. Mettez à jour la plage d’adresses en choisissant la plage d’adresses existante et en la modifiant, ou en ajoutant une plage d’adresses supplémentaire. Vérifiez que la nouvelle plage d’adresses est dans une plage d’adresses IP privées. Quand vous êtes prêt, **Enregistrez** les modifications.
1. Sélectionnez **Sous-réseaux** dans la navigation de gauche.
1. Choisissez le sous-réseau que vous souhaitez modifier ou créez-en un supplémentaire.
1. Mettez à jour ou spécifiez une plage d’adresses IP privées, puis **Enregistrez** vos modifications.
1. [Créez un domaine managé de remplacement](tutorial-create-instance.md). Veillez à sélectionner le sous-réseau de réseau virtuel mis à jour avec une plage d’adresses IP privées.

L’intégrité du domaine managé se met automatiquement à jour dans les deux heures, et l’alerte est supprimée.

## <a name="aadds106-your-azure-subscription-is-not-found"></a>AADDS106 : Votre abonnement Azure est introuvable

### <a name="alert-message"></a>Message d’alerte

*Votre abonnement Azure associé à votre domaine managé a été supprimé.  Azure AD Domain Services nécessite un abonnement actif pour continuer à fonctionner correctement.*

### <a name="resolution"></a>Résolution

Azure AD DS nécessite un abonnement actif et ne peut pas être déplacé vers un autre abonnement. Si l’abonnement Azure auquel le domaine managé a été associé est supprimé, vous devez recréer un abonnement Azure et un domaine managé.

1. [Créez un abonnement Azure](../cost-management-billing/manage/create-subscription.md).
1. [Supprimez le domaine managé](delete-aadds.md) de votre annuaire Azure AD existant.
1. [Créez un domaine managé de remplacement](tutorial-create-instance.md).

## <a name="aadds107-your-azure-subscription-is-disabled"></a>AADDS107 : Votre abonnement Azure est désactivé

### <a name="alert-message"></a>Message d’alerte

*Votre abonnement Azure associé à votre domaine managé n’est pas actif.  Azure AD Domain Services nécessite un abonnement actif pour continuer à fonctionner correctement.*

### <a name="resolution"></a>Résolution

Azure AD DS nécessite un abonnement actif. Si l’abonnement Azure auquel le domaine managé était associé n’est pas actif, vous devez le renouveler pour réactiver l’abonnement.

1. [Renouvelez votre abonnement Azure](../cost-management-billing/manage/subscription-disabled.md).
2. Une fois l’abonnement renouvelé, une notification Azure AD DS vous permet de réactiver le domaine managé.

Quand le domaine managé est réactivé, l’intégrité du domaine managé se met automatiquement à jour dans les deux heures et l’alerte est supprimée.

## <a name="aadds108-subscription-moved-directories"></a>AADDS108 : Annuaires déplacés d’abonnements

### <a name="alert-message"></a>Message d’alerte

*L’abonnement utilisé par Azure AD Domain Services a été déplacé dans un autre répertoire. Azure AD Domain Services nécessite un abonnement actif dans le même répertoire pour continuer à fonctionner correctement.*

### <a name="resolution"></a>Résolution

Azure AD DS nécessite un abonnement actif et ne peut pas être déplacé vers un autre abonnement. Si l’abonnement Azure auquel le domaine managé était associé est déplacé, replacez l’abonnement dans l’annuaire précédent, ou [supprimez votre domaine managé](delete-aadds.md) de l’annuaire existant et [créez un domaine managé de remplacement dans l’abonnement choisi](tutorial-create-instance.md).

## <a name="aadds109-resources-for-your-managed-domain-cannot-be-found"></a>AADDS109 : Les ressources pour votre domaine managé sont introuvables

### <a name="alert-message"></a>Message d’alerte

*Une ressource utilisée pour votre domaine managé a été supprimée. Cette ressource est requise pour qu’Azure AD Domain Services fonctionne correctement.*

### <a name="resolution"></a>Résolution

Azure AD DS crée des ressources supplémentaires pour fonctionner correctement, telles que des adresses IP publiques, des interfaces de réseau virtuel et un équilibreur de charge. Si l’une de ces ressources est supprimée, le domaine managé est dans un état non pris en charge et il ne peut pas être géré. Pour plus d’informations sur ces ressources, consultez [Ressources réseau utilisées par Azure AD DS](network-considerations.md#network-resources-used-by-azure-ad-ds).

Cette alerte est générée quand l’une de ces ressources requises est supprimée. Si la ressource a été supprimée il y a moins de quatre heures, il est possible que la plateforme Azure puisse la recréer automatiquement. Les étapes suivantes décrivent comment vérifier l’état d’intégrité et l’horodatage pour la suppression des ressources :

1. Dans le portail Azure, recherchez et sélectionnez **Services de domaine**. Choisissez votre domaine managé, par exemple *aaddscontoso.com*.
1. Dans le volet de navigation gauche, sélectionnez **Intégrité**.
1. Dans la page d’intégrité, sélectionnez l’alerte avec l’ID *AADDS109*.
1. L’alerte a un horodatage correspondant au moment où elle a été initialement détectée. Si cet horodatage est inférieur à quatre heures, la plateforme Azure peut être en mesure de recréer automatiquement la ressource et de résoudre l’alerte par elle-même.

    Si l’alerte a plus de 4 heures, le domaine managé est dans un état irrécupérable. [Supprimez le domaine managé](delete-aadds.md), puis [créez un domaine managé de remplacement](tutorial-create-instance.md).

## <a name="aadds110-the-subnet-associated-with-your-managed-domain-is-full"></a>AADDS110 : Le sous-réseau associé à votre domaine managé est plein

### <a name="alert-message"></a>Message d’alerte

*Le sous-réseau sélectionné pour le déploiement d’Azure AD Domain Services est plein et n’a pas l’espace nécessaire pour le contrôleur de domaine supplémentaire qui doit être créé.*

### <a name="resolution"></a>Résolution

Le sous-réseau de réseau virtuel pour Azure AD DS nécessite suffisamment d’adresses IP pour les ressources créées automatiquement. Cet espace d’adressage IP inclut la nécessité de créer des ressources de remplacement en cas d’événement de maintenance. Pour réduire le risque de manquer d’adresses IP disponibles, ne déployez pas d’autres ressources (par exemple, vos propres machines virtuelles) sur le même sous-réseau de réseau virtuel que le domaine managé.

Cette erreur est irrécupérable. Pour résoudre l’alerte, [supprimez votre domaine managé existant](delete-aadds.md), puis recréez-le. Si vous rencontrez des problèmes lors de la suppression du domaine managé, [créez une demande de support Azure][azure-support] pour obtenir une assistance supplémentaire.

## <a name="aadds111-service-principal-unauthorized"></a>AADDS111 : Principal du service non autorisé

### <a name="alert-message"></a>Message d’alerte

*Un principal de service qu’Azure AD Domain Services utilise pour gérer votre domaine n’est pas autorisé à gérer les ressources de l’abonnement Azure. Le principal du service doit obtenir les autorisations nécessaires pour gérer votre domaine managé.*

### <a name="resolution"></a>Résolution

Certains principaux de service générés automatiquement sont utilisés afin de gérer et de créer des ressources pour un domaine managé. Si les autorisations d’accès pour l’un de ces principaux de service sont modifiées, le domaine ne pourra pas gérer correctement les ressources. Les étapes suivantes montrent comment comprendre et accorder des autorisations d’accès à un principal de service :

1. Apprenez-en davantage sur le [contrôle d’accès en fonction du rôle et sur la façon d’accorder un accès aux applications dans le portail Azure](../role-based-access-control/role-assignments-portal.md).
2. Passez en revue l’accès dont dispose le principal de service avec l’ID *abba844e-bc0e-44b0-947a-dc74e5d09022* et accordez l’accès qui a été refusé précédemment.

## <a name="aadds112-not-enough-ip-address-in-the-managed-domain"></a>AADDS112 : Nombre insuffisant d’adresses IP dans le domaine managé

### <a name="alert-message"></a>Message d’alerte

*Nous avons détecté que le sous-réseau du réseau virtuel dans ce domaine n’a peut-être pas suffisamment d’adresses IP. Azure AD Domain Services a besoin d’au moins deux adresses IP disponibles au sein du sous-réseau où il est activé. Nous vous recommandons d’avoir au moins 3 à 5 adresses IP auxiliaires au sein de ce sous-réseau. Cela peut se produire si d’autres machines virtuelles sont déployées au sein du sous-réseau, épuisant ainsi le nombre d’adresses IP disponibles, ou s’il existe une restriction sur le nombre d’adresses IP disponibles dans le sous-réseau.*

### <a name="resolution"></a>Résolution

Le sous-réseau de réseau virtuel pour Azure AD DS nécessite suffisamment d’adresses IP pour les ressources créées automatiquement. Cet espace d’adressage IP inclut la nécessité de créer des ressources de remplacement en cas d’événement de maintenance. Pour réduire le risque de manquer d’adresses IP disponibles, ne déployez pas d’autres ressources (par exemple, vos propres machines virtuelles) sur le même sous-réseau de réseau virtuel que le domaine managé.

Pour résoudre cette alerte, supprimez votre domaine managé existant et recréez-le sur un réseau virtuel avec une plage d’adresses IP suffisamment grande. Ce processus entraîne des perturbations, car le domaine managé est indisponible, et toutes les ressources personnalisées que vous avez créées, comme les unités d’organisation ou les comptes de service, sont perdues.

1. [Supprimez le domaine managé](delete-aadds.md) de votre annuaire.
1. Pour mettre à jour la plage d’adresses IP du réseau virtuel, recherchez et sélectionnez *Réseau virtuel* dans le portail Azure. Sélectionnez le réseau virtuel du domaine managé qui a la petite plage d’adresses IP.
1. Sous **Paramètres**, sélectionnez *Espace d’adressage*.
1. Mettez à jour la plage d’adresses en choisissant la plage d’adresses existante et en la modifiant, ou en ajoutant une plage d’adresses supplémentaire. Vérifiez que la nouvelle plage d’adresses IP est suffisamment grande pour la plage de sous-réseau du domaine managé. Quand vous êtes prêt, **Enregistrez** les modifications.
1. Sélectionnez **Sous-réseaux** dans la navigation de gauche.
1. Choisissez le sous-réseau que vous souhaitez modifier ou créez-en un supplémentaire.
1. Mettez à jour ou spécifiez une plage d’adresses IP suffisamment grande, puis **Enregistrez** vos modifications.
1. [Créez un domaine managé de remplacement](tutorial-create-instance.md). Veillez à sélectionner le sous-réseau de réseau virtuel mis à jour avec une plage d’adresses IP suffisamment importante.

L’intégrité du domaine managé se met automatiquement à jour dans les deux heures, et l’alerte est supprimée.

## <a name="aadds113-resources-are-unrecoverable"></a>AADDS113 : Des ressources sont irrécupérables

### <a name="alert-message"></a>Message d’alerte

*Les ressources utilisées par Azure AD Domain Services ont été détectées dans un état inattendu et ne peuvent pas être récupérées.*

### <a name="resolution"></a>Résolution

Cette erreur est irrécupérable. Pour résoudre l’alerte, [supprimez votre domaine managé existant](delete-aadds.md), puis recréez-le. Si vous rencontrez des problèmes lors de la suppression du domaine managé, [créez une demande de support Azure][azure-support] pour obtenir une assistance supplémentaire.

## <a name="aadds114-subnet-invalid"></a>AADDS114 : Sous-réseau non valide

### <a name="alert-message"></a>Message d’alerte

*Le sous-réseau sélectionné pour le déploiement d’Azure AD Domain Services n’est pas valide et ne peut pas être utilisé.*

### <a name="resolution"></a>Résolution

Cette erreur est irrécupérable. Pour résoudre l’alerte, [supprimez votre domaine managé existant](delete-aadds.md), puis recréez-le. Si vous rencontrez des problèmes lors de la suppression du domaine managé, [créez une demande de support Azure][azure-support] pour obtenir une assistance supplémentaire.

## <a name="aadds115-resources-are-locked"></a>AADDS115 : Des ressources sont verrouillées

### <a name="alert-message"></a>Message d’alerte

*Il est impossible d’opérer sur une ou plusieurs des ressources réseau utilisées par le domaine managé, car l’étendue cible a été verrouillée.*

### <a name="resolution"></a>Résolution

Les verrous de ressources peuvent être appliqués aux ressources Azure pour en empêcher la modification ou la suppression. Azure AD DS étant un service managé, la plateforme Azure doit pouvoir apporter des modifications à la configuration. Si un verrou de ressource est appliqué à certains des composants Azure AD DS, la plateforme Azure ne peut pas effectuer ses tâches de gestion.

Pour rechercher des verrous de ressources sur les composants Azure AD DS et les supprimer, effectuez les étapes suivantes :

1. Pour chacun des composants réseau du domaine managé de votre groupe de ressources (par exemple, réseau virtuel, carte réseau ou adresse IP publique), consultez les journaux des opérations sur le Portail Azure. Ces journaux des opérations doivent indiquer la raison pour laquelle une opération échoue et où un verrou de ressource est appliqué.
1. Sélectionnez la ressource où un verrou est appliqué puis, sous **Verrous**, sélectionnez et supprimez le ou les verrous.

## <a name="aadds116-resources-are-unusable"></a>AADDS116 : Des ressources sont inutilisables

### <a name="alert-message"></a>Message d’alerte

*En raison d’une ou de plusieurs restrictions de stratégie, il est impossible d’opérer sur une ou plusieurs des ressources réseau utilisées par le domaine managé.*

### <a name="resolution"></a>Résolution

Les stratégies sont appliquées aux ressources et aux groupes de ressources Azure qui contrôlent les actions de configuration autorisées. Azure AD DS étant un service managé, la plateforme Azure doit pouvoir apporter des modifications à la configuration. Si une stratégie est appliquée à certains des composants Azure AD DS, la plateforme Azure risque de ne pas pouvoir effectuer ses tâches de gestion.

Pour rechercher les stratégies appliquées sur les composants Azure AD DS et les mettre à jour, effectuez les étapes suivantes :

1. Pour chacun des composants réseau du domaine managé de votre groupe de ressources (par exemple, réseau virtuel, carte NIC ou adresse IP publique), consultez les journaux des opérations sur le Portail Azure. Ces journaux des opérations doivent indiquer la raison pour laquelle une opération échoue et où une stratégie restrictive est appliquée.
1. Sélectionnez la ressource où une stratégie est appliquée puis, sous **Stratégies**, sélectionnez et modifiez la stratégie pour qu’elle soit moins restrictive.

## <a name="aadds500-synchronization-has-not-completed-in-a-while"></a>AADDS500 : La synchronisation ne parvient pas à se terminer depuis un certain temps

### <a name="alert-message"></a>Message d’alerte

*La dernière synchronisation du domaine managé avec Azure AD a eu lieu le [date]. Les utilisateurs sont peut-être dans l’impossibilité de se connecter au domaine managé, ou les appartenances aux groupes ne sont peut-être pas synchronisées avec Azure AD.*

### <a name="resolution"></a>Résolution

[Vérifiez l’intégrité d’Azure AD DS](check-health.md) pour toutes les alertes qui indiquent des problèmes au niveau de la configuration du domaine managé. Les problèmes de configuration réseau peuvent bloquer la synchronisation à partir d’Azure AD. Si vous êtes en mesure de résoudre les alertes qui indiquent un problème de configuration, attendez deux heures, puis vérifiez à nouveau si la synchronisation s’est terminée avec succès.

Voici quelques raisons courantes qui provoquent l’arrêt de la synchronisation dans un domaine managé :

* La connectivité réseau requise est bloquée. Pour savoir comment vérifier si le réseau virtuel Azure présente des problèmes et connaître les exigences, consultez [Résolution des problèmes relatifs aux groupes de sécurité réseau](alert-nsg.md) et [Exigences réseau d’Azure AD DS](network-considerations.md).
*  La synchronisation de mot de passe n’a pas été configurée ou ne s’est pas terminée avec succès quand le domaine managé a été déployé. Vous pouvez configurer la synchronisation de mot de passe pour les [utilisateurs du cloud uniquement](tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds) ou pour les [utilisateurs hybrides locaux](tutorial-configure-password-hash-sync.md).

## <a name="aadds501-a-backup-has-not-been-taken-in-a-while"></a>AADDS501 : Il n’y a pas eu de sauvegarde depuis un certain temps

### <a name="alert-message"></a>Message d’alerte

*La dernière sauvegarde du domaine managé a eu lieu le [date].*

### <a name="resolution"></a>Résolution

[Vérifiez l’intégrité d’Azure AD DS](check-health.md) pour les alertes qui indiquent des problèmes avec la configuration du domaine géré. Les problèmes liés à la configuration réseau peuvent empêcher la plateforme Azure d’effectuer des sauvegardes. Si vous êtes en mesure de résoudre les alertes qui indiquent un problème de configuration, attendez deux heures, puis vérifiez à nouveau si la synchronisation s’est terminée avec succès.

## <a name="aadds503-suspension-due-to-disabled-subscription"></a>AADDS503 : Suspension en raison de l’abonnement désactivé

### <a name="alert-message"></a>Message d’alerte

*Le domaine managé est suspendu, car l’abonnement Azure associé au domaine n’est pas actif.*

### <a name="resolution"></a>Résolution

> [!WARNING]
> Si un domaine managé est suspendu pendant une période prolongée, il y a un risque qu’il soit supprimé. Résolvez le motif de suspension le plus rapidement possible. Pour plus d’informations, consultez [Présentation des états suspendus pour Azure AD DS](suspension.md).

Azure AD DS nécessite un abonnement actif. Si l’abonnement Azure auquel le domaine managé était associé n’est pas actif, vous devez le renouveler pour réactiver l’abonnement.

1. [Renouvelez votre abonnement Azure](../cost-management-billing/manage/subscription-disabled.md).
2. Une fois l’abonnement renouvelé, une notification Azure AD DS vous permet de réactiver le domaine managé.

Quand le domaine managé est réactivé, l’intégrité du domaine managé se met automatiquement à jour dans les deux heures et l’alerte est supprimée.

## <a name="aadds504-suspension-due-to-an-invalid-configuration"></a>AADDS504 : Suspension en raison d’une configuration non valide

### <a name="alert-message"></a>Message d’alerte

*Le domaine managé est suspendu en raison d’une configuration non valide. Le service n’a pas pu gérer, corriger ou mettre à jour les contrôleurs du domaine managé depuis un certain temps.*

### <a name="resolution"></a>Résolution

> [!WARNING]
> Si un domaine managé est suspendu pendant une période prolongée, il y a un risque qu’il soit supprimé. Résolvez le motif de suspension le plus rapidement possible. Pour plus d’informations, consultez [Présentation des états suspendus pour Azure AD DS](suspension.md).

[Vérifiez l’intégrité d’Azure AD DS](check-health.md) pour les alertes qui indiquent des problèmes avec la configuration du domaine géré. Si vous êtes en mesure de résoudre les alertes qui indiquent un problème de configuration, attendez deux heures, puis vérifiez de nouveau que la synchronisation est terminée. Quand vous êtes prêt, [créez une demande de support Azure][azure-support] pour réactiver le domaine managé.

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez toujours des problèmes, [formulez une demande de support Azure][azure-support] pour bénéficier d’une aide supplémentaire.

<!-- INTERNAL LINKS -->
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md
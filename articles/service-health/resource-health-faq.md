---
title: FAQ Azure Resource Health
description: Vue d’ensemble d’Azure Resource Health
ms.topic: conceptual
ms.date: 01/29/2019
ms.openlocfilehash: a5ad8867b110039e4dcc34b20c8a61fccfd39eaf
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032862"
---
# <a name="azure-resource-health-faq"></a>FAQ Azure Resource Health
Découvrez les réponses aux questions courantes sur Azure Resource Health.

## <a name="what-is-azure-resource-health"></a>Qu’est-ce qu’Azure Resource Health ?
Resource Health vous permet d’établir des diagnostics et d’obtenir de l’aide lorsqu’un problème touchant Azure a des répercussions sur vos ressources. Il vous informe de l’intégrité (actuelle et passée) de vos ressources et vous aide à atténuer les problèmes. Resource Health propose un support technique dès lors que vous êtes confronté à des problèmes de service Azure et que vous avez besoin d’aide.  

## <a name="what-is-the-resource-health-intended-for"></a>À quoi sert Resource Health ?
Une fois qu’un problème lié à une ressource a été détecté, Resource Health peut vous aider à en diagnostiquer la cause première. Il fournit de l’aide pour résoudre le problème et un support technique si vous avez besoin d’aide supplémentaire concernant des problèmes liés aux services Azure.

## <a name="what-health-checks-are-performed-by-resource-health"></a>Quels contrôles d’intégrité sont effectués par Resource Health ?
Resource Health effectue plusieurs vérifications selon le [type de ressource](resource-health-checks-resource-types.md). Ces contrôles sont conçus pour résoudre trois types de problèmes : 
- Événements non planifiés, par exemple le redémarrage d’un hôte inattendu
- Événements planifiés, telles que les mises à jour planifiées du système d’exploitation hôte
- Événements déclenchés par des actions de l’utilisateur, par exemple le redémarrage d’une machine virtuelle

## <a name="what-does-each-of-the-health-status-mean"></a>Que signifie chaque état Resource Health ?
Il existe trois états différents :
- Disponible : aucun problème connu sur la plateforme Azure susceptible d’impacter cette ressource
- Non disponible : Resource Health a détecté des problèmes qui impactent la ressource
- Inconnu : Resource Health ne peut pas déterminer l’intégrité d’une ressource, car il ne reçoit plus d’informations la concernant. 

## <a name="what-does-the-unknown-status-mean-is-something-wrong-with-my-resource"></a>Que signifie l’état inconnu ? Y a-t-il un problème avec ma ressource ?
L’état d’intégrité est défini sur Inconnu lorsque Resource Health ne reçoit plus d’informations concernant une ressource spécifique. Si cet état n’est pas une indication définitive de l’état de la ressource, cela peut indiquer un problème Azure si vous rencontrez des difficultés.

## <a name="how-can-i-get-help-for-a-resource-that-is-unavailable"></a>Comment puis-je obtenir de l’aide pour une ressource qui n’est pas disponible ?
Vous pouvez envoyer une demande de support à partir du panneau Resource Health. Vous n’avez pas besoin d’un contrat de support Microsoft pour ouvrir une demande lorsque la ressource n’est pas disponible suite à des événements de plateforme.

## <a name="does-resource-health-differentiate-between-unavailability-caused-by-platform-problems-versus-something-i-did"></a>Resource Health fait-il la différence entre une indisponibilité causée par des problèmes de plateforme et une mauvaise manipulation de l’utilisateur ?
Oui, lorsqu’une ressource n’est pas disponible, Resource Health identifie l’origine du problème selon les catégories suivantes : 
-   Action initiée par l’utilisateur
-   Événement planifié 
-   Événement non planifié

Sur le portail, les actions initiées par l’utilisateur sont affichées à l’aide d’une icône de notification bleue, tandis que les événements planifiés et non planifiés sont affichés à l’aide d’une icône d’avertissement rouge. Pour plus d’informations, voir [Vue d’ensemble d’Azure Resource Health](Resource-health-overview.md).  

## <a name="can-i-integrate-resource-health-with-my-monitoring-tools"></a>Puis-je intégrer Resource Health à mes outils d’analyse ?
Resource Health [prend en charge](resource-health-alert-arm-template-guide.md) les alertes basées sur le journal d’activité. Les alertes du journal d'activité utilisent des [groupes d'actions](../azure-monitor/platform/action-groups.md) pour avertir les utilisateurs qu'une alerte a été déclenchée. Les groupes d'actions prennent en charge différents canaux de notification (e-mail, SMS, webhook, actions de gestion des services informatiques, etc.).

## <a name="where-do-i-find-resource-health"></a>Où trouver Resource Health ?
Une fois connecté au portail Azure, vous pouvez accéder à Resource Health de plusieurs manières :
- Accédez à votre ressource. Dans le volet de navigation de gauche, sélectionnez **Resource Health**.
- Accédez au panneau Azure Service Health.  Dans le volet de navigation de gauche, sélectionnez **Resource Health**.

Vous pouvez également utiliser l’API Resource Health pour obtenir des informations sur l’intégrité de vos ressources.

## <a name="is-resource-health-available-for-all-resource-types"></a>Resource Health est-il disponible pour tous les types de ressources ?
Vous trouverez la liste des contrôles d’intégrité et des types de ressources pris en charge par Resource Health [ici](resource-health-checks-resource-types.md).

## <a name="what-should-i-do-if-my-resource-is-showing-available-but-i-believe-it-is-not"></a>Que dois-je faire si ma ressource apparaît comme disponible, alors que ce n’est pas le cas ?
Lors de la vérification de l’intégrité d’une ressource, cliquez sur **Rapport d’état d’intégrité incorrect** sous l’état d’intégrité. Avant d’envoyer le rapport, vous avez la possibilité de fournir des détails supplémentaires sur la raison pour laquelle vous pensez que l’état actuel est incorrect.

## <a name="is-resource-health-available-for-all-azure-regions"></a>Resource Health est-il disponible pour toutes les régions Azure ? 
Resource Health est disponible dans toutes les zones géographiques Azure.

## <a name="how-is-resource-health-different-from-azure-status-or-the-service-health-dashboard"></a>En quoi Resource Health diffère-t-il de l’état d’Azure ou du tableau de bord Service Health ?
Les informations fournies par Resource Health sont plus précises que celles de l’état d’Azure ou du tableau de bord Service Health.

Tandis que [l’état d’Azure](https://status.azure.com) et le tableau de bord Service Health vous informent des problèmes de service affectant un large éventail de clients (par exemple, une région Azure), Resource Health expose des événements plus granulaires qui ne concernent que la ressource spécifique. Par exemple, si un hôte redémarre inopinément, Resource Health n’avertit que les clients dont les machines virtuelles étaient exécutées sur cet hôte.

## <a name="do-i-need-to-activate-resource-health-for-each-resource"></a>Dois-je activer Resource Health pour chaque ressource ?
Non, les informations d’intégrité sont disponibles pour tous les types de ressources disponibles via Resource Health. 

## <a name="do-we-need-to-enable-resource-health-for-my-organization"></a>Faut-il activer Resource Health pour mon organisation ?
Non.  Azure Resource Health est accessible via le portail Azure sans conditions requises d’installation.

## <a name="is-resource-health-available-free-of-charge"></a>Resource Health est-il disponible gratuitement ?
Oui.  Azure Resource Health est gratuit.

## <a name="what-are-the-recommendations-that-resource-health-provides"></a>Quelles sont les recommandations que fournit Resource Health ?
En fonction de l’état d’intégrité, Resource Health fournit des recommandations dans le but de réduire le temps passé à résolution les problèmes. Pour les ressources disponibles, les recommandations sont axées sur la façon de résoudre les principaux problèmes rencontrés par les clients. Si la ressource est indisponible en raison d’un événement Azure non planifié, l’objectif principal est de vous assister pendant et après le processus de récupération. 

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur Resource Health :
-  [Vue d’ensemble d’Azure Resource Health](Resource-health-overview.md)
-  [Types de ressource et contrôles d’intégrité disponibles par le biais d’Azure Resource Health](resource-health-checks-resource-types.md)

---
title: Azure Traffic Manager avec Azure Site Recovery | Microsoft Docs
description: Explique comment utiliser Azure Traffic Manager avec Azure Site Recovery pour la migration et la reprise après sinistre
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: mayg
ms.openlocfilehash: bb112e0b2d1c64e65ecaf6749a25707d8632c0cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86134958"
---
# <a name="azure-traffic-manager-with-azure-site-recovery"></a>Azure Traffic Manager avec Azure Site Recovery

Azure Traffic Manager vous permet de contrôler la répartition du trafic entre les points de terminaison de votre application. Un point de terminaison est tout service côté Internet hébergé à l’intérieur ou à l’extérieur d’Azure.

Traffic Manager utilise le système DNS pour diriger les requêtes des clients vers le point de terminaison le plus approprié, en fonction de la méthode de routage du trafic et de l’intégrité des points de terminaison. Traffic Manager fournit un large éventail de [méthodes de routage du trafic](../traffic-manager/traffic-manager-routing-methods.md) et [d’option de surveillance des points de terminaison](../traffic-manager/traffic-manager-monitoring.md) pour répondre aux besoins variés des applications et aux divers modèles de basculement automatique. Les clients se connectent directement au point de terminaison sélectionné. Traffic Manager n’est pas un proxy ni une passerelle, et ne voit pas le trafic qui transite entre le client et le service.

Cet article explique comment combiner le routage intelligent d’Azure Traffic Manager avec les fonctionnalités puissantes de migration et de reprise après sinistre d’Azure Site Recovery.

## <a name="on-premises-to-azure-failover"></a>Basculement de l’environnement local vers Azure

Pour le premier scénario, prenons le cas d’une **société A** qui exécute l’ensemble de son infrastructure d’applications dans un environnement local. Pour des raisons de conformité et de continuité d’activité, la **société A** décide d’utiliser Azure Site Recovery pour protéger ses applications.

La **société A** exécute des applications avec des points de terminaison publics et souhaite pouvoir rediriger facilement le trafic vers Azure en cas d’incident. Dans Azure Traffic Manager, la méthode de routage du trafic [Priorité](../traffic-manager/traffic-manager-configure-priority-routing-method.md) permet à la société A d’implémenter facilement ce modèle de basculement.

La configuration est la suivante :
- La **société A** crée un [profil Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md).
- En utilisant la méthode de routage **Priorité**, la **société A** crée deux points de terminaison : **Principal** pour l’environnement local et **Basculement** pour Azure. **Principal** reçoit la Priorité 1 et **Basculement** reçoit la Priorité 2.
- Étant donné que le point de terminaison **Principal** est hébergé en dehors d’Azure, il est créé comme un point de terminaison [Externe](../traffic-manager/traffic-manager-endpoint-types.md#external-endpoints).
- Avec Azure Site Recovery, le site Azure ne comprend pas de machines virtuelles ou d’applications qui s’exécutent avant le basculement. Par conséquent, le point de terminaison **Basculement** est également créé comme un point de terminaison **Externe**.
- Par défaut, le trafic utilisateur est dirigé vers l’application locale, car ce point de terminaison a la priorité la plus élevée. Aucun trafic n’est dirigé vers Azure si le point de terminaison **Principal** est sain.

![Local vers Azure avant le basculement](./media/concepts-traffic-manager-with-site-recovery/on-premises-failover-before.png)

En cas d’incident, la société A peut déclencher un [basculement](site-recovery-failover.md) vers Azure et récupérer ses applications sur Azure. Quand Azure Traffic Manager détecte que le point de terminaison **Principal** n’est plus sain, il utilise automatiquement le point de terminaison **Basculement** dans la réponse DNS, et les utilisateurs se connectent à l’application qui a été récupérée sur Azure.

![Local vers Azure après le basculement](./media/concepts-traffic-manager-with-site-recovery/on-premises-failover-after.png)

En fonction de ses besoins, la **société A** peut choisir une [fréquence de détection](../traffic-manager/traffic-manager-monitoring.md) plus ou moins élevée pour basculer d’un environnement local vers Azure en cas d’incident, et garantir un temps d’arrêt minimal pour les utilisateurs.

Lorsque l’incident est maîtrisé, la **société A** peut effectuer une restauration automatique d’Azure vers son environnement local ([VMware](vmware-azure-failback.md) ou [Hyper-V](hyper-v-azure-failback.md)) à l’aide d’Azure Site Recovery. Quand Traffic Manager détecte que le point de terminaison **Principal** est à nouveau sain, il utilise automatiquement le point de terminaison **Principal** dans les réponses DNS.

## <a name="on-premises-to-azure-migration"></a>Migration d’un environnement local vers Azure

En plus de la reprise après sinistre, Azure Site Recovery permet également d’effectuer des [migrations vers Azure](migrate-overview.md). Grâce aux puissantes fonctionnalités de test de basculement d’Azure Site Recovery, les clients peuvent évaluer les performances des applications sur Azure, sans affecter leur environnement local. Lorsque les clients sont prêts à effectuer la migration, ils peuvent choisir de migrer plusieurs charges de travail entières, ou d’effectuer une migration et une mise à l’échelle progressivement.

La méthode de routage [Pondérée](../traffic-manager/traffic-manager-configure-weighted-routing-method.md) d’Azure Traffic Manager peut être utilisée pour diriger une partie du trafic entrant et diriger la plus grande partie du trafic vers l’environnement local. Cette approche peut aider à évaluer les performances de mise à l’échelle, car elle vous permet d’accroître la pondération affectée à Azure à mesure que vous migrez vos charges de travail vers Azure.

Par exemple, la **société B** choisit une migration en plusieurs phases, en déplaçant une partie de son environnement d’application tout en conservant le reste localement. Au cours des phases initiales, lorsque la majeure partie de l’environnement est locale, une pondération supérieure est attribuée à l’environnement local. Traffic Manager retourne un point de terminaison en fonction des pondérations affectées aux points de terminaison disponibles.

![Migration d’un environnement local vers Azure](./media/concepts-traffic-manager-with-site-recovery/on-premises-migration.png)

Pendant la migration, les deux points de terminaison sont actifs, et la plupart du trafic est dirigé vers l’environnement local. Au fil de la migration, une pondération supérieure peut être attribuée au point de terminaison Azure, et le point de terminaison local peut être désactivé après la migration.

## <a name="azure-to-azure-failover"></a>Basculement d’Azure vers Azure

Pour cet exemple, prenons le cas d’une **société C** qui exécute l’ensemble de son infrastructure d’applications dans Azure. Pour des raisons de conformité et de continuité d’activité, la **société C** décide d’utiliser Azure Site Recovery pour protéger ses applications.

La **société C** exécute des applications avec des points de terminaison publics et souhaite pouvoir rediriger facilement le trafic vers une autre région Azure en cas d’incident. La méthode de routage du trafic [Priorité](../traffic-manager/traffic-manager-configure-priority-routing-method.md) permet à la **société C** d’implémenter facilement ce modèle de basculement.

La configuration est la suivante :
- La **société C** crée un [profil Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md).
- En utilisant la méthode de routage **Priorité**, la **société C** crée deux points de terminaison : **Principal** pour la région source (Azure Asie Est) et **Basculement** pour la région de récupération (Azure Asie Sud-Est). **Principal** reçoit la Priorité 1 et **Basculement** reçoit la Priorité 2.
- Étant donné que le point de terminaison **Principal** est hébergé dans Azure, il peut également être un point de terminaison [Azure](../traffic-manager/traffic-manager-endpoint-types.md#azure-endpoints).
- Avec Azure Site Recovery, le site Azure de récupération ne comprend pas de machines virtuelles ou d’applications qui s’exécutent avant le basculement. Par conséquent, le point de terminaison **Basculement** peut également être créé comme un point de terminaison [Externe](../traffic-manager/traffic-manager-endpoint-types.md#external-endpoints).
- Par défaut, le trafic utilisateur est dirigé vers la région source (Asie Est), car ce point de terminaison a la priorité la plus élevée. Aucun trafic n’est dirigé vers la région de récupération si le point de terminaison **Principal** est sain.

![Azure vers Azure avant le basculement](./media/concepts-traffic-manager-with-site-recovery/azure-failover-before.png)

En cas d’incident, la **société C** peut déclencher un [basculement](azure-to-azure-tutorial-failover-failback.md) et récupérer ses applications dans la région Azure de récupération. Quand Azure Traffic Manager détecte que le point de terminaison Principal n’est plus sain, il utilise automatiquement le point de terminaison **Basculement** dans la réponse DNS, et les utilisateurs se connectent à l’application qui a été récupérée dans la région Azure de récupération (Asie Sud-Est).

![Azure vers Azure après le basculement](./media/concepts-traffic-manager-with-site-recovery/azure-failover-after.png)

En fonction de ses besoins, la **société C** peut choisir une [fréquence de détection](../traffic-manager/traffic-manager-monitoring.md) plus ou moins élevée pour basculer entre la région source et la région de récupération, et garantir un temps d’arrêt minimal pour les utilisateurs.

Lorsque l’incident est maîtrisé, la **société C** peut effectuer une restauration automatique à partir de la région Azure de récupération à l’aide d’Azure Site Recovery. Quand Traffic Manager détecte que le point de terminaison **Principal** est à nouveau sain, il utilise automatiquement le point de terminaison **Principal** dans les réponses DNS.

## <a name="protecting-multi-region-enterprise-applications"></a>Protection des applications d’entreprise multirégions

Les entreprises multinationales améliorent souvent l’expérience utilisateur en adaptant leurs applications aux besoins de chaque région. La localisation et la réduction de la latence peuvent entraîner une division de l’infrastructure des applications entre plusieurs régions. Les entreprises sont également soumises aux lois de leur pays concernant les données, et choisissent d’isoler une partie de leur infrastructure d’applications au sein de limites régionales.  

Prenons l’exemple de la **société D** qui a divisé ses points de terminaison d’application pour fournir séparément l’Allemagne et le reste du monde. Pour une telle configuration, la **société D** utilise la méthode de routage [Géographique](../traffic-manager/traffic-manager-configure-geographic-routing-method.md) d’Azure Traffic Manager. Tout le trafic en provenance d’Allemagne est dirigé vers le **point de terminaison 1** et tout le reste du trafic est dirigé vers le **point de terminaison 2**.

Le problème avec cette configuration est que si le **point de terminaison 1**  cesse de fonctionner pour une raison quelconque, aucune redirection du trafic n’est effectuée vers le **point de terminaison 2**. Le trafic en provenance d’Allemagne continue alors d’être dirigé vers le **point de terminaison 1**, quel que soit l’état d’intégrité du point de terminaison. Les utilisateurs allemands n’ont donc plus accès à l’application de la **société D**. De même, si le **point de terminaison 2** est hors connexion, aucune redirection du trafic n’est effectuée vers le **point de terminaison 1**.

![Application multirégion avant](./media/concepts-traffic-manager-with-site-recovery/geographic-application-before.png)

Pour éviter ce problème et garantir la résilience des applications, la **société D** utilise des [profils imbriqués Traffic Manager](../traffic-manager/traffic-manager-nested-profiles.md) avec Azure Site Recovery. Dans une configuration de profil imbriqué, le trafic n’est pas dirigé vers des points de terminaison, mais vers d’autres profils Traffic Manager. Voici comment fonctionne cette configuration :
- Au lieu d’utiliser le routage géographique avec des points de terminaison, la **société D** l’utilise avec des profils Traffic Manager.
- Chaque profil enfant Traffic Manager utilise le routage **Priorité** avec un point de terminaison principal et un point de terminaison de récupération, d’où l’imbrication du routage **Priorité** dans le routage **Géographique**.
- Pour permettre la résilience des applications, chaque distribution de charge de travail utilise Azure Site Recovery pour basculer vers une région de récupération en cas d’incident.
- Lorsque le profil parent Traffic Manager reçoit une requête DNS, celle-ci est dirigée vers le profil enfant Traffic Manager adapté, qui lui répond avec un point de terminaison disponible.

![Application multirégion après](./media/concepts-traffic-manager-with-site-recovery/geographic-application-after.png)

Par exemple, si le point de terminaison Allemagne Centre échoue, l’application peut rapidement être récupérée vers la région Allemagne Nord-Est. Le nouveau point de terminaison gère le trafic en provenance d’Allemagne avec un temps d’arrêt minimal pour les utilisateurs. De même, une panne de point de terminaison dans la région Europe Ouest peut être gérée en récupérant la charge de travail d’application vers la région Europe Nord, et en utilisant Azure Traffic Manager pour gérer les redirections DNS vers le point de terminaison disponible.

La configuration ci-dessus peut être développée pour inclure autant de combinaisons de régions et de points de terminaison que nécessaire. Traffic Manager permet jusqu’à 10 niveaux de profils imbriqués et n’autorise pas les boucles au sein de la configuration imbriquée.

## <a name="recovery-time-objective-rto-considerations"></a>Remarques sur les objectifs de délai de récupération

Dans la plupart des entreprises, l’ajout et la modification des enregistrements DNS sont gérés par une équipe distincte ou par une personne extérieure à l’entreprise. Cela rend très difficile la modification des enregistrements DNS. La durée nécessaire à la mise à jour des enregistrements DNS, par d’autres équipes ou par des entreprises qui gèrent l’infrastructure DNS, dépend de l’entreprise et impacte sur l’objectif de délai de récupération de l’application.

En utilisant Traffic Manager, vous pouvez débuter en amont le travail nécessaire aux mises à jour DNS. Aucune action manuelle ou scriptée n’est nécessaire au moment du basculement. Cette approche permet un basculement rapide (et donc un objectif de délai de récupération plus court), et permet d’éviter les erreurs de modification DNS longues et coûteuses en cas d’incident. Avec Traffic Manager, même l’étape de restauration automatique est automatisée. Si ce n’était pas le cas, elle devrait être gérée séparément.

Le fait de configurer un [intervalle de détection](../traffic-manager/traffic-manager-monitoring.md) adapté, au moyen de vérifications d’intégrité plus ou moins rapprochées, peut considérablement réduire l’objectif de délai de récupération pendant le basculement et réduire le temps d’arrêt pour les utilisateurs.

Vous pouvez optimiser davantage la valeur de durée de vie (TTL) du DNS pour le profil Traffic Manager. La durée de vie correspond à la durée de mise en cache d’une entrée DNS par un client. Pendant cette durée de vie, le DNS n’est pas interrogé deux fois pour un même enregistrement. Chaque enregistrement DNS a sa propre durée de vie. La diminution de cette valeur entraîne l’augmentation des requêtes DNS à Traffic Manager, mais peut raccourcir l’objectif de délai de récupération en découvrant les pannes plus rapidement.

La durée de vie subie par le client n’augmente pas en cas d’augmentation du nombre de programmes de résolution DNS entre le client et le serveur DNS de référence. Les programmes de résolution DNS effectuent le décompte de la durée de vie et passent uniquement une valeur de durée de vie qui reflète le temps écoulé depuis la mise en cache de l’enregistrement. Ainsi, l’enregistrement DNS est actualisé sur le client une fois la durée de vie terminée, quel que soit le nombre de programmes de résolution DNS.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [méthodes de routage](../traffic-manager/traffic-manager-routing-methods.md)de Traffic Manager
- En savoir plus sur les [profils imbriqués Traffic Manager](../traffic-manager/traffic-manager-nested-profiles.md)
- En savoir plus sur la [surveillance du point de terminaison](../traffic-manager/traffic-manager-monitoring.md).
- En savoir plus sur les [plans de récupération](site-recovery-create-recovery-plans.md) pour automatiser le basculement d’application

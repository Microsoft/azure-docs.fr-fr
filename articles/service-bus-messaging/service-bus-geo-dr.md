---
title: Géorécupération d’urgence Azure Service Bus | Microsoft Docs
description: Utiliser les régions géographiques pour le basculement et la récupération d’urgence dans Azure Service Bus
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 8c203ed197c1e5bfb15cfb503a04df79b85c630e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91372521"
---
# <a name="azure-service-bus-geo-disaster-recovery"></a>Géorécupération d’urgence Azure Service Bus

Si tout un centre de données ou une région Azure complète (si aucune [zone de disponibilité](../availability-zones/az-overview.md) n’est utilisée) connaît un temps d’arrêt, il est essentiel que le traitement des données puisse continuer dans les autres régions ou centres de données. Pour cette raison, la *géo-reprise d’activité après sinistre* est une fonctionnalité importante pour toutes les entreprises. Azure Service Bus prend en charge la géo-reprise d’activité après sinistre au niveau de l’espace de noms.

La fonctionnalité de géorécupération d’urgence est disponible de manière globale pour la référence SKU Premium de Service Bus. 

>[!NOTE]
> La géo-reprise d’activité après sinistre garantit uniquement la copie des métadonnées (files d’attente, rubriques, abonnements, filtres) entre l’espace de noms principal et l’espace de noms secondaire, lorsque ceux-ci sont associés.

## <a name="outages-and-disasters"></a>Pannes et sinistres

Il est important de noter la différence entre « panne » et « sinistre ». 

Une *panne* se définit comme l’indisponibilité temporaire d’Azure Service Bus. La panne impacte certains composants du service, comme une banque de messages, ou le centre de données entier. Toutefois, une fois le problème résolu, Service Bus redevient disponible. En règle générale, une panne ne provoque aucune perte de messages ou d’autres données. Une coupure de courant dans le centre de données est un exemple de panne. Certaines pannes sont uniquement dues à une courte perte de la connexion en raison de problèmes réseau ou de soucis temporaires. 

Un *sinistre* se définit comme une perte définitive ou à long terme d’un cluster Service Bus, d’une région Azure ou d’un centre de données. La région ou le centre de données peut redevenir disponible ou pas, et rester arrêté(e) pendant plusieurs heures ou jours. Les incendies, les inondations ou les tremblements de terre sont des exemples de sinistres. Un sinistre qui devient permanent peut entraîner la perte de certains messages, événements ou d’autres données. Toutefois, dans la plupart des cas, il ne doit y avoir aucune perte de données et les messages peuvent être récupérés une fois que le centre de données est sauvegardé.

La fonctionnalité de géorécupération d’urgence d’Azure Service Bus est une solution de récupération d’urgence. Les concepts et le workflow décrits dans cet article concernent des scénarios de sinistres, pas des pannes temporaires ou transitoires. Pour obtenir une présentation détaillée de la récupération d’urgence dans Microsoft Azure, consultez [cet article](/azure/architecture/resiliency/disaster-recovery-azure-applications).   

## <a name="basic-concepts-and-terms"></a>Concepts et terminologie de base

La fonctionnalité de récupération d’urgence implémente la récupération d’urgence des métadonnées, en s’appuyant sur les espaces de noms de récupération d’urgence principal et secondaire. Notez que la fonctionnalité de géorécupération d’urgence est disponible uniquement pour la [référence SKU Premium](service-bus-premium-messaging.md). Vous n’avez pas besoin de modifier la chaîne de connexion, car la connexion est établie à l’aide d’un alias.

Cet article emploie les termes suivants :

-  *Alias* : nom d’une configuration de récupération d’urgence que vous avez configurée. L’alias fournit une chaîne de connexion de nom de domaine complet (FQDN) stable. Les applications utilisent cet alias de chaîne de connexion pour se connecter à un espace de noms. L’utilisation d’un alias garantit que la chaîne de connexion restera inchangée après le déclenchement du basculement.

-  *Espace de noms principal/secondaire* : espaces de noms qui correspondent à l’alias. L’espace de noms principal est « actif » et reçoit les messages (il peut s’agir d’un espace de noms existant ou nouveau). L’espace de noms secondaire est « passif » et ne reçoit pas de messages. Les métadonnées sont synchronisées entre ces deux espaces de noms, qui peuvent ainsi accepter facilement les messages sans aucune modification du code d’application ou de la chaîne de connexion. Pour vous assurer que seul l’espace de noms actif reçoit des messages, vous devez utiliser l’alias. 

-  *Métadonnées* : entités telles que les files d'attentes, les rubriques et les abonnements ; incluent également leurs propriétés sur le service associé à l'espace de noms. Notez que seules les entités et leurs paramètres sont automatiquement répliqués. Les messages ne sont pas répliqués.

-  *Basculement* : processus d’activation de l’espace de noms secondaire.

## <a name="setup"></a>Programme d’installation

La section suivante est une présentation de l’association de deux espaces de noms.

![1][]

Le processus est le suivant :

1. Provisionnez un espace de noms ***Principal*** pour Service Bus Premium.

2. Provisionnez un espace de noms ***Secondaire*** pour Service Bus Premium dans une région *différente de celle où est provisionné l’espace de noms principal*. Vous obtiendrez ainsi un isolement des pannes dans toutes les régions du centre de données.

3. Créez une association entre l’espace de noms principal et l’espace de noms secondaire pour obtenir l’***alias***.

    >[!NOTE] 
    > Si vous avez [migré votre espace de noms Azure Service Bus Standard vers Azure Service Bus Premium](service-bus-migrate-standard-premium.md), vous devez utiliser l’alias préexistant (par exemple, la chaîne de connexion de l’espace de noms Service Bus Standard) pour créer la configuration de récupération d’urgence via **PS/CLI** ou **l’API REST**.
    >
    >
    > Cela tient au fait que lors de la migration, le nom DNS/la chaîne de connexion de l’espace de noms Azure Service Bus Standard devient un alias pour l’espace de noms Azure Service Bus Premium.
    >
    > Vos applications clientes doivent utiliser cet alias (par exemple, la chaîne de connexion de l’espace de noms Azure Service Bus Standard) pour se connecter à l’espace de noms Premium là où le couplage de récupération d’urgence a été configuré.
    >
    > Si vous utilisez le portail pour la configuration de la récupération d’urgence, le portail extrait cette notification pour vous.


4. Utilisez l’***alias*** obtenu à l’étape 3 pour connecter vos applications clientes à l’espace de noms principal où est activée la géo-reprise d’activité après sinistre. Initialement, l’alias pointe vers l’espace de noms principal.

5. [Facultatif] Ajoutez un système de supervision pour détecter si un basculement est nécessaire.

## <a name="failover-flow"></a>Flux de basculement

Le basculement est déclenché manuellement par le client (soit explicitement à l’aide d’une commande, soit via une logique métier appartenant au client qui déclenche cette commande) et jamais par Azure. Ainsi, le client dispose d’une pleine propriété et d’une visibilité complète pour la résolution des pannes sur le segment principal d’Azure.

![4][]

Après le déclenchement du basculement :

1. La chaîne de connexion ***alias*** est mise à jour pour pointer vers l’espace de noms secondaire Premium.

2. Les clients (expéditeurs et destinataires) se connectent automatiquement à l’espace de noms secondaire.

3. L’association existante entre les espaces de noms Premium principal et secondaire est rompue.

Après le lancement du basculement :

1. Vous voulez être sûr de pouvoir refaire un basculement en cas de nouvelle panne. Pour cela, configurez un autre espace de noms passif et mettez à jour l’association. 

2. Tirez (pull) les messages à partir de l’ancien espace de noms principal une fois qu’il est de nouveau disponible. Après cela, utilisez cet espace de noms pour les messages réguliers en dehors de votre configuration de géorécupération ou supprimez l’ancien espace de noms principal.

> [!NOTE]
> Seule la sémantique de transfert du basculement est prise en charge. Dans ce scénario, vous basculez puis effectuez un nouveau couplage avec un nouvel espace de noms. La restauration automatique n’est pas prise en charge ; par exemple, dans un cluster SQL. 

Vous pouvez automatiser le basculement à l’aide de systèmes de surveillance ou à l’aide de solutions de surveillance personnalisées. Toutefois, cette automatisation nécessite des tâches de planification et du travail supplémentaires, qui ne seront pas abordés dans cet article.

![2][]

## <a name="management"></a>Gestion

Si vous avez fait une erreur (par exemple, vous avez associé les mauvaises régions lors de la configuration initiale), vous pouvez rompre le couplage des deux espaces de noms à tout moment. Si vous souhaitez utiliser les espaces de noms couplés comme des espaces de noms standard, supprimez l’alias.

## <a name="use-existing-namespace-as-alias"></a>Utiliser l’espace de noms existant en tant qu’alias

Si vous avez un scénario dans lequel vous ne pouvez pas modifier les connexions des producteurs et des consommateurs, vous pouvez réutiliser le nom de votre espace de noms en tant que nom d’alias. Voir l’[exemple de code sur GitHub ici](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/SBGeoDR2/SBGeoDR_existing_namespace_name).

## <a name="samples"></a>Exemples

Les [exemples sur GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/SBGeoDR2/) montrent comment configurer et lancer un basculement. Ces exemples illustrent les concepts suivants :

- Un exemple .NET et des paramètres requis dans Azure Active Directory pour utiliser Azure Resource Manager avec Service Bus afin d’installer et d’activer la géo-reprise d'activité après sinistre.
- Étapes requises pour exécuter l’exemple de code.
- Comment utiliser un espace de noms existant en tant qu’alias.
- Exécuter pas à pas pour activer de façon alternative la géorécupération d’urgence via PowerShell ou CLI.
- [Envoyer et recevoir](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/TestGeoDR/ConsoleApp1) à partir de l’espace de noms principal ou secondaire actuel à l’aide de l’alias.

## <a name="considerations"></a>Considérations

Notez les points suivants pour cette version :

1. Dans votre planification de basculement, vous devez également tenir compte du facteur temps. Par exemple, si vous perdez la connectivité pendant plus de 15 à 20 minutes, vous pouvez décider de lancer le basculement.

2. Le fait qu’aucune donnée ne soit répliquée signifie que les sessions actuellement actives ne sont pas répliquées. En outre, la détection des doublons et les messages planifiés peuvent ne pas fonctionner. Les nouvelles sessions, les nouveaux messages planifiés et les nouveaux doublons fonctionneront. 

3. Le basculement d’une infrastructure distribuée complexe doit être [répétée](/azure/architecture/reliability/disaster-recovery#disaster-recovery-plan) au moins une fois.

4. La synchronisation des entités peut prendre un certain temps, à raison d’environ 50 à 100 entités par minute. Les abonnements et les règles comptent également comme des entités.

## <a name="availability-zones"></a>Zones de disponibilité

La référence SKU de Service Bus Premium prend également en charge les [zones de disponibilité](../availability-zones/az-overview.md), fournissant des emplacements isolés des défaillances au sein d’une région Azure.

> [!NOTE]
> Dans Azure Service Bus Premium, la prise en charge des zones de disponibilité est fournie uniquement pour les [régions Azure](../availability-zones/az-region.md) qui comprennent déjà des zones de disponibilité.

Vous pouvez activer les Zones de disponibilité sur les nouveaux espaces de noms uniquement, à l’aide du portail Azure. Service Bus ne prend pas en charge la migration des espaces de noms existants. Vous ne pouvez pas désactiver la redondance de zone après l’avoir activée sur votre espace de noms.

![3][]

## <a name="private-endpoints"></a>Instances Private Endpoint
Cette section fournit des informations supplémentaires concernant l’utilisation de la géo-reprise d’activité après sinistre avec des espaces de noms qui utilisent des points de terminaison privés. Pour en savoir plus sur l’utilisation des points de terminaison privés avec Service Bus en général, consultez [Intégrer Azure Service Bus à Azure Private Link](private-link-service.md).

### <a name="new-pairings"></a>Nouveaux pairages
Si vous tentez de créer un pairage entre un espace de noms principal avec un point de terminaison privé et un espace de noms secondaire sans point de terminaison privé, le pairage échoue. Le pairage ne fonctionnera que si les espaces de noms principal et secondaire ont des points de terminaison privés. Nous vous recommandons d’utiliser les mêmes configurations sur les espaces de noms principal et secondaire, ainsi que sur les réseaux virtuels dans lesquels des points de terminaison privés sont créés. 

> [!NOTE]
> Lorsque vous tentez d’appairer l’espace de noms principal avec un point de terminaison privé et l’espace de noms secondaire, le processus de validation vérifie uniquement s’il existe un point de terminaison privé sur l’espace de noms secondaire. Il ne vérifie pas si le point de terminaison fonctionne ou fonctionnera après le basculement. Il vous incombe de vous assurer que l’espace de noms secondaire avec un point de terminaison privé fonctionnera comme prévu après le basculement.
>
> Pour vérifier que les configurations de point de terminaison privé sont identiques, envoyez une requête [Get queues](/rest/api/servicebus/stable/queues/get) à l’espace de noms secondaire depuis l’extérieur du réseau virtuel et vérifiez que vous recevez un message d’erreur de la part du service.

### <a name="existing-pairings"></a>Pairages existants
Si un pairage entre les espaces de noms principal et secondaire existe déjà, la création d’un point de terminaison privé sur l’espace de noms principal échoue. Pour résoudre ce problème, commencez par créer un point de terminaison privé sur l’espace de noms secondaire, puis créez-en un pour l’espace de noms principal.

> [!NOTE]
> Bien que nous autorisions un accès en lecture seule à l’espace de noms secondaire, les mises à jour des configurations de point de terminaison privé sont autorisées. 

### <a name="recommended-configuration"></a>Configuration recommandée
Lorsque vous créez une configuration de récupération d’urgence pour votre application et Service Bus, vous devez créer des points de terminaison privés pour les espaces de noms Service Bus principal et secondaire sur les réseaux virtuels hébergeant des instances principales et secondaires de votre application.

Supposons que vous disposiez de deux réseaux virtuels, VNET-1 et VNET-2, et d’espaces de noms principal et secondaire, ServiceBus-Namespace1-Primary et ServiceBus-Namespace2-Secondary. Vous devez procéder comme suit : 

- Sur ServiceBus-Namespace1-Primary, créez deux points de terminaison privés qui utilisent des sous-réseaux de VNET-1 et VNET-2.
- Sur ServiceBus-Namespace2-Secondary, créez deux points de terminaison privés qui utilisent les mêmes sous-réseaux de VNET-1 et VNET-2. 

![Points de terminaison privés et réseaux virtuels](./media/service-bus-geo-dr/private-endpoints-virtual-networks.png)


L’avantage de cette approche est que le basculement peut se produire au niveau de la couche Application, indépendamment de l’espace de noms Service Bus. Examinez les scénarios suivants : 

**Basculement de l’application uniquement :** Ici, l’application n’existera pas dans VNET-1, mais passera à VNET-2. Comme les deux points de terminaison privés sont configurés sur VNET-1 et VNET-2 pour les espaces de noms principal et secondaire, l’application fonctionnera normalement. 

**Basculement de l’espace de noms Service Bus uniquement :** Ici encore, étant donné que les deux points de terminaison privés sont configurés sur les deux réseaux virtuels pour les espaces de noms principal et secondaire, l’application fonctionnera normalement. 

> [!NOTE]
> Pour obtenir des conseils sur la géo-reprise d’activité après sinistre d’un réseau virtuel, consultez [Réseau virtuel : Continuité d’activité](../virtual-network/virtual-network-disaster-recovery-guidance.md).

## <a name="next-steps"></a>Étapes suivantes

- Consultez les [informations de référence de l’API REST](/rest/api/servicebus/stable/disasterrecoveryconfigs) sur la géorécupération d’urgence.
- Exécutez [l’exemple de géorécupération d’urgence sur GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/SBGeoDR2/SBGeoDR2).
- Consultez [l’exemple d’envoi de messages à un alias](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoDR/TestGeoDR/ConsoleApp1) pour la géorécupération d’urgence.

Pour en savoir plus sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
* [API REST](/rest/api/servicebus/) 

[1]: ./media/service-bus-geo-dr/geodr_setup_pairing.png
[2]: ./media/service-bus-geo-dr/geo2.png
[3]: ./media/service-bus-geo-dr/az.png
[4]: ./media/service-bus-geo-dr/geodr_failover_alias_update.png

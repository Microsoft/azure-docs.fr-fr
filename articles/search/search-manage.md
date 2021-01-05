---
title: Administration des services sur le portail
titleSuffix: Azure Cognitive Search
description: Gérez Recherche cognitive Azure, un service de recherche cloud hébergé sur Microsoft Azure, à l'aide du portail Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
tags: azure-portal
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/24/2020
ms.openlocfilehash: 84ddc4b427f6dc168c044f34b41e81e3b0ff19e5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935039"
---
# <a name="service-administration-for-azure-cognitive-search-in-the-azure-portal"></a>Administration des services pour Recherche cognitive Azure sur le portail Azure

> [!div class="op_single_selector"]
>
> * [PowerShell](search-manage-powershell.md)
> * [REST API](/rest/api/searchmanagement/)
> * [Kit de développement logiciel (SDK) .NET](/dotnet/api/microsoft.azure.management.search)
> * [Portail](search-manage.md)
> * [Python](https://pypi.python.org/pypi/azure-mgmt-search/0.1.0)> 

Recherche cognitive Azure est un service de recherche entièrement géré, basé sur le cloud, utilisé pour la création d’une expérience de recherche riche dans des applications personnalisées. Cet article aborde les tâches d’administration des services que vous pouvez effectuer dans le [portail Azure](https://portal.azure.com) pour un service de recherche que vous avez déjà provisionné. L’administration des services est légère de par sa conception et se limite aux tâches suivantes :

* Vérifiez le stockage à l’aide du lien **Utilisation** en milieu de page.
* Vérifiez les volumes et la latence des requêtes à l’aide du lien **Surveillance** en milieu de page et vérifiez si les requêtes ont été limitées.
* Gérez l’accès à l’aide de la page **Clés** sur la gauche.
* Ajustez la capacité à l’aide de la page **Mettre à l’échelle** sur la gauche.

Les mêmes tâches effectuées dans le portail peuvent également être gérées par programmation par le biais des [API de gestion](/rest/api/searchmanagement/) et du [module AZ. Search PowerShell](search-manage-powershell.md). Les tâches d’administration sont entièrement représentées sur le portail et les interfaces de programmation. Aucune tâche d’administration spécifique n’est disponible que dans une seule modalité.

Recherche cognitive Azure tire parti d’autres services Azure pour effectuer une surveillance et une gestion plus poussées. Les seules données stockées avec un service de recherche sont le contenu (index, indexeur et définitions de sources de données et autres objets). Les mesures exposées sur les pages du portail sont tirées de journaux internes sur un cycle de 30 jours continu. Pour la rétention des journaux et des événements supplémentaires contrôlés par l’utilisateur, vous devez disposer d’[Azure Monitor](../azure-monitor/index.yml). 

## <a name="fixed-service-properties"></a>Propriétés du service fixe

Plusieurs aspects d’un service de recherche sont déterminés lorsque le service est approvisionné et ne peuvent pas être modifiés ultérieurement :

* Nom du service (vous ne pouvez pas renommer un service)
* Emplacement du service (vous ne pouvez pas déplacer actuellement un service intact vers une autre région)
* Nombre maximal de réplicas et de partitions (déterminé par le niveau, De base ou Standard)

Si vous avez démarré avec le niveau De base et le nombre maximal de partitions associé, et qu’il vous faut plus de partitions, vous devez [créer un service](search-create-service-portal.md) à un niveau supérieur et recréer votre contenu sur le nouveau service. 

## <a name="administrator-rights"></a>Droits d’administrateur

L’approvisionnement ou le retrait du service lui-même peut être effectué par un administrateur d’abonnement Azure ou un coadministrateur.

En ce qui concerne l’accès au point de terminaison, toute personne ayant accès à l’URL du service et une clé API a accès au contenu. Pour plus d’informations sur les clés, consultez la page [Gestion des clés API](search-security-api-keys.md).

* L’accès en lecture seule au service est un droit de requête, généralement accordé à une application cliente en lui fournissant l’URL et une clé API de requête.
* L’accès en lecture-écriture permet d’ajouter, de supprimer ou de modifier des objets serveur, notamment des clés API, des index, des indexeurs, des sources de données et des planifications. L’accès en lecture-écriture est accordé par la fourniture de l’URL, d’une clé API administrateur.

Les droits d’accès à l’appareil d’approvisionnement de services sont accordés par le biais d’attributions de rôles. Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) est un système d’autorisation basé sur [Azure Resource Manager](../azure-resource-manager/management/overview.md) pour l’approvisionnement de ressources Azure. 

Dans le cadre de Recherche cognitive Azure, les [attributions de rôles Azure](search-security-rbac.md) déterminent les personnes qui peut effectuer les tâches, qu’elles utilisent le [portail](search-manage.md), [PowerShell](search-manage-powershell.md) ou les [API REST de gestion](/rest/api/searchmanagement/search-howto-management-rest-api) :

* Créer ou supprimer un service
* Mettre à l’échelle le service
* Supprimer ou régénérer les clés API
* Activer la journalisation des diagnostics (créer des services)
* Activer Traffic Analytics (créer des services)

> [!TIP]
> En utilisant des mécanismes à l’échelle d’Azure, vous pouvez verrouiller un abonnement ou une ressource pour empêcher la suppression accidentelle ou non autorisée de votre service de recherche par les utilisateurs disposant de droits d’administration. Pour plus d’informations, consultez [Verrouiller les ressources pour en empêcher la suppression](../azure-resource-manager/management/lock-resources.md).

## <a name="logging-and-system-information"></a>Journalisation et informations système

Au niveau de base et supérieur, Microsoft surveille tous les services Recherche cognitive Azure pour vérifier la disponibilité de 99,9 % par contrat de niveau de service (SLA). Si le service est lent ou si le débit des demandes tombe en dessous des seuils de contrat SLA, les équipes de support passent en revue les fichiers journaux à leur disposition et résolvent le problème.

Recherche cognitive Azure tire parti d’[Azure Monitor](../azure-monitor/index.yml) pour collecter et stocker les activités d’indexation et de requête. Un service de recherche en lui-même stocke uniquement son contenu (index, définitions d’indexeurs, définitions de sources de données, définitions d’ensembles de compétences, cartes de synonymes). Les informations de mise en cache et journalisées sont stockées hors service, souvent dans un compte de stockage Azure. Pour plus d’informations sur la journalisation des charges de travail d’indexation et de requête, consultez [Collecter et analyser les données de journal](search-monitor-logs.md).

En matière d’informations générales sur votre service, en utilisant seulement les fonctionnalités intégrées à Recherche cognitive Azure lui-même, vous permet d’obtenir des informations des manières suivantes :

* À l’aide du service depuis la page **Présentation**, via les notifications, les propriétés et les messages d’état.
* À l’aide de [PowerShell](search-manage-powershell.md) ou de l’[API REST de gestion](/rest/api/searchmanagement/) pour [obtenir les propriétés du service](/rest/api/searchmanagement/services). Aucune nouvelle information ou opération n’est fournie au niveau de la couche de programmation. Des interfaces vous permettent d’écrire des scripts.

## <a name="monitor-resource-usage"></a>surveiller l’utilisation des ressources ;

Sur le tableau de bord, l’analyse des ressources se limite aux informations affichées dans le tableau de bord des services et à quelques mesures que vous pouvez obtenir en interrogeant le service. Dans la section Utilisation du tableau de bord des services, vous pouvez déterminer rapidement si les niveaux des ressources de partition sont adaptés à votre application. Vous pouvez provisionner des ressources externes, telles que la supervision Azure, si vous souhaitez capturer et conserver les événements enregistrés. Pour plus d’informations, consultez [Supervision de Recherche cognitive Azure](search-monitor-usage.md).

L’API REST du service de recherche vous permet d’obtenir le nombre de documents et d’index par programmation : 

* [Obtention de statistiques d’index](/rest/api/searchservice/Get-Index-Statistics)
* [Nombre de documents](/rest/api/searchservice/count-documents)

## <a name="disaster-recovery-and-service-outages"></a>Récupération d’urgence et pannes de service

Bien que nous puissions récupérer vos données, Recherche cognitive Azure ne fournit pas de basculement instantané du service en cas de panne au niveau du centre de données ou du cluster. Si un cluster tombe en panne dans le centre de données, l’équipe d’exploitation le détecte et tente de restaurer le service. Vous rencontrerez un temps d’arrêt pendant la restauration de service, mais vous pouvez demander des crédits de service afin de compenser une indisponibilité du service conformément au [Contrat de niveau de service (SLA)](https://azure.microsoft.com/support/legal/sla/search/v1_0/). 

Si le service ne doit pas être interrompu même en cas de défaillances catastrophiques qui échappent au contrôle de Microsoft, vous pouvez [approvisionner un service supplémentaire](search-create-service-portal.md) dans une autre région et mettre en œuvre une stratégie de géoréplication pour assurer une redondance complète des index sur tous les services.

Les clients qui utilisent des [indexeurs](search-indexer-overview.md) pour remplir et actualiser les index peuvent gérer la récupération d’urgence par le biais d’indexeurs propres à la région qui exploitent la même source de données. Deux services situés dans des régions différentes, chacun exécutant un indexeur, peuvent indexer la même source de données pour bénéficier de la géoredondance. Si vous indexez à partir de sources de données qui sont aussi géoredondantes, sachez que les indexeurs de Recherche cognitive Azure ne peuvent assurer qu’une indexation incrémentielle (en fusionnant les mises à jour de documents nouveaux, modifiés ou supprimés) à partir de réplicas principaux. À l’occasion d’un basculement, veillez à refaire pointer l’indexeur vers le nouveau réplica principal. 

Si vous n’utilisez pas d’indexeurs, vous devez utiliser le code de votre application pour effectuer une transmission de type push des objets et des données vers différents services de recherche en parallèle. Pour plus d’informations, consultez [Performance and optimization in Azure Search](search-performance-optimization.md)(Performances et optimisation dans Recherche cognitive Azure).

## <a name="backup-and-restore"></a>Sauvegarde et restauration

Recherche cognitive Azure n’est pas une solution de stockage de données principal, c’est pourquoi nous ne fournissons pas de mécanisme formel de sauvegarde et de restauration en libre-service. Toutefois, vous pouvez utiliser l’exemple de code **index-backup-restore** dans cet [exemple de dépôt .NET Recherche cognitive Azure](https://github.com/Azure-Samples/azure-search-dotnet-samples) pour sauvegarder la définition et l’instantané d’un index dans une série de fichiers JSON, puis utiliser ces fichiers pour restaurer l’index, si nécessaire. Cet outil peut également déplacer des index entre les niveaux de service.

Sinon, le code de votre application utilisé pour créer et remplir un index est l’option de restauration de facto si vous supprimez un index par erreur. Pour reconstruire un index, vous devez le supprimer (s’il existe), recréer l’index dans le service et le recharger en récupérant les données à partir de votre banque de données principale.

## <a name="scale-up-or-down"></a>Augmentation ou réduction d'échelle

Au départ, chaque service de recherche comporte, au minimum, un réplica et une partition. Si vous vous êtes inscrit à un [niveau qui prend en charge plus de capacités](search-limits-quotas-capacity.md), cliquez sur **Mettre à l’échelle** dans le volet de navigation gauche pour ajuster l’utilisation des ressources.

Lorsque vous ajoutez des capacités à travers l’une des ressources, le service les utilise automatiquement. Aucune autre action n’est nécessaire de votre part. Cependant, un léger décalage est à prévoir avant que l’effet de la nouvelle ressource soit perceptible. L’approvisionnement des ressources supplémentaires demande au moins 15 minutes.

### <a name="add-replicas"></a>Ajout de réplicas

Pour augmenter le nombre de requêtes par seconde (RPS) ou parvenir à une haute disponibilité, il convient d'ajouter des réplicas. Chaque réplica comporte une copie d’un index. L’ajout d’un réplica se traduit donc par un ou plusieurs index supplémentaires disponibles pour satisfaire les demandes de requête. Au moins 3 réplicas sont nécessaires pour la haute disponibilité (pour plus d’informations, consultez [Régler la capacité](search-capacity-planning.md)).

Un service de recherche qui comporte davantage de réplicas peut équilibrer la charge des demandes de requête sur un plus grand nombre d'index. Pour un nombre de requêtes donné, le débit sera plus élevé si davantage de copies de l'index sont disponibles pour leur traitement. Si vous constatez une latence des requêtes, la mise en ligne des répliques supplémentaires aura, à n'en pas douter, un effet positif sur les performances.

Bien que l'ajout de réplicas à votre service entraîne une augmentation du débit des requêtes, celui-ci n'est pas exactement multiplié par deux ou par trois. Toutes les applications de recherche sont, en effet, soumises à des facteurs externes susceptibles d'affecter les performances des requêtes. La complexité des requêtes et la latence réseau, notamment, sont deux facteurs qui contribuent à la fluctuation des temps de réponse des requêtes.

### <a name="add-partitions"></a>Ajout de partitions

Il est plus courant d’ajouter des réplicas, mais lorsque le stockage est restreint, vous pouvez ajouter des partitions pour obtenir plus de capacité. La possibilité d’ajouter ou non des partitions dépend du niveau auquel vous avez approvisionné le service. Le niveau De base est verrouillé sur une partition. Les niveaux Standard et supérieurs prennent en charge des partitions supplémentaires.

Les partitions sont ajoutées par diviseurs de 12 (à savoir, 1, 2, 3, 4, 6 ou 12). Il s’agit d’un artefact de partitionnement. Un index est créé dans 12 fragments (ou shards) qui peuvent tous être stockés dans 1 partition ou répartis équitablement dans 2, 3, 4, 6 ou 12 partitions (un fragment par partition).

### <a name="remove-replicas"></a>Suppression de réplicas

Après une période de traitement de requêtes intensive, vous pouvez utiliser le curseur pour réduire le nombre de réplicas une fois la charge de requêtes de recherche revenue à la normale (à la fin d’une période de soldes, par exemple). Rien de plus ! La réduction du nombre de réplicas entraîne l'abandon des machines virtuelles dans le centre de données. Désormais, vos opérations de requête et d'ingestion de données s'exécuteront sur un nombre moins élevé de machines virtuelles. La configuration minimale nécessite un réplica.

### <a name="remove-partitions"></a>Suppression de partitions

Contrairement à la suppression de réplicas, qui n'exige aucune opération supplémentaire de votre part, utiliser plus de volume de stockage que la capacité disponible après réduction peut entraîner une surcharge de travail. Par exemple, si votre solution utilise trois partitions, le fait de passer à une ou deux partitions générera une erreur si le nouvel espace de stockage est inférieur à celui nécessaire pour héberger votre index. Comme vous pouvez vous y attendre, deux solutions s’offrent alors à vous : soit supprimer des index ou documents au sein d’un index associé afin de libérer de l’espace, soit conserver la configuration actuelle.

Aucune méthode de détection ne vous permet d'identifier les fragments d'index qui sont stockés sur des partitions spécifiques. Chaque partition fournit environ 25 Go de stockage. Vous devrez donc réduire l'espace de stockage à une taille pouvant être prise en charge par le nombre de partitions dont vous disposez. Si vous souhaitez revenir à une seule partition, celle-ci devra contenir les 12 fragments.

Pour faciliter la planification, vous pouvez vérifier le stockage (voir la page [Obtenir des statistiques d'index](/rest/api/searchservice/Get-Index-Statistics)) afin de connaître l'espace réellement utilisé. 

## <a name="next-steps"></a>Étapes suivantes

* Automatiser avec [PowerShell](search-manage-powershell.md)

* Réviser le [niveau de performance et les techniques d’optimisation](search-performance-optimization.md)

* Réviser les [fonctionnalité de sécurité](search-security-overview.md) pour protéger le contenu et les opérations

* Activer la [journalisation des diagnostics](search-monitor-logs.md) pour surveiller les charges de travail de requête et d’indexation
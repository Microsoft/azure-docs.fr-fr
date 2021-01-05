---
title: Sélectionnez un niveau tarifaire
titleSuffix: Azure Cognitive Search
description: 'Le service Recherche cognitive Azure peut être approvisionné aux niveaux suivants : Gratuit, De base et Standard, sachant que la référence Standard est disponible en différentes configurations de ressources et différents niveaux de capacité.'
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/15/2020
ms.custom: contperf-fy21q2
ms.openlocfilehash: 062bd41b0803cbb08f74fbcbcebb89bbddeb0d45
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97559801"
---
# <a name="choose-a-pricing-tier-for-azure-cognitive-search"></a>Choisir un niveau tarifaire pour Recherche cognitive Azure

Lorsque vous [créez un service de recherche](search-create-service-portal.md), vous choisissez un niveau tarifaire qui est fixé pour toute la durée de vie du service. Le niveau que vous sélectionnez détermine ce qui suit :

+ Quantité d’index et d’autres objets que vous pouvez créer (limites maximales)
+ Taille et la vitesse des partitions (stockage physique)
+ Taux facturable, coût mensuel fixe, mais également coût incrémentiel si vous ajoutez des partitions ou des réplicas

En outre, quelques [fonctionnalités Premium](#premium-features) sont fournies avec les exigences de niveau.

## <a name="tier-descriptions"></a>Descriptions des niveaux

Les niveaux disponibles sont : **Gratuit**, **De base**, **Standard** et **À stockage optimisé**. Les niveaux Standard et À stockage optimisé sont proposés dans diverses configurations et capacités.

La capture d’écran suivante du portail Azure indique les niveaux disponibles, après la déduction indiquée sur le portail et dans la [page de tarification](https://azure.microsoft.com/pricing/details/search/). 

![Niveaux tarifaires de Recherche cognitive Azure](media/search-sku-tier/tiers.png "Niveaux tarifaires de Recherche cognitive Azure")

Le niveau **Gratuit** crée un service de recherche limité pour les projets plus petits, notamment les tutoriels et les exemples de code. En interne, les réplicas et les partitions sont partagées entre plusieurs abonnés. Vous ne pouvez pas mettre à l’échelle un service gratuit ni exécuter des charges de travail importantes.

Les niveau **De base** et **Standard** sont les niveaux facturables les plus utilisés, **Standard** étant le niveau par défaut. Grâce à des ressources dédiées sous votre contrôle, vous pouvez déployer des projets plus volumineux, optimiser les performances et augmenter la capacité.

Certains niveaux sont optimisés pour certains types de travaux. Par exemple, le niveau **Standard 3 High Density (S3 HD)** est un *mode d’hébergement* pour S3, où le matériel sous-jacent est optimisé pour un grand nombre d’index plus petits, qui est destiné aux scénarios d’architecture mutualisée. Le niveau S3 HD présente les mêmes frais à l’unité que S3, mais le matériel est optimisé pour les lectures de fichiers rapides sur un grand nombre d’index plus petits.

Les niveaux **À stockage optimisé** offrent une capacité de stockage plus importante et à moindre coût par To que les niveaux Standard. Le principal compromis impliqué par ces niveaux réside dans une latence de requête plus élevée, ce que vous devez valider pour vos exigences applicatives spécifiques. Pour plus d’informations sur les considérations en matière de performances de ce niveau, consultez l’article [Considérations sur les performances et l’optimisation](search-performance-optimization.md).

Des informations complémentaires sur les différents niveaux sont disponibles sur la [page de tarification](https://azure.microsoft.com/pricing/details/search/), dans l’article [Service limits in Azure Search](search-limits-quotas-capacity.md) (Limites du service Recherche cognitive Azure), ainsi que sur la page du portail lorsque vous approvisionnez un service.

<a name="premium-features"></a>

## <a name="feature-availability-by-tier"></a>Disponibilité des fonctionnalités par niveau

La plupart des fonctionnalités sont disponibles dans tous les niveaux, y compris le niveau gratuit. Dans certains cas, le niveau que vous choisissez aura un impact sur votre capacité à implémenter une fonctionnalité. Le tableau suivant décrit les contraintes de fonctionnalités liées au niveau de service.

| Fonctionnalité | Limites |
|---------|-------------|
| [indexeurs](search-indexer-overview.md) | Les indexeurs ne sont pas disponibles sur S3 HD.  |
| [Enrichissement par IA](search-security-manage-encryption-keys.md) | Fonctionne au niveau Gratuit, mais n’est pas recommandé. |
| [Clés de chiffrement gérées par le client](search-security-manage-encryption-keys.md) | Non disponibles au niveau Gratuit. |
| [Accès au pare-feu IP](service-configure-firewall.md) | Non disponibles au niveau Gratuit. |
| [Point de terminaison privé (intégration à Azure Private Link)](service-create-private-endpoint.md) | Pour les connexions entrantes à un service de recherche ; non disponible au niveau Gratuit. Pour les connexions sortantes, par des indexeurs, à d'autres ressources Azure ; non disponible aux niveaux Gratuit et S3 HD. Pour les indexeurs qui utilisent des ensembles de compétences ; non disponible aux niveaux Gratuit, De base, S1 et S3 HD.|

Les fonctionnalités gourmandes en ressources risquent de ne pas fonctionner correctement si vous ne leur attribuez pas une capacité suffisante. Par exemple, [l’enrichissement par IA](cognitive-search-concept-intro.md) implique des qualifications à long terme qui dépassent le délai d’attente sur un service Gratuit, sauf si le jeu de données est restreint.

## <a name="billable-events"></a>Événements facturables

Une solution reposant sur Recherche cognitive Azure peut occasionner des coûts de l’une des manières suivantes :

+ [Coût du service](#service-costs) proprement dit, exécuté 24 h sur 24 et 7 j sur 7 avec une configuration minimale (une partition et un réplica), au taux de base. Vous pouvez y penser comme au coût fixe de fonctionnement du service.

+ Ajout de capacité (réplicas ou partitions), où les coûts augmentent par incréments du taux facturable

+ Frais de bande passante (transfert de données sortant)

+ Services complémentaires requis pour des capacités ou fonctionnalités spécifiques :

  + enrichissement par IA (nécessite [Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/))
  + base de connaissances (nécessite [Stockage Azure](https://azure.microsoft.com/pricing/details/storage/))
  + enrichissement incrémentiel (nécessite [Stockage Azure](https://azure.microsoft.com/pricing/details/storage/), s’applique à l’enrichissement par IA)
  + clés gérées par le client et double chiffrement (nécessite [Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/))
  + points de terminaison privés pour un modèle sans accès à Internet (nécessite [Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/))

### <a name="service-costs"></a>Coûts de service

Contrairement à des machines virtuelles ou à d’autres ressources qui peuvent être « mises en pause » pour éviter des frais, un service Recherche cognitive Azure est toujours disponible sur du matériel dédié à votre usage exclusif. En tant que telle, la création d’un service est un événement facturable qui commence lorsque vous créez le service et se termine lorsque vous le supprimez. 

Le coût minimal est la première unité de recherche (une réplique x une partition) au tarif facturable. Ces frais minimaux sont fixes pendant toute la durée de vie du service, car il est impossible d’exécuter le service sur une configuration inférieure à cette dernière. Au-delà des frais minimaux, vous pouvez ajouter des réplicas et des partitions indépendants les uns des autres. Les augmentations incrémentielles de capacité via des réplicas et partitions augmentent votre facture selon la formule suivante : [ (réplicas x partitions x tarif) ](#search-units), où le tarif facturé dépendant du niveau de tarification sélectionné.

Lorsque vous estimez le coût d’une solution de recherche, gardez à l’esprit que la tarification et la capacité ne sont pas linéaires. (Par exemple, la multiplication par deux de la capacité coûte plus du double.) Pour découvrir un exemple du mode de fonctionnement de la formule, consultez la section [How to allocate replicas and partitions](search-capacity-planning.md#how-to-allocate-replicas-and-partitions) (Allocation de réplicas et de partitions).

### <a name="bandwidth-charges"></a>Frais liés à la bande passante

L’utilisation d’[indexeurs](search-indexer-overview.md) peut avoir un impact sur la facturation si la source de données Azure se trouve dans une région différente de celle de Recherche cognitive Azure. Dans ce scénario, il s’agit d’un coût pour le déplacement des données sortantes de la source de données Azure vers Recherche cognitive Azure. 

Vous pouvez éliminer totalement les frais de sortie de données si vous créez le service Recherche cognitive Azure dans la même région que vos données. Voici quelques informations issues de la [page de tarification de la bande passante](https://azure.microsoft.com/pricing/details/bandwidth/) :

+ Microsoft ne facture aucune donnée entrante dans un quelconque service Azure.
+ Aucune donnée sortante provenant de Recherche cognitive Azure n’est facturée. Par exemple, si votre service de recherche se trouve dans la région USA Ouest et qu’une application web Azure se trouve dans la région USA Est, Microsoft ne facture pas les charges utiles des réponses aux requêtes qui proviennent de la région USA Ouest.
+ Dans les solutions multiservice, il n’existe aucuns frais pour les données qui transitent par le réseau lorsque tous les services se trouvent dans la même région.

Des frais s’appliquent pour les données sortantes si les services sont situés dans des régions différentes. Ces frais ne figurent pas sur votre facture Recherche cognitive Azure. Ils sont mentionnés ici, car si vous utilisez des données ou des indexeurs IA pour extraire les données de différentes régions, les coûts associés apparaîtront sur votre facture globale.

### <a name="ai-enrichment-with-cognitive-services"></a>Enrichissement de l’IA avec Cognitive Services

Pour [l’enrichissement de l’IA](cognitive-search-concept-intro.md), prévoyez d’[attacher une ressource Azure Cognitive Services facturable](cognitive-search-attach-cognitive-services.md) dans la même région que le service Recherche cognitive Azure, au niveau tarifaire S0 pour le traitement du paiement à l’utilisation. L’attachement de Cognitive Services n’entraîne aucun coût fixe. Vous payez uniquement pour le traitement dont vous avez besoin.

| Opération | Impact sur la facturation |
|-----------|----------------|
| Craquage de document, extraction de texte | Gratuit |
| Craquage de document, extraction d’image | Facturée en fonction du nombre d’images extraites de vos documents. Dans une [configuration d’indexeur](/rest/api/searchservice/create-indexer#indexer-parameters), **imageAction** est le paramètre qui déclenche l’extraction d’images. Si **imageAction** est défini sur « none » (valeur par défaut), l’extraction d’images ne vous sera pas facturée. Le tarif de l’extraction d’image est mentionné sur la page des [détails de tarification](https://azure.microsoft.com/pricing/details/search/) du service Recherche cognitive Azure.|
| [Compétences cognitives prédéfinies](cognitive-search-predefined-skills.md) | Facturées au même tarif que si vous aviez exécuté la tâche directement avec Cognitive Services. |
| Compétences personnalisées | Une compétence personnalisée est une fonctionnalité que vous fournissez. Le coût d’utilisation d’une compétence personnalisée dépend entièrement du fait qu’un code personnalisé appelle d’autres services mesurés. |

La fonctionnalité [d’enrichissement incrémentiel](cognitive-search-incremental-indexing-conceptual.md) (préversion) permet de fournir un cache qui augmente l’efficacité de l’indexeur en exécutant uniquement les compétences cognitives nécessaires en cas de modification ultérieure de l’ensemble de compétences, ce qui représente un gain de temps et d’argent.

<a name="search-units"></a>

## <a name="billing-formula-r-x-p--su"></a>Formule de facturation (R x P = SU)

La notion de facturation la plus importante à comprendre pour les opérations du service Recherche cognitive Azure est *l’Unité Recherche* (SU, Search Unit). Comme Recherche cognitive Azure dépend à la fois des réplicas et des partitions pour l’indexation et les requêtes, facturer uniquement par rapport à l’un ou à l’autre n’aurait aucun sens. Au lieu de cela, la facturation est basée sur un composite de ces deux facteurs.

L’Unité Recherche est le produit des *réplicas* et des *partitions* utilisés par un service : **(R x P = SU)** .

Chaque service commence avec une Unité Recherche (un réplica multiplié par une partition) au minimum. Le montant maximal de tout service est de 36 SU. Ce montant maximal peut être atteint de plusieurs façons : 6 partitions x 6 réplicas, ou 3 partitions x 12 réplicas, par exemple. La capacité utilisée est généralement inférieure à la capacité totale (par exemple, un service de 3 réplicas et 3 partitions facturé avec un montant de 9 SU). Pour connaître les combinaisons valides, consultez l’article [Partition and replica combinations](search-capacity-planning.md#chart) (Combinaisons de partitions et de réplicas).

Le tarif de facturation est un tarif horaire par SU. Le tarif augmente progressivement à chaque niveau. Les niveaux supérieurs proposent des partitions plus volumineuses et plus rapides, ce qui entraîne un tarif horaire globalement plus élevé pour ces niveaux. Vous pouvez consulter les tarifs de chaque niveau sur la page des [détails de tarification](https://azure.microsoft.com/pricing/details/search/).

La plupart des clients mettent seulement une partie de la capacité totale en ligne et gardent le reste en réserve. Pour la facturation, le nombre de partitions et de réplicas que vous mettez en ligne, calculé à l’aide de la formule SU, détermine le tarif horaire qui vous est facturé.

## <a name="next-steps"></a>Étapes suivantes

La gestion des coûts fait partie intégrante de la planification de la capacité. À l’étape suivante, poursuivez avec l’article suivant pour obtenir des conseils sur la façon d’estimer la capacité et de gérer les coûts.

> [!div class="nextstepaction"]
> [Gestion des coûts et estimation de la capacité dans Recherche cognitive Azure](search-sku-manage-costs.md)
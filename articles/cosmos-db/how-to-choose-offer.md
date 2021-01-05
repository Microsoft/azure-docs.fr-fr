---
title: Comment choisir entre mise à l’échelle manuelle et automatique sur Azure Cosmos DB
description: Découvrez comment choisir entre un débit approvisionné standard (manuel) et un débit approvisionné avec mise à l’échelle automatique pour votre charge de travail.
author: deborahc
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.author: dech
ms.openlocfilehash: d8a6471d53ad4b2428504f9c53cbec6bc1967c49
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93089636"
---
# <a name="how-to-choose-between-standard-manual-and-autoscale-provisioned-throughput"></a>Comment choisir entre le débit approvisionné standard (manuel) et le débit approvisionné avec mise à l’échelle automatique 
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB prend en charge deux types ou offres de débit approvisionné : standard (manuel) et avec mise à l’échelle automatique. Ces deux types de débit sont adaptés aux charges de travail stratégiques qui requièrent de hautes performances et à grande échelle, et s’appuient les mêmes contrats SLA Azure Cosmos DB pour le débit, la disponibilité, la latence et la cohérence.

Cet article décrit comment choisir entre le débit approvisionné standard (manuel) et celui avec mise à l’échelle automatique pour votre charge de travail. 

## <a name="overview-of-provisioned-throughput-types"></a>Vue d’ensemble des types de débits approvisionnés
Avant de se pencher sur la différence entre l’approvisionnement standard (manuel) et avec mise à l’échelle automatique, il est important de commencer par comprendre le fonctionnement du débit approvisionné dans Azure Cosmos DB. 

Lorsque vous utilisez le débit approvisionné, vous définissez le débit, mesuré en unités de requête par seconde (RU/s) requis pour votre charge de travail. Le service offre la capacité nécessaire pour prendre en charge les exigences liées au débit. Les opérations de base de données sur le service, telles que les lectures, les écritures et les requêtes, consomment un certain nombre d’unités de requête (RU). En savoir plus sur les [unités de requête](request-units.md).

Le tableau suivant présente une comparaison générale entre l’approvisionnement standard (manuel) et avec mise à l’échelle automatique.

|Description|Standard (manuel)|Mise à l’échelle automatique|
|-------------|------|-------|
|Idéale pour|Charges de travail avec trafic stable ou prévisible|Charges de travail avec trafic variable ou imprévisible. Voir [cas d’usage de la mise à l’échelle automatique](provision-throughput-autoscale.md#use-cases-of-autoscale).|
|Fonctionnement|Vous configurez une quantité définie de RU/s `T` qui ne change pas avec le temps, sauf si vous la modifiez manuellement. Chaque seconde, vous pouvez utiliser jusqu’à `T` RU/s de débit. <br/><br/>Par exemple, si vous définissez 400 RU/s standard (manuel) , le débit ne dépassera pas 400 RU/s.|Choisissez la valeur la plus élevée ou le nombre maximum de RU/s `Tmax` que vous ne souhaitez pas que le système dépasse. Le système met automatiquement à l’échelle le débit `T` de manière à ce que `0.1* Tmax <= T <= Tmax`. <br/><br/>Par exemple, si vous définissez le nombre maximum de RU/s pour la mise échelle à 4 000 RU/s, le système procédera à la mise à l’échelle entre 400-4 000 RU/s.|
|Quand utiliser cette fonctionnalité ?|Vous souhaitez gérer manuellement votre capacité de débit (RU/s) et la mettre à l’échelle vous-même.<br/><br/>Vous faites une utilisation élevée et cohérente des RU/s approvisionnées. Sur l’ensemble des heures du mois, si vous configurez un nombre de RU/s approvisionnées `T` et que vous utilisez 66 % des heures du montant total ou plus, les RU/s approvisionnées standard (manuel) vous permettrons de faire des économies.<br/><br/>Ceci se base sur une comparaison entre un total défini sur `T` en mode standard (manuel) et la même quantité `Tmax` avec la mise à l’échelle automatique. |Vous souhaitez qu’Azure Cosmos DB gère la capacité de débit (RU/s) et la mise à l’échelle en fonction de l’utilisation.<br/><br/>L’utilisation des RU/s est variable ou difficile à prédire. Sur l’ensemble des heures du mois, si vous définissez le nombre maximal de RU/s avec mise à l’échelle automatique `Tmax` et que vous utilisez la totalité de la `Tmax` pour 66 % des heures ou moins, la mise à l’échelle automatique vous permettra de faire des économies.<br/><br/>Ceci se base sur une comparaison entre sur une comparaison entre un total défini sur `Tmax` avec la mise à l’échelle automatique et la même quantité `T` en mode débit standard (manuel).|
|Modèle de facturation|La facturation s’effectue à l’heure pour les RU/s approvisionnés, quel que soit le nombre d’unités de requêtes consommées.<br/><br/>Exemple : <li>Approvisionnement de 400 RU/s</li><li>Heure 1 : aucune requête</li><li>Heure 2 : requêtes valant 400 RU/s</li><br/><br/>Pour les heures 1 et 2, vous êtes facturé 400 RU/s pour les deux heures au [tarif standard (manuel)](https://azure.microsoft.com/pricing/details/cosmos-db/).|La facturation s’effectue à l’heure, pour les RU/s les plus élevées que le système a mis à l’échelle pendant l’heure. <br/><br/>Exemple : <li>Approvisionnement avec mise à l’échelle automatique de 4 000 RU/s maximum (mise à l’échelle entre 400-4 000 RU/s)</li><li>Heure 1 : la système a été mis à l’échelle pour atteindre la valeur la plus élevée de 3 500 RU/s</li><li>Heure 2 : le système a effectué un scale-down pour atteindre un minimum de 400 RU/s (toujours 10 % de `Tmax`), à cause de l’absence d’utilisation</li><br/><br/>Vous serez facturé pour 3 500 RU/s pour l’heure 1 et 400 RU/s pour l’heure 2 avec le [débit approvisionné avec mise à l’échelle automatique](https://azure.microsoft.com/pricing/details/cosmos-db/). Le tarif pour la mise à l’échelle automatique par RU/s représente 1,5 * le tarif standard (manuel).
|Que se passe-t-il si vous dépassez les RU/s approvisionnées ?|Le nombre de RU/s reste statique par rapport à ce qui a été approvisionné. Toutes les requêtes qui consomment plus que les unités de requête approvisionnées par seconde feront l’objet d’une limitation de débit, avec une réponse qui recommande d’attendre avant d’effectuer une nouvelle tentative. Vous pouvez augmenter ou diminuer manuellement la taille les RU/s si besoin.| Le système met à l’échelle les RU/s jusqu’au nombre maximal de RU/s de la mise à l’échelle automatique. Toutes les requêtes qui consomment plus que les unités de requête approvisionnées avec mise à l’échelle automatique par seconde feront l’objet d’une limitation de débit, avec une réponse qui recommande d’attendre avant d’effectuer une nouvelle tentative.|

## <a name="understand-your-traffic-patterns"></a>Comprendre vos modèles de trafic

### <a name="new-applications"></a>Nouvelles applications ###

Si vous générez une nouvelle application et que vous ne connaissez pas encore votre modèle de trafic, vous pouvez commencer à partir du point d’entrée RU/s (ou RU/s minimum) pour éviter le surapprovisionnement au début. Autrement, si votre application est petite et n’a pas besoin d’une grande échelle, vous pourrez configurer uniquement le point d’entrée RU/s minimal pour optimiser les coûts. Pour les petites applications dont le trafic est faible, vous pouvez également envisager le mode de capacité [serverless](throughput-serverless.md).

Que vous prévoyiez d’utiliser la mise à l’échelle standard (manuelle) ou automatique, voici ce que vous devez prendre en compte :

Si vous configurez les RU/s standard (manuel) au point d’entrée de 400 RU/s, vous ne pourrez pas utiliser plus de 400 RU/s, sauf si vous modifiez le débit manuellement. Vous serez facturé pour 400 RU/s avec le débit approvisionné standard (manuel), à l’heure.

Si vous configurez le débit avec mise à l’échelle automatique au point d’entrée maximum de 4 000 RU/s, la ressource sera mise à l’échelle entre 400 et 4 000 RU/s. Étant donné que le taux de facturation du débit avec mise à l’échelle automatique par RU/s représente 1,5 x celui du débit standard (manuel), pour les heures où le système a effectué un scale-down jusqu’à la valeur minimale de 400 RU/s, votre facture sera plus élevée que si vous approvisionnez 400 RU/s manuellement. Toutefois, avec la mise à l’échelle automatique, à tout moment, en cas de pic du trafic de votre application, vous pouvez consommer jusqu’à 4 000 RU/s sans aucune intervention de l’utilisateur. De manière général, il vous faut évaluer les avantages présentés par le fait de pouvoir consommer des RU/s jusqu’à la limite maximale à tout moment grâce à la fréquence de 1,5 x la mise à l’échelle automatique.

Utilisez la [calculatrice de capacité](estimate-ru-with-capacity-planner.md) d’Azure Cosmos DB pour estimer vos besoins en débit. 

### <a name="existing-applications"></a>Applications existantes ###

Si vous disposez d’une application existante utilisant un débit approvisionné standard (manuel), vous pouvez utiliser les [métriques Azure Monitor](../azure-monitor/insights/cosmosdb-insights-overview.md) pour déterminer si votre modèle de trafic est adapté à la mise à l’échelle automatique. 

Tout d’abord, recherchez la [métrique de consommation d’unités de requête normalisées](monitor-normalized-request-units.md#view-the-normalized-request-unit-consumption-metric) de votre base de données ou de votre conteneur. L’utilisation normalisée mesure votre utilisation actuelle du débit approvisionné standard (manuel). Plus le nombre est proche de 100 %, plus l’usage que vous faites de vos RU/s approvisionnées est complet. [En savoir plus](monitor-normalized-request-units.md#view-the-normalized-request-unit-consumption-metric) sur la métrique.

Ensuite, déterminez comment l’utilisation normalisée va varier dans le temps. Recherchez l’utilisation normalisée la plus élevée pour chaque heure. Calculez ensuite l’utilisation normalisée moyenne sur toutes les heures. Si vous constatez que votre utilisation moyenne est inférieure à 66 %, songez à activer la mise à l’échelle automatique sur votre base de données ou conteneur. En revanche, si l’utilisation moyenne est supérieure à 66 %, il est recommandé de conserver le débit approvisionné standard (manuel).

> [!TIP]
> Si votre compte est configuré pour utiliser des écritures multirégions et qu’il a plus d’une région, le taux pour 100 RU/s est le même pour la mise à l’échelle manuelle et automatique. Cela signifie que l’activation de la mise à l’échelle automatique n’entraîne aucun coût supplémentaire quelle que soit l’utilisation. Par conséquent, il est toujours recommandé d’utiliser la mise à l’échelle automatique avec des écritures multirégions lorsque vous avez plus d’une région, afin de tirer parti des économies générées en ne payant que pour le taux de RU/s auquel votre application est mise à l’échelle. Si vous avez des écritures multirégions et une région, utilisez l’utilisation moyenne pour déterminer si la mise à l’échelle automatique génèrera des économies. 

#### <a name="examples"></a>Exemples

Observons deux exemples de charges de travail différents et analysons s’ils sont adaptés à la mise à l’échelle manuelle ou automatique du débit. Pour illustrer l’approche générale, nous allons analyser trois heures d’historique pour déterminer la différence de coût entre la mise à l’échelle manuelle et automatique. Pour les charges de travail de production, il est recommandé d’utiliser 7 à 30 jours d’historique (ou plus, le cas échéant) pour établir un modèle d’utilisation des RU/s.

> [!NOTE]
> Tous les exemples présentés dans ce document se basent sur le prix d’un compte Azure Cosmos déployé dans une région non gouvernementale aux États-Unis. Le tarif et le calcul varient en fonction de la région. Pour connaître les dernières informations tarifaires, consultez la [page des tarifs](https://azure.microsoft.com/pricing/details/cosmos-db/) Azure Cosmos DB.

Il est supposé que :
- Supposons que nous disposons actuellement d’un débit manuel de 30 000 RU/s. 
- Notre région est configurée avec des écritures à une seule région, avec une région. Si nous avions plusieurs régions, nous devrions multiplier le coût horaire par le nombre de régions.
- Utilisez les tarifs publics pour la mise à l’échelle manuelle (0,008 $ USD par 100 RU/s par heure) et la mise à l’échelle automatique (0,012 $ USD par 100 RU/s par heure) du débit dans les comptes avec des écritures à une seule région. Pour plus d’informations, consultez la [page des tarifs](https://azure.microsoft.com/pricing/details/cosmos-db/). 

#### <a name="example-1-variable-workload-autoscale-recommended"></a>Exemple 1 : Charge de travail variable (mise à l’échelle automatique recommandée)

Commençons par observer la consommation du RU normalisée. Cette charge de travail a un trafic variable, avec une consommation de RU normalisée comprise entre 6 % et 100 %. Il existe des pics occasionnels de 100 % qui sont difficiles à prédire, mais de nombreuses heures avec une faible utilisation. 

:::image type="content" source="media/how-to-choose-offer/variable-workload_use_autoscale.png" alt-text="Charge de travail à trafic variable : la consommation de RU normalisée est comprise entre 6 % et 100 % pendant toutes les heures":::

Comparons le coût de l’approvisionnement d’un débit manuel de 30 000 RU/s et une définition de la mise à l’échelle automatique sur 30 000 RU/s maximum (mise à l’échelle comprise entre 3 000 et 30 000 RU/s). 

Analysons maintenant l’historique. Supposons que nous disposons de l’utilisation décrite dans le tableau suivant. L’utilisation moyenne sur ces trois heures est de 39 %. Étant donné que la consommation RU normalisée moyenne est inférieure à 66 %, la mise à l’échelle automatique nous permet de réaliser des économies. 

Notez que, pendant l’heure 1 et une utilisation de 6 %, la mise à l’échelle automatique facturera les RU/s pour 10 % du nombre maximal de RU/s, soit la valeur minimale par heure. Bien que le coût de la mise à l’échelle automatique puisse être supérieur à celui du débit manuel à certaines heures, tant que l’utilisation moyenne est inférieure à 66 % sur toutes les heures, la mise à l’échelle automatique est moins avantageuse.

|Période  | Utilisation |RU/s avec mise à l’échelle automatique facturées  |Option 1 : 30 000 RU/s manuelles  | Option n°2 : Mise à l’échelle automatique entre 3 000 et 30 000 RU/s |
|---------|---------|---------|---------|---------|
|Heure 1  | 6 %  |     3000  |  30 000 * 0,008 / 100 = 2,40 $        |   3 000 * 0,012 / 100 = 0,36 $      |
|Heure 2  | 100 %  |     30,000    |  30 000 * 0,008 / 100 = 2,40 $       |  30 000 * 0,012 / 100 = 3,60 $      |
|Heure 3 |  11%  |     3300    |  30 000 * 0,008 / 100 = 2,40 $       |    3 300 * 0,012 / 100 = 0,40 $     |
|**Total**   |  |        |  7,20 $       |    4,36 $ (39 % d’économies)    |

#### <a name="example-2-steady-workload-manual-throughput-recommended"></a>Exemple 2 : Charge de travail stable (débit manuel recommandé)

Cette charge de travail a un trafic stable, avec une consommation de RU normalisée comprise entre 72 % et 100 %. Avec 30 000 RU/s approvisionnées, cela signifie que nous consommons entre 21 600 et 30 000 RU/s.

:::image type="content" source="media/how-to-choose-offer/steady_workload_use_manual_throughput.png" alt-text="Charge de travail à trafic stable : la consommation de RU normalisée est comprise entre 72 % et 100 % pendant toutes les heures":::

Comparons le coût de l’approvisionnement d’un débit manuel de 30 000 RU/s et une définition de la mise à l’échelle automatique sur 30 000 RU/s maximum (mise à l’échelle comprise entre 3 000 et 30 000 RU/s).

Supposons que nous disposons de l’historique d’utilisation décrit dans le tableau suivant. Notre utilisation moyenne sur ces trois heures est de 88 %. Étant donné que la consommation RU normalisée moyenne est supérieure à 66 %, l’utilisation du débit manuel nous permet de réaliser des économies.

En général, si l’utilisation moyenne sur l’ensemble des 730 heures pendant un mois est supérieure à 66 %, l’utilisation du débit manuel sera plus avantageuse. 

| Période | Utilisation |RU/s avec mise à l’échelle automatique facturées  |Option 1 : 30 000 RU/s manuelles  | Option n°2 : Mise à l’échelle automatique entre 3 000 et 30 000 RU/s |
|---------|---------|---------|---------|---------|
|Heure 1  | 72 %  |     21 600   |  30 000 * 0,008 / 100 = 2,40 $        |   21 600 * 0,012 / 100 = 2,59 $      |
|Heure 2  | 93 %  |     28 000    |  30 000 * 0,008 / 100 = 2,40 $       |  28 000 * 0,012 / 100 = 3,36 $       |
|Heure 3 |  100 %  |     30,000    |  30 000 * 0,008 / 100 = 2,40 $       |    30 000 * 0,012 / 100 = 3,60 $     |
|**Total**   |  |        |  7,20 $       |    9,55 $     |

> [!TIP]
> Avec le débit standard (manuel), vous pouvez utiliser la métrique d’utilisation normalisée pour estimer les RU/s réels que vous utiliserez potentiellement si vous basculez vers la mise à l’échelle automatique. Multipliez l’utilisation normalisée à un moment donné dans le temps par les RU/s standard approvisionnées (manuel) actuellement. Par exemple, si vous avez approvisionné 5 000 RU/s et que l’utilisation normalisée représente 90 %, l’utilisation des RU/s est de 0,9 * 5000 = 4 500 RU/s. Si vous constatez que votre modèle de trafic est variable, mais que vous avez sur ou sous-approvisionné, vous pouvez activer la mise à l’échelle automatique, puis modifier le paramètre du nombre maximal de RU/s pour la mise à l’échelle automatique en conséquence.

#### <a name="how-to-calculate-average-utilization"></a>Comment calculer l’utilisation moyenne
La mise à l’échelle automatique facture le plus grand nombre de RU/s mises à l’échelle pendant une heure. Lors de l’analyse de la consommation de RU normalisée dans le temps, il est important d’utiliser l’utilisation la plus élevée par heure dans le calcul de la moyenne. 

Pour calculer la moyenne de l’utilisation la plus élevée sur toutes les heures :
1. Définissez l’ **agrégation** sur la métrique de consommation de RU normalisée sur **Max**.
1. Sélectionnez la **granularité temporelle** sur 1 heure.
1. Accédez aux **options graphiques**.
1. Sélectionnez l’option de graphique à barres. 
1. Sous **Partager** , sélectionnez l’option **Télécharger vers Excel**. À partir de la feuille de calcul générée, calculez l’utilisation moyenne sur toutes les heures. 

:::image type="content" source="media/how-to-choose-offer/variable-workload-highest-util-by-hour.png" alt-text="Pour afficher la consommation de RU normalisée par heure, 1) sélectionnez une granularité temporelle de 1 heure ; 2) modifiez les paramètres de graphique ; 3) sélectionnez l’option de graphique à barres ; 4) sous Partager, sélectionnez l’option Télécharger vers Excel pour calculer la moyenne sur toutes les heures. ":::

## <a name="measure-and-monitor-your-usage"></a>Mesurer et surveiller votre utilisation
Par la suite, une fois que vous avez choisi le type de débit, vous devez surveiller votre application et effectuer les ajustements nécessaires. 

Lorsque vous utilisez la mise à l’échelle automatique, utilisez Azure Monitor pour afficher le nombre maximal de RU/s approvisionnées avec mise à l’échelle automatique ( **Débit maximal avec mise à l’échelle automatique** ) et les RU/s sur lesquelles le système est actuellement mis à l’échelle ( **Débit approvisionné** ). Vous trouverez ci-dessous un exemple de charge de travail variable ou imprévisible utilisant la mise à l’échelle automatique. Notez qu’en l’absence de trafic, le système met à l’échelle les RU/s vers la valeur minimale de 10 % du nombre maximal de RU/s, soit respectivement 5 000 RU/s et 50 000 RU/s. 

:::image type="content" source="media/how-to-choose-offer/autoscale-metrics-azure-monitor.png" alt-text="Exemple de charge de travail utilisant la mise à l’échelle automatique, avec une mise à l’échelle automatique de 50 000 RU/s maximum et un débit compris entre 5 000 et 50 000 RU/s":::

> [!NOTE]
> Lorsque vous utilisez un débit approvisionné standard (manuel), la métrique **Débit approvisionné** fait référence à ce que vous, en tant qu’utilisateur, avez défini. Lorsque vous utilisez le débit avec mise à l’échelle automatique, cette mesure fait référence aux RU/s sur lesquelles le système est actuellement mis à l’échelle.

## <a name="next-steps"></a>Étapes suivantes
* Utilisez la [calculatrice d’unités de requête](https://cosmos.azure.com/capacitycalculator/) pour estimer le débit de vos nouvelles charges de travail.
* Utilisez [Azure Monitor](monitor-cosmos-db.md#view-operation-level-metrics-for-azure-cosmos-db) pour surveiller vos charges de travail existantes.
* Découvrez comment [approvisionner le débit avec mise à l’échelle automatique sur un conteneur ou une base de données Azure Cosmos](how-to-provision-autoscale-throughput.md).
* Examinez le [FAQ sur la mise à l’échelle automatique](autoscale-faq.md).
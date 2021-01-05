---
title: FAQ sur la tarification d’Azure Network Performance Monitor | Microsoft Docs
description: Forum aux questions - Azure Network Performance Monitor
ms.subservice: logs
ms.topic: conceptual
author: agummadi
ms.author: agummadi
ms.date: 04/02/2018
ms.openlocfilehash: f44afd84c58c94c6a8d3e6145e8a4f66e0e2e782
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86539650"
---
# <a name="pricing-changes-for-azure-network-performance-monitor"></a>Modification de la tarification d’Azure Network Performance Monitor

Nous avons pris note de vos commentaires et récemment introduit une [nouvelle expérience de tarification](https://azure.microsoft.com/blog/introducing-a-new-way-to-purchase-azure-monitoring-services/) pour différents services de surveillance sur Azure. Cet article présente les modifications de tarification relatives à Azure [Network Performance Monitor](../../networking/network-monitoring-overview.md) (NPM) dans un format de questions/réponses simple à lire.

Network Performance Monitor comporte trois composants :
* [Analyseur de performances](../../networking/network-monitoring-overview.md#performance-monitor)
* [Moniteur de points de terminaison de service](../../networking/network-monitoring-overview.md)
* [Moniteur ExpressRoute](../../networking/network-monitoring-overview.md#expressroute-monitor)

Les sections suivantes expliquent les modifications de tarification des composants NPM.

## <a name="performance-monitor"></a>Analyseur de performances

**Comment l’analyseur de performances était-il facturé dans l’ancien modèle ?**

La facturation de NPM était basée sur l’utilisation et la consommation de deux composants :
* **Nœuds** : toutes les transactions synthétiques commencent et se terminent au niveau des nœuds. Les nœuds sont également appelés agents ou Microsoft Management Agents.
* **Données** : les résultats des différents tests de réseau sont stockés dans l'espace de travail Log Analytics.

Dans l’ancien modèle, la facture était calculée en fonction du nombre de nœuds et du volume de données généré. 

**Comment l’utilisation de l’Analyseur de performances est-elle facturée dans le nouveau modèle ?**

La fonctionnalité de l’Analyseur de performances dans NPM est maintenant facturée en fonction d’une combinaison de : 

* Liens de sous-réseau surveillés
* Volume de données

**Qu’est un lien de sous-réseau ?**

L’Analyseur de performances surveille la connectivité entre deux emplacements ou plus sur le réseau. La connexion entre un groupe de nœuds ou d’agents sur un sous-réseau et un groupe de nœuds sur un autre sous-réseau est appelée un lien de sous-réseau.

**J’ai deux sous-réseaux (A et B) et plusieurs agents sur chaque sous-réseau. L’Analyseur de performances surveille la connectivité entre tous les agents du sous-réseau A et tous les agents du sous-réseau B. Serai-je facturé en fonction du nombre de connexions entre sous-réseaux ?**

Non. Pour la facturation, toutes les connexions du sous-réseau A au sous-réseau B sont regroupées dans un même lien de sous-réseau. Vous êtes facturé pour une seule connexion. L’Analyseur de performances continue à surveiller la connectivité entre différents agents sur chaque sous-réseau.

**Quels sont les coûts de surveillance d’un lien de sous-réseau ?**

Pour connaître le coût de surveillance d’un seul lien de sous-réseau pour un mois entier, reportez-vous à la section intitulée [Ping Mesh](https://azure.microsoft.com/pricing/details/network-watcher/).

**Quels sont les frais relatifs aux données générées par l’Analyseur de performances ?**

Les frais d'ingestion (chargement des données dans l'espace de travail Log Analytics d'Azure Monitor, traitement et indexation) sont disponibles sur la [page de tarification](https://azure.microsoft.com/pricing/details/log-analytics/) de Log Analytics, dans la section relative à l'ingestion des données. Les frais de conservation des données (à savoir, les données conservées à la discrétion du client, après le premier mois) sont également disponibles sur la [page de tarification](https://azure.microsoft.com/pricing/details/log-analytics/) dans la section relative à la conservation des données.


## <a name="expressroute-monitor"></a>Moniteur ExpressRoute

**Quels sont les frais d’utilisation du moniteur ExpressRoute ?**

Les frais relatifs au moniteur ExpressRoute sont facturés en fonction du volume de données généré pendant la surveillance. Pour plus d’informations, reportez-vous à la question « Quels sont les frais relatifs aux données générées par l’Analyseur de performances ? ».

**J’utilise le Moniteur ExpressRoute pour surveiller plusieurs circuits ExpressRoute. Suis-je facturé en fonction du nombre de circuits surveillés ?**

Vous n’êtes pas facturé en fonction du nombre de circuits ou du type de peering (par exemple, peering privé, peering Microsoft). Vous êtes facturé en fonction du volume de données, comme expliqué précédemment.

**Quel est le volume de données généré lorsque le service ExpressRoute surveille un seul circuit ?**

Le volume de données généré par mois, lorsque le service ExpressRoute surveille une connexion de peering privé, est le suivant :

|Percentile      |Données/mois (Mo)|
| :---:          |           ---:|
|50<sup>e</sup> |            192|
|60<sup>e</sup> |            256|
|70<sup>e</sup> |            360|
|80<sup>e</sup> |            498|
|90<sup>e</sup> |            870|
|95<sup>e</sup> |           1560|


D’après ce tableau, les clients au 50e percentile payent 192 Mo de données. À 2,30 USD/Go pour le premier mois, le coût de la surveillance d’un circuit est de 0,43 USD (= 192 * 2,30 / 1024).

**Quelles sont certaines raisons des variations du volume de données ?**

Le volume de données de surveillance généré dépend de plusieurs facteurs :
* Nombre d’agents. La précision de l’isolation des défaillances augmente à mesure que le nombre d’agents augmente.
* Nombre de tronçons sur le réseau.
* Nombre de chemins d’accès entre la source et la destination.

Les clients aux percentiles supérieurs (dans le tableau précédent) surveillent généralement leurs circuits à partir de plusieurs points stratégiques sur leur réseau local. Plusieurs agents sont également placés plus en profondeur dans le réseau, loin du routeur de périphérie du fournisseur de services. Ils sont souvent placés au niveau de plusieurs sites utilisateur, branches et racks de centres de données.

## <a name="service-endpoint-monitor"></a>Moniteur de points de terminaison de service

**Quels sont les frais d’utilisation du moniteur de points de terminaison de service ?**

Les frais d’utilisation du moniteur de points de terminaison de service sont calculés en fonction de :
* Nombre de connexions
* Volume de données

**Qu’est-ce qu’une connexion ?**

Une connexion est un test d’accessibilité à un point de terminaison (URL ou service réseau) à partir d’un seul agent pour le mois entier. Par exemple, la surveillance d’une connexion à bing.com à partir de trois agents correspond à trois connexions.

**Quels sont les coûts du moniteur de points de terminaison de service ?**

Reportez-vous à la section [Contrôle de la connexion](https://azure.microsoft.com/pricing/details/network-watcher/) pour connaître le coût de la surveillance d’un point de terminaison pour un mois entier. Les frais de données sont disponibles sur la [page de tarification](https://azure.microsoft.com/pricing/details/log-analytics/) de Log Analytics, dans la section relative à l’ingestion des données.

## <a name="references"></a>References

[FAQ sur la tarification de Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/) : la section FAQ contient des informations sur le niveau gratuit, la tarification par nœud, ainsi que d’autres détails de tarification.

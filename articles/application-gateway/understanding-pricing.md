---
title: Compréhension de la tarification – Azure Application Gateway
description: Cet article décrit le processus de facturation pour Azure Application Gateway et le pare-feu d’applications web (WAF) pour les références (SKU) v1 à v2
services: application-gateway
author: azhar2005
ms.service: application-gateway
ms.topic: conceptual
ms.custom: references_regions
ms.date: 09/01/2020
ms.author: azhussai
ms.openlocfilehash: 89ba6b7a69c95951a083628f23be68d811c7768c
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96601609"
---
# <a name="understanding-pricing-for-azure-application-gateway-and-web-application-firewall"></a>Compréhension de la tarification d’Azure Application Gateway et du pare-feu d’applications web

>[!NOTE]
>Les prix indiqués dans cet article sont des exemples fournis uniquement à titre d’illustration. Pour obtenir des informations sur la tarification en fonction de votre région, consultez la [page relative à la tarification](https://azure.microsoft.com/pricing/details/application-gateway/).

Azure Application Gateway est une solution d’équilibrage de charge de couche 7 qui permet une livraison d’application web évolutive, hautement disponible et sécurisée sur Azure.

Aucun coût initial ou coût d’arrêt n’est associé à Application Gateway.
Vous êtes facturé uniquement pour les ressources préapprovisionnées et utilisées en fonction de la consommation horaire réelle. Les coûts associés à Application Gateway sont classés en deux composantes : coûts fixes et coûts variables. Les coûts réels au sein de chaque composant varient en fonction de la référence (SKU) utilisée.

Cet article décrit les coûts associés à chaque référence (SKU) et il est recommandé aux utilisateurs d’utiliser ce document pour planifier et gérer les coûts associés à Azure Application Gateway.

## <a name="v1-skus"></a>Références (SKU) V1

Les références (SKU) V1 d’Application Gateway Standard et WAF sont facturées comme une combinaison de :

* Coût fixe

    Coût basé sur le temps pendant lequel un type particulier d’Application Gateway/WAF est approvisionné et opérationnel pour traiter les demandes.
    La composante coût fixe prend en compte les facteurs suivants :
    * Niveau – Application Gateway Standard ou WAF
    * Taille – Petit, Moyen et Grand
    * Nombre d’instances – Nombre d’instances à déployer

    Pour N instances, les coûts associés sont de N * coût d’une instance d’un niveau particulier (Standard et WAF) & Taille (Petite, Moyenne et Grande).

* Coût variable

    Coût basé sur la quantité de données traitées par Application Gateway/WAF. Les octets de demande et de réponse traités par Application Gateway sont pris en compte pour la facturation.

Coût total = coût fixe + coût variable

### <a name="standard-application-gateway"></a>Application Gateway Standard

Le coût fixe et le coût variable sont calculés en fonction du type d’Application Gateway.
Le tableau suivant présente des exemples de prix basés sur un capture instantanée de la tarification pour la région USA Est, fournis uniquement à des fins d’illustration.

#### <a name="fixed-cost-east-us-region-pricing"></a>Coût fixe (tarification de la région USA Est)

|              Type d’Application Gateway             |  Coûts (USD/heure)  |
| ------------------------------------------------- | ---------------|
|                     Petite                         |    0,025 USD      |
|                     Moyenne                        |    0,07 USD       |
|                     Grande                         |    0,32 USD       |

Les estimations de prix mensuel estimé sont basées sur 730 heures d’utilisation par mois.

#### <a name="variable-cost-east-us-region-pricing"></a>Coût variable (tarification de la région USA Est)

|              Données traitées             |  Petit (USD/Go)  |  Moyen (USD/Go) |  Grand (USD/Go) |
| --------------------------------------- | -------------- | -------------- | ------------- |
| 10 premiers To/mois                       |     0,008 USD     |      Gratuit      |     Gratuit      |
| 30 To suivants (10–40 To)/mois             |     0,008 USD     |     0,007 USD     |     Gratuit      |
| Plus de 40 To/mois                        |     0,008 USD     |     0,007 USD     |     0,0035 USD   |

Pour plus d’informations sur la tarification en fonction de votre région, consultez la [page relative à la tarification](https://azure.microsoft.com/pricing/details/application-gateway/).

### <a name="waf-v1"></a>WAF V1

Le coût fixe et le coût variable sont calculés en fonction du type d’Application Gateway approvisionné.

Le tableau suivant présente des exemples de prix basés sur une capture instantanée de la tarification pour la région USA Est, qui sont fournis uniquement à titre d’illustration.

#### <a name="fixed-cost-east-us-region-pricing"></a>Coût fixe (tarification de la région USA Est)

|              Type d’Application Gateway             |  Coûts (USD/heure)  |
| ------------------------------------------------- | ---------------|
|                     Petite                         |       N/D       |
|                     Moyenne                        |     0,126 USD     |
|                     Grande                         |     0,448 USD     |

Les estimations de prix mensuel estimé sont basées sur 730 heures d’utilisation par mois.

#### <a name="variable-cost-east-us-region-pricing"></a>Coût variable (tarification de la région USA Est)

|              Données traitées             |  Petit (USD/Go)  |  Moyen (USD/Go) |  Grand (USD/Go) |
| --------------------------------------- | -------------- | -------------- | ------------- |
| 10 premiers To/mois                       |     0,008 USD     |      Gratuit      |     Gratuit      |
| 30 To suivants (10–40 To)/mois             |     0,008 USD     |     0,007 USD     |     Gratuit      |
| Plus de 40 To/mois                        |     0,008 USD     |     0,007 USD     |     0,0035 USD   |

Pour plus d’informations sur la tarification en fonction de votre région, consultez la [page relative à la tarification](https://azure.microsoft.com/pricing/details/application-gateway/).

> [!NOTE]
> Transferts de données sortantes : les données sortant de centres de données Azure en provenance de passerelles d’application sont facturées aux [taux de transfert de données](https://azure.microsoft.com/pricing/details/bandwidth/) standard.

### <a name="example-1-a--standard-application-gateway-with-1-instance-count"></a>Exemple 1 (a) – Application Gateway Standard avec 1 instance

Supposons que vous avez approvisionné Application Gateway Standard de type moyen avec 1 instance et qu’il traite 500 Go par mois. Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

Prix fixe = 0,07 USD * 730 (heures) = 51,1 USD Les estimations de prix mensuel sont basées sur 730 heures d’utilisation par mois.

Coûts variables = Gratuit (le niveau moyen est gratuit pour les 10 premiers To traités par mois) Coût total = 51,1 USD + 0 = 51,1 USD 

> [!NOTE]
> Pour prendre en charge les scénarios de haute disponibilité, il est nécessaire de configurer au minimum 2 instances pour les références (SKU) V1. Voir [SLA pour Application Gateway](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_2/).

### <a name="example-1-b--standard-application-gateway-with--1-instance-count"></a>Exemple 1 (b) – Application Gateway Standard avec plus de 1 instance

Supposons que vous avez approvisionné Application Gateway Standard de type moyen avec 5 instances et qu’il traite 500 Go par mois. Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

Prix fixe = 5 (nombre d’instances) * 0.07 USD * 730 (heures) = 255,5 USD Les estimations de prix mensuel sont basées sur 760 heures d’utilisation par mois.

Coûts variables = Gratuit (le niveau moyen est gratuit pour les 10 premiers To traités par mois) Coût total = 255,5 USD + 0 = 255,5 USD 

### <a name="example-2--waf-application-gateway"></a>Exemple 2 – Application Gateway WAF

Supposons que vous avez approvisionné une Application Gateway Standard de petite taille et une Application Gateway WAF de grande taille pour les 15 premiers jours du mois. La passerelle d’application de petite taille traite 15 To pendant sa durée d’activité, et la passerelle d’application WAF de grande taille traite 100 To pendant sa durée d’activité. Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit : 

###### <a name="small-instance-standard-application-gateway"></a>Petite instance Application Gateway Standard

24 heures * 15 jours = 360 heures

Prix fixe = 0,025 USD * 360 (heures) = 9 USD

Coûts variables = 10 * 1000 * 0,008 USD/Go + 5 * 1000 * 0,008 USD/Go = 120 USD

Coût total = 9 USD + 120 USD = 129 USD

###### <a name="large-instance-waf-application-gateway"></a>Grande instance Application Gateway WAF
24 heures * 15 jours = 360 heures

Prix fixe = 0,448 USD * 360 (heures) = 161,28 USD

Coûts variables = 60 * 1000 * 0,0035 USD/Go = 210 USD (le niveau Grand est gratuit pour les 40 premiers To traités par mois)

Coût total = 161,28 USD + 210 USD = 371,28 USD

## <a name="v2-skus"></a>Références (SKU) V2  

Les SKU Application Gateway V2 et WAF V2 prennent en charge la mise à l’échelle automatique et garantissent une haute disponibilité par défaut.

### <a name="key-terms"></a>Termes clés

##### <a name="capacity-unit"></a>Unité de capacité 
L’unité de capacité est la mesure de l’utilisation de la capacité pour une instance Application Gateway sur plusieurs paramètres.

Une unité de capacité unique est constituée des paramètres suivants :
* 2500 connexions persistantes
* Débit de 2,22 Mbits/s
* 1 unité Compute

Si l’un de ces paramètres est dépassé, n autres unités de capacité sont nécessaires, même si les deux autres paramètres ne dépassent pas les limites de cette unité de capacité.
Le paramètre dont l’utilisation est la plus élevée parmi les trois ci-dessus est utilisé en interne pour le calcul des unités de capacité, qui est facturé à son tour.

##### <a name="compute-unit"></a>Unité Compute
L’unité Compute est la mesure de la capacité de calcul consommée. Les facteurs affectant la consommation d’unité Compute sont les connexions TLS/s, les calculs de réécriture d’URL et le traitement de règle WAF. Le nombre de requêtes qu’une unité Compute peut gérer dépend de différents critères parmi lesquels, la taille de clé du certificat TLS, l’algorithme d’échange de clés, les réécritures d’en-têtes et la taille de la requête entrante WAF, le cas échéant.

Aide relative aux unités de calcul :
* Standard_v2 - Chaque unité Compute permet environ 50 connexions par seconde avec un certificat TLS de clé RSA de 2048 bits.

* WAF_v2 - Chaque unité Compute peut prendre en charge environ 10 requêtes simultanées par seconde pour une combinaison de trafic de 70-30 % avec 70 % des demandes inférieures à 2 Ko GET/POST et restant plus élevées. Actuellement, le niveau de performance WAF n’est pas affecté par la taille de la réponse.

##### <a name="instance-count"></a>Nombre d'instances 
Le pré-approvisionnement des ressources pour les SKU Application Gateway V2 est défini en termes de nombre d’instances. Chaque instance garantit un minimum de 10 unités de capacité en termes de capabilité de processus. La même instance peut potentiellement prendre en charge plus de 10 unités de capacité pour différents modèles de trafic en fonction des paramètres d’unité de capacité.

L’échelle et les limites définies manuellement pour la mise à l’échelle automatique (minimale ou maximale) sont définies en termes de nombre d’instances. L’échelle définie manuellement pour le nombre d’instances et le nombre minimal d’instances dans la configuration de mise à l’échelle automatique réserve 10 unités de capacité/instance. Ces unités de capacité réservées sont facturées tant que l’Application Gateway est active, quelle que soit la consommation réelle de ressources. Si la consommation réelle dépasse le seuil de 10 unités de capacité/instance, les unités de capacité supplémentaires sont facturées sous la composante des coûts variables.

Les références (SKU) V2 sont facturées en fonction de la consommation, et sont constituées de deux composantes :

* Coûts fixes

    Coût basé sur le temps pendant lequel Application Gateway V2/WAF V2 est approvisionné et disponible pour le traitement des demandes. Cela garantit une haute disponibilité, même si aucune instance n’est réservée en spécifiant 0 dans le nombre minimal d’instances dans le cadre de la mise à l’échelle automatique. 
    
    Le coût fixe comprend également le coût associé à l’adresse IP publique attachée à l’Application Gateway.

    Le nombre d’instances en cours d’exécution à un moment donné n’est pas considéré comme un facteur pour les coûts fixes des références (SKU) V2. Le coût fixe horaire lié à l’exécution d’une référence (SKU) Standard_V2 (ou WAF_V2) ne change pas, quel que soit le nombre d’instances exécutées dans la même région Azure.

* Coûts d’unité de capacité 

    Coût en fonction du nombre d’unités de capacité réservées ou utilisées en supplément, comme requis pour le traitement des requêtes entrantes.  Les coûts basés sur la consommation sont calculés sur une base horaire.

Coût total = Coûts fixes + Coûts d’unités de capacité

> [!NOTE]
> Une heure partielle est facturée comme une heure complète.

Le tableau suivant présente des exemples de prix basés sur une capture instantanée de la tarification pour la région USA Est, qui sont fournis uniquement à titre d’illustration.

#### <a name="fixed-costs-east-us-region-pricing"></a>Coûts fixes (tarification de la région USA Est)

|              Référence (SKU) V2             |  Coûts (USD/heure)  |
| ------------------------------- | ---------------|
|            Standard_v2          |     0,246 USD     |
|              WAF_V2             |     0,443 USD     |

Les estimations de prix mensuel estimé sont basées sur 730 heures d’utilisation par mois.

#### <a name="variable-costs-east-us-region-pricing"></a>Coûts variables (tarification de la région USA Est)

|        Unité de capacité         |  Standard_v2 (USD/heure)  |  WAF_V2 (USD/heure) |
| ---------------------------- | -------------------- | -------------- |
|             1 unité de capacité             |        0,008 USD        |     0,0144 USD    |

Pour plus d’informations sur la tarification en fonction de votre région, consultez la [page relative à la tarification](https://azure.microsoft.com/pricing/details/application-gateway/).

> [!NOTE]
> Transferts de données sortantes : les données sortant de centres de données Azure en provenance de passerelles d’application sont facturées aux [taux de transfert de données](https://azure.microsoft.com/pricing/details/bandwidth/) standard.

### <a name="example-1-a--manual-scaling"></a>Exemple 1 (a) – Mise à l’échelle manuelle 
Supposons que vous avez approvisionné une instance Application Gateway Standard_V2 avec une mise à l’échelle manuelle définie sur 8 instances pour le mois entier. Pendant ce temps, elle reçoit un transfert de données moyen de 88,8 Mbits/s.

Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

1 unité de capacité peut gérer un débit de 2,22 Mbits/s.

Unités de capacité requises pour gérer 88,8 Mbits/s = 88,8/2,22 = 40 unités de capacité 

Unités de capacité préapprovisionnées = 8 (nombre d’instances) * 10 = 80 

Comme 80 (capacité réservée) > 40 (capacité requise), aucune unité de capacité supplémentaire n’est requise. 

Prix fixe = 0,246 USD * 730 (heures) = 179,58 USD

Coûts variables = 0,008 USD * 8 (unités d’instance) * 10 (unités de capacité) * 730 (heures) = 467,2 USD

Coût total = 179,58 USD + 467,2 USD = 646,78 USD

![Diagramme de l’échelle manuelle 1.](./media/pricing/manual-scale-1.png)

### <a name="example-1-b--manual-scaling-with-traffic-going-beyond-provisioned-capacity"></a>Exemple 1 (b) – Mise à l’échelle manuelle avec trafic allant au-delà de la capacité approvisionnée

Supposons que vous avez approvisionné une instance Application Gateway Standard_V2 avec une mise à l’échelle manuelle définie sur 3 instances pour le mois entier. Pendant ce temps, elle reçoit un transfert de données moyen de 88,8 Mbits/s.

Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

1 unité de capacité peut gérer un débit de 2,22 Mbits/s.

Unités de capacité requises pour gérer 88,8 Mbits/s = 88,8/2,22 = 40 unités de capacité 

Unités de capacité préapprovisionnées = 3 (nombre d’instances) * 10 = 30 

Étant donné que 40 (capacité requise) > 30 (capacité réservée), des unités de capacité supplémentaires sont requises.
Le nombre d’unités de capacité supplémentaires dépend de la capacité libre disponible avec chaque instance.

Si une capacité de traitement équivalant à 10 unités de capacité supplémentaires était disponible pour une utilisation dans les 3 instances réservées :

Prix fixe = 0,246 USD * 730 (heures) = 179,58 USD

Coûts variables = 0,008 USD * (3 (unités d’instance) * 10 (unités de capacité) + 10 (unités de capacité supplémentaires)) * 730 (heures) = 233,6 USD

Coût total = 179,58 USD + 233,6 USD = 413,18 USD

Toutefois, si une capacité de traitement équivalant à 7 unités de capacité supplémentaires était disponible pour une utilisation dans les 3 instances réservées :
Dans ce scénario, la ressource Application Gateway est mise à l’échelle et peut entraîner une augmentation de la latence ou la suppression des demandes.

Prix fixe = 0,246 USD * 730 (heures) = 179,58 USD

Coûts variables = 0,008 USD * (3 (unités d’instance) * 10 (unités de capacité) + 7 (unités de capacité supplémentaires)) * 730 (heures) = 216,08 USD

Coût total = 179,58 USD + 216,08 USD = 395,66 USD


![Diagramme de l’échelle manuelle 2.](./media/pricing/manual-scale-2.png)

> [!NOTE]
> En cas de mise à l’échelle manuelle, toutes les demandes supplémentaires dépassant la capacité de traitement maximale des instances réservées peuvent avoir un impact sur la disponibilité de votre application. Dans les situations de forte charge, des instances réservées peuvent fournir plus de 10 unités de capacité de traitement en fonction de la configuration et du type des demandes entrantes. Il est cependant recommandé d’approvisionner le nombre d’instances conformément aux besoins de votre trafic.

### <a name="example-2--waf_v2-instance-with-autoscaling"></a>Exemple 2 – Instance WAF_V2 avec mise à l’échelle automatique

Supposons que vous avez approvisionné une instance WAF_V2 avec la mise à l’échelle automatique activée et défini le nombre d’instances minimal sur 6 pour le mois entier. La charge de la demande a entraîné une montée en puissance parallèle de l’instance WAF, et l’utilisation de 65 unités de capacité (montée en puissance parallèle de 5 unités de capacité, et 60 unités réservées) pendant le mois entier.
Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

Les estimations de prix mensuel estimé sont basées sur 730 heures d’utilisation par mois.

Prix fixe = 0,443 USD * 730 (heures) = 323,39 USD

Coûts variables = 0,0144 USD * 65 (unités de capacité) * 730 (heures) = 683,28 USD

Coût total = 323,39 USD + 683,28 USD = 1006,67 USD

![Diagramme de mise à l’échelle automatique 2.](./media/pricing/auto-scale-1.png)

> [!NOTE]
> Le trafic réel observé pour votre Application Gateway est peu susceptible de présenter un profil de trafic aussi constant, et la charge observée sur votre Application Gateway fluctue en fonction de l’utilisation réelle.

### <a name="example-3-a--waf_v2-instance-with-autoscaling-and-0-min-scale-config"></a>Exemple 3 (a) – Instance WAF_V2 avec mise à l’échelle automatique et configuration de l’échelle minimale 0

Supposons que vous avez approvisionné une instance WAF_V2 avec la mise à l’échelle automatique activée et défini le nombre d’instances minimal sur 0 pour le mois entier. La charge de la demande sur l’instance WAF est minimale, mais systématiquement présente à chaque heure pendant le mois entier. La charge est inférieure à la capacité d’une unité de capacité.
Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

Les estimations de prix mensuel estimé sont basées sur 730 heures d’utilisation par mois.

Prix fixe = 0,443 USD * 730 (heures) = 323,39 USD

Coûts variables = 0,0144 USD * 1 (unités de capacité) * 730 (heures) = 10,512 USD

Coût total = 323,39 USD + 10,512 USD = 333,902 USD

### <a name="example-3-b--waf_v2-instance-with-autoscaling-with-0-min-instance-count"></a>Exemple 3 (b) – Instance WAF_V2 avec mise à l’échelle automatique et nombre d’instances minimal de 0

Supposons que vous avez approvisionné une instance WAF_V2 avec la mise à l’échelle automatique activée et défini le nombre d’instances minimal sur 0 pour le mois entier. Toutefois, aucun trafic n’est dirigé vers l’instance WAF pendant le mois entier.
Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

Prix fixe = 0,443 USD * 730 (heures) = 323,39 USD

Coûts variables = 0,0144 USD * 0 (unités de capacité) * 730 (heures) = 0 USD

Coût total = 323,39 USD + 0 USD = 323,39 USD

### <a name="example-3-c--waf_v2-instance-with-manual-scaling-set-to-1-instance"></a>Exemple 3 (c) – Instance WAF_V2 avec une mise à l’échelle manuelle définie sur 1 instance

Supposons que vous avez approvisionné une instance WAF_V2 et l’avez définie sur une mise à l’échelle manuelle avec la valeur minimale acceptable de 1 instance pour le mois entier. Toutefois, aucun trafic n’est dirigé vers l’instance WAF pour le mois entier.
Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

Les estimations de prix mensuel estimé sont basées sur 730 heures d’utilisation par mois.

Prix fixe = 0,443 USD * 730 (heures) = 323,39 USD

Coûts variables = 0,0144 USD * 1 (nombre d’instances) * 10 (unités de capacité) * 730 (heures) = 105,12 USD

Coût total = 323,39 USD + 105,12 USD = 428,51 USD

### <a name="example-4--waf_v2-with-autoscaling-capacity-unit-calculations"></a>Exemple 4 – Instance WAF_V2 avec mise à l’échelle automatique, calculs d’unités de capacité

Supposons que vous avez approvisionné une instance WAF_V2 avec la mise à l’échelle automatique activée et défini le nombre d’instances minimal sur 0 pour le mois entier. Pendant ce temps, l’instance reçoit 25 nouvelles connexions TLS/s, avec un transfert de données moyen de 8,88 Mbits/s.
Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

Les estimations de prix mensuel estimé sont basées sur 730 heures d’utilisation par mois.

Prix fixe = 0,443 USD * 730 (heures) = 323,39 USD

Coûts variables = 0,0144 USD * 730 (heures) * {max (25/50, 8.88/2,22)} = 23,36 USD (4 unités de capacité requises pour gérer 8,88 Mbps)

Coût total = 323,39 USD + 23,36 USD = 346,75 USD

### <a name="example-5-a--standard_v2-with-autoscaling-time-based-calculations"></a>Exemple 5 (a) – Instance Standard_V2 avec mise à l’échelle automatique, calculs basés sur le temps

Supposons que vous avez approvisionné une instance Standard_V2 avec la mise à l’échelle automatique activée et défini le nombre d’instances minimal sur 0 et que cette passerelle applicative est active pendant 2 heures.
Au cours de la première heure, l’instance reçoit le trafic qui peut être géré par 10 unités de capacité et, pendant la deuxième heure, elle reçoit un trafic nécessitant 20 unités de capacité pour gérer la charge.
Vos coûts d’Application Gateway selon la tarification précitée sont calculés comme suit :

Prix fixe = 0.246 USD * 2 (heures) = 0,492 USD

Coûts variables = 0,008 USD * 10 (unités de capacité) * 1 (heure) + 0,008 USD * 20 (unités de capacité) * 1 (heure) = 0,24 USD

Coût total = 0,492 USD + 0,24 USD = 0,732 USD


## <a name="monitoring-billed-usage"></a>Surveillance de l’utilisation facturée

Vous pouvez afficher la quantité de consommation pour différents paramètres (unité Compute, débit et connexions persistantes), ainsi que les unités de capacité utilisées dans le cadre des métriques d’Application Gateway dans la section **Surveillance**.

![Diagramme de la section des métriques.](./media/pricing/metrics-1.png)

### <a name="useful-metrics-for-cost-estimation"></a>Métriques utiles pour l’estimation des coûts

* Unités de capacité actuelles

    Nombre d’unités de capacité consommées pour équilibrer la charge du trafic entre les trois paramètres : Connexions persistantes, débit et unité Compute.

* Unités de capacité facturables fixes

    Nombre minimal d’unités de capacité gardées approvisionnées conformément à la valeur du paramètre Nombre d’instances minimal (une instance se traduit par 10 unités de capacité) dans la configuration d’Application Gateway.

* Unités de capacité facturées estimées

    Les unités de capacité facturées estimées indiquent le nombre d’unités de capacité à l’aide desquelles la facturation est estimée. Cette valeur est calculée comme étant la plus grande valeur entre Unités de capacité actuelles (unités de capacité requises pour équilibrer la charge du trafic) et Unités de capacité facturables fixes (unités de capacité minimales approvisionnées conservées).

D’autres métriques telles que le débit, les connexions actuelles et les unités Compute sont également disponibles pour comprendre les goulots d’étranglement et estimer le nombre d’unités de capacité requises. Des informations détaillées sont disponibles dans [Métriques d’Application Gateway](application-gateway-metrics.md)

#### <a name="example---estimating-capacity-units-being-utilized"></a>Exemple – Estimation des unités de capacité utilisées

**Métriques observées :**

* Unités Compute = 17,38
* Débit = 1 370 000 octets/s - 10,96 Mbits/s
* Connexions en cours = 123 080
* Unités de capacité calculées = Max (17,38, 10,96/2,22, 123 080/2 500) = 49,232

Unités de capacité observées dans les métriques = 49,23

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les tarifs dans Azure Application Gateway, consultez les articles suivants :

* [Page relative à la tarification d’Application Gateway](https://azure.microsoft.com/pricing/details/application-gateway/)
* [Calculatrice de prix d’Azure Application Gateway](https://azure.microsoft.com/en-us/pricing/calculator/?service=application-gateway)

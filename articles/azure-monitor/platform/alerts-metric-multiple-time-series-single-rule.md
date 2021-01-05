---
title: Surveiller plusieurs séries chronologiques dans une seule règle d’alerte de métrique
description: Alerte à l’échelle à l’aide d’une règle d’alerte unique pour plusieurs séries chronologiques
author: harelbr
ms.author: harelbr
ms.topic: conceptual
ms.date: 11/12/2020
ms.subservice: alerts
ms.openlocfilehash: 66987a28acc8a2c9ae71d89ff5760fa508e32963
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566501"
---
# <a name="monitor-multiple-time-series-in-a-single-metric-alert-rule"></a>Surveiller plusieurs séries chronologiques dans une seule règle d’alerte de métrique

Une seule règle d’alerte de métrique peut être utilisée pour analyser une ou plusieurs séries de temps de métriques, ce qui facilite la surveillance des ressources à l’échelle.

## <a name="metric-time-series"></a>Série chronologique de métriques

Une série chronologique de métriques est une série de mesures (ou « valeurs de métriques ») capturées sur une période donnée. 

Par exemple :

- Utilisation du processeur par une machine virtuelle
- Octets entrants dans un compte de stockage
- Nombre de requêtes ayant échoué pour une application web



## <a name="alert-rule-on-a-single-time-series"></a>Règle d’alerte sur une seule série chronologique
Une règle d’alerte surveille une série chronologique unique lorsqu’elle remplit toutes les conditions suivantes :
-   La règle analyse une ressource cible unique 
-   Contient une condition unique
-   Évalue une métrique sans choisir de dimensions (en supposant que la métrique prend en charge les dimensions)

Exemple de règle d’alerte (avec uniquement les propriétés pertinentes affichées) :
-   Ressource cible : *myVM1*
-   Mesure : *Pourcentage d’UC*
-   Opérateur : *Supérieur à*
-   Seuil : *70*


Pour cette règle d’alerte, une seule série chronologique est analysée :
-   Pourcentage d’UC où *Resource*= 'myVM1 ' > 70%

![Une règle d’alerte sur une seule série chronologique](media/alerts-metric-multiple-time-series-single-rule/simple-alert-rule.png)

## <a name="alert-rule-on-multiple-time-series"></a>Règle d’alerte sur plusieurs séries chronologiques
Une règle d’alerte surveille plusieurs séries chronologiques si elle utilise au moins l’une des fonctionnalités suivantes : 
-   Plusieurs ressources
-   Plusieurs conditions 
-   Plusieurs dimensions


## <a name="multiple-resources-multi-resource"></a>Plusieurs ressources

Une règle d’alerte de mesure unique peut surveiller plusieurs ressources, à condition que les ressources soient du même type et se trouvent dans la même région Azure. L’utilisation de ce type de règle réduit la complexité et le nombre total de règles d’alerte que vous devez gérer. 

Exemple d’une telle règle d’alerte :
-   Ressource cible : *myVM1, myVM2*
-   Mesure : *Pourcentage d’UC*
-   Opérateur : *Supérieur à*
-   Seuil : *70*

Pour cette règle d’alerte, deux séries chronologiques de métriques sont surveillées séparément :
-   Pourcentage d’UC où *Resource*= 'myVM1 ' > 70%
-   Pourcentage d’UC où *Resource*= 'myVM2 ' > 70%

![Une règle d’alerte à plusieurs ressources](media/alerts-metric-multiple-time-series-single-rule/multi-resource-alert-rule.png)
 
Dans une règle d’alerte à plusieurs ressources, la condition est évaluée  **séparément** pour chacune des ressources (ou plus précisément, pour chacune des séries chronologiques de métriques correspondant à chaque ressource). Cela signifie que les alertes sont également déclenchées pour chaque ressource séparément.

Supposons, par exemple, que nous ayons défini la règle d’alerte ci-dessus pour surveiller l’UC au-dessus de 70 %. Dans la période évaluée (c’est-à-dire, les 5 dernières minutes)
-   Le *Pourcentage d'UC* de *myVM1* est supérieur à 70 % 
-   Le *Pourcentage d'UC* de *myVM2* est à 50 %

La règle d’alerte se déclenche sur *myVM1*, mais pas sur *myVM2*. Ces alertes déclenchées sont indépendantes. Elles peuvent également être résolues à des moments différents selon le comportement individuel de chacune des machines virtuelles.

Pour plus d’informations sur les règles d’alerte sur plusieurs ressources et les types de ressources pris en charge pour cette fonctionnalité, consultez [Surveillance à l’échelle à l’aide d’alertes de métrique dans Azure Monitor](alerts-metric-overview.md#monitoring-at-scale-using-metric-alerts-in-azure-monitor).

> [!NOTE] 
> Dans une règle d’alerte métrique qui surveille plusieurs ressources, une seule condition est autorisée.

## <a name="multiple-conditions-multi-condition"></a>Conditions multiples

Une règle d’alerte de métrique unique peut également analyser jusqu’à cinq conditions par règle d’alerte. 

Par exemple :

- Ressource cible : *myVM1*
- Condition1
  - Mesure : *Pourcentage d’UC*
  - Opérateur : *Supérieur à*
  - Seuil : *70*
- Condition2
  - Mesure : *Octets entrants réseau totaux*
  - Opérateur : *Supérieur à*
  - Seuil : *20 Mo*

Pour cette règle d’alerte, deux séries chronologiques de métriques sont surveillées :

- Pourcentage d’UC où *Resource*= 'myVM1 ' > 70%
- Octets entrants réseau totaux où *Resource*=’myVM1’ > 20 Mo

![Une règle d’alerte à plusieurs conditions](media/alerts-metric-multiple-time-series-single-rule/multi-condition-alert-rule.png)
 
Un opérateur « AND » est utilisé entre les conditions. La règle d’alerte déclenche une alerte lorsque **toutes les conditions** sont remplies. L’alerte déclenchée est résolue si au moins une des conditions n’est plus remplie. 

> [!NOTE]
> Des restrictions lors de l’utilisation de dimensions dans une règle d’alerte des indicateurs de performance avec plusieurs conditions. Pour plus d’informations, consultez [Restrictions lors de l’utilisation de dimensions dans une règle d’alerte des indicateurs de performance avec plusieurs conditions](alerts-troubleshoot-metric.md#restrictions-when-using-dimensions-in-a-metric-alert-rule-with-multiple-conditions).


## <a name="multiple-dimensions-multi-dimension"></a>Plusieurs dimensions

Une règle d’alerte de métrique unique peut également surveiller plusieurs valeurs de dimension d’une métrique. Les dimensions d’une métrique sont des paires nom-valeur qui transmettent des données supplémentaires pour décrire la valeur de la métrique. Par exemple, la métrique **Transactions** d’un compte de stockage a une dimension appelée **Nom de l’API**, décrivant le nom de l’API appelée par chaque transaction (par exemple, GetBlob, DeleteBlob, PutPage). L’utilisation de dimensions est facultative, mais elle permet de filtrer la mesure et d’analyser uniquement des séries chronologiques spécifiques, au lieu de surveiller la métrique sous la forme d’un agrégat de toutes les valeurs dimensionnelles rassemblées. 

Par exemple, vous pouvez choisir de déclencher une alerte lorsque le nombre de transactions est élevé sur tous les noms d’API (c’est-à-dire les données agrégées), ou de la décomposer en une seule alerte lorsque le nombre de transactions est élevé pour des noms d’API spécifiques. 

Voici un exemple de règle d’alerte qui surveille plusieurs dimensions :

- Ressource cible : *myStorage1*
- Mesure : *Transactions*
- Dimensions
  * API name = *GetBlob, DeleteBlob, PutPage*
- Opérateur : *Supérieur à*
- Seuil : *70*

Pour cette règle d’alerte, trois séries chronologiques de métriques sont surveillées :

- Les transactions où *Resource*= 'myStorage1' et *API Name*= 'GetBlob' > 70
- Les transactions où *Resource*= 'myStorage1' et *API Name*= 'DeleteBlob' > 70
- Les transactions où *Resource*= 'myStorage1' et *API Name*= 'PutPage' > 70

![Règle d’alerte à plusieurs dimensions avec des valeurs d’une dimension](media/alerts-metric-multiple-time-series-single-rule/multi-dimension-1.png)

Une règle d’alerte de métrique à plusieurs dimensions peut également analyser plusieurs valeurs de dimension à partir de **différentes** dimensions d’une mesure. Dans ce cas, la règle d’alerte surveille **séparément** toutes les combinaisons de valeurs de dimensions des valeurs de dimension sélectionnées.

Voici un exemple de ce type d’alerte :

- Ressource cible : *myStorage1*
- Mesure : *Transactions*
- Dimensions
  * API name = *GetBlob, DeleteBlob, PutPage*
  * Authentication = *SAS, AccountKey*
- Opérateur : *Supérieur à*
- Seuil : *70*

Pour cette règle d’alerte, six séries chronologiques de métriques sont surveillées séparément :

- Les transactions où *Resource*= 'myStorage1', *API Name*= 'GetBlob' et *Authentication*=’SAS’ > 70
- Les transactions où *Resource*= 'myStorage1', *API Name*= 'GetBlob' et *Authentication*=’AccountKey’ > 70
- Les transactions où *Resource*= 'myStorage1', *API Name*= 'DeleteBlob' et *Authentication*=’SAS’ > 70
- Les transactions où *Resource*= 'myStorage1', *API Name*= 'DeleteBlob' et *Authentication*=’AccountKey’ > 70
- Les transactions où *Resource*= 'myStorage1', *API Name*= 'PutPage' et *Authentication*=’SAS’ > 70
- Les transactions où *Resource*= 'myStorage1', *API Name*= 'PutPage' et *Authentication*=’AccountKey’ > 70

![Règle d’alerte à plusieurs dimensions avec des valeurs de plusieurs dimensions](media/alerts-metric-multiple-time-series-single-rule/multi-dimension-2.png)
 
### <a name="advanced-multi-dimension-features"></a>Fonctionnalités avancées pour plusieurs dimensions

1.  **Sélection de toutes les dimensions actuelles et futures** : vous pouvez choisir de surveiller toutes les valeurs possibles d’une dimension, y compris les valeurs futures. Une telle règle d’alerte est automatiquement mise à l’échelle pour analyser toutes les valeurs de la dimension sans que vous ayez besoin de modifier la règle d’alerte chaque fois qu’une valeur de dimension est ajoutée ou supprimée.
2.  **Exclusion de dimensions** : la sélection de l’opérateur « ≠ » (exclure) pour une valeur de dimension équivaut à sélectionner toutes les autres valeurs de cette dimension, y compris les valeurs futures.
3.  **Dimensions nouvelles et personnalisées** : les valeurs de dimension affichées dans le portail Azure sont basées sur les données de mesure collectées au cours des trois derniers jours. Si la valeur de dimension que vous recherchez n’est pas encore émise, vous pouvez ajouter une valeur de dimension personnalisée.
4. **Correspondance des dimensions avec un préfixe** : vous pouvez choisir de surveiller toutes les valeurs de dimension qui commencent par un modèle spécifique, en sélectionnant l’opérateur « Commence par » et en entrant un préfixe personnalisé.

![Fonctionnalités avancées pour plusieurs dimensions](media/alerts-metric-multiple-time-series-single-rule/advanced-features.png)


## <a name="metric-alerts-pricing"></a>Tarification des alertes de métriques

La tarification des règles d’alerte de métrique est disponible sur la [page de tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

Lors de la création d’une règle d’alerte de métrique, l’estimation de prix fournie est basée sur les fonctionnalités sélectionnées et le nombre de séries chronologiques analysées, qui est déterminé à partir de la configuration de la règle et des valeurs actuelles des métriques. Toutefois, les frais mensuels sont basés sur les évaluations réelles de la série chronologique et peuvent donc différer de l’estimation d’origine si certaines séries chronologiques n’ont pas de données à évaluer ou si la règle d’alerte utilise des fonctionnalités qui peuvent la faire évoluer dynamiquement.

Par exemple, une règle d’alerte peut afficher une estimation de prix élevée si elle s’appuie sur la fonctionnalité à plusieurs dimensions et qu’un grand nombre de combinaisons de valeurs de dimension sont sélectionnées, ce qui entraîne la surveillance de nombreuses séries chronologiques. Toutefois, les frais réels de cette règle d’alerte peuvent être inférieurs si toutes les séries chronologiques résultant des combinaisons de valeurs de dimension n’ont pas réellement de données à évaluer.

## <a name="number-of-time-series-monitored-by-a-single-alert-rule"></a>Nombre de séries chronologiques analysées par une règle d’alerte unique

Pour éviter les coûts excessifs, chaque règle d’alerte peut surveiller jusqu’à 5 000 séries chronologiques par défaut. Pour lever cette limite à partir de votre abonnement, ouvrez un ticket de support.


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la surveillance à l’échelle à l’aide des alertes de métrique et des [seuils dynamiques](alerts-dynamic-thresholds.md).

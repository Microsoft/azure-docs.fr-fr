---
title: Forum Aux Questions sur Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Forum aux questions sur le service Metrics Advisor.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 11/05/2020
ms.author: mbullwin
ms.openlocfilehash: 0c4c296cb1454ed89eef102732533589b1c8ca0d
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420957"
---
# <a name="metrics-advisor-frequently-asked-questions"></a>Forum Aux Questions sur Metrics Advisor

### <a name="what-is-the-cost-of-my-instance"></a>Quel est le coût de mon instance ?

Il n’y a actuellement pas de coût d’utilisation de votre instance au cours de la préversion.

### <a name="why-cant-i-create-the-resource-the-pricing-tier-is-unavailable-and-it-says-you-have-already-created-1-s0-for-this-subscription"></a>Pourquoi ne puis-je pas créer la ressource ? Le « Niveau tarifaire » n’est pas disponible et il indique « Vous avez déjà créé 1 S0 pour cet abonnement » ?

:::image type="content" source="media/pricing.png" alt-text="Message lorsqu’une ressource F0 existe déjà":::

Pendant la préversion publique, une seule instance de Metrics Advisor peut être créée par région dans le cadre d’un abonnement.

Si vous avez déjà créé une instance dans la même région à l’aide du même abonnement, vous pouvez essayer une autre région ou un autre abonnement pour créer une nouvelle instance. Vous pouvez également supprimer une instance existante pour en créer une nouvelle.

Si vous avez déjà supprimé l’instance existante mais que l’erreur persiste, patientez environ 20 minutes après la suppression de la ressource avant de créer une nouvelle instance.

## <a name="basic-concepts"></a>Concepts de base

### <a name="what-is-multi-dimensional-time-series-data"></a>Qu’est-ce que les données de séries chronologiques multidimensionnelles ?

Consultez la définition de la [Métrique multidimensionnelle](glossary.md#multi-dimensional-metric) dans le glossaire.

### <a name="how-much-data-is-needed-for-metrics-advisor-to-start-anomaly-detection"></a>Quelle quantité de données est nécessaire pour que Metrics Advisor démarrent la détection des anomalies ?

Au minimum, un point de données peut déclencher la détection d’anomalie. Toutefois, cela n’apporte pas la meilleure précision. Le service envisage une fenêtre de points de données précédents en utilisant la valeur que vous avez spécifiée comme règle de remplissage lors de la création du flux de données.

Nous vous recommandons d’avoir des données avant le timestamp sur lequel vous souhaitez effectuer la détection.
En fonction de la granularité de vos données, la quantité de données recommandée varie comme indiqué ci-dessous.

| Granularité | Quantité de données recommandée pour la détection |
| ----------- | ------------------------------------- |
| moins de 5 minutes | 4 jours de données |
| de 5 minutes à 1 jour | 28 jours de données |
| de plus d’1 jour à 31 jours | 4 ans de données |
| plus de 31 jours | 48 jours de données |

### <a name="why-metrics-advisor-doesnt-detect-anomalies-from-historical-data"></a>Pourquoi Metrics Advisor ne détecte-t-il pas les anomalies à partir des données historiques ?

Metrics Advisor est conçu pour détecter les données de streaming en direct. Il existe une limitation de la longueur maximale des données d’historique que le service examinera pour exécuter la détection des anomalies. Cela signifie que seuls les points de données après un timestamp précédent donné auront des résultats de détection d’anomalie. Ce timestamp précédent dépend de la granularité de vos données.

En fonction de la granularité de vos données, les longueurs des données historiques qui auront des résultats de détection d’anomalie sont les suivantes.

| Granularité | Longueur maximale des données d’historique pour la détection d’anomalie |
| ----------- | ------------------------------------- |
| moins de 5 minutes | heure d’intégration : 13 heures |
| de 5 minutes à moins de 1 heure | heure d’intégration : 4 heures  |
| de 1 heure à moins de 1 jour | heure d’intégration : 14 heures  |
| 1 jour | heure d’intégration : 28 heures  |
| de plus d’1 jour à moins de 31 jours | heure d’intégration : 2 heures  |
| plus de 31 jours | heure d’intégration : 24 heures   |

### <a name="more-concepts-and-technical-terms"></a>Plus de concepts et de termes techniques

Pour plus d’informations, consultez également le [Glossaire](glossary.md).

###  <a name="how-do-i-write-a-valid-query-for-ingesting-my-data"></a>Comment écrire une requête valide pour l’ingestion de mes données ?  

Pour que Metrics Advisor ingère vos données, vous devez créer une requête qui retourne les dimensions de vos données à un horodatage unique. Metrics Advisor exécute cette requête plusieurs fois pour obtenir les données de chaque horodatage. 

Notez que la requête doit retourner au plus un enregistrement pour chaque combinaison de dimensions, à un horodatage donné. Tous les enregistrements retournés doivent avoir le même horodatage. Aucun enregistrement en double ne doit être retourné par la requête.

Par exemple, supposons que vous créez la requête ci-dessous, pour une métrique quotidienne : 
 
`select timestamp, city, category, revenue from sampledata where Timestamp >= @StartTime and Timestamp < dateadd(DAY, 1, @StartTime)`

Veillez à utiliser la précision appropriée pour votre série chronologique. Pour une métrique horaire, vous utilisez : 

`select timestamp, city, category, revenue from sampledata where Timestamp >= @StartTime and Timestamp < dateadd(hour, 1, @StartTime)`

Notez que ces requêtes retournent uniquement des données à un horodatage unique et contiennent toutes les combinaisons de dimensions à ingérer par Metrics Advisor. 

:::image type="content" source="media/query-result.png" alt-text="Résultat d’une requête avec un horodatage" lightbox="media/query-result.png":::


### <a name="how-do-i-detect-spikes--dips-as-anomalies"></a>Comment détecter des mines et pixels indépendants du périphérique comme des anomalies ?

Si des seuils définitifs sont prédéfinis, vous pouvez définir manuellement le « seuil définitif » dans [configurations de la détection d’anomalie](how-tos/configure-metrics.md#anomaly-detection-methods).
S’il n’y a aucun seuil, vous pouvez utiliser la « détection intelligente » qui est alimentée par l’intelligence artificielle. Pour plus d’informations, reportez-vous à [régler la configuration de détection](how-tos/configure-metrics.md#tune-the-detecting-configuration).

### <a name="how-do-i-detect-inconformity-with-regular-seasonal-patterns-as-anomalies"></a>Comment faire détecter la non-conformité avec les modèles standard (saisonnier) comme des anomalies ?

La « Détection intelligente » peut apprendre le modèle de vos données, y compris les modèles saisonniers. Elle détecte ensuite les points de données qui ne sont pas conformes aux modèles standard comme des anomalies. Pour plus d’informations, reportez-vous à [régler la configuration de détection](how-tos/configure-metrics.md#tune-the-detecting-configuration).

### <a name="how-do-i-detect-flat-lines-as-anomalies"></a>Comment détecter les lignes plates comme des anomalies ?

Si vos données sont normalement assez instables et fluctuent beaucoup et que vous souhaitez être alerté lorsqu’elles deviennent trop stables ou qu’elles se transforment en une ligne plate, le « Seuil de modification » peut être configuré pour détecter ces points de données lorsque la modification est trop légère.
Pour plus d’informations, consultez les [configurations de détection d’anomalie](how-tos/configure-metrics.md#anomaly-detection-methods).

### <a name="how-to-set-up-email-settings-and-enable-alerting-by-email"></a>Comment configurer des paramètres de messagerie et activer les alertes par e-mail ?

1.  Un utilisateur doté de privilèges d’administrateur d’abonnement ou d’administrateur de groupe de ressources a besoin d’accéder à la ressource Metrics Advisor créée dans le portail Azure, puis de sélectionner l’onglet **Contrôle d’accès (IAM)** . 
2.  Sélectionnez **Ajouter des attributions de rôle**.
3.  Sélectionnez un rôle **Administrateur Metrics Advisor Cognitive Services**, puis sélectionnez votre compte comme dans l’image ci-dessous.
4.  Cliquez sur le bouton **Enregistrer**. Vous avez été correctement ajouté en tant qu’administrateur de ressource Metrics Advisor. Notez que toutes les actions ci-dessus doivent être effectuées par un administrateur d’abonnement ou un administrateur de groupe de ressources. 

:::image type="content" source="media/access-control.png" alt-text="Page du menu Contrôle d’accès (IAM) avec l’option Ajouter une attribution de rôle encadrée, puis une zone avec l’option Attribuer l’accès à définie sur un utilisateur auquel le rôle attribué est Administrateur Metrics Advisor Cognitive Services, puis le bouton Enregistrer de l’interface utilisateur encadré pour illustrer les étapes de la recherche d’un utilisateur et de l’ajout d’un niveau d’autorisation particulier." lightbox="media/access-control.png":::


5.  La propagation des autorisations peut prendre jusqu’à une minute. Ensuite, sélectionnez votre espace de travail Metrics Advisor, puis sélectionnez l’option **Paramètre de messagerie** dans le volet de navigation gauche. Renseignez les éléments nécessaires, en particulier les informations relatives au protocole SMTP. 
6.  Sélectionnez **Enregistrer** et vous avez terminé de configurer la messagerie. Vous pouvez créer des hooks et vous abonner à des anomalies de métrique pour obtenir des alertes en quasi-temps réel. 

## <a name="advanced-concepts"></a>Concepts avancés

### <a name="how-does-metric-advisor-build-an-incident-tree-for-multi-dimensional-metrics"></a>Comment Metric Advisor crée-t-il une arborescence de l’incident pour les métriques multidimensionnelles ?

Une métrique peut être fractionnée en plusieurs séries chronologiques par dimensions. Par exemple, la métrique `Response latency` est surveillée pour tous les services appartenant à l’équipe. La catégorie `Service` peut être utilisée comme dimension pour enrichir la métrique, de sorte que `Response latency` est fractionnée par `Service1`, `Service2`, etc. Chaque service pouvant être déployé sur différents ordinateurs dans plusieurs centres de données, la métrique peut être divisée davantage par `Machine` et `Data center`.

|Service| Centre de données| Machine  | 
|----|------|----------------   |
| S1 |  DC1 |   M1 |
| S1 |  DC1 |   M2 |
| S1 |  DC2 |   M3 |
| S1 |  DC2 |   M4 |
| S2 |  DC1 |   M1 |
| S2 |  DC1 |   M2 |
| S2 |  DC2 |   M5 |
| S2 |  DC2 |   M6 |
| ...|      |      |

À partir de la `Response latency` totale, nous pouvons explorer la mesure au niveau du détail par `Service`, `Data center` et `Machine`. Toutefois, il est peut-être plus judicieux pour les propriétaires de service d’utiliser le chemin d’accès `Service` -> `Data center` -> `Machine`, ou plus logique pour les ingénieurs d’infrastructure d’utiliser le chemin d’accès `Data Center` -> `Machine` -> `Service`. Tout dépend des besoins professionnels de vos utilisateurs. 

Dans Metric Advisor, les utilisateurs peuvent spécifier n’importe quel chemin d’accès qu’ils veulent explorer au niveau du détail ou dont ils veulent remonter à partir d’un nœud de la topologie hiérarchique. Plus précisément, la topologie hiérarchique est un graphe orienté acyclique plutôt qu’une arborescence. Il existe une topologie hiérarchique complète qui se compose de toutes les combinaisons de dimensions potentielles, comme celle-ci : 

:::image type="content" source="media/dimension-combinations-view.png" alt-text="Diagramme de topologie hiérarchique constitué de plusieurs sommets et arêtes interconnectés à dimensions multiples étiquetées S, DC et M avec des numéros correspondants allant de 1 à 6" lightbox="media/dimension-combinations-view.png":::

En théorie, si la dimension `Service` a `Ls` valeurs distinctes, que la dimension `Data center` a `Ldc` valeurs distinctes et que la dimension `Machine` a `Lm` valeurs distinctes, alors il peut y avoir `(Ls + 1) * (Ldc + 1) * (Lm + 1)` combinaisons de dimensions dans la topologie hiérarchique. 

Mais généralement, toutes les combinaisons de dimensions ne sont pas valides, ce qui peut réduire considérablement la complexité. Actuellement, si les utilisateurs agrègent eux-mêmes la métrique, nous ne limitons pas le nombre de dimensions. Si vous devez utiliser les fonctionnalités de cumul fournies par Metrics Advisor, le nombre de dimensions ne devrait pas être supérieur à 6. Toutefois, nous limitons à moins de 10 000 le nombre de séries chronologiques développées par des dimensions pour une métrique.

L’outil **Arborescence de l’incident** de la page de diagnostics affiche uniquement les nœuds dans lesquels une anomalie a été détectée, plutôt que la topologie entière. Cela vous permet de vous concentrer sur le problème actuel. Il se peut également qu’il ne montre pas toutes les anomalies dans la métrique et affiche plutôt les principales anomalies en fonction de leur contribution. De cette façon, nous pouvons rapidement déterminer l’impact, la portée et le chemin de propagation des données anormales. Cela réduit considérablement le nombre d’anomalies sur lesquelles nous devons nous concentrer et aide les utilisateurs à comprendre et à localiser leurs problèmes clés. 
 
Par exemple, lorsqu’une anomalie se produit sur `Service = S2 | Data Center = DC2 | Machine = M5`, l’écart de l’anomalie a un impact sur le nœud parent `Service= S2` qui a également détecté l’anomalie, mais l’anomalie ne concerne pas l’ensemble du centre de données au niveau de `DC2` ni tous les services sur `M5`. L’arborescence de l’incident est générée comme dans la capture d’écran ci-dessous : la première anomalie est capturée sur `Service = S2` et la cause racine peut être analysée dans deux chemins qui mènent tous deux à `Service = S2 | Data Center = DC2 | Machine = M5`.

 :::image type="content" source="media/root-cause-paths.png" alt-text="Cinq sommets étiquetés avec deux chemins distincts reliés par des arêtes avec un nœud commun étiqueté S2. La première anomalie est capturée sur Service = S2 et la cause racine peut être analysée par les deux chemins qui mènent tous deux à Service = S2 | Data Center = DC2 | Machine = M5" lightbox="media/root-cause-paths.png":::

## <a name="next-steps"></a>Étapes suivantes
- [Présentation de Metrics Advisor](overview.md)
- [Utiliser le Portail web](quickstarts/web-portal.md)
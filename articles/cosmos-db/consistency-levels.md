---
title: Niveaux de cohérence dans Azure Cosmos DB
description: Azure Cosmos DB offre cinq niveaux de cohérence qui permettent de faire des compromis avisés entre cohérence éventuelle, disponibilité et latence.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 12/09/2020
ms.openlocfilehash: a480c8f2dfdda0ce7a1eb879554fb79c96adbe1e
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347810"
---
# <a name="consistency-levels-in-azure-cosmos-db"></a>Niveaux de cohérence dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Les bases de données distribuées qui reposent sur la réplication afin d’offrir une haute disponibilité, une faible latence ou les deux, constituent le compromis fondamental entre la cohérence de la lecture et la disponibilité, la latence et le débit comme déterminé par le [théorème PACLC](https://en.wikipedia.org/wiki/PACELC_theorem). La linéarisabilité du modèle de cohérence fort constitue la référence en matière de programmabilité des données. Mais cela augmente considérablement le coût des latences d’écriture en raison des données qui doivent être répliquées et validées sur de grandes distances. Une cohérence forte peut également souffrir d’une disponibilité réduite (pendant les défaillances), car les données ne peuvent pas être répliquées et validées dans chaque région. La cohérence éventuelle offre une disponibilité accrue et de meilleures performances, mais il est plus difficile de programmer des applications, car les données peuvent ne pas être entièrement cohérentes dans toutes les régions.

La plupart des bases de données NoSQL distribuées disponibles sur le marché offrent uniquement une cohérence forte et éventuelle. Azure Cosmos DB offre cinq niveaux bien définis. De la plus forte à la plus faible cohérence, les niveaux sont les suivants :

- *Fort*
- *Obsolescence limitée*
- *Session*
- *Préfixe cohérent*
- *Éventuel*

Chaque niveau propose des compromis entre disponibilité et performances. L’illustration suivante montre les différents niveaux de cohérence en tant que spectre.

:::image type="content" source="./media/consistency-levels/five-consistency-levels.png" alt-text="Spectre de la cohérence" border="false" :::

Les niveaux de cohérence sont indépendants des régions et garantis pour toutes les opérations, indépendamment de la région d’émission des lectures et des écritures, du nombre de régions associées à votre compte Azure Cosmos ou de la configuration de votre compte avec une ou plusieurs régions d’écriture.

## <a name="consistency-levels-and-azure-cosmos-db-apis"></a>Niveaux de cohérence et API Azure Cosmos DB

Azure Cosmos DB fournit la prise en charge native des API compatibles avec les protocoles filaires pour les bases de données les plus courantes. Ces bases de données incluent le stockage Table Azure, MongoDB, Apache Cassandra et Gremlin. Pour les API Gremlin et Table, le niveau de cohérence configuré par défaut sur le compte Azure Cosmos est utilisé. Pour plus d’informations sur le mappage des niveaux de cohérence entre l’API Cassandra ou l’API pour MongoDB et les niveaux de cohérence d’Azure Cosmos DB, consultez [Mappage de cohérence pour l’API Cassandra](cassandra-consistency.md) et [Mappage de cohérence pour l’API MongoDB](mongodb-consistency.md).

## <a name="scope-of-the-read-consistency"></a>Étendue de la cohérence de lecture

La cohérence de lecture s’applique à une même opération de lecture dans une partition logique. L’opération de lecture peut être émise par un client distant ou une procédure stockée.

## <a name="configure-the-default-consistency-level"></a>Configurer le niveau de cohérence par défaut

Vous pouvez configurer le niveau de cohérence par défaut sur votre compte Azure Cosmos DB à tout moment. Le niveau de cohérence par défaut configuré sur votre compte s’applique à toutes les bases de données Azure Cosmos et tous les conteneurs sous ce compte. Toutes les lectures et requêtes émises vers un conteneur ou une base de données utilisent le niveau de cohérence par défaut spécifié. Pour en savoir plus, consultez [Configurer le niveau de cohérence par défaut](how-to-manage-consistency.md#configure-the-default-consistency-level). Vous pouvez également remplacer le niveau de cohérence par défaut pour une requête spécifique. Pour plus d’informations, consultez [Guide pratique pour remplacer le niveau de cohérence par défaut](how-to-manage-consistency.md?#override-the-default-consistency-level).

> [!IMPORTANT]
> Il est nécessaire de recréer une instance de Kit de développement logiciel (SDK) après avoir modifié le niveau de cohérence par défaut. Pour ce faire, vous pouvez redémarrer l’application. Cela garantit que le Kit de développement logiciel (SDK) utilise le nouveau niveau de cohérence par défaut.

## <a name="guarantees-associated-with-consistency-levels"></a>Garanties associées aux niveaux de cohérence

Azure Cosmos DB garantit que 100 % des requêtes de lecture respecteront la garantie de cohérence dans le cadre du niveau de cohérence choisi. Les définitions précises des cinq niveaux de cohérence dans Azure Cosmos DB (en utilisant le langage de spécification TLA +) sont fournies dans le référentiel GitHub [azure-cosmos-tla](https://github.com/Azure/azure-cosmos-tla).

La sémantique des cinq niveaux de cohérence est décrite ici :

- **Remarque**: une cohérence forte offre une garantie de linéarisabilité. La linéarisabilité fait référence aux demandes de traitement simultanées. Garantit que les lectures retournent la version validée la plus récente d’un élément. Un client ne voit jamais une écriture partielle ou non validée. Les utilisateurs sont toujours assurés de lire la toute dernière écriture validée.

  Le graphique suivant illustre la cohérence forte avec des notes musicales. Une fois les données écrites dans la région « USA Ouest 2 », quand vous lisez les données à partir d’autres régions, vous obtenez la valeur la plus récente :

  :::image type="content" source="media/consistency-levels/strong-consistency.gif" alt-text="Illustration d’un niveau de cohérence fort":::

- **Obsolescence limitée**: les lectures honoreront la garantie de préfixe cohérent. Les lectures risquent d’accuser un retard par rapport aux écritures d’au maximum *« K »* versions (c’est-à-dire « mises à jour ») d’un élément ou d’un intervalle de temps *« T »* , suivant le seuil qui est atteint en premier. En d’autres termes, lorsque vous choisissez l’obsolescence limitée, « l’obsolescence » peut être configurée de deux manières :

- Nombre de versions (*K*) de l’élément
- Intervalle de temps (*T*) pendant lequel les lectures peuvent être en retard par rapport aux écritures

Pour un compte à région unique, la valeur minimale de *K* et de *T* est de 10 opérations d’écriture ou de 5 secondes. Pour les comptes à plusieurs régions, la valeur minimale de *K* et *T* est de 100 000 opérations d’écriture ou de 300 secondes.

La cohérence de type Obsolescence limitée offre un ordre global total, en dehors de la « fenêtre d’obsolescence ». Lorsqu’un client effectue des opérations de lecture dans une région acceptant les écritures, les garanties fournies par une cohérence à obsolescence limitée sont identiques à celles à forte cohérence. Quand la fenêtre d’obsolescence s’approche pour l’heure ou les mises à jour, selon la valeur la plus proche, le service limite les nouvelles écritures pour permettre à la réplication de rattraper son retard et d’honorer la garantie de cohérence.

À l’intérieur de la fenêtre d’obsolescence, l’obsolescence limitée fournit les garanties de cohérence suivantes :

- Cohérence pour les clients de la même région pour un compte à une seule région d’écriture = Forte
- Cohérence pour les clients de régions différentes pour un compte à une seule région d’écriture = Préfixe cohérent
- Cohérence pour les clients écrivant dans une région unique pour un compte à plusieurs régions d’écriture = Préfixe cohérent
- Cohérence pour les clients écrivant dans différentes régions pour un compte à plusieurs régions d’écriture = Éventuelle

  L’obsolescence limitée est souvent choisie par des applications mondialement distribuées qui souhaitent de faibles latences en écriture, mais nécessitent des garanties d’ordre totales à l’échelle mondiale. L’obsolescence limitée est idéale pour les applications qui incluent la collaboration et le partage de groupes, les cotations boursières, la publication-abonnement/la mise en file d’attente, etc. Le graphique suivant illustre la cohérence à obsolescence limitée avec des notes musicales. Une fois les données écrites dans la région « USA Ouest 2 », les régions « USA Est 2 » et « Australie Est » lisent la valeur écrite en fonction de la durée de latence maximum configurée ou du nombre maximal d’opérations :

  :::image type="content" source="media/consistency-levels/bounded-staleness-consistency.gif" alt-text="Illustration du niveau de cohérence d’obsolescence limitée":::

- **Session**:  Dans une session à un seul client, il est garanti que les lectures honorent le préfixe cohérent, les lectures unitones, les écritures unitones, les lectures de vos écritures et les écritures suivant les lectures. Cela suppose une session avec un seul « processus d’écriture » ou le partage du jeton de session pour plusieurs processus d’écriture.

Les clients en dehors de la session effectuant des écritures verront les garanties suivantes :

- Cohérence pour les clients de la même région pour un compte à une seule région d’écriture = Préfixe cohérent
- Cohérence pour les clients de régions différentes pour un compte à une seule région d’écriture = Préfixe cohérent
- Cohérence pour les clients écrivant dans une région unique pour un compte à plusieurs régions d’écriture = Préfixe cohérent
- Cohérence pour les clients écrivant dans plusieurs régions pour un compte à plusieurs régions d’écriture = Éventuelle

  La cohérence de session est le niveau de cohérence le plus utilisé pour les applications limitées à une seule région comme pour les applications distribuées dans le monde entier. Elle propose des latences en écriture, une disponibilité et un débit de lecture comparables à ceux de la cohérence éventuelle, mais fournit aussi des garanties de cohérence qui répondent aux besoins des applications écrites pour agir dans le contexte de l’utilisateur. Le graphique suivant illustre la cohérence de session avec des notes musicales. Le « processus d’écriture USA Ouest 2 » et le « processus de lecture USA Est 2 » utilisent la même session (session A) et lisent donc les mêmes données en même temps. En revanche, la région « Australie Est » utilise « Session B » et reçoit les données plus tard, mais dans le même ordre que les écritures.

  :::image type="content" source="media/consistency-levels/session-consistency.gif" alt-text="Illustration du niveau de cohérence de session":::

- **Préfixe cohérent** : les mises à jour retournées contiennent un préfixe de toutes les mises à jour, sans interruption. Le niveau de cohérence Préfixe cohérent garantit que les lectures ne voient jamais les écritures non ordonnées.

Si les écritures ont été effectuées dans l’ordre `A, B, C`, un client peut voir `A`, `A,B` ou `A,B,C`, mais jamais de permutations dans le désordre comme `A,C` ou `B,A,C`. La cohérence avec préfixe cohérent fournit des latences d’écriture, une disponibilité et un débit de lecture comparables à ceux de la cohérence éventuelle, mais offre également des garanties d’ordre adaptées aux besoins des scénarios dans lesquels l’ordre est important.

Voici les garanties de cohérence pour le niveau Préfixe cohérent :

- Cohérence pour les clients de la même région pour un compte à une seule région d’écriture = Préfixe cohérent
- Cohérence pour les clients de régions différentes pour un compte à une seule région d’écriture = Préfixe cohérent
- Cohérence pour les clients écrivant dans une région unique pour un compte à plusieurs régions d’écriture = Préfixe cohérent
- Cohérence pour les clients écrivant dans plusieurs régions pour un compte à plusieurs régions d’écriture = Éventuelle

Le graphique suivant illustre la cohérence avec préfixe cohérent avec des notes musicales. Dans toutes les régions, les lectures ne voient jamais d’écritures dans le désordre :

  :::image type="content" source="media/consistency-levels/consistent-prefix.gif" alt-text="Illustration d’un préfixe cohérent":::

- **Eventual (Éventuel)** : Il n’existe aucune garantie de classement pour les lectures. En l’absence d’autres écritures, les réplicas finissent par converger.  
La cohérence éventuelle est la forme de cohérence la plus faible, car un client peut lire des valeurs plus anciennes que celles qu’il a déjà lues. La cohérence éventuelle est idéale lorsque l’application ne nécessite pas de garantie d’ordre. Comme exemples, citons le nombre de retweets, de mentions J’aime ou de commentaires non liés à un thread. Le graphique suivant illustre la cohérence éventuelle avec des notes musicales.

  :::image type="content" source="media/consistency-levels/eventual-consistency.gif" alt-text="Illustration de la cohérence éventuelle":::

## <a name="consistency-guarantees-in-practice"></a>Garanties de cohérence en pratique

En pratique, vous pouvez obtenir de meilleures garanties de cohérence. Les garanties de cohérence pour une opération de lecture correspondent à l’actualisation et au classement de l’état de la base de données que vous demandez. La cohérence de lecture est liée au classement et à la propagation des opérations d’écriture/mise à jour.  

Si aucune opération d’écriture n’est effectuée sur la base de données, une opération de lecture avec un niveau de cohérence **éventuelle**, **session** ou **préfixe cohérent** risque de produire les mêmes résultats qu’une opération de lecture avec un niveau de cohérence forte.

Si votre compte Azure Cosmos est configuré avec un niveau de cohérence autre que la cohérence forte, vous pouvez déterminer la probabilité que vos clients bénéficieront de lectures fortes et cohérentes pour vos charges de travail en examinant la métrique *Probabilistically Bounded Staleness* (PBS). Cette métrique est exposée dans le portail Azure. Pour en savoir plus, consultez [Surveiller la métrique de probabilités en fonction de l’obsolescence limitée (PBS)](how-to-manage-consistency.md#monitor-probabilistically-bounded-staleness-pbs-metric).

La métrique de probabilités en fonction de l’obsolescence limitée montre comment la valeur éventuelle représente la cohérence éventuelle. Cette métrique fournit une idée de la fréquence à laquelle vous pouvez obtenir une cohérence plus forte que le niveau de cohérence que vous avez configuré sur votre compte Azure Cosmos. En d’autres termes, vous pouvez voir la probabilité (mesurée en millisecondes) d’obtenir des lectures fortement cohérentes pour une combinaison de régions d’écriture et de lecture.

## <a name="consistency-levels-and-latency"></a>Niveaux de cohérence et latence

La latence de lecture est toujours garantie inférieure à 10 millisecondes au 99e centile pour tous les niveaux de cohérence. La latence de lecture moyenne (au 50e centile) est généralement inférieure ou égale à 4 millisecondes.

La latence d’écriture est toujours garantie inférieure à 10 millisecondes au 99e centile pour tous les niveaux de cohérence. La latence d’écriture moyenne (au 50e centile) est généralement inférieure ou égale à 5 millisecondes. Les comptes Azure Cosmos qui s’étendent sur plusieurs régions et sont configurés avec une cohérence forte constituent une exception à cette garantie.

### <a name="write-latency-and-strong-consistency"></a>Latence d’écriture et cohérence forte

Pour les comptes Azure Cosmos configurés avec une cohérence forte et plusieurs régions, la latence d’écriture est égale à deux allers-retours (RTT) entre deux des régions les plus éloignées, plus 10 millisecondes au 99e centile. Un temps d’aller-retour réseau élevé entre les régions se traduira par une latence plus élevée pour les requêtes Cosmos DB, car la cohérence forte effectue une opération seulement après avoir vérifié qu’elle a été validée dans toutes les régions d’un compte.

La latence exacte de la durée des boucles s’exprime en fonction de la distance à la vitesse de la lumière et de la topologie de réseau Azure. La mise en réseau Azure ne fournit pas de contrats SLA de latence pour le RTT entre deux régions Azure. Toutefois, il publie des [Statistiques de latence aller-retour réseau Azure](../networking/azure-network-latency.md). Pour votre compte Azure Cosmos, les latences de réplication sont affichées dans le portail Azure. Vous pouvez utiliser le portail Azure (accédez au panneau Métriques, puis sélectionnez l’onglet Cohérence) pour superviser les latences de réplication entre les différentes régions associées à votre compte Azure Cosmos.

> [!IMPORTANT]
> Une cohérence forte pour les comptes dont les régions sont distantes de plus de 8 000 kilomètres est bloquée par défaut en raison d’une latence d’écriture élevée. Pour activer cette fonctionnalité, contactez le support technique.

## <a name="consistency-levels-and-throughput"></a>Niveaux de cohérence et débit

- Pour une obsolescence forte et limitée, les lectures sont effectuées sur deux réplicas dans un jeu de quatre réplicas (quorum minoritaire) pour fournir des garanties de cohérence. Les niveaux Session, Préfixe cohérent et À terme font des lectures sur un seul réplica. Le résultat est que, pour le même nombre d’unités de requête, le débit de lecture pour l’obsolescence forte et limitée est la moitié de celui des autres niveaux de cohérence.

- Pour un type donné d’opération d’écriture, tel qu’insert, replace, upsert et delete, le débit d’écriture des unités de requête est identique pour tous les niveaux de cohérence.

|**Niveau de cohérence**|**Lectures de quorum**|**Écritures de quorum**|
|--|--|--|
|**Fort**|Minorité locale|Majorité globale|
|**Obsolescence limitée**|Minorité locale|Majorité locale|
|**Session**|Réplica unique (avec un jeton de session)|Majorité locale|
|**Préfixe cohérent**|Réplica unique|Majorité locale|
|**Éventuel**|Réplica unique|Majorité locale|

> [!NOTE]
> Le coût des lectures en RU/s pour les lectures en minorité locale est le double des niveaux de cohérence plus faibles, car les lectures sont effectuées à partir de deux réplicas pour fournir des garanties de cohérence en cas d’obsolescence forte et limitée.

## <a name="consistency-levels-and-data-durability"></a><a id="rto"></a>Niveaux de cohérence et durabilité des données

Dans un environnement de base de données globalement distribuée, il existe une relation directe entre le niveau de cohérence et la durabilité des données en situation de panne à l'échelle d'une région. Au moment de l'élaboration de votre plan de continuité d'activité, vous devez identifier le délai maximal acceptable nécessaire à la récupération complète de l'application après un événement perturbateur. Ce délai s’appelle l’**objectif de délai de récupération** (**RTO**, recovery time objective). Vous devez également déterminer sur quelle période maximale l'application peut accepter de perdre les mises à jour de données récentes lors de la récupération après l'événement perturbateur. Il s’agit de l’**objectif de point de récupération** (**RPO**, recovery point objective).

Le tableau ci-dessous définit la relation entre le modèle de cohérence et la durabilité des données en situation de panne à l’échelle d’une région. Il est important de noter que dans un système distribué, même avec une cohérence forte, il s’avère impossible d’avoir une base de données distribuée avec un RPO de zéro en raison du [théorème CAP](https://en.wikipedia.org/wiki/CAP_theorem).

|**Région(s)**|**Mode de réplication**|**Niveau de cohérence**|**RPO**|**RTO**|
|---------|---------|---------|---------|---------|
|1|Une ou plusieurs régions d’écriture|Tous les niveaux de cohérence|< 240 minutes|< 1 semaine|
|>1|Région d’écriture unique|Session, Préfixe cohérent et Éventuel|< 15 minutes|< 15 minutes|
|>1|Région d’écriture unique|Obsolescence limitée|*K* & *T*|< 15 minutes|
|>1|Région d’écriture unique|Remarque|0|< 15 minutes|
|>1|Régions d’écriture multiples|Session, Préfixe cohérent et Éventuel|< 15 minutes|0|
|>1|Régions d’écriture multiples|Obsolescence limitée|*K* & *T*|0|

*K* = nombre de versions *« K »* (à savoir, mises à jour) d’un élément.

*T* = intervalle de temps *« T »* depuis la dernière mise à jour.

Pour un compte à région unique, la valeur minimale de *K* et de *T* est de 10 opérations d’écriture ou de 5 secondes. Pour les comptes à plusieurs régions, la valeur minimale de *K* et *T* est de 100 000 opérations d’écriture ou de 300 secondes. Cela définit le RPO minimal pour les données lors de l’utilisation de l’obsolescence limitée.

## <a name="strong-consistency-and-multiple-write-regions"></a>Cohérence forte et régions d’écriture multiples

Les comptes Cosmos configurés pour avec plusieurs régions d’écriture ne peuvent pas être configurés pour une cohérence forte, car il n’est pas possible qu’un système distribué fournisse un objectif de point de récupération (RPO) et un objectif de délai de récupération (RTO) tous de valeur zéro. En outre, il n’y a pas d’avantage en termes de latence d’écriture à utiliser une cohérence forte avec les régions d’écriture multiples, car l’écriture dans une région doit être répliquée et validée dans toutes les régions configurées au sein du compte. Cela aboutit à une latence d’écriture identique à celle d’une région d’écriture unique.

## <a name="additional-reading"></a>Documentation supplémentaire

Pour en savoir plus sur les concepts de cohérence, lisez les articles suivants :

- [Spécifications TLA + de haut niveau pour les cinq niveaux de cohérence proposés par Azure Cosmos DB](https://github.com/Azure/azure-cosmos-tla)
- [La cohérence des données répliquées expliquée par le base-ball (vidéo) par Doug Terry](https://www.youtube.com/watch?v=gluIh8zd26I)
- [La cohérence des données répliquées expliquée par le base-ball (livre blanc) par Doug Terry](https://www.microsoft.com/research/publication/replicated-data-consistency-explained-through-baseball/)
- [Garanties de session pour des données répliquées peu cohérentes](https://dl.acm.org/citation.cfm?id=383631)
- [Compromis de cohérence en termes de conception de systèmes de base de données distribuée modernes : CAP n’est qu’une partie de l’histoire](https://www.computer.org/csdl/magazine/co/2012/02/mco2012020037/13rRUxjyX7k)
- [Probabilistic Bounded Staleness (PBS) for Practical Partial Quorums](https://vldb.org/pvldb/vol5/p776_peterbailis_vldb2012.pdf) (Probabilités en fonction de l’obsolescence limitée (PBS) pour les quorums partiels pratiques)
- [Niveau de cohérence éventuel repensé](https://www.allthingsdistributed.com/2008/12/eventually_consistent.html)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les niveaux de cohérence dans Azure Cosmos DB, lisez les articles suivants :

- [Configurer le niveau de cohérence par défaut](how-to-manage-consistency.md#configure-the-default-consistency-level)
- [Remplacer le niveau de cohérence par défaut](how-to-manage-consistency.md#override-the-default-consistency-level)
- [SLA pour Azure Cosmos DB](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_3/)

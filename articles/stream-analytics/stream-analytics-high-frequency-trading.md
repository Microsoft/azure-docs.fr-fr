---
title: Échange à fréquence élevée à l’aide d’Azure Stream Analytics
description: Comment effectuer l’apprentissage de modèle de régression linéaire et la notation dans une tâche Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/07/2018
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: e5d346a6f412b5764400a42e2bebbafec610009b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89015422"
---
# <a name="high-frequency-trading-simulation-with-stream-analytics"></a>Simulation d’échange à fréquence élevée avec Stream Analytics
La combinaison du langage SQL, des fonctions JavaScript définies par l’utilisateur (UDF) et des agrégats définis par l’utilisateur (UDA) dans Azure Stream Analytics permet aux utilisateurs d’effectuer des analyses avancées. Les analyses avancées peuvent inclure l’apprentissage automatique en ligne et la notation, ainsi que la simulation des processus avec état. Cet article décrit comment effectuer une régression linéaire dans une tâche Azure Stream Analytics qui exécute la formation continue et la notation dans un scénario commercial haute fréquence.

## <a name="high-frequency-trading"></a>Échange à fréquence élevée
Le flux logique des échanges à fréquence élevée concerne :
1. L’obtention de devis en temps réel à partir d’un échange de sécurité.
2. La création d’un modèle prédictif autour des devis, afin d’anticiper les mouvements de prix.
3. Le placement d’ordres d’achat ou de vente pour générer des revenus à partir de la prédiction réussie des mouvements de prix. 

Par conséquent, nous avons besoin :
* D’un flux de devis en temps réel.
* D’un modèle de prévision pouvant opérer sur les devis en temps réel.
* D’une simulation commerciale illustrant les profits ou les pertes de l’algorithme commercial.

### <a name="real-time-quote-feed"></a>Flux de devis en temps réel
IEX permet l’affichage gratuit des [offres en temps réel et la demande de devis](https://iextrading.com/developer/docs/#websockets) à l’aide de socket.io. Un programme de console simple peut être écrit pour recevoir les devis en temps réel et les transmettre aux Event Hubs Azure en tant que source de données. Le code suivant est la structure du programme. Le code omet la gestion des erreurs pour des raisons de concision. Vous devez également inclure les packages NuGet SocketIoClientDotNet et WindowsAzure.ServiceBus dans votre projet.

```csharp
using Quobject.SocketIoClientDotNet.Client;
using Microsoft.ServiceBus.Messaging;
var symbols = "msft,fb,amzn,goog";
var eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, eventHubName);
var socket = IO.Socket("https://ws-api.iextrading.com/1.0/tops");
socket.On(Socket.EVENT_MESSAGE, (message) =>
{
    eventHubClient.Send(new EventData(Encoding.UTF8.GetBytes((string)message)));
});
socket.On(Socket.EVENT_CONNECT, () =>
{
    socket.Emit("subscribe", symbols);
});
```

Voici quelques exemples d’événements générés :

```json
{"symbol":"MSFT","marketPercent":0.03246,"bidSize":100,"bidPrice":74.8,"askSize":300,"askPrice":74.83,volume":70572,"lastSalePrice":74.825,"lastSaleSize":100,"lastSaleTime":1506953355123,lastUpdated":1506953357170,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"GOOG","marketPercent":0.04825,"bidSize":114,"bidPrice":870,"askSize":0,"askPrice":0,volume":11240,"lastSalePrice":959.47,"lastSaleSize":60,"lastSaleTime":1506953317571,lastUpdated":1506953357633,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"MSFT","marketPercent":0.03244,"bidSize":100,"bidPrice":74.8,"askSize":100,"askPrice":74.83,volume":70572,"lastSalePrice":74.825,"lastSaleSize":100,"lastSaleTime":1506953355123,lastUpdated":1506953359118,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"FB","marketPercent":0.01211,"bidSize":100,"bidPrice":169.9,"askSize":100,"askPrice":170.67,volume":39042,"lastSalePrice":170.67,"lastSaleSize":100,"lastSaleTime":1506953351912,lastUpdated":1506953359641,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"GOOG","marketPercent":0.04795,"bidSize":100,"bidPrice":959.19,"askSize":0,"askPrice":0,volume":11240,"lastSalePrice":959.47,"lastSaleSize":60,"lastSaleTime":1506953317571,lastUpdated":1506953360949,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"FB","marketPercent":0.0121,"bidSize":100,"bidPrice":169.9,"askSize":100,"askPrice":170.7,volume":39042,"lastSalePrice":170.67,"lastSaleSize":100,"lastSaleTime":1506953351912,lastUpdated":1506953362205,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"GOOG","marketPercent":0.04795,"bidSize":114,"bidPrice":870,"askSize":0,"askPrice":0,volume":11240,"lastSalePrice":959.47,"lastSaleSize":60,"lastSaleTime":1506953317571,lastUpdated":1506953362629,"sector":"softwareservices","securityType":"commonstock"}
```

>[!NOTE]
>L’horodatage de l’événement est **lastUpdated**, en heure d’époque.

### <a name="predictive-model-for-high-frequency-trading"></a>Modèle de prédictif pour les échanges à fréquence élevée
À des fins de démonstration, nous utilisons un modèle linéaire décrit par Darryl Shen dans [sa thèse](https://docplayer.net/23038840-Order-imbalance-based-strategy-in-high-frequency-trading.html).

Le déséquilibre du volume de commande (VOI) est une fonction d’offres/demandes qui concerne les prix et le volume, elle s’applique aux offres/demandes en cours ou depuis le dernier cycle. Dans la thèse, sont identifiées les corrélations entre le déséquilibre du volume de commande et les mouvements de prix futurs. On y distingue un modèle linéaire entre les 5 dernières valeurs du VOI et les mouvements de prix des 10 cycles suivants. L’apprentissage du modèle est effectué à l’aide des données du jour précédent et de la régression linéaire. 

Le modèle formé est ensuite utilisé pour effectuer des prédictions de prix sur les devis du jour de bourse actuel, en temps réel. Lorsqu’une variation de prix suffisamment importante est prévue, une transaction est exécutée. Selon le paramètre de seuil, des milliers de transactions peuvent se produire pour une action unique en un seul jour de bourse.

![Définition de déséquilibre de commande de volume](./media/stream-analytics-high-frequency-trading/volume-order-imbalance-formula.png)

Maintenant, nous allons exprimer les opérations d’apprentissage et de prédiction dans une tâche Azure Stream Analytics.

Tout d’abord, les entrées sont nettoyées. L’heure d’époque est convertie au format datetime via **DATEADD**. **TRY_CAST** est utilisé pour convertir de force les types de données sans faire échouer la requête. Il est toujours conseillé d’effectuer une conversion de type des champs d’entrée vers les types de données attendus ; cela permet d’éviter les comportements inattendus lors de la manipulation ou de la comparaison des champs.

```SQL
WITH
typeconvertedquotes AS (
    /* convert all input fields to proper types */
    SELECT
        System.Timestamp AS lastUpdated,
        symbol,
        DATEADD(millisecond, CAST(lastSaleTime as bigint), '1970-01-01T00:00:00Z') AS lastSaleTime,
        TRY_CAST(bidSize as bigint) AS bidSize,
        TRY_CAST(bidPrice as float) AS bidPrice,
        TRY_CAST(askSize as bigint) AS askSize,
        TRY_CAST(askPrice as float) AS askPrice,
        TRY_CAST(volume as bigint) AS volume,
        TRY_CAST(lastSaleSize as bigint) AS lastSaleSize,
        TRY_CAST(lastSalePrice as float) AS lastSalePrice
    FROM quotes TIMESTAMP BY DATEADD(millisecond, CAST(lastUpdated as bigint), '1970-01-01T00:00:00Z')
),
timefilteredquotes AS (
    /* filter between 7am and 1pm PST, 14:00 to 20:00 UTC */
    /* clean up invalid data points */
    SELECT * FROM typeconvertedquotes
    WHERE DATEPART(hour, lastUpdated) >= 14 AND DATEPART(hour, lastUpdated) < 20 AND bidSize > 0 AND askSize > 0 AND bidPrice > 0 AND askPrice > 0
),
```

Ensuite, nous utilisons la fonction **LAG** pour obtenir les valeurs depuis le dernier cycle. La durée d’une heure pour la valeur **LIMIT DURATION** est choisie arbitrairement. Étant donné la fréquence des devis, on peut supposer que vous trouverez le cycle précédent en retournant en arrière d’une heure.  

```SQL
shiftedquotes AS (
    /* get previous bid/ask price and size in order to calculate VOI */
    SELECT
        symbol,
        (bidPrice + askPrice)/2 AS midPrice,
        bidPrice,
        bidSize,
        askPrice,
        askSize,
        LAG(bidPrice) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS bidPricePrev,
        LAG(bidSize) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS bidSizePrev,
        LAG(askPrice) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS askPricePrev,
        LAG(askSize) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS askSizePrev
    FROM timefilteredquotes
),
```

Nous pouvons ensuite calculer la valeur pour VOI. Nous filtrons les valeurs null si le cycle précédent n’existe pas, au cas où.

```SQL
currentPriceAndVOI AS (
    /* calculate VOI */
    SELECT
        symbol,
        midPrice,
        (CASE WHEN (bidPrice < bidPricePrev) THEN 0
            ELSE (CASE WHEN (bidPrice = bidPricePrev) THEN (bidSize - bidSizePrev) ELSE bidSize END)
         END) -
        (CASE WHEN (askPrice < askPricePrev) THEN askSize
            ELSE (CASE WHEN (askPrice = askPricePrev) THEN (askSize - askSizePrev) ELSE 0 END)
         END) AS VOI
    FROM shiftedquotes
    WHERE
        bidPrice IS NOT NULL AND
        bidSize IS NOT NULL AND
        askPrice IS NOT NULL AND
        askSize IS NOT NULL AND
        bidPricePrev IS NOT NULL AND
        bidSizePrev IS NOT NULL AND
        askPricePrev IS NOT NULL AND
        askSizePrev IS NOT NULL
),
```

À présent, nous utilisons **LAG** à nouveau pour créer une séquence avec 2 valeurs VOI consécutives, suivies de 10 valeurs consécutives de prix moyen.

```SQL
shiftedPriceAndShiftedVOI AS (
    /* get 10 future prices and 2 previous VOIs */
    SELECT
        symbol,
        midPrice AS midPrice10,
        LAG(midPrice, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice9,
        LAG(midPrice, 2) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice8,
        LAG(midPrice, 3) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice7,
        LAG(midPrice, 4) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice6,
        LAG(midPrice, 5) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice5,
        LAG(midPrice, 6) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice4,
        LAG(midPrice, 7) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice3,
        LAG(midPrice, 8) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice2,
        LAG(midPrice, 9) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice1,
        LAG(midPrice, 10) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice,
        LAG(VOI, 10) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS VOI1,
        LAG(VOI, 11) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS VOI2
    FROM currentPriceAndVOI
),
```

Ensuite, nous remodelons les données en entrées dans un modèle linéaire à deux variables. De nouveau, nous filtrons les événements pour lesquels les données sont incomplètes.

```SQL
modelInput AS (
    /* create feature vector, x being VOI, y being delta price */
    SELECT
        symbol,
        (midPrice1 + midPrice2 + midPrice3 + midPrice4 + midPrice5 + midPrice6 + midPrice7 + midPrice8 + midPrice9 + midPrice10)/10.0 - midPrice AS y,
        VOI1 AS x1,
        VOI2 AS x2
    FROM shiftedPriceAndShiftedVOI
    WHERE
        midPrice1 IS NOT NULL AND
        midPrice2 IS NOT NULL AND
        midPrice3 IS NOT NULL AND
        midPrice4 IS NOT NULL AND
        midPrice5 IS NOT NULL AND
        midPrice6 IS NOT NULL AND
        midPrice7 IS NOT NULL AND
        midPrice8 IS NOT NULL AND
        midPrice9 IS NOT NULL AND
        midPrice10 IS NOT NULL AND
        midPrice IS NOT NULL AND
        VOI1 IS NOT NULL AND
        VOI2 IS NOT NULL
),
```

Comme Azure Stream Analytics n’a pas de fonction intégrée de régression linéaire, vous devez utiliser les agrégats **SUM** et **AVG** pour calculer les coefficients pour le modèle linéaire.

![Formule mathématique de régression linéaire](./media/stream-analytics-high-frequency-trading/linear-regression-formula.png)

```SQL
modelagg AS (
    /* get aggregates for linear regression calculation,
     http://faculty.cas.usf.edu/mbrannick/regression/Reg2IV.html */
    SELECT
        symbol,
        SUM(x1 * x1) AS x1x1,
        SUM(x2 * x2) AS x2x2,
        SUM(x1 * y) AS x1y,
        SUM(x2 * y) AS x2y,
        SUM(x1 * x2) AS x1x2,
        AVG(y) AS avgy,
        AVG(x1) AS avgx1,
        AVG(x2) AS avgx2
    FROM modelInput
    GROUP BY symbol, TumblingWindow(hour, 24, -4)
),
modelparambs AS (
    /* calculate b1 and b2 for the linear model */
    SELECT
        symbol,
        (x2x2 * x1y - x1x2 * x2y)/(x1x1 * x2x2 - x1x2 * x1x2) AS b1,
        (x1x1 * x2y - x1x2 * x1y)/(x1x1 * x2x2 - x1x2 * x1x2) AS b2,
        avgy,
        avgx1,
        avgx2
    FROM modelagg
),
model AS (
    /* calculate a for the linear model */
    SELECT
        symbol,
        avgy - b1 * avgx1 - b2 * avgx2 AS a,
        b1,
        b2
    FROM modelparambs
),
```

Pour utiliser le modèle du jour précédent pour la notation de l’événement actuel, nous souhaitons joindre les devis au modèle. Mais au lieu d’utiliser **JOIN**, nous utilisons **UNION** sur les événements de modèle et les événements de devis. Ensuite, nous utilisons **LAG** pour coupler les événements avec le modèle du jour précédent, afin obtenir exactement une correspondance. En raison du weekend-end, il faut revenir trois jours en arrière. En utilisant simplement **JOIN**, nous obtiendrions trois modèles pour chaque événement de devis.

```SQL
shiftedVOI AS (
    /* get two consecutive VOIs */
    SELECT
        symbol,
        midPrice,
        VOI AS VOI1,        
        LAG(VOI, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS VOI2
    FROM currentPriceAndVOI
),
VOIAndModel AS (
    /* combine VOIs and models */
    SELECT
        'voi' AS type,
        symbol,
        midPrice,
        VOI1,
        VOI2,
        0.0 AS a,
        0.0 AS b1,
        0.0 AS b2
    FROM shiftedVOI
    UNION
    SELECT
        'model' AS type,
        symbol,
        0.0 AS midPrice,
        0 AS VOI1,
        0 AS VOI2,
        a,
        b1,
        b2
    FROM model
),
VOIANDModelJoined AS (
    /* match VOIs with the latest model within 3 days (72 hours, to take the weekend into account) */
    SELECT
        symbol,
        midPrice,
        VOI1 as x1,
        VOI2 as x2,
        LAG(a, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 72) WHEN type = 'model') AS a,
        LAG(b1, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 72) WHEN type = 'model') AS b1,
        LAG(b2, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 72) WHEN type = 'model') AS b2
    FROM VOIAndModel
    WHERE type = 'voi'
),
```

Ainsi, nous pouvons faire des prédictions et générer des signaux d’achat/vente basés sur le modèle, avec une valeur de seuil de 0,02. Une valeur commerciale de 10 correspond à un achat. Une valeur commerciale de -10 correspond à une vente.

``` SQL
prediction AS (
    /* make prediction if there is a model */
    SELECT
        symbol,
        midPrice,
        a + b1 * x1 + b2 * x2 AS efpc
    FROM VOIANDModelJoined
    WHERE
        a IS NOT NULL AND
        b1 IS NOT NULL AND
        b2 IS NOT NULL AND
        x1 IS NOT NULL AND
        x2 IS NOT NULL
),
tradeSignal AS (
    /* generate buy/sell signals */
    SELECT
        DateAdd(hour, -7, System.Timestamp) AS time,
        symbol,     
        midPrice,
        efpc,
        CASE WHEN (efpc > 0.02) THEN 10 ELSE (CASE WHEN (efpc < -0.02) THEN -10 ELSE 0 END) END AS trade,
        DATETIMEFROMPARTS(DATEPART(year, System.Timestamp), DATEPART(month, System.Timestamp), DATEPART(day, System.Timestamp), 0, 0, 0, 0) as date
    FROM prediction
),
```

### <a name="trading-simulation"></a>Simulation commerciale
Après avoir obtenu les signaux commerciaux, nous voulons tester l’efficacité de la stratégie commerciale, sans commercer réellement. 

Nous réalisons ce test à l’aide d’un UDA, avec une fenêtre récurrente, faisant des bonds à chaque minute. Le regroupement supplémentaire sur la date et la clause Having permettent les comptes de fenêtre pour les événements appartenant au même jour. Pour une fenêtre récurrente étendue sur deux jours, la date **GROUP BY** permet de séparer le regroupement entre jour précédent et jour actuel. La clause **HAVING** filtre les fenêtres qui se terminent durant le jour en cours, mais se regroupent durant la journée précédente.

```SQL
simulation AS
(
    /* perform trade simulation for the past 7 hours to cover an entire trading day, and generate output every minute */
    SELECT
        DateAdd(hour, -7, System.Timestamp) AS time,
        symbol,
        date,
        uda.TradeSimulation(tradeSignal) AS s
    FROM tradeSignal
    GROUP BY HoppingWindow(minute, 420, 1), symbol, date
    Having DateDiff(day, date, time) < 1 AND DATEPART(hour, time) < 13
)
```

L’UDA JavaScript initialise les accumulateurs de la fonction `init`, calcule la transition d’état prenant en compte chaque événement ajouté à la fenêtre et retourne les résultats de la simulation à la fin de la fenêtre. Le processus commercial général consiste à :

- Acheter des actions lors de la réception d’un signal d’achat sans détention d’actions.
- Vendre des actions lors de la réception d’un signal de vente avec détention d’actions.
- Position courte s’il n’y a pas de détention d’actions. 

Si un signal d’achat est reçu en position courte, il s’agit d’un achat de couverture. Nous ne détenons jamais 10 parts d’une action dans cette simulation. Le coût de transaction est fixe à 8 $.

```javascript
function main() {
    var TRADE_COST = 8.0;
    var SHARES = 10;
    this.init = function () {
        this.own = false;
        this.pos = 0;
        this.pnl = 0.0;
        this.tradeCosts = 0.0;
        this.buyPrice = 0.0;
        this.sellPrice = 0.0;
        this.buySize = 0;
        this.sellSize = 0;
        this.buyTotal = 0.0;
        this.sellTotal = 0.0;
    }
    this.accumulate = function (tradeSignal, timestamp) {
        if(!this.own && tradeSignal.trade == 10) {
          // Buy to open
          this.own = true;
          this.pos = 1;
          this.buyPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.buySize += SHARES;
          this.buyTotal += SHARES * tradeSignal.midprice;
        } else if(!this.own && tradeSignal.trade == -10) {
          // Sell to open
          this.own = true;
          this.pos = -1
          this.sellPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.sellSize += SHARES;
          this.sellTotal += SHARES * tradeSignal.midprice;
        } else if(this.own && this.pos == 1 && tradeSignal.trade == -10) {
          // Sell to close
          this.own = false;
          this.pos = 0;
          this.sellPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.pnl += (this.sellPrice - this.buyPrice)*SHARES - 2*TRADE_COST;
          this.sellSize += SHARES;
          this.sellTotal += SHARES * tradeSignal.midprice;
          // Sell to open
          this.own = true;
          this.pos = -1;
          this.sellPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.sellSize += SHARES;        
          this.sellTotal += SHARES * tradeSignal.midprice;
        } else if(this.own && this.pos == -1 && tradeSignal.trade == 10) {
          // Buy to close
          this.own = false;
          this.pos = 0;
          this.buyPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.pnl += (this.sellPrice - this.buyPrice)*SHARES - 2*TRADE_COST;
          this.buySize += SHARES;
          this.buyTotal += SHARES * tradeSignal.midprice;
          // Buy to open
          this.own = true;
          this.pos = 1;
          this.buyPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.buySize += SHARES;         
          this.buyTotal += SHARES * tradeSignal.midprice;
        }
    }
    this.computeResult = function () {
        var result = {
            "pnl": this.pnl,
            "buySize": this.buySize,
            "sellSize": this.sellSize,
            "buyTotal": this.buyTotal,
            "sellTotal": this.sellTotal,
            "tradeCost": this.tradeCost
            };
        return result;
    }
}
```

Enfin, nous générons dans le tableau de bord Power BI pour la visualisation.

```SQL
SELECT * INTO tradeSignalDashboard FROM tradeSignal /* output tradeSignal to PBI */
SELECT
    symbol,
    time,
    date,
    TRY_CAST(s.pnl as float) AS pnl,
    TRY_CAST(s.buySize as bigint) AS buySize,
    TRY_CAST(s.sellSize as bigint) AS sellSize,
    TRY_CAST(s.buyTotal as float) AS buyTotal,
    TRY_CAST(s.sellTotal as float) AS sellTotal
    INTO pnlDashboard
FROM simulation /* output trade simulation to PBI */
```

![Visuel graphique Power BI : transactions](./media/stream-analytics-high-frequency-trading/trades-power-bi-chart.png)

![Visuel graphique Power BI : PNL](./media/stream-analytics-high-frequency-trading/pnl-power-bi-chart.png)


## <a name="summary"></a>Résumé
Nous pouvons implémenter un modèle commercial réaliste à fréquence élevée avec une requête peu complexe dans Azure Stream Analytics. Nous devons simplifier le modèle de cinq à deux variables d’entrée, en raison de l’absence d’une fonction de régression linéaire intégrée. Mais pour un utilisateur déterminé, les algorithmes de dimensions et de sophistication plus élevée peuvent éventuellement être implémentés, comme pour l’UDA JavaScript. 

À noter que la plupart de la requête, en dehors de l’UDA JavaScript, peut être testée et déboguée dans Visual Studio à l’aide des [Outils Azure Stream Analytics pour Visual Studio](stream-analytics-tools-for-visual-studio-install.md). Une fois que la requête initiale a été écrite, l’auteur a passé moins de 30 minutes à tester et déboguer la requête dans Visual Studio. 

Actuellement, l’UDA ne peut pas être débogué dans Visual Studio. Nous essayons de rendre cela possible avec la capacité d’examiner le code JavaScript. En outre, notez que les champs atteignant l’UDA ont des noms en minuscule. Ce comportement n’était pas prévu durant le test de la requête. Cependant, avec le niveau de compatibilité 1.1 d’Azure Stream Analytics, nous préservons la casse du nom de champ, par conséquent, le comportement est plus naturel.

J’espère que cet article inspirera les utilisateurs d’Azure Stream Analytics qui peuvent utiliser notre service pour effectuer des analyses avancées en quasi temps réel, en continu. Faites-nous part de vos commentaires afin de faciliter l’implémentation des requêtes pour les scénarios d’analyses avancées.

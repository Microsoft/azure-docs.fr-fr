---
title: Traiter des flux de données IoT en temps réel à l’aide d’Azure Stream Analytics
description: Flux de données et balises de capteur IoT avec analyses de flux et traitement des données en temps réel
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 11/26/2019
ms.openlocfilehash: 311aca139220622a0436d490e73a536c3fc898c9
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93129012"
---
# <a name="process-real-time-iot-data-streams-with-azure-stream-analytics"></a>Traiter des flux de données IoT en temps réel à l’aide d’Azure Stream Analytics

Dans cet article, vous allez découvrir comment créer une logique de traitement de flux pour collecter des données à partir d’appareils IoT (Internet des objets). Vous allez recourir à un cas d’utilisation réel de l’IoT pour découvrir comment générer votre solution rapidement et à moindre coût.

## <a name="prerequisites"></a>Prérequis

* Créez un [abonnement Azure](https://azure.microsoft.com/pricing/free-trial/) gratuit.
* Téléchargez des exemples de fichiers de requête et de données à partir de [GitHub](https://aka.ms/azure-stream-analytics-get-started-iot).

## <a name="scenario"></a>Scénario

Entreprise du secteur de l’automatisation industrielle, Contoso a complètement automatisé son processus de fabrication. Les machines de cette usine possèdent des capteurs capables de générer des flux de données en temps réel. Dans ce scénario, un responsable d’atelier de production souhaite obtenir des informations en temps réel à partir des données des capteurs pour rechercher des modèles et effectuer des actions sur ceux-ci. Vous pouvez utiliser le langage de requête Stream Analytics (SAQL, Stream Analytics Query Language) sur les données des capteurs pour rechercher des modèles intéressants à partir du flux de données entrant.

Dans cet exemple, les données sont générées à partir d’un appareil Texas Instruments SensorTag. La charge utile des données est au format JSON et ressemble à ceci :

```json
{
    "time": "2016-01-26T20:47:53.0000000",  
    "dspl": "sensorE",  
    "temp": 123,  
    "hmdt": 34  
}  
```

Dans un scénario réel, des centaines de capteurs de ce type pourraient générer des événements sous forme de flux. Dans l’idéal, un appareil de passerelle exécuterait du code pour transmettre ces événements à [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) ou [Azure IoT Hubs](https://azure.microsoft.com/services/iot-hub/). Votre tâche Stream Analytics ingèrerait ces événements depuis Event Hubs et exécuterait des requêtes d’analyse en temps réel sur les flux. Vous pourriez ensuite envoyer les résultats à l’une des [sorties prises en charge](stream-analytics-define-outputs.md).

Pour plus de convivialité, ce guide de mise en route fournit un exemple de fichier de données capturé à partir de balises de capteur réelles. Vous pouvez exécuter des requêtes sur les exemples de données et afficher les résultats. Dans les didacticiels suivants, vous apprendrez comment connecter votre travail aux entrées et sorties et le déployer sur le service Azure.

## <a name="create-a-stream-analytics-job"></a>Création d’un travail Stream Analytics

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **+ Créer une ressource** dans le menu de navigation de gauche. Sélectionnez ensuite **Tâche Stream Analytics** sous **Analytics**.
   
    ![Créer une tâche Stream Analytics](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-02.png)

1. Saisissez un nom de tâche unique et vérifiez que l’abonnement est approprié pour votre tâche. Créez un groupe de ressources ou sélectionnez un groupe existant dans votre abonnement.

1. Sélectionnez l’emplacement de votre tâche. Utilisez le même emplacement pour votre groupe de ressources et toutes les ressources pour augmenter la vitesse de traitement et réduire les coûts. Une fois les configurations effectuées, sélectionnez **Créer**.
   
    ![Créer une tâche Stream Analytics - détails](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-03.png)

## <a name="create-an-azure-stream-analytics-query"></a>Création d’une requête Azure Stream Analytics
L’étape suivante après la création de votre tâche consiste à écrire une requête. Vous pouvez tester des requêtes sur des exemples de données sans connecter une entrée ou une sortie à votre tâche.

Téléchargez [HelloWorldASA-InputStream.json](https://github.com/Azure/azure-stream-analytics/blob/master/Samples/GettingStarted/HelloWorldASA-InputStream.json
) à partir de GitHub. Accédez ensuite à votre tâche Azure Stream Analytics dans le portail Azure.

Sélectionnez **Requête** sous **Topologie de la tâche** dans le menu de gauche. Ensuite, sélectionnez **Charger un exemple de requête**. Téléchargez le fichier `HelloWorldASA-InputStream.json`, puis sélectionnez **OK**.

![Mosaïque de requête du tableau de bord Stream Analytics](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-05.png)

Notez qu’un aperçu des données est automatiquement renseigné dans la table **Aperçu de l’entrée**.

![Aperçu des exemples de données d’entrée](./media/stream-analytics-get-started-with-iot-devices/input-preview.png)

### <a name="query-archive-your-raw-data"></a>Requête : Archiver vos données brutes

La forme de requête la plus simple est une requête directe qui archive toutes les données d’entrée dans sa sortie désignée. Cette requête est la requête par défaut renseignée dans une nouvelle tâche Azure Stream Analytics.

```sql
SELECT 
    *
INTO
    Output
FROM
    InputStream
```

Sélectionnez **Tester la requête** et affichez les résultats dans la table **Résultats du test**.

![Résultat de test pour une requête Stream Analytics](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-07.png)

### <a name="query-filter-the-data-based-on-a-condition"></a>Requête : Filtrer les données en fonction d’une condition

Essayons de filtrer les résultats en fonction d’une condition. Nous souhaitons limiter les résultats aux événements qui proviennent de « SensorA ».

```sql
SELECT 
    time,
    dspl AS SensorName,
    temp AS Temperature,
    hmdt AS Humidity
INTO
   Output
FROM
    InputStream
WHERE dspl='sensorA'
```

Collez la requête dans l’éditeur et sélectionnez **Tester la requête** pour passer en revue les résultats.

![Filtrage d’un flux de données](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-08.png)

### <a name="query-alert-to-trigger-a-business-workflow"></a>Requête : Alerte déclenchant un flux de travail métier

Nous allons maintenant affiner notre requête. Pour chaque type de capteur, nous souhaitons surveiller la température moyenne par fenêtre de 30 secondes et afficher les résultats uniquement si la température moyenne se situe au-dessus de 100 degrés.

```sql
SELECT 
    System.Timestamp AS OutputTime,
    dspl AS SensorName,
    Avg(temp) AS AvgTemperature
INTO
   Output
FROM
    InputStream TIMESTAMP BY time
GROUP BY TumblingWindow(second,30),dspl
HAVING Avg(temp)>100
```

![Requête de filtre de 30 secondes](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-10.png)

Les résultats doivent contenir uniquement 245 lignes et répertorier les noms des capteurs pour lesquels la température moyenne est supérieure à 100 degrés. Dans cette requête, nous avons regroupé le flux d’événements en fonction du champ **dspl** , qui correspond au nom du capteur, et par valeur **Intervalle de temps** de 30 secondes. Les requêtes temporelles doivent indiquer le mode de calcul du temps. À l’aide de la clause **TIMESTAMP BY** , vous avez spécifié la colonne **OUTPUTTIME** pour associer les valeurs de temps avec l’ensemble des calculs temporels. Pour obtenir des informations détaillées, consultez [Gestion du temps](/stream-analytics-query/time-management-azure-stream-analytics) et [Fonctions de fenêtrage](/stream-analytics-query/windowing-azure-stream-analytics).

### <a name="query-detect-absence-of-events"></a>Requête : Détection de l’absence d’événements

Comment écrire une requête pour rechercher une absence d’événements d’entrée ? Recherchons la dernière fois qu’un capteur a envoyé des données, puis qu’il n’a pas envoyé d’événements au cours des 5 secondes suivantes.

```sql
SELECT 
    t1.time,
    t1.dspl AS SensorName
INTO
   Output
FROM
    InputStream t1 TIMESTAMP BY time
LEFT OUTER JOIN InputStream t2 TIMESTAMP BY time
ON
    t1.dspl=t2.dspl AND
    DATEDIFF(second,t1,t2) BETWEEN 1 and 5
WHERE t2.dspl IS NULL
```

![Détection de l’absence d’événements](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-11.png)

Ici, nous utilisons une jointure **LEFT OUTER** sur le même flux de données (jointure réflexive). Pour une jointure **INNER** , un résultat n’est renvoyé que si une correspondance est trouvée.  Par contre, pour une jointure **LEFT OUTER** , si un événement du côté gauche de la jointure n’a pas de correspondance, une ligne contenant la valeur NULL pour toutes les colonnes de la ligne de droite est renvoyée. Cette technique est très utile pour rechercher une absence d’événements. Pour plus d'informations, consultez [JOIN](/stream-analytics-query/join-azure-stream-analytics).

## <a name="conclusion"></a>Conclusion

L’objectif de cet article est de montrer comment écrire différentes requêtes en langage de requête Stream Analytics et afficher les résultats dans le navigateur. Toutefois, il ne s’agit que d’une prise en main. Ce langage prend en charge un large éventail d’entrées et de sorties et peut même tirer parti de fonctions d’Azure Machine Learning, ce qui en fait un outil robuste pour l’analyse des flux de données. Pour en savoir plus sur la méthode à suivre pour écrire des requêtes, voir [Exemples de requête pour les modes d’utilisation courants dans Stream Analytics](stream-analytics-stream-analytics-query-patterns.md).
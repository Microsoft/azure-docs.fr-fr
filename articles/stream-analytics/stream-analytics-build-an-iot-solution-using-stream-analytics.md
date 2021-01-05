---
title: Créer une solution IoT à l’aide d’Azure Stream Analytics
description: Prise en main du didacticiel pour la solution IoT Stream Analytics d’un scénario de station de péage
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/06/2018
ms.custom: seodec18
ms.openlocfilehash: 87ec59d19fb442293fb7f14d110cf513015ec9f7
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130797"
---
# <a name="build-an-iot-solution-by-using-stream-analytics"></a>Créer une solution IoT à l’aide de Stream Analytics

## <a name="introduction"></a>Introduction
Dans cette solution, vous allez apprendre à utiliser Azure Stream Analytics pour obtenir des informations en temps réel à partir de vos données. Les développeurs peuvent aisément combiner des flux de données, tels que des flux de clics, des journaux d’activité et des événements générés par des appareils, avec des enregistrements d’historique ou des données de référence pour recueillir des perspectives. En tant que service de calcul de flux en temps réel entièrement géré hébergé dans Microsoft Azure, Azure Stream Analytics offre la résilience, la faible latence et la scalabilité nécessaires pour être opérationnel en quelques minutes.

À l’issue de cette solution, vous serez en mesure de :

* vous familiariser avec le portail Azure Stream Analytics ;
* configurer et déployer un travail de diffusion en continu ;
* exposer des problèmes concrets et les résoudre à l’aide du langage de requête Stream Analytics ;
* développer des solutions de streaming pour vos clients à l’aide de Stream Analytics en toute confiance ;
* vous appuyer sur l’expérience de surveillance et de journalisation pour résoudre les problèmes.

## <a name="prerequisites"></a>Prérequis
Pour effectuer cette solution, vous avez besoin des prérequis suivants :
* Un [abonnement Azure](https://azure.microsoft.com/pricing/free-trial/)

## <a name="scenario-introduction-hello-toll"></a>Présentation du scénario : « Gare de péage »
Une gare de péage est un dispositif très répandu. Il s’en trouve sur de nombreux autoroutes, ponts et tunnels dans le monde entier. Chaque station de péage compte plusieurs guichets. Aux guichets manuels, vous vous arrêtez pour régler le montant du péage à un guichetier. Aux guichets automatisés, un capteur scanne une carte RFID apposée sur le pare-brise de votre véhicule lorsque vous franchissez le péage. Il est facile de visualiser le franchissement de ces stations de péage par les véhicules sous la forme d’un flux d’événements sur lequel des opérations intéressantes peuvent être effectuées.

![Image de voitures à des postes de péage](media/stream-analytics-build-an-iot-solution-using-stream-analytics/cars-in-toll-booth.jpg)

## <a name="incoming-data"></a>Données entrantes
Cette solution fonctionne avec deux flux de données. Des capteurs installés à l’entrée et à la sortie des stations de péage produisent le premier flux. Le deuxième flux de données est un jeu de données de recherche statique contenant les données d’inscription des véhicules.

### <a name="entry-data-stream"></a>Flux des données d’entrée
Le flux des données d’entrée contient les informations relatives aux véhicules qui arrivent aux stations de péage. Les événements de données de sortie sont diffusés en temps réel dans une file d’attente Event Hub à partir d’une application web incluse dans l’exemple d’application.

| TollID | EntryTime | LicensePlate | State | Marque | Modèle | VehicleType | VehicleWeight | Toll | Tag |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 |2014-09-10 12:01:00.000 |JNB 7001 |NY |Honda |CRV |1 |0 |7 | |
| 1 |2014-09-10 12:02:00.000 |YXZ 1001 |NY |Toyota |Camry |1 |0 |4 |123456789 |
| 3 |2014-09-10 12:02:00.000 |ABC 1004 |CT |Ford |Taurus |1 |0 |5 |456789123 |
| 2 |2014-09-10 12:03:00.000 |XYZ 1003 |CT |Toyota |Corolla |1 |0 |4 | |
| 1 |2014-09-10 12:03:00.000 |BNJ 1007 |NY |Honda |CRV |1 |0 |5 |789123456 |
| 2 |2014-09-10 12:05:00.000 |CDE 1007 |NJ |Toyota |4x4 |1 |0 |6 |321987654 |

Voici une brève description des colonnes :

| Colonne | Description |
| --- | --- |
| TollID |ID de guichet identifiant de façon unique un poste de péage |
| EntryTime |Date et heure UTC d’entrée du véhicule en poste de péage |
| LicensePlate |Numéro de plaque d’immatriculation du véhicule |
| State |État aux États-Unis |
| Marque |Fabricant du véhicule |
| Modèle |Numéro de modèle du véhicule |
| VehicleType |1 pour les véhicules de transport de personnes, 2 pour les véhicules commerciaux |
| WeightType |Poids du véhicule en tonnes ; 0 pour les véhicules de tourisme |
| Toll |Prix du péage en dollar américain |
| Tag |Balise électronique sur la voiture, qui automatise le paiement ; vide si le paiement a été effectué manuellement |

### <a name="exit-data-stream"></a>Flux des données de sortie
Le flux des données de sortie contient des informations sur les véhicules qui quittent la gare de péage. Les événements de données de sortie sont diffusés en temps réel dans une file d’attente Event Hub à partir d’une application web incluse dans l’exemple d’application.

| **TollId** | **ExitTime** | **LicensePlate** |
| --- | --- | --- |
| 1 |2014-09-10T12:03:00.0000000Z |JNB 7001 |
| 1 |2014-09-10T12:03:00.0000000Z |YXZ 1001 |
| 3 |2014-09-10T12:04:00.0000000Z |ABC 1004 |
| 2 |2014-09-10T12:07:00.0000000Z |XYZ 1003 |
| 1 |2014-09-10T12:08:00.0000000Z |BNJ 1007 |
| 2 |2014-09-10T12:07:00.0000000Z |CDE 1007 |

Voici une brève description des colonnes :

| Colonne | Description |
| --- | --- |
| TollID |ID de guichet identifiant de façon unique un poste de péage |
| ExitTime |Date et heure UTC auxquelles le véhicule quitte le poste de péage |
| LicensePlate |Numéro de plaque d’immatriculation du véhicule |

### <a name="commercial-vehicle-registration-data"></a>Données d’inscription des véhicules commerciaux
La solution utilise un instantané statique d’une base de données d’inscription de véhicules commerciaux. Ces données sont enregistrées dans un fichier JSON dans le stockage d’objets blob Azure, inclus dans l’exemple.

| LicensePlate | ID d’inscription | Expiré |
| --- | --- | --- |
| SVT 6023 |285429838 |1 |
| XLZ 3463 |362715656 |0 |
| BAC 1005 |876133137 |1 |
| RIV 8632 |992711956 |0 |
| SNY 7188 |592133890 |0 |
| ELH 9896 |678427724 |1 |

Voici une brève description des colonnes :

| Colonne | Description |
| --- | --- |
| LicensePlate |Numéro de plaque d’immatriculation du véhicule |
| ID d’inscription |ID d’inscription du véhicule |
| Expiré |État d’inscription du véhicule : 0 si l’inscription est active, 1 si elle a expiré |

## <a name="set-up-the-environment-for-azure-stream-analytics"></a>Configurer l’environnement pour Azure Stream Analytics
Pour effectuer cette solution, vous avez besoin d’un abonnement Microsoft Azure. Si vous n’avez pas de compte Azure, vous pouvez [demander un essai gratuit](https://azure.microsoft.com/pricing/free-trial/).

Pour tirer pleinement parti de votre crédit Azure, suivez les instructions de la section « Nettoyer votre compte Azure » à la fin de cet article.

## <a name="deploy-the-sample"></a>Déployer l'exemple
Il existe plusieurs ressources qui peuvent facilement être déployées ensemble dans un groupe de ressources en quelques clics. La définition de la solution est hébergée dans le référentiel GitHub sous [https://github.com/Azure/azure-stream-analytics/tree/master/Samples/TollApp](https://github.com/Azure/azure-stream-analytics/tree/master/Samples/TollApp).

### <a name="deploy-the-tollapp-template-in-the-azure-portal"></a>Déployer le modèle TollApp dans le portail Azure
1. Pour déployer l’environnement TollApp sur Azure, utilisez ce lien pour [déployer un modèle TollApp Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-stream-analytics%2Fmaster%2FSamples%2FTollApp%2FVSProjects%2FTollAppDeployment%2Fazuredeploy.json).

2. Connectez-vous au portail Azure, si vous y êtes invité.

3. Sélectionnez l’abonnement dans lequel les différentes ressources sont facturées.

4. Spécifiez un nouveau groupe de ressources en lui attribuant un nom unique (par exemple, `MyTollBooth`).

5. Sélectionnez un emplacement Azure.

6. Spécifiez un **intervalle** (en secondes). Cette valeur est utilisée dans l’exemple d’application web et désigne la fréquence à laquelle envoyer les données à Event Hub.

7. **Cochez la case** pour accepter les conditions générales.

8. Sélectionnez **Épingler au tableau de bord** pour pouvoir localiser facilement les ressources ultérieurement.

9. Sélectionnez **Acheter** pour déployer l’exemple de modèle.

10. Après quelques instants, une notification s’affiche pour confirmer la **réussite du déploiement**.

### <a name="review-the-azure-stream-analytics-tollapp-resources"></a>Passer en revue les ressources TollApp d’Azure Stream Analytics

1. Connectez-vous au portail Azure.

2. Localisez le groupe de ressources que vous avez nommé dans la section précédente.

3. Vérifiez que les ressources suivantes sont répertoriées dans le groupe de ressources :
   - Un compte Cosmos DB
   - Une tâche Azure Stream Analytics
   - Un compte de stockage Azure
   - Un Azure Event Hub
   - Deux applications Web

## <a name="examine-the-sample-tollapp-job"></a>Examinez l’exemple de tâche TollApp.
1. À partir du groupe de ressources dans la section précédente, sélectionnez la tâche de diffusion en continu Stream Analytics commençant par le nom **tollapp** (le nom contient des caractères aléatoires à des fins d’unicité).

2. Sur la page **Vue d’ensemble** de la tâche, regardez la case **Requête** pour voir la syntaxe de la requête.

   ```sql
   SELECT TollId, System.Timestamp AS WindowEnd, COUNT(*) AS Count
   INTO CosmosDB
   FROM EntryStream TIMESTAMP BY EntryTime
   GROUP BY TUMBLINGWINDOW(minute, 3), TollId
   ```

   Pour paraphraser le but de la requête, imaginons que vous ayez besoin de compter le nombre de véhicules entrant dans un poste de péage. Un poste de péage autoroutier accueillant un flux continu de véhicules, ces événements d’entrée peuvent être comparés à un flux qui ne s’arrête jamais. Pour quantifier le flux, vous devez définir la « durée » pendant laquelle vous allez effectuer la mesure. Nous pouvons affiner la question de la sorte : « Combien de véhicules passent par le poste de péage en trois minutes ? ». C’est ce qu’on appelle communément le « nombre bascule ».

   Comme vous pouvez le voir, Azure Stream Analytics utilise un langage de requête similaire à SQL et ajoute quelques extensions pour spécifier les aspects de la requête liés au temps.  Pour plus de détails, consultez les constructions de [gestion du temps](/stream-analytics-query/time-management-azure-stream-analytics) et de [fenêtrage](/stream-analytics-query/windowing-azure-stream-analytics) utilisées dans la requête.

3. Examinez les entrées de l’exemple de tâche TollApp. Seule l’entrée EntryStream est utilisée dans la requête actuelle.
   - L’entrée **EntryStream** est une connexion de l'Event Hub qui met en file d’attente les données représentant chaque passage d’un véhicule par un poste de péage autoroutier. Une application web qui fait partie de l’exemple crée les événements, et ces données sont mises en file d’attente dans cet Event Hub. Notez que cette entrée est interrogée dans la clause FROM de la requête de diffusion en continu.
   - L’entrée **ExitStream** est une connexion de l'Event Hub qui met en file d’attente les données représentant chaque sortie d’un véhicule d’un poste de péage autoroutier. Cette entrée de flux sera utilisée dans des variantes ultérieures de la syntaxe de requête.
   - L’entrée **Enregistrement** est une connexion du stockage d'objets blob Azure, pointant vers un fichier .json d’enregistrement statique, utilisé pour les recherches en fonction des besoins. Cette entrée de données de référence sera utilisée dans des variantes ultérieures de la syntaxe de requête.

4. Examinez les sorties de l’exemple de tâche TollApp.
   - La sortie **Cosmos DB** est un conteneur de base de données Cosmos qui reçoit les événements du récepteur de sortie. Notez que cette sortie est utilisée dans la clause INTO de la requête de diffusion en continu.

## <a name="start-the-tollapp-streaming-job"></a>Démarrer la tâche de diffusion en continu TollApp
Suivez ces étapes pour démarrer la tâche de diffusion en continu :

1. Sur la page **Vue d’ensemble** de la tâche, sélectionnez **Démarrer**.

2. Sur le volet **Démarrer la tâche** , sélectionnez **Maintenant**.

3. Après quelques instants, une fois la tâche en cours d’exécution, sur la page **Vue d’ensemble** de la tâche de diffusion en continu, affichez le graphique **Analyse**. Le graphique doit afficher plusieurs milliers d’événements d’entrée et des dizaines d’événements de sortie.

## <a name="review-the-cosmosdb-output-data"></a>Passer en revue les données de sortie CosmosDB
1. Localisez le groupe de ressources qui contient les ressources TollApp.

2. Sélectionnez le compte Azure Cosmos DB avec le modèle de nom **tollapp\<random\>-cosmos**.

3. Sélectionnez l’en-tête **Explorateur de données** pour ouvrir la page Explorateur de données.

4. Développez le chemin **tollAppDatabase** > **tollAppCollection** > **Documents**.

5. Dans la liste des identifiants, plusieurs documents s’affichent une fois que la sortie est disponible.

6. Sélectionnez chacun des identifiants pour passer en revue le document JSON. Notez chaque TollID, l’heure Windowend et le nombre de voitures depuis cette fenêtre.

7. Au bout de trois minutes supplémentaires, un autre jeu de quatre documents est disponible, soit un document par TollID.


## <a name="report-total-time-for-each-car"></a>Signaler la durée totale pour chaque véhicule
Le temps moyen nécessaire pour qu’une voiture franchisse le péage vous aide à évaluer l’efficacité du processus et l’expérience du client.

Pour connaître la durée totale, ajoutez le flux EntryTime au flux ExitTime. Ajoutez les deux flux d’entrée aux colonnes TollId et LicencePlate égales correspondantes. L’opérateur **JOIN** exige que vous spécifiiez une marge de manœuvre temporelle décrivant l’écart de temps acceptable entre les événements joints. Utilisez la fonction **DATEDIFF** pour spécifier que les événements ne doivent pas être séparés de plus de 15 minutes. Appliquez également la fonction **DATEDIFF** aux heures de sortie et d’entrée pour calculer le temps réel qu’un véhicule passe en gare de péage. Notez que la fonction **DATEDIFF** n’est pas utilisée de la même façon dans une instruction **SELECT** et dans une condition **JOIN**.

```sql
SELECT EntryStream.TollId, EntryStream.EntryTime, ExitStream.ExitTime, EntryStream.LicensePlate, DATEDIFF (minute, EntryStream.EntryTime, ExitStream.ExitTime) AS DurationInMinutes
INTO CosmosDB
FROM EntryStream TIMESTAMP BY EntryTime
JOIN ExitStream TIMESTAMP BY ExitTime
ON (EntryStream.TollId= ExitStream.TollId AND EntryStream.LicensePlate = ExitStream.LicensePlate)
AND DATEDIFF (minute, EntryStream, ExitStream ) BETWEEN 0 AND 15
```

### <a name="to-update-the-tollapp-streaming-job-query-syntax"></a>Pour mettre à jour la syntaxe de requête d’une tâche de diffusion en continu TollApp :

1. Sur la page **Vue d’ensemble** de la tâche, sélectionnez **Arrêter**.

2. Attendez quelques instants que la notification de la tâche s’arrête.

3. Sous l’en-tête TOPOLOGIE DE LA TÂCHE, sélectionnez **< > requête**.

4. Collez la requête SQL de diffusion en continu ajustée.

5. Sélectionnez **Enregistrer** pour enregistrer la requête. Cliquez sur **Oui** pour enregistrer les modifications.

6. Sur la page **Vue d’ensemble** de la tâche, sélectionnez **Démarrer**.

7. Sur le volet **Démarrer la tâche** , sélectionnez **Maintenant**.

### <a name="review-the-total-time-in-the-output"></a>Passer en revue la durée totale de la sortie
Répétez les étapes décrites dans la section précédente pour passer en revue les données de sortie CosmosDB à partir de la tâche de diffusion en continu. Passez en revue les derniers documents JSON.

Par exemple, ce document montre un exemple de voiture avec une certaine plaque d’immatriculation, l’heure d’entrée et de sortie ainsi que le champ de la durée en minutes calculée DATEDIFF qui affiche une durée au péage de deux minutes :
```JSON
{
    "tollid": 4,
    "entrytime": "2018-04-05T06:51:39.0491173Z",
    "exittime": "2018-04-05T06:53:09.0491173Z",
    "licenseplate": "JVR 9425",
    "durationinminutes": 2,
    "id": "ff52eb25-d580-7566-2879-1f52bba6601e",
    "_rid": "+8E4AI1DZgBjAAAAAAAAAA==",
    "_self": "dbs/+8E4AA==/colls/+8E4AI1DZgA=/docs/+8E4AI1DZgBjAAAAAAAAAA==/",
    "_etag": "\"ad02f6b8-0000-0000-0000-5ac5c8330000\"",
    "_attachments": "attachments/",
    "_ts": 1522911283
}
```

## <a name="report-vehicles-with-expired-registration"></a>Signaler les véhicules dont l’inscription a expiré
Azure Stream Analytics peut utiliser des instantanés statiques des données de référence pour joindre des flux de données temporelles. Pour illustrer cette fonctionnalité, nous allons utiliser l’exemple de question suivant. L’entrée Inscription est un fichier json blob statique qui répertorie les dates d’expiration des plaques d’immatriculation. En s’aidant de la plaque d’immatriculation, les données de référence sont comparées à chaque véhicule passant par le poste de péage.

Si un véhicule commercial est inscrit auprès de l’entreprise de péage, il peut franchir directement le poste de péage sans être arrêté pour inspection. Utilisez la table de recherche contenant les inscriptions de véhicules commerciaux pour identifier tous ceux dont l’inscription a expiré.

```sql
SELECT EntryStream.EntryTime, EntryStream.LicensePlate, EntryStream.TollId, Registration.RegistrationId
INTO CosmosDB
FROM EntryStream TIMESTAMP BY EntryTime
JOIN Registration
ON EntryStream.LicensePlate = Registration.LicensePlate
WHERE Registration.Expired = '1'
```

1. Répétez les étapes décrites dans la section précédente pour mettre à jour la syntaxe de requête de la tâche de diffusion en continu TollApp.

2. Répétez les étapes décrites dans la section précédente pour passer en revue les données de sortie CosmosDB à partir de la tâche de diffusion en continu.

Exemple de sortie :
```json
    {
        "entrytime": "2018-04-05T08:01:28.0252168Z",
        "licenseplate": "GMT 3221",
        "tollid": 1,
        "registrationid": "763220582",
        "id": "47db0535-9716-4eb2-db58-de7886966cbf",
        "_rid": "y+F8AJ9QWACSAQAAAAAAAA==",
        "_self": "dbs/y+F8AA==/colls/y+F8AJ9QWAA=/docs/y+F8AJ9QWACSAQAAAAAAAA==/",
        "_etag": "\"88007d8d-0000-0000-0000-5ac5d7e20000\"",
        "_attachments": "attachments/",
        "_ts": 1522915298
    }
```

## <a name="scale-out-the-job"></a>Effectuer un scale-out de la tâche
Azure Stream Analytics est conçu pour adapter son échelle en souplesse afin de pouvoir gérer de grands volumes de données. La requête d’Azure Stream Analytics peut utiliser une clause **PARTITION BY** pour informer le système que cette étape fait l’objet d’une montée en charge. **PartitionId** est une colonne spéciale ajoutée par le système, qui correspond à l’ID de partition de l’entrée (hub d’événements).

Pour effectuer un scale-out de la requête aux partitions, modifiez la syntaxe de requête pour qu’elle suive le code suivant :
```sql
SELECT TollId, System.Timestamp AS WindowEnd, COUNT(*)AS Count
INTO CosmosDB
FROM EntryStream
TIMESTAMP BY EntryTime
PARTITION BY PartitionId
GROUP BY TUMBLINGWINDOW(minute,3), TollId, PartitionId
```

Pour monter en puissance la tâche de diffusion en continu à davantage d’unités de diffusion en continu :

1. **Arrêtez** la tâche en cours.

2. Mettez à jour la syntaxe de la requête dans la page **< > Requête** et enregistrez les modifications.

3. Sous l’en-tête CONFIGURER de la tâche de diffusion en continu, sélectionnez **Mettre à l'échelle**.

4. Faites glisser le curseur **Unités de diffusion en continu** de 1 à 6. Les unités de streaming définissent la quantité de puissance de calcul allouable à la tâche. Sélectionnez **Enregistrer**.

5. **Démarrez** la tâche de diffusion en continu afin d’illustrer la mise à l'échelle supplémentaire. Azure Stream Analytics distribue le travail entre un plus grand nombre de ressources de calcul et obtient un meilleur débit, en partitionnant le travail entre les ressources à l’aide de la colonne désignée dans la clause PARTITION BY.

## <a name="monitor-the-job"></a>Surveiller la tâche
La zone **Surveiller** contient des statistiques sur le travail en cours d’exécution. Une configuration initiale est nécessaire pour utiliser le compte de stockage dans la même région (nommer le numéro comme le reste de ce document).

![Surveillance des tâches Azure Stream Analytics](media/stream-analytics-build-an-iot-solution-using-stream-analytics/stream-analytics-job-monitoring.png)

Vous pouvez également accéder aux **Journaux d’activité** à partir de la zone **Paramètres** du tableau de bord du travail.

## <a name="clean-up-the-tollapp-resources"></a>Nettoyer les ressources TollApp
1. Arrêtez la tâche Stream Analytics dans le portail Azure.

2. Localisez le groupe de ressources contenant huit ressources relatives au modèle TollApp.

3. Sélectionnez **Supprimer le groupe de ressources**. Tapez le nom du groupe de ressources pour confirmer la suppression.

## <a name="conclusion"></a>Conclusion
Cette solution vous a présenté le service Azure Stream Analytics. Il vous a expliqué comment configurer les entrées et sorties de la tâche Stream Analytics. À l’aide du scénario sur les données de péage, la solution vous a expliqué les types courants de problèmes qui surviennent dans l’espace de données en mouvement et comment les résoudre avec de simples requêtes de type SQL dans Azure Stream Analytics. Elle a décrit des constructions d’extension SQL en vue d’une utilisation avec des données temporelles. Il a montré comment joindre des flux de données et enrichir le flux de données avec des données de référence statiques, et comment effectuer un scale-out d’une requête pour obtenir un débit plus élevé.

Bien que cette solution constitue une bonne introduction, elle n’est en aucun cas complète. Vous pouvez trouver plusieurs modèles de requête utilisant le langage SAQL dans [Exemples de requête pour les modes d’utilisation courants dans Stream Analytics](stream-analytics-stream-analytics-query-patterns.md).
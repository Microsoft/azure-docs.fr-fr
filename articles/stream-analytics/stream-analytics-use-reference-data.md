---
title: Utiliser des données de référence pour effectuer des recherches dans Azure Stream Analytics
description: Cet article décrit comment utiliser des données de référence pour rechercher ou mettre en corrélation des données dans le cadre d’une conception de requête de travail Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 12/2/2020
ms.openlocfilehash: 2cfd391daa13a100a56bb10b79b27eda80902374
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533603"
---
# <a name="using-reference-data-for-lookups-in-stream-analytics"></a>Utiliser des données de référence pour effectuer des recherches dans Stream Analytics

Les données de référence (également appelées « tables de choix ») sont un jeu de données finies, statiques ou variant lentement par nature, utilisé pour effectuer des recherches ou pour augmenter vos flux de données. Par exemple, dans un scénario IoT, vous pourriez stocker des métadonnées sur les capteurs (qui ne changent pas souvent) dans les données de référence et les associer à des flux de données IoT en temps réel. Azure Stream Analytics charge les données de référence dans la mémoire pour obtenir un traitement de flux à faible latence. Pour utiliser des données de référence dans votre travail Azure Stream Analytics, vous utiliserez généralement une [jointure de données de référence](/stream-analytics-query/reference-data-join-azure-stream-analytics) dans votre requête. 

## <a name="example"></a>Exemple  
Vous pouvez avoir un flux d’événements en temps réel généré quand les voitures passent un poste de péage. Le poste de péage peut capturer la plaque de licence en temps réel et associer un jeu de données statique qui contient les détails de l’inscription pour identifier les plaques de licence qui ont expiré.  
  
```SQL  
SELECT I1.EntryTime, I1.LicensePlate, I1.TollId, R.RegistrationId  
FROM Input1 I1 TIMESTAMP BY EntryTime  
JOIN Registration R  
ON I1.LicensePlate = R.LicensePlate  
WHERE R.Expired = '1'
```  

Stream Analytics prend en charge le stockage Blob Azure et Azure SQL Database comme couche de stockage pour les données de référence. Vous pouvez également transformer et/ou copier des données de référence dans le stockage d’objets Blob à partir d’Azure Data Factory pour utiliser [n’importe quel nombre de banques de données cloud et locales](../data-factory/copy-activity-overview.md).

## <a name="azure-blob-storage"></a>Stockage Blob Azure

Les données de référence sont modélisées en tant que séquence d'objets Blob (définie dans la configuration d'entrée) dans l'ordre croissant de la date/l'heure spécifiée dans le nom de l'objet blob. Elles prennent en charge **uniquement** l’ajout à la fin de la séquence à l’aide d’une date/heure **ultérieure** à celle indiquée par le dernier objet blob dans la séquence.

### <a name="configure-blob-reference-data"></a>Configurer les données de référence d’objet blob

Pour configurer vos données de référence, vous devez d'abord créer une entrée de type **Données de référence**. Le tableau ci-dessous explique chaque propriété que vous devez fournir lors de la création de l’entrée des données de référence avec sa description :

|**Nom de la propriété**  |**Description**  |
|---------|---------|
|Alias d’entrée   | Nom convivial qui servira dans la requête de tâche pour faire référence à cette entrée.   |
|Compte de stockage   | Nom du compte de stockage dans lequel se trouvent vos blobs. S’il se trouve dans le même abonnement que votre travail Stream Analytics, vous pouvez le sélectionner dans la liste déroulante.   |
|Clé du compte de stockage   | Clé secrète associée au compte de stockage. Elle est remplie automatiquement si le compte de stockage se trouve dans le même abonnement que votre travail Stream Analytics.   |
|Conteneur de stockage   | Les conteneurs fournissent un regroupement logique des objets blob stockés dans le service d’objets blob Microsoft Azure. Lorsque vous téléchargez un objet blob dans le service d'objets Blob, vous devez spécifier un conteneur pour cet objet blob.   |
|Modèle de chemin d'accès   | C’est une propriété obligatoire qui est utilisée pour localiser vos objets blob dans le conteneur spécifié. Dans le chemin d’accès, vous pouvez choisir de spécifier une ou plusieurs instances de l’une des 2 variables suivantes :<BR>{date}, {time}<BR>Exemple 1 : products/{date}/{time}/product-list.csv<BR>Exemple 2 : products/{date}/product-list.csv<BR>Exemple 3 : product-list.csv<BR><br> Si l’objet blob n’existe pas dans le chemin d’accès spécifié, le travail Stream Analytics attend indéfiniment que l’objet blob devienne disponible.   |
|Format de la date [facultatif]   | Si vous avez utilisé {date} dans le modèle de chemin d’accès que vous avez spécifié, vous pouvez sélectionner le format de date selon lequel vos blobs sont organisés dans la liste déroulante des formats pris en charge.<BR>Exemple : AAAA/MM/JJ, MM/JJ/AAAA, etc.   |
|Format de l’heure [facultatif]   | Si vous avez utilisé {time} dans le modèle de chemin d’accès que vous avez spécifié, vous pouvez sélectionner le format d’heure selon lequel vos blobs sont organisés dans la liste déroulante des formats pris en charge.<BR>Exemple : HH, HH/mm ou HH-mm.  |
|Format de sérialisation de l’événement   | Pour s’assurer que vos requêtes fonctionnent comme prévu, Stream Analytics a besoin de connaître le format de sérialisation que vous utilisez pour les flux de données d’entrée. Pour les données de référence, les formats pris en charge sont CSV et JSON.  |
|Encodage   | UTF-8 est le seul format de codage actuellement pris en charge.  |

### <a name="static-reference-data"></a>Données de référence statiques

Si vos données de référence ne sont pas supposées changer, la prise en charge des données de référence statiques est activée en spécifiant un chemin d’accès statique dans la configuration d’entrée. Azure Stream Analytics récupère l’objet blob à partir du chemin spécifié. Les jetons de substitution {date} et {time} ne sont pas nécessaires. Étant donné que les données de référence sont immuables dans Stream Analytics, le remplacement d’un objet blob de données de référence statiques n’est pas recommandé.

### <a name="generate-reference-data-on-a-schedule"></a>Générer des données de référence dans une planification

Si vos données de référence sont un jeu de données variant lentement, la prise en charge de l’actualisation des données de référence peut être activée en spécifiant un modèle de chemin d’accès dans la configuration d’entrée à l’aide des jetons de substitution {date} et {time}. Stream Analytics collectera les définitions de données de référence mises à jour en fonction de ce modèle de chemin d’accès. Par exemple, un modèle `sample/{date}/{time}/products.csv` avec un format de date **« AAAA-MM-JJ »** et un format d’heure **« HH:mm »** donne pour instruction à Stream Analytics de sélectionner l’objet blob `sample/2015-04-16/17-30/products.csv` mis à jour à 17:30 le 16 avril 2015 (UTC).

Azure Stream Analytics analyse automatiquement les objets blob de données de référence actualisées à un intervalle d’une minute. Si un objet blob avec l’horodatage 10:30:00 est chargé avec un léger décalage (par exemple, 10:30:30), vous remarquerez un petit retard dans le travail Stream Analytics faisant référence à cet objet blob. Pour éviter de tels scénarios, il est recommandé de charger l’objet blob avant l’heure réelle cible (10:30:00 dans cet exemple), afin d’octroyer au travail Stream Analytics suffisamment de temps pour le découvrir et le charger en mémoire, puis d’effectuer des opérations. 

> [!NOTE]
> Actuellement, les tâches Stream Analytics recherchent l’actualisation des objets blob uniquement lorsque l’heure machine s’approche de l’heure encodée dans le nom de l’objet blob. Par exemple la tâche recherche `sample/2015-04-16/17-30/products.csv` dès que possible, mais aucune version antérieure à 17 h 30 le 16 avril 2015 (UTC). Elle ne recherche *jamais* un blob avec une heure encodée antérieure à la dernière heure détectée.
> 
> Par exemple, une fois que la tâche trouve l’objet blob `sample/2015-04-16/17-30/products.csv`, elle ignore tout fichier dont la date encodée est antérieure à 17 h 30 le 16 avril 2015. Donc, si un objet blob `sample/2015-04-16/17-25/products.csv` arrivant ultérieurement est créé dans le même conteneur, la tâche ne l’utilise pas.
> 
> De même, si `sample/2015-04-16/17-30/products.csv` est produit uniquement à 23 h 03 16 avril 2015 alors qu’aucun objet blob avec une date antérieure n’est présent dans le conteneur, la tâche utilise ce fichier en commençant à 23 h 03 le 16 avril 2015 et les données de référence antérieures à ce moment.
> 
> Une exception est faite à cette règle lorsque la tâche doit traiter de nouveau des données en revenant en arrière dans le temps ou lors du premier démarrage de la tâche. Au moment du démarrage, la tâche recherche l'objet blob le plus récent produit avant l'heure de début de la tâche spécifiée. Cela permet de garantir la présence d'un jeu de données de référence **non vide** au démarrage de la tâche. Si un aucun jeu de données n’est trouvé, la tâche affiche le diagnostic suivant : `Initializing input without a valid reference data blob for UTC time <start time>`.

[Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) peut être utilisé pour orchestrer la tâche de création de blobs mis à jour requise par Stream Analytics pour mettre à jour les définitions de données de référence. Data Factory est un service d’intégration de données dans le cloud qui gère et automatise le déplacement et la transformation des données. Data Factory prend en charge la [connexion à un grand nombre de magasins de données cloud et en local](../data-factory/copy-activity-overview.md) et le déplacement facile de données à intervalles réguliers que vous spécifiez. Pour plus d'informations et des instructions étape par étape sur la façon de configurer un pipeline Data Factory pour générer des données de référence pour Stream Analytics qui est actualisé selon une planification prédéfinie, consultez cet [exemple GitHub](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ReferenceDataRefreshForASAJobs).

### <a name="tips-on-refreshing-blob-reference-data"></a>Conseils pour l'actualisation des données de référence d’objets blob

1. N’écrasez pas les objets blob des données de référence car ils sont immuables.
2. La méthode recommandée pour actualiser les données de référence consiste à :
    * Utiliser {date} / {time} dans le modèle de chemin d’accès
    * Ajouter un nouvel objet blob à l’aide du même conteneur et du même chemin d’accès que ceux définis dans l’entrée du travail
    * Utiliser une date/heure **supérieure** à celle spécifiée par le dernier objet blob dans la séquence.
3. Les objets blob de données de référence ne sont **pas** triés selon l’heure de la « Dernière modification », mais uniquement selon l’heure et la date spécifiées dans le nom d’objet blob en utilisant les substitutions {date} et {time}.
3. Afin d’éviter de se retrouver avec une liste de blobs volumineuse, envisagez de supprimer les blobs très anciens pour lesquels le traitement ne se fera plus. Veuillez noter qu’il se peut qu’ASA doive traiter à nouveau une petite quantité dans certains scénarios, par exemple un redémarrage.

## <a name="azure-sql-database"></a>Azure SQL Database

Les données de référence Azure SQL Database sont récupérées par votre tâche Stream Analytics et sont stockées en tant qu’instantané en mémoire en vue de leur traitement. La capture instantanée de vos données de référence est également stockée dans un conteneur dans un compte de stockage que vous spécifiez dans les paramètres de configuration. Le conteneur est créé automatiquement lorsque le travail démarre. Si le travail est arrêté ou entre dans un état d’échec, les conteneurs créés automatiquement sont supprimés une fois le travail redémarré.  

Si vos données de référence sont un jeu de données à variation lente, vous devez régulièrement actualiser l’instantané qui est utilisé dans votre tâche. Stream Analytics vous permet de définir une fréquence de rafraîchissement lorsque vous configurez votre connexion d’entrée Azure SQL Database. Le runtime Stream Analytics va interroger votre base de données Azure SQL à l’intervalle spécifié par la fréquence d’actualisation. La fréquence de rafraîchissement la plus rapide prise en charge correspond à une fois par minute. Pour chaque actualisation, Stream Analytics stocke un nouvel instantané dans le compte de stockage fourni.

Stream Analytics fournit deux options pour l’interrogation de votre base de données Azure SQL. Une requête de capture instantanée est obligatoire et doit être incluse dans chaque tâche. Stream Analytics exécute la requête de capture instantanée régulièrement selon votre intervalle d’actualisation et utilise le résultat de la requête (la capture instantanée) en tant que jeu de données de référence. La requête de capture instantanée devrait convenir à la plupart des scénarios, mais si vous rencontrez des problèmes de performances avec les jeux de données volumineux et les fréquences d’actualisation rapides, vous pouvez utiliser l’option de requête delta. Les requêtes qui prennent plus de 60 secondes pour retourner le jeu de données de référence se traduisent par un délai d’attente.

Avec l’option de requête delta, Stream Analytics exécute la requête de capture instantanée initialement pour obtenir un jeu de données de référence de ligne de base. Ensuite, Stream Analytics exécute la requête delta régulièrement selon votre intervalle d’actualisation pour récupérer les modifications incrémentielles. Ces modifications incrémentielles sont appliquées en permanence au jeu de données de référence pour le maintenir à jour. L’utilisation de requêtes delta peut permettre de réduire les coûts de stockage et les opérations d’e/s réseau.

### <a name="configure-sql-database-reference"></a>Configurer une référence SQL Database

Pour configurer vos données de référence SQL Database, vous devez d'abord créer une entrée **Données de référence**. Le tableau ci-dessous explique chaque propriété que vous devez fournir lors de la création de l’entrée des données de référence avec sa description. Pour plus d'informations, consultez [Utiliser les données de référence d'une instance de SQL Database pour une tâche Azure Stream Analytics](sql-reference-data.md).

Vous pouvez utiliser [Azure SQL Managed Instance](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md) comme entrée de données de référence. Vous devez [configurer un point de terminaison public dans SQL Managed Instance](../azure-sql/managed-instance/public-endpoint-configure.md), puis configurer manuellement les paramètres suivants dans Azure Stream Analytics. Une machine virtuelle Azure exécutant SQL Server auquel une base de données est attachée est également prise en charge en configurant manuellement les paramètres ci-dessous.

|**Nom de la propriété**|**Description**  |
|---------|---------|
|Alias d’entrée|Nom convivial qui servira dans la requête de tâche pour faire référence à cette entrée.|
|Abonnement|Choisir votre abonnement|
|Base de données|Base de données Azure SQL qui contient vos données de référence. Pour SQL Managed Instance, il est nécessaire de spécifier le port 3342. Par exemple, *sampleserver.public.database.windows.net,3342*|
|Nom d’utilisateur|Nom d’utilisateur associé à votre base de données Azure SQL.|
|Mot de passe|Mot de passe associé à votre base de données Azure SQL.|
|Actualiser régulièrement|Cette option vous permet de choisir une fréquence d’actualisation. L’activation de cette option vous permet de spécifier la fréquence d’actualisation au format DD:HH:MM.|
|Requête d'instantané|Il s’agit de l’option de requête par défaut qui récupère les données de référence à partir de votre base de données SQL.|
|Requête delta|Pour les scénarios avancés avec des jeux de données volumineux et une courte fréquence d’actualisation, choisissez d’ajouter une requête delta.|

## <a name="size-limitation"></a>Limite de taille

Nous vous recommandons d’utiliser des jeux de données de référence qui sont inférieurs à 300 Mo pour des performances optimales. Les jeux de données de référence de 5 Go ou moins sont pris en charge dans les travaux avec 6 unités de diffusion en continu (SU) ou plus. L’utilisation de données de référence très volumineuses peut affecter la latence de bout en bout de votre travail. À mesure que la complexité des requêtes augmente pour inclure un traitement avec état comme les agrégations fenêtrées, les jointures temporelles et les fonctions analytiques temporelles, il est probable que la taille maximale prise en charge pour les données de référence diminue. Si Azure Stream Analytics ne peut pas charger les données de référence et effectuer des opérations complexes, le travail manque de mémoire et échoue. Dans ce cas, la métrique d’utilisation SU % atteindra 100 %.    

|**Nombre d’unités de streaming**  |**Taille recommandée**  |
|---------|---------|
|1   |50 Mo ou moins   |
|3   |150 Mo ou moins   |
|6 et au-delà   |5 Go ou moins.    |

La prise en charge de la compression n’est pas disponible pour les données de référence.

## <a name="joining-multiple-reference-datasets-in-a-job"></a>Jointure de plusieurs jeux de données de référence dans un travail
Vous pouvez joindre une seule entrée de flux avec une entrée de données de référence en une seule étape de votre requête. Toutefois, vous pouvez joindre plusieurs jeux de données de référence en fractionnant la requête en plusieurs étapes. Voici un exemple.

```SQL  
With Step1 as (
    --JOIN input stream with reference data to get 'Desc'
    SELECT streamInput.*, refData1.Desc as Desc
    FROM    streamInput
    JOIN    refData1 ON refData1.key = streamInput.key 
)
--Now Join Step1 with second reference data
SELECT *
INTO    output 
FROM    Step1
JOIN    refData2 ON refData2.Desc = Step1.Desc 
``` 

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.get.started]: ./stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: /stream-analytics-query/stream-analytics-query-language-reference
[stream.analytics.rest.api.reference]: /rest/api/streamanalytics/

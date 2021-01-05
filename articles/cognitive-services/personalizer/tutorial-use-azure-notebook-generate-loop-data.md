---
title: 'Tutoriel : Azure Notebook - Personalizer'
titleSuffix: Azure Cognitive Services
description: Ce tutoriel simule un système de boucle Personalizer dans un notebook Azure, qui suggère le type de café que doit commander un client. Les utilisateurs et leurs préférences sont stockés dans un jeu de données d’utilisateurs. Des informations sur le café sont également disponibles et stockées dans un jeu de données de cafés.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: tutorial
ms.date: 04/27/2020
ms.custom: devx-track-python
ms.openlocfilehash: 0cf2aa504fd7a36a55740b3a59eeb3759e67f469
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88509907"
---
# <a name="tutorial-use-personalizer-in-azure-notebook"></a>Tutoriel : Utiliser Personalizer dans Azure Notebook

Ce tutoriel exécute une boucle Personalizer dans un notebook Azure, illustrant le cycle de vie de bout en bout d’une boucle Personalizer.

La boucle suggère le type de café qu’un client doit commander. Les utilisateurs et leurs préférences sont stockés dans un jeu de données d’utilisateurs. Des informations sur le café sont stockées dans un jeu de données de cafés.

## <a name="users-and-coffee"></a>Utilisateurs et café

Le notebook, qui simule l’interaction utilisateur avec un site web, sélectionne un utilisateur aléatoire, l’heure de la journée et le type de météo dans le jeu de données. Voici un résumé des informations sur les utilisateurs :

|Clients - caractéristiques de contexte|Heures de la journée|Types de météo|
|--|--|--|
|Alice<br>Bob<br>Cathy<br>Dave|Matin<br>Après-midi<br>Soir|Ensoleillé<br>Pluvieux<br>Neige|

Pour aider Personalizer à apprendre, au fil du temps, le _système_ connaît également les détails concernant la sélection de café de chaque personne.

|Café - caractéristiques d’action|Types d’entités temperature|Lieu d’origine|Type de torréfaction|Bio|
|--|--|--|--|--|
|Cappuccino|À chaud|Kenya|Foncé|Bio|
|Infusion à froid|Froid|Brésil|Léger|Bio|
|Moka frappée|Froid|Éthiopie|Léger|Non bio|
|Latte|À chaud|Brésil|Foncé|Non bio|

L’**objectif** de la boucle Personalizer est de trouver le plus souvent possible la meilleure correspondance entre les utilisateurs et le café.

Le code de ce tutoriel est disponible dans le [dépôt GitHub d’exemples Personalizer](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/tree/master/samples/azurenotebook).

## <a name="how-the-simulation-works"></a>Fonctionnement de la simulation

Au début, seules 20 à 30 % des suggestions de Personalizer sont correctes. Cette réussite est indiquée par la récompense renvoyée à l’API de récompense (Reward) de Personalizer, avec un score de 1. Après un certain nombre d’appels de classement et de récompense, le système s’améliore.

Après les requêtes initiales, exécutez une évaluation hors connexion. Cela permet à Personalizer d’examiner les données et de suggérer une meilleure stratégie d’apprentissage. Appliquez la nouvelle stratégie d’apprentissage et réexécutez le notebook avec 20 % du nombre de requêtes précédentes. La boucle sera plus performante avec la nouvelle stratégie d’apprentissage.

## <a name="rank-and-reward-calls"></a>Appels de classement et de récompense

Pour chacun des quelques milliers d’appels au service Personalizer, le notebook Azure envoie la requête de **classement** à l’API REST :

* Un ID unique de l’événement de classement/requête
* Caractéristiques de contexte : un choix aléatoire d’utilisateur, de météo et d’heure de la journée, simulant un utilisateur sur un site web ou un appareil mobile
* Actions avec caractéristiques : _toutes_ les données sur le café, à partir desquelles Personalizer fait une suggestion

Le système reçoit la demande, puis compare cette prédiction avec le choix connu de l’utilisateur pour la même heure de la journée et la même météo. Si le choix connu est le même que le choix prédit, la **récompense** de 1 est renvoyée à Personalizer. Sinon, la récompense renvoyée est 0.

> [!Note]
> S’agissant d’une simulation, l’algorithme de récompense est simple. Dans un scénario réel, l’algorithme doit utiliser une logique métier, éventuellement avec des pondérations pour différents aspects de l’expérience du client, afin de déterminer le score de récompense.


## <a name="prerequisites"></a>Prérequis

* Un compte [Azure Notebook](https://notebooks.azure.com/)
* Une [ressource Azure Personalizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer)
    * Si vous avez déjà utilisé la ressource Personalizer, veillez à [effacer les données](how-to-settings.md#clear-data-for-your-learning-loop) dans le portail Azure pour la ressource.
* Charger tous les fichiers de [cet exemple](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/tree/master/samples/azurenotebook) dans un projet Azure Notebook

Descriptions des fichiers :

* [Personalizer.ipynb](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/blob/master/samples/azurenotebook/Personalizer.ipynb) est le notebook Jupyter pour ce tutoriel.
* Le [jeu de données d’utilisateurs](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/blob/master/samples/azurenotebook/users.json) est stocké dans un objet JSON.
* Le [jeu de données de cafés](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/blob/master/samples/azurenotebook/coffee.json) est stocké dans un objet JSON.
* L’[exemple de requête JSON](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/blob/master/samples/azurenotebook/example-rankrequest.json) est le format attendu pour une requête POST à l’API de classement.

## <a name="configure-personalizer-resource"></a>Configurer la ressource Personalizer

Dans le portail Azure, configurez votre ressource [Personalizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer) avec la **Fréquence de mise à jour du modèle** définie sur 15 secondes et une **Durée d’attente de la récompense** de 15 secondes. Ces valeurs figurent dans la page **[Configuration](how-to-settings.md#configure-service-settings-in-the-azure-portal)** .

|Paramètre|Valeur|
|--|--|
|Fréquence de mise à jour du modèle|15 secondes|
|Durée d’attente de la récompense|15 secondes|

Ces valeurs sont très brèves afin de pouvoir observer les changements dans ce tutoriel. Vous ne devez pas les utiliser dans un scénario de production sans avoir vérifié qu’elles vous permettent d’atteindre votre objectif avec votre boucle Personalizer.

## <a name="set-up-the-azure-notebook"></a>Configurer le notebook Azure

1. Remplacez le noyau par `Python 3.6`.
1. Ouvrez le fichier `Personalizer.ipynb` .

## <a name="run-notebook-cells"></a>Exécuter les cellules du notebook

Exécutez chaque cellule exécutable et attendez qu’elle retourne. Vous savez qu’elle est terminée quand les crochets à côté de la cellule affichent un nombre au lieu de `*`. Les sections suivantes expliquent ce que fait chaque cellule programmatiquement et ce qui est attendu pour la sortie.

### <a name="include-the-python-modules"></a>Inclure les modules python

Incluez les modules python requis. La cellule n’a pas de sortie.

```python
import json
import matplotlib.pyplot as plt
import random
import requests
import time
import uuid
```

### <a name="set-personalizer-resource-key-and-name"></a>Définir le nom et la clé de ressource Personalizer

À partir du portail Azure, recherchez votre clé et votre point de terminaison dans la page **Démarrage rapide** de votre ressource Personalizer. Remplacez la valeur de `<your-resource-name>` par le nom de votre ressource Personalizer. Remplacez la valeur de `<your-resource-key>` par votre clé Personalizer.

```python
# Replace 'personalization_base_url' and 'resource_key' with your valid endpoint values.
personalization_base_url = "https://<your-resource-name>.cognitiveservices.azure.com/"
resource_key = "<your-resource-key>"
```

### <a name="print-current-date-and-time"></a>Imprimer la date et l’heure actuelles
Utilisez cette fonction pour noter les heures de début et de fin de la fonction itérative.

Ces cellules n’ont pas de sortie. La fonction génère la date et l’heure actuelles quand elle est appelée.

```python
# Print out current datetime
def currentDateTime():
    currentDT = datetime.datetime.now()
    print (str(currentDT))
```

### <a name="get-the-last-model-update-time"></a>Obtenir l’heure de la dernière mise à jour du modèle

Quand la fonction `get_last_updated` est appelée, elle imprime la date et l’heure de la dernière modification du modèle.

Ces cellules n’ont pas de sortie. La fonction génère la date du dernier entraînement du modèle quand elle est appelée.

La fonction utilise une API REST pour [obtenir les propriétés du modèle](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/GetModelProperties).

```python
# ititialize variable for model's last modified date
modelLastModified = ""
```

```python
def get_last_updated(currentModifiedDate):

    print('-----checking model')

    # get model properties
    response = requests.get(personalization_model_properties_url, headers = headers, params = None)

    print(response)
    print(response.json())

    # get lastModifiedTime
    lastModifiedTime = json.dumps(response.json()["lastModifiedTime"])

    if (currentModifiedDate != lastModifiedTime):
        currentModifiedDate = lastModifiedTime
        print(f'-----model updated: {lastModifiedTime}')
```

### <a name="get-policy-and-service-configuration"></a>Obtenir la configuration du service et de la stratégie

Validez l’état du service à l’aide de ces deux appels REST.

Ces cellules n’ont pas de sortie. La fonction génère les valeurs du service quand elle est appelée.

```python
def get_service_settings():

    print('-----checking service settings')

    # get learning policy
    response = requests.get(personalization_model_policy_url, headers = headers, params = None)

    print(response)
    print(response.json())

    # get service settings
    response = requests.get(personalization_service_configuration_url, headers = headers, params = None)

    print(response)
    print(response.json())
```

### <a name="construct-urls-and-read-json-data-files"></a>Construire des URL et lire des fichiers de données JSON

Cette cellule

* Génère les URL utilisées dans les appels REST.
* Définit l’en-tête de sécurité à l’aide de votre clé de ressource Personalizer.
* Définit la valeur de départ aléatoire pour l’ID d’événement de classement.
* Lit les fichiers de données JSON.
* Appelle la méthode `get_last_updated` (la stratégie d’apprentissage a été supprimée dans l’exemple de sortie).
* Appelle la méthode `get_service_settings`.

La cellule contient la sortie de l’appel aux fonctions `get_last_updated` et `get_service_settings`.

```python
# build URLs
personalization_rank_url = personalization_base_url + "personalizer/v1.0/rank"
personalization_reward_url = personalization_base_url + "personalizer/v1.0/events/" #add "{eventId}/reward"
personalization_model_properties_url = personalization_base_url + "personalizer/v1.0/model/properties"
personalization_model_policy_url = personalization_base_url + "personalizer/v1.0/configurations/policy"
personalization_service_configuration_url = personalization_base_url + "personalizer/v1.0/configurations/service"

headers = {'Ocp-Apim-Subscription-Key' : resource_key, 'Content-Type': 'application/json'}

# context
users = "users.json"

# action features
coffee = "coffee.json"

# empty JSON for Rank request
requestpath = "example-rankrequest.json"

# initialize random
random.seed(time.time())

userpref = None
rankactionsjsonobj = None
actionfeaturesobj = None

with open(users) as handle:
    userpref = json.loads(handle.read())

with open(coffee) as handle:
    actionfeaturesobj = json.loads(handle.read())

with open(requestpath) as handle:
    rankactionsjsonobj = json.loads(handle.read())

get_last_updated(modelLastModified)
get_service_settings()

print(f'User count {len(userpref)}')
print(f'Coffee count {len(actionfeaturesobj)}')
```

Vérifiez que les valeurs `rewardWaitTime` et `modelExportFrequency` de la sortie sont toutes deux définies sur 15 secondes.

```console
-----checking model
<Response [200]>
{'creationTime': '0001-01-01T00:00:00+00:00', 'lastModifiedTime': '0001-01-01T00:00:00+00:00'}
-----model updated: "0001-01-01T00:00:00+00:00"
-----checking service settings
<Response [200]>
{...learning policy...}
<Response [200]>
{'rewardWaitTime': '00:00:15', 'defaultReward': 0.0, 'rewardAggregation': 'earliest', 'explorationPercentage': 0.2, 'modelExportFrequency': '00:00:15', 'logRetentionDays': -1}
User count 4
Coffee count 4
```

### <a name="troubleshooting-the-first-rest-call"></a>Résolution des problèmes liés au premier appel REST

Cette cellule précédente est la première cellule qui appelle Personalizer. Vérifiez que le code d’état REST dans la sortie est `<Response [200]>`. Si vous recevez une erreur, telle que 404, mais que vous êtes sûr que la clé de ressource et le nom sont corrects, rechargez le notebook.

Vérifiez qu’il y a bien quatre cafés et quatre utilisateurs. Si vous recevez une erreur, vérifiez que vous avez chargé les trois fichiers JSON.

### <a name="set-up-metric-chart-in-azure-portal"></a>Configurer le graphique de métriques dans le portail Azure

Plus loin dans ce tutoriel, le processus longue durée des 10 000 requêtes est visible à partir du navigateur avec une zone de texte mise à jour. Il peut être plus simple d’afficher un graphique ou une somme totale pour savoir quand le processus longue durée se termine. Pour voir ces informations, utilisez les métriques fournies avec la ressource. Vous pouvez créer le graphique maintenant que vous avez terminé une requête adressée au service, puis actualiser le graphique régulièrement pendant le processus longue durée.

1. Dans le portail Azure, sélectionnez votre ressource Personalizer.
1. Dans la barre de navigation des ressources, sélectionnez **Métriques** sous Supervision.
1. Dans le graphique, sélectionnez **Ajouter une métrique**.
1. Les espaces de noms de ressources et de métriques sont déjà définis. Il vous suffit de sélectionner la métrique des **appels ayant abouti** et l’agrégation de **somme**.
1. Modifiez le filtre de temps sur les quatre dernières heures.

    ![Configurez le graphique des métriques dans le portail Azure, en ajoutant une métrique pour les appels ayant abouti pour les quatre dernières heures.](./media/tutorial-azure-notebook/metric-chart-setting.png)

    Vous devez voir trois appels ayant abouti dans le graphique.

### <a name="generate-a-unique-event-id"></a>Générer un ID d’événement unique

Cette fonction génère un ID unique pour chaque appel de classement. L’ID est utilisé pour identifier les informations d’appel de récompense et de classement. Cette valeur peut provenir d’un processus métier, tel qu’un ID de vue web ou un ID de transaction.

La cellule n’a pas de sortie. La fonction génère l’ID unique quand elle est appelée.

```python
def add_event_id(rankjsonobj):
    eventid = uuid.uuid4().hex
    rankjsonobj["eventId"] = eventid
    return eventid
```

### <a name="get-random-user-weather-and-time-of-day"></a>Obtenir un utilisateur, une météo et une heure de la journée aléatoires

Cette fonction sélectionne un utilisateur, une météo et une heure de la journée, puis ajoute ces éléments à l’objet JSON à envoyer à la requête de classement.

La cellule n’a pas de sortie. Quand la fonction est appelée, elle retourne le nom de l’utilisateur aléatoire, la météo aléatoire et l’heure aléatoire de la journée.

La liste de quatre utilisateurs et leurs préférences (seules certaines préférences sont affichées par souci de concision) :

```json
{
  "Alice": {
    "Sunny": {
      "Morning": "Cold brew",
      "Afternoon": "Iced mocha",
      "Evening": "Cold brew"
    }...
  },
  "Bob": {
    "Sunny": {
      "Morning": "Cappucino",
      "Afternoon": "Iced mocha",
      "Evening": "Cold brew"
    }...
  },
  "Cathy": {
    "Sunny": {
      "Morning": "Latte",
      "Afternoon": "Cold brew",
      "Evening": "Cappucino"
    }...
  },
  "Dave": {
    "Sunny": {
      "Morning": "Iced mocha",
      "Afternoon": "Iced mocha",
      "Evening": "Iced mocha"
    }...
  }
}
```

```python
def add_random_user_and_contextfeatures(namesoption, weatheropt, timeofdayopt, rankjsonobj):
    name = namesoption[random.randint(0,3)]
    weather = weatheropt[random.randint(0,2)]
    timeofday = timeofdayopt[random.randint(0,2)]
    rankjsonobj['contextFeatures'] = [{'timeofday': timeofday, 'weather': weather, 'name': name}]
    return [name, weather, timeofday]
```


### <a name="add-all-coffee-data"></a>Ajouter toutes les données sur les cafés

Cette fonction ajoute la liste entière de cafés à l’objet JSON à envoyer à la requête de classement.

La cellule n’a pas de sortie. La fonction modifie `rankjsonobj` quand elle est appelée.


Voici un exemple des caractéristiques d’un café :

```json
{
    "id": "Cappucino",
    "features": [
    {
        "type": "hot",
        "origin": "kenya",
        "organic": "yes",
        "roast": "dark"

    }
}
```

```python
def add_action_features(rankjsonobj):
    rankjsonobj["actions"] = actionfeaturesobj
```

### <a name="compare-prediction-with-known-user-preference"></a>Comparer la prédiction avec la préférence connue de l’utilisateur

Cette fonction est appelée après l’appel de l’API de classement, pour chaque itération.

Elle compare la préférence de l’utilisateur en terme de café, en fonction de la météo et de l’heure de la journée, avec la suggestion de Personalizer pour l’utilisateur et pour ces filtres. Si la suggestion correspond, un score de 1 est retourné ; sinon, le score est 0. La cellule n’a pas de sortie. La fonction génère le score quand elle est appelée.

```python
def get_reward_from_simulated_data(name, weather, timeofday, prediction):
    if(userpref[name][weather][timeofday] == str(prediction)):
        return 1
    return 0
```

### <a name="loop-through-calls-to-rank-and-reward"></a>Parcourir les appels de classement et de récompense en boucle

La cellule suivante correspond au travail _principal_ du notebook : obtenir un utilisateur aléatoire, obtenir la liste des cafés, envoyer ces deux informations à l’API de classement, comparer la prédiction avec les préférences connues de l’utilisateur, puis renvoyer la récompense au service Personalizer.

La boucle s’exécute `num_requests` fois. Personalizer a besoin de quelques milliers d’appels de classement et de récompense pour créer un modèle.

Voici un exemple de code JSON envoyé à l’API de classement. La liste des cafés n’est pas complète, par souci de concision. Vous pouvez voir la totalité du JSON pour le café dans `coffee.json`.

JSON envoyé à l’API de classement :

```json
{
   'contextFeatures':[
      {
         'timeofday':'Evening',
         'weather':'Snowy',
         'name':'Alice'
      }
   ],
   'actions':[
      {
         'id':'Cappucino',
         'features':[
            {
               'type':'hot',
               'origin':'kenya',
               'organic':'yes',
               'roast':'dark'
            }
         ]
      }
        ...rest of coffee list
   ],
   'excludedActions':[

   ],
   'eventId':'b5c4ef3e8c434f358382b04be8963f62',
   'deferActivation':False
}
```

Réponse JSON de l’API de classement :

```json
{
    'ranking': [
        {'id': 'Latte', 'probability': 0.85 },
        {'id': 'Iced mocha', 'probability': 0.05 },
        {'id': 'Cappucino', 'probability': 0.05 },
        {'id': 'Cold brew', 'probability': 0.05 }
    ],
    'eventId': '5001bcfe3bb542a1a238e6d18d57f2d2',
    'rewardActionId': 'Latte'
}
```

Pour finir, chaque boucle affiche la sélection aléatoire d’utilisateur, de météo et d’heure de la journée, ainsi que la récompense déterminée. La récompense 1 indique que la ressource Personalizer a sélectionné le bon type de café pour l’utilisateur, la météo et l’heure de la journée donnés.

```console
1 Alice Rainy Morning Latte 1
```

La fonction utilise :

* Classement : une API REST POST pour [obtenir le classement](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Rank).
* Récompense : une API REST POST pour [indiquer la récompense](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Reward).

```python
def iterations(n, modelCheck, jsonFormat):

    i = 1

    # default reward value - assumes failed prediction
    reward = 0

    # Print out dateTime
    currentDateTime()

    # collect results to aggregate in graph
    total = 0
    rewards = []
    count = []

    # default list of user, weather, time of day
    namesopt = ['Alice', 'Bob', 'Cathy', 'Dave']
    weatheropt = ['Sunny', 'Rainy', 'Snowy']
    timeofdayopt = ['Morning', 'Afternoon', 'Evening']


    while(i <= n):

        # create unique id to associate with an event
        eventid = add_event_id(jsonFormat)

        # generate a random sample
        [name, weather, timeofday] = add_random_user_and_contextfeatures(namesopt, weatheropt, timeofdayopt, jsonFormat)

        # add action features to rank
        add_action_features(jsonFormat)

        # show JSON to send to Rank
        print('To: ', jsonFormat)

        # choose an action - get prediction from Personalizer
        response = requests.post(personalization_rank_url, headers = headers, params = None, json = jsonFormat)

        # show Rank prediction
        print ('From: ',response.json())

        # compare personalization service recommendation with the simulated data to generate a reward value
        prediction = json.dumps(response.json()["rewardActionId"]).replace('"','')
        reward = get_reward_from_simulated_data(name, weather, timeofday, prediction)

        # show result for iteration
        print(f'   {i} {currentDateTime()} {name} {weather} {timeofday} {prediction} {reward}')

        # send the reward to the service
        response = requests.post(personalization_reward_url + eventid + "/reward", headers = headers, params= None, json = { "value" : reward })

        # for every N rank requests, compute total correct  total
         total =  total + reward

        # every N iteration, get last updated model date and time
        if(i % modelCheck == 0):

            print("**** 10% of loop found")

            get_last_updated(modelLastModified)

        # aggregate so chart is easier to read
        if(i % 10 == 0):
            rewards.append( total)
            count.append(i)
             total = 0

        i = i + 1

    # Print out dateTime
    currentDateTime()

    return [count, rewards]
```

## <a name="run-for-10000-iterations"></a>Exécuter pour 10 000 itérations
Exécutez la boucle Personalizer pour 10 000 itérations. Il s’agit d’un événement de longue durée. Ne fermez pas le navigateur qui exécute le notebook. Actualisez régulièrement le graphique des métriques dans le portail Azure pour voir le nombre total d’appels au service. Quand vous avez environ 20 000 appels, un appel de classement et de récompense pour chaque itération de la boucle, les itérations sont terminées.

```python
# max iterations
num_requests = 200

# check last mod date N% of time - currently 10%
lastModCheck = int(num_requests * .10)

jsonTemplate = rankactionsjsonobj

# main iterations
[count, rewards] = iterations(num_requests, lastModCheck, jsonTemplate)
```



## <a name="chart-results-to-see-improvement"></a>Afficher un graphique des résultats pour voir l’amélioration

Créez un graphique à partir de `count` et `rewards`.

```python
def createChart(x, y):
    plt.plot(x, y)
    plt.xlabel("Batch of rank events")
    plt.ylabel("Correct recommendations per batch")
    plt.show()
```

## <a name="run-chart-for-10000-rank-requests"></a>Exécuter le graphique pour 10 000 requêtes de classement

Exécutez la fonction `createChart`.

```python
createChart(count,rewards)
```

## <a name="reading-the-chart"></a>Lecture du graphique

Ce graphique montre la réussite du modèle pour la stratégie d’apprentissage par défaut actuelle.

![Ce graphique montre la réussite de la stratégie d’apprentissage actuelle pendant la durée du test.](./media/tutorial-azure-notebook/azure-notebook-chart-results.png)


La cible idéale est qu’à la fin du test, la boucle obtienne un taux de réussite moyen proche de 100 % moins l’exploration. La valeur d’exploration par défaut est 20 %.

`100-20=80`

Cette valeur d’exploration figure dans le portail Azure pour la ressource Personalizer, dans la page **Configuration**.

Pour trouver une meilleure stratégie d’apprentissage, basée sur vos données envoyées à l’API de classement, exécutez une [évaluation hors connexion](how-to-offline-evaluation.md) dans le portail pour votre boucle Personalizer.

## <a name="run-an-offline-evaluation"></a>Exécuter une évaluation hors connexion

1. Dans le portail Azure, ouvrez la page **Évaluations** de la ressource Personalizer.
1. Sélectionnez **Créer une évaluation**.
1. Entrez les données requises (nom de l’évaluation et plage de dates pour l’évaluation de la boucle). La plage de dates doit inclure uniquement les jours sur lesquels vous vous concentrez pour votre évaluation.
    ![Dans le portail Azure, ouvrez la page Évaluations de la ressource Personalizer. Sélectionnez Créer une évaluation. Entrez le nom de l’évaluation et la plage de dates.](./media/tutorial-azure-notebook/create-offline-evaluation.png)

    L’objectif de cette évaluation hors connexion est de déterminer s’il existe une meilleure stratégie d’apprentissage pour les caractéristiques et actions utilisées dans cette boucle. Pour trouver cette meilleure stratégie d’apprentissage, n’oubliez pas d’activer l’option **Découverte d’optimisation**.

1. Sélectionnez **OK** pour commencer l’évaluation.
1. Cette page **Évaluations** montre la nouvelle évaluation et son état actuel. En fonction de la quantité de données dont vous disposez, cette évaluation peut prendre un certain temps. Vous pouvez revenir à cette page après quelques minutes pour voir les résultats.
1. Une fois l’évaluation terminée, sélectionnez-la, puis sélectionnez **Comparaison des différentes stratégies d’apprentissage**. Cela montre les stratégies d’apprentissage disponibles et comment elles se comporteraient avec les données.
1. Sélectionnez la stratégie d’apprentissage qui figure en haut du tableau, puis sélectionnez **Appliquer**. La _meilleure_ stratégie d’apprentissage est alors appliquée à votre modèle, et un nouvel entraînement est effectué.

## <a name="change-update-model-frequency-to-5-minutes"></a>Régler la fréquence de mise à jour du modèle sur cinq minutes

1. Dans le portail Azure, toujours sur la ressource Personalizer, sélectionnez la page **Configuration**.
1. Réglez la **Fréquence de mise à jour du modèle** et la **Durée d’attente de la récompense** sur cinq minutes et sélectionnez **Enregistrer**.

Apprenez-en davantage sur la [durée d’attente de la récompense](concept-rewards.md#reward-wait-time) et sur la [fréquence de mise à jour du modèle](how-to-settings.md#model-update-frequency).

```python
#Verify new learning policy and times
get_service_settings()
```

Vérifiez que les valeurs `rewardWaitTime` et `modelExportFrequency` de la sortie sont toutes deux définies sur cinq minutes.
```console
-----checking model
<Response [200]>
{'creationTime': '0001-01-01T00:00:00+00:00', 'lastModifiedTime': '0001-01-01T00:00:00+00:00'}
-----model updated: "0001-01-01T00:00:00+00:00"
-----checking service settings
<Response [200]>
{...learning policy...}
<Response [200]>
{'rewardWaitTime': '00:05:00', 'defaultReward': 0.0, 'rewardAggregation': 'earliest', 'explorationPercentage': 0.2, 'modelExportFrequency': '00:05:00', 'logRetentionDays': -1}
User count 4
Coffee count 4
```

## <a name="validate-new-learning-policy"></a>Valider la nouvelle stratégie d’apprentissage

Revenez au fichier Azure Notebooks et continuez en exécutant la même boucle, mais seulement pendant 2000 itérations. Actualisez régulièrement le graphique des métriques dans le portail Azure pour voir le nombre total d’appels au service. Quand vous avez environ 4 000 appels, un appel de classement et de récompense pour chaque itération de la boucle, les itérations sont terminées.

```python
# max iterations
num_requests = 2000

# check last mod date N% of time - currently 10%
lastModCheck2 = int(num_requests * .10)

jsonTemplate2 = rankactionsjsonobj

# main iterations
[count2, rewards2] = iterations(num_requests, lastModCheck2, jsonTemplate)
```

## <a name="run-chart-for-2000-rank-requests"></a>Exécuter le graphique pour 2 000 requêtes de classement

Exécutez la fonction `createChart`.

```python
createChart(count2,rewards2)
```

## <a name="review-the-second-chart"></a>Examiner le deuxième graphique

Le deuxième graphique doit montrer une augmentation visible de la correspondance entre les prédictions de classement et les préférences de l’utilisateur.

![Le deuxième graphique doit montrer une augmentation visible de la correspondance entre les prédictions de classement et les préférences de l’utilisateur.](./media/tutorial-azure-notebook/azure-notebook-chart-results-happy-graph.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’envisagez pas de continuer la série de tutoriels, nettoyez les ressources suivantes :

* Supprimez votre projet Azure Notebook.
* Supprimez votre ressource Personalizer.

## <a name="next-steps"></a>Étapes suivantes

Le [notebook Jupyter et les fichiers de données](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/tree/master/samples/azurenotebook) utilisés dans cet exemple sont disponibles dans le dépôt GitHub pour Personalizer.


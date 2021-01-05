---
title: 'ML Studio (classique) : Gérer les services web à l’aide de Gestion des API – Azure'
description: Guide montrant comment gérer les services web AzureML à l’aide de la gestion des API Gérez vos points de terminaison d’API REST en définissant l’accès utilisateur, la limitation d’utilisation et la surveillance du tableau de bord.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 11/03/2017
ms.openlocfilehash: b00e75c5fda8a05f4ed0f3a756ba20cca570ba5c
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305995"
---
# <a name="manage-azure-machine-learning-studio-classic-web-services-using-api-management"></a>Gérer les services web Azure Machine Learning Studio (classique) à l’aide de Gestion des API

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


## <a name="overview"></a>Vue d’ensemble
Ce guide décrit la prise en main rapide de la gestion de vos services web Azure Machine Learning Studio (classique) grâce à Gestion des API.

## <a name="what-is-azure-api-management"></a>Qu’est-ce que Gestion des API Azure ?
Gestion des API Azure est un service Azure qui vous permet de gérer vos points de terminaison d’API REST en définissant l’accès utilisateur, la limitation d’utilisation et la surveillance du tableau de bord. Pour plus d’informations, rendez-vous sur le [site de Gestion des API Azure](https://azure.microsoft.com/services/api-management/). Pour prendre en main Gestion des API Azure, consultez [le guide d’importation et de publication](../../api-management/import-and-publish.md). Cet autre guide, sur lequel ce guide est basé, couvre plus de rubriques, notamment les configurations de notification, la tarification, la gestion des réponses, l’authentification des utilisateurs, la création de produits, les abonnements pour développeur et le tableau de bord d’utilisation.

## <a name="prerequisites"></a>Prérequis
Pour utiliser ce guide, il vous faut :

* Un compte Azure.
* Un compte AzureML.
* L’espace de travail, le service et l’api_key pour l’expérience AzureML déployés sous forme de service web. Pour plus d’informations sur la création d’une expérience AzureML, consultez le guide de [démarrage rapide Studio](create-experiment.md). Pour plus d’informations sur le déploiement d’une expérience Studio (classique) en tant que service web, consultez le [guide pratique d’un déploiement Studio](deploy-a-machine-learning-web-service.md), à la section relative au déploiement d’une expérience AzureML en tant que service web. L’annexe A contient également des instructions sur la façon de créer et de tester une expérience AzureML simple et de la déployer en tant que service web.

## <a name="create-an-api-management-instance"></a>Création d'une instance Gestion des API

Vous pouvez gérer votre service web Azure Machine Learning au moyen d’une instance Gestion des API.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **+ Créer une ressource**.
3. Dans la zone de recherche, tapez « Gestion des API », puis sélectionnez la ressource « Gestion des API ».
4. Cliquez sur **Créer**.
5. La valeur **Nom** est utilisée plus tard pour créer une URL unique (cet exemple utilise « demoazureml »).
6. Sélectionnez un **Abonnement** , un **Groupe de ressources** et un **Emplacement** pour votre instance de service.
7. Spécifiez une valeur pour le **Nom de l’organisation** (cet exemple utilise « demoazureml »).
8. Entrez votre **E-mail de l’administrateur** , cet e-mail est utilisé plus tard pour les notifications depuis le système Gestion des API.
9. Cliquez sur **Créer**.

La création d’un service peut durer jusqu’à 30 minutes.

![Capture d’écran montrant la boîte de dialogue du service Gestion des API avec les options requises pour créer un service.](./media/manage-web-service-endpoints-using-api-management/create-service.png)


## <a name="create-the-api"></a>Création de l’API
Une fois l’instance de service créée, l’étape suivante consiste à créer une API. Une API se compose d’un ensemble d’opérations pouvant être appelées à partir d’une application cliente. Les opérations de l’API sont transmises par proxy aux services web existants. Ce guide crée des API comme proxy pour les services web AzureML RRS et BES existants.

Pour créer l’API :

1. Dans le Portail Azure, ouvrez l’instance de service que vous avez créée.
2. Dans le volet de navigation de gauche, sélectionnez **API**.

   ![api-management-menu](./media/manage-web-service-endpoints-using-api-management/api-management.png)

1. Cliquez sur **Ajouter l’API**.
2. Entrez un **Nom de l’API web** (cet exemple utilise « API de démonstration AzureML »).
3. Pour **URL du service Web** , entrez « `https://ussouthcentral.services.azureml.net` ».
4. Indiquez un « **Suffixe de l’URL de l’API web ». Il constitue la dernière partie de l’URL que les clients vont utiliser pour envoyer des requêtes à l’instance de service (cet exemple utilise « azureml-demo »).
5. Pour **Modèle d’URL de l’API Web** , sélectionnez **HTTPS**.
6. Pour **Produits** , sélectionnez **Starter**.
7. Cliquez sur **Enregistrer**.


## <a name="add-the-operations"></a>Ajout des opérations

Les opérations sont ajoutées et configurées dans une API sur le portail des éditeurs. Pour accéder au portail des éditeurs, cliquez sur **Portail des éditeurs** dans le portail Azure de votre service Gestion des API, sélectionnez **API** , **Opérations** , puis cliquez sur **Ajouter une opération**.

![add-operation](./media/manage-web-service-endpoints-using-api-management/add-an-operation.png)

La fenêtre **Nouvelle opération** s’affiche et l’onglet **Signature** est sélectionné par défaut.

## <a name="add-rrs-operation"></a>Ajout d’une opération RRS
En premier lieu, créez une opération pour le service AzureML RRS :

1. Pour le **Verbe HTTP** , sélectionnez **POST**.
2. Pour le **Modèle d’URL** , tapez « `/workspaces/{workspace}/services/{service}/execute?api-version={apiversion}&details={details}` ».
3. Entrez un **Nom d’affichage** (cet exemple utilise « Exécution RRS »).

   ![Capture d’écran montrant la page Signature dans laquelle vous pouvez entrer un nom d’affichage.](./media/manage-web-service-endpoints-using-api-management/add-rrs-operation-signature.png)

4. Cliquez sur **Réponses** > **AJOUTER** sur la gauche et sélectionnez **200 OK**.
5. Cliquez sur **Enregistrer** pour enregistrer cette opération.

   ![Capture d’écran montrant la page d’exécution d’opération RRS avec un bouton Enregistrer.](./media/manage-web-service-endpoints-using-api-management/add-rrs-operation-response.png)

## <a name="add-bes-operations"></a>Ajout d’opérations BES

> [!NOTE]
> Les captures d’écran ne sont pas incluses ici pour les opérations BES, car elles sont très similaires à celles de l’ajout de l’opération RRS.

### <a name="submit-but-not-start-a-batch-execution-job"></a>Envoi (et non démarrage) d’un travail d’exécution de lot

1. Cliquez sur **Ajouter une opération** pour ajouter une opération BES à l’API.
2. Pour le **Verbe HTTP** , sélectionnez **POST**.
3. Pour le **Modèle d’URL** , tapez « `/workspaces/{workspace}/services/{service}/jobs?api-version={apiversion}` ».
4. Entrez un **Nom d’affichage** (cet exemple utilise « Envoi BES »).
5. Cliquez sur **Réponses** > **AJOUTER** sur la gauche et sélectionnez **200 OK**.
6. Cliquez sur **Enregistrer**.

### <a name="start-a-batch-execution-job"></a>Démarrage d’une tâche d’exécution de lot

1. Cliquez sur **Ajouter une opération** pour ajouter une opération BES à l’API.
2. Pour le **Verbe HTTP** , sélectionnez **POST**.
3. Pour le **verbe HTTP** , tapez « `/workspaces/{workspace}/services/{service}/jobs/{jobid}/start?api-version={apiversion}` ».
4. Entrez un **Nom d’affichage** (cet exemple utilise « Démarrage BES »).
6. Cliquez sur **Réponses** > **AJOUTER** sur la gauche et sélectionnez **200 OK**.
7. Cliquez sur **Enregistrer**.

### <a name="get-the-status-or-result-of-a-batch-execution-job"></a>Obtention de l’état ou du résultat d’une tâche d’exécution de lot

1. Cliquez sur **Ajouter une opération** pour ajouter une opération BES à l’API.
2. Pour le **Verbe HTTP** , sélectionnez **GET**.
3. Pour le **Modèle d’URL** , tapez « `/workspaces/{workspace}/services/{service}/jobs/{jobid}?api-version={apiversion}` ».
4. Entrez un **Nom d’affichage** (cet exemple utilise « État BES »).
6. Cliquez sur **Réponses** > **AJOUTER** sur la gauche et sélectionnez **200 OK**.
7. Cliquez sur **Enregistrer**.

### <a name="delete-a-batch-execution-job"></a>Suppression d’une tâche d’exécution de lot

1. Cliquez sur **Ajouter une opération** pour ajouter une opération BES à l’API.
2. Pour le **Verbe HTTP** , sélectionnez **DELETE**.
3. Pour le **Modèle d’URL** , tapez « `/workspaces/{workspace}/services/{service}/jobs/{jobid}?api-version={apiversion}` ».
4. Entrez un **Nom d’affichage** (cet exemple utilise « Suppression BES »).
5. Cliquez sur **Réponses** > **AJOUTER** sur la gauche et sélectionnez **200 OK**.
6. Cliquez sur **Enregistrer**.

## <a name="call-an-operation-from-the-developer-portal"></a>Appel d’une opération à partir du portail des développeurs

Les opérations peuvent être directement appelées depuis le portail des développeurs, ce qui permet d'afficher et de tester les opérations d'une API. À cette étape, vous appellerez la méthode **Exécution RRS** qui a été ajoutée à l’ **API de démonstration AzureML**. 

1. Cliquez sur **Portail des développeurs**.

   ![Capture d’écran montrant le lien du portail des développeurs.](./media/manage-web-service-endpoints-using-api-management/developer-portal.png)

2. Cliquez sur **API** dans le menu en haut, puis sur **API de démonstration AzureML** pour afficher les opérations disponibles.

   ![Capture d’écran montrant le lien de l’API de démonstration d’Azure ML.](./media/manage-web-service-endpoints-using-api-management/demoazureml-api.png)

3. Sélectionnez **Exécution RRS** pour l’opération. Cliquez sur **Essayer**.

   ![Capture d’écran montrant la boîte de dialogue de l’API de démonstration d’Azure ML, avec l’exécution post RRS sélectionnée et un bouton Essayer.](./media/manage-web-service-endpoints-using-api-management/try-it.png)

4. Pour les **Paramètres de requête** , tapez votre **espace de travail** et votre **service** , tapez « 2.0 » pour la **versionapi** et « true » pour les **détails**. Vous pouvez trouver vos **espace de travail** et **service** dans le tableau de bord du service web AzureML (voir **Test du service web** dans l’annexe A).

   Pour **En-têtes de demande** , cliquez sur **Ajouter un en-tête** , tapez « Content-Type » et « application/json ». Cliquez de nouveau sur **Ajouter un en-tête** , puis tapez « Authorization » et « Bearer *\<your service API-KEY\>*  ». Vous pouvez trouver cette clé API dans le tableau de bord du service web AzureML (voir **Tester le service web dans l’annexe A** ).

   Pour le **Corps de la demande** , tapez `{"Inputs": {"input1": {"ColumnNames": ["Col2"], "Values": [["This is a good day"]]}}, "GlobalParameters": {}}`.

   ![Capture d’écran montrant les paramètres de demande, les en-têtes de demande, le corps de la demande et l’autorisation de l’API de démonstration d’Azure ML.](./media/manage-web-service-endpoints-using-api-management/azureml-demo-api.png)

5. Cliquez sur **Envoyer**.

   ![Capture d’écran montrant un bouton Envoyer.](./media/manage-web-service-endpoints-using-api-management/send.png)

Après l’appel d’une opération, le portail des développeurs affiche **l’URL requise** à partir du service principal, **l’état de la réponse** , les **en-têtes de la réponse** et tout **contenu de la réponse**.

![Capture d’écran montrant le portail des développeurs affichant l’état de la réponse, la latence de réponse, les en-têtes de réponse et le contenu de réponse.](./media/manage-web-service-endpoints-using-api-management/response-status.png)

## <a name="appendix-a---creating-and-testing-a-simple-azureml-web-service"></a>Annexe A - Création et test d’un service web AzureML simple
### <a name="creating-the-experiment"></a>Création de l’expérience
Vous trouverez ci-dessous les étapes de création d’une expérience AzureML simple et de son déploiement comme service web. Le service web prend comme entrée une colonne de texte arbitraire et retourne un ensemble de fonctionnalités représentées sous forme d’entiers. Par exemple :

| Texte | Texte haché |
| --- | --- |
| C’est une belle journée |1 1 2 2 0 2 0 1 |

Tout d’abord, à l’aide du navigateur de votre choix, accédez à [https://studio.azureml.net/](https://studio.azureml.net/) et entrez vos informations d’identification pour vous connecter. Ensuite, créez une nouvelle expérience vide.

![Capture d’écran montrant une NOUVELLE page avec EXPERIMENT sélectionné et une recherche de texte.](./media/manage-web-service-endpoints-using-api-management/search-experiment-templates.png)

Renommez-la **SimpleFeatureHashingExperiment**. Développez **Datasets enregistrés** et faites glisser **Critiques de livres d’Amazon** sur votre expérimentation.

![Capture d’écran montrant des exemples sur le côté gauche et un volet SimpleFeatureHashingExperiment à droite avec l’instruction Faire glisser les éléments ici.](./media/manage-web-service-endpoints-using-api-management/simple-feature-hashing-experiment.png)

Développez **Transformation des données** et **Manipulation** et faites glisser **Sélectionner des colonnes dans le jeu de données** sur votre expérimentation. Connectez **Critiques de livres d’Amazon** à **Sélectionner des colonnes dans le jeu de données**.

![Connecter le module de jeu de données de critiques de livres à un module de colonnes de projet](./media/manage-web-service-endpoints-using-api-management/project-columns.png)

Cliquez sur **Sélectionner des colonnes dans le jeu de données** , puis sur **Exécuter le sélecteur de colonne** et sélectionnez **Col2**. Cliquez sur la coche pour appliquer ces modifications.

![Sélectionner des colonnes à l'aide de leurs noms](./media/manage-web-service-endpoints-using-api-management/select-columns.png)

Développez **Analyse de texte** et faites glisser **Fonction de hachage** sur l’expérimentation. Connectez **Sélectionner des colonnes dans le jeu de données** à **Fonction de hachage**.

![Capture d’écran montrant l’ajout d’un élément de hachage de fonctionnalité à l’espace de travail.](./media/manage-web-service-endpoints-using-api-management/connect-project-columns.png)

Tapez **3** pour la **Taille de bits de hachage**. Cela crée 8 (23) colonnes.

![Capture d’écran montrant des propriétés avec le hachage des fonctionnalités sélectionné, où vous pouvez entrer une taille de bit hachage.](./media/manage-web-service-endpoints-using-api-management/hashing-bitsize.png)

À ce stade, vous pouvez cliquer sur **Exécuter** pour tester l’expérience.

![Capture d’écran montrant un bouton EXÉCUTER.](./media/manage-web-service-endpoints-using-api-management/run.png)

### <a name="create-a-web-service"></a>Création d’un service web
Maintenant, créez un service web. Développez **Service Web** et faites glisser **Entrée** sur votre expérimentation. Connectez **Entrée** à **Fonction de hachage**. Faites également glisser **Sortie** sur votre expérience. Connectez **Sortie** à **Fonction de hachage**.

![Capture d’écran  montrant l’espace de travail après les modifications spécifiées.](./media/manage-web-service-endpoints-using-api-management/output-to-feature-hashing.png)

Cliquez sur **Publier le service Web**.

![Capture d’écran montrant un bouton PUBLIER LE SERVICE WEB.](./media/manage-web-service-endpoints-using-api-management/publish-web-service.png)

Cliquez sur **Oui** pour publier l’expérience.

![Capture d’écran montrant un message de confirmation et l’option permettant de publier ou non.](./media/manage-web-service-endpoints-using-api-management/yes-to-publish.png)

### <a name="test-the-web-service"></a>Test du service web
Un service web AzureML se compose de points de terminaison RRS (service de requête/réponse) et BES (service d’exécution de lot). RSS est conçu pour l’exécution synchrone. BES est conçu pour l’exécution de tâches asynchrone. Pour tester un service web avec l’exemple de source Python ci-dessous, vous devrez peut-être télécharger et installer le Kit de développement logiciel (SDK) Microsoft Azure pour Python (voir : [Installation de Python](/azure/developer/python/azure-sdk-install)).

Il vous faudra également **l’espace de travail** , le **service** et **l’api_key** de votre expérimentation pour l’exemple de source ci-dessous. Vous pouvez trouver l’espace de travail et le service en cliquant sur **Requête/réponse** ou **Exécution de lot** pour votre expérimentation dans le tableau de bord de service web.

![Capture d’écran montrant le volet Demande dans lequel vous pouvez trouver les valeurs d’espace de travail et de service.](./media/manage-web-service-endpoints-using-api-management/find-workspace-and-service.png)

Vous pouvez trouver **l’api_key** en cliquant sur votre expérimentation dans le tableau de bord de service web.

![Capture d’écran montrant l’expérience dans le tableau de bord du service web, où vous pouvez trouver la clé API.](./media/manage-web-service-endpoints-using-api-management/find-api-key.png)

#### <a name="test-rrs-endpoint"></a>Test du point de terminaison RRS
##### <a name="test-button"></a>Bouton de test
Un moyen simple de tester le point de terminaison RRS consiste à cliquer sur **Test** sur le tableau de bord du service web.

![Capture d’écran montrant l’expérience dans le tableau de bord du service web, qui contient le bouton Test.](./media/manage-web-service-endpoints-using-api-management/test.png)

Tapez **C’est une belle journée** en **col2**. Cliquez sur la coche.

![Capture d’écran montrant la boîte de dialogue Entrer les données à prédire, dans laquelle vous pouvez entrer du texte comme dans l’exemple C’est une belle journée.](./media/manage-web-service-endpoints-using-api-management/enter-data.png)

Le résultat suivant doit s’afficher :

![Capture d’écran montrant le résultat de l’expérience, qui lit C’est une belle journée et quelques chiffres entre guillemets.](./media/manage-web-service-endpoints-using-api-management/sample-output.png)

##### <a name="sample-code"></a>Exemple de code
Vous pouvez également tester votre RRS à partir de votre code client. Si vous cliquez sur **Requête/réponse** sur le tableau de bord et faites défiler la liste vers le bas, vous trouverez des exemples de code pour C#, Python et R. Vous trouverez également la syntaxe de la requête RRS, y compris l’URI, les en-têtes et le corps de la requête.

Ce guide fournit un exemple Python opérationnel. Vous devrez le modifier avec les **espace de travail** , **service** et **api_key** de votre expérimentation.

```python
import urllib2
import json
workspace = "<REPLACE WITH YOUR EXPERIMENT'S WEB SERVICE WORKSPACE ID>"
service = "<REPLACE WITH YOUR EXPERIMENT'S WEB SERVICE SERVICE ID>"
api_key = "<REPLACE WITH YOUR EXPERIMENT'S WEB SERVICE API KEY>"
data = {
"Inputs": {
    "input1": {
        "ColumnNames": ["Col2"],
        "Values": [ [ "This is a good day" ] ]
    },
},
"GlobalParameters": { }
}
url = "https://ussouthcentral.services.azureml.net/workspaces/" + workspace + "/services/" + service + "/execute?api-version=2.0&details=true"
headers = {'Content-Type':'application/json', 'Authorization':('Bearer '+ api_key)}
body = str.encode(json.dumps(data))
try:
    req = urllib2.Request(url, body, headers)
    response = urllib2.urlopen(req)
    result = response.read()
    print "result:" + result
        except urllib2.HTTPError, error:
    print("The request failed with status code: " + str(error.code))
    print(error.info())
    print(json.loads(error.read()))
```

#### <a name="test-bes-endpoint"></a>Test du point de terminaison BES
Cliquez sur **Exécution de lot** sur le tableau de bord et faites défiler la liste vers le bas. Vous trouverez des exemples de code pour C#, Python et R. Vous trouverez également la syntaxe des requêtes BES pour soumettre une tâche, démarrer une tâche, obtenir l’état ou les résultats d’une tâche et supprimer une tâche.

Ce guide fournit un exemple Python opérationnel. Vous devez le modifier avec les **espace de travail** , **service** et **api_key** de votre expérimentation. En outre, vous devez modifier les **nom de compte de stockage** , **clé de compte de stockage** et **nom de conteneur de stockage**. Enfin, vous devez modifier l’emplacement du **fichier d’entrée** et l’emplacement du **fichier de sortie**.

```python
import urllib2
import json
import time
from azure.storage import *
workspace = "<REPLACE WITH YOUR WORKSPACE ID>"
service = "<REPLACE WITH YOUR SERVICE ID>"
api_key = "<REPLACE WITH THE API KEY FOR YOUR WEB SERVICE>"
storage_account_name = "<REPLACE WITH YOUR AZURE STORAGE ACCOUNT NAME>"
storage_account_key = "<REPLACE WITH YOUR AZURE STORAGE KEY>"
storage_container_name = "<REPLACE WITH YOUR AZURE STORAGE CONTAINER NAME>"
input_file = "<REPLACE WITH THE LOCATION OF YOUR INPUT FILE>" # Example: C:\\mydata.csv
output_file = "<REPLACE WITH THE LOCATION OF YOUR OUTPUT FILE>" # Example: C:\\myresults.csv
input_blob_name = "mydatablob.csv"
output_blob_name = "myresultsblob.csv"
def printHttpError(httpError):
print("The request failed with status code: " + str(httpError.code))
print(httpError.info())
print(json.loads(httpError.read()))
return
def saveBlobToFile(blobUrl, resultsLabel):
print("Reading the result from " + blobUrl)
try:
    response = urllib2.urlopen(blobUrl)
except urllib2.HTTPError, error:
    printHttpError(error)
    return
with open(output_file, "wb+") as f:
    f.write(response.read())
print(resultsLabel + " have been written to the file " + output_file)
return
def processResults(result):
first = True
results = result["Results"]
for outputName in results:
    result_blob_location = results[outputName]
    sas_token = result_blob_location["SasBlobToken"]
    base_url = result_blob_location["BaseLocation"]
    relative_url = result_blob_location["RelativeLocation"]
    print("The results for " + outputName + " are available at the following Azure Storage location:")
    print("BaseLocation: " + base_url)
    print("RelativeLocation: " + relative_url)
    print("SasBlobToken: " + sas_token)
    if (first):
        first = False
        url3 = base_url + relative_url + sas_token
        saveBlobToFile(url3, "The results for " + outputName)
return

def invokeBatchExecutionService():
url = "https://ussouthcentral.services.azureml.net/workspaces/" + workspace +"/services/" + service +"/jobs"
blob_service = BlobService(account_name=storage_account_name, account_key=storage_account_key)
print("Uploading the input to blob storage...")
data_to_upload = open(input_file, "r").read()
blob_service.put_blob(storage_container_name, input_blob_name, data_to_upload, x_ms_blob_type="BlockBlob")
print "Uploaded the input to blob storage"
input_blob_path = "/" + storage_container_name + "/" + input_blob_name
connection_string = "DefaultEndpointsProtocol=https;AccountName=" + storage_account_name + ";AccountKey=" + storage_account_key
payload =  {
    "Input": {
        "ConnectionString": connection_string,
        "RelativeLocation": input_blob_path
    },
    "Outputs": {
        "output1": { "ConnectionString": connection_string, "RelativeLocation": "/" + storage_container_name + "/" + output_blob_name },
    },
    "GlobalParameters": {
    }
}
    body = str.encode(json.dumps(payload))
headers = { "Content-Type":"application/json", "Authorization":("Bearer " + api_key)}
print("Submitting the job...")
# submit the job
req = urllib2.Request(url + "?api-version=2.0", body, headers)
try:
    response = urllib2.urlopen(req)
except urllib2.HTTPError, error:
    printHttpError(error)
    return
result = response.read()
job_id = result[1:-1] # remove the enclosing double-quotes
print("Job ID: " + job_id)
# start the job
print("Starting the job...")
req = urllib2.Request(url + "/" + job_id + "/start?api-version=2.0", "", headers)
try:
    response = urllib2.urlopen(req)
except urllib2.HTTPError, error:
    printHttpError(error)
    return
url2 = url + "/" + job_id + "?api-version=2.0"

while True:
    print("Checking the job status...")
    # If you are using Python 3+, replace urllib2 with urllib.request in the following code
    req = urllib2.Request(url2, headers = { "Authorization":("Bearer " + api_key) })
    try:
        response = urllib2.urlopen(req)
    except urllib2.HTTPError, error:
        printHttpError(error)
        return
    result = json.loads(response.read())
    status = result["StatusCode"]
    print "status:" + status
    if (status == 0 or status == "NotStarted"):
        print("Job " + job_id + " not yet started...")
    elif (status == 1 or status == "Running"):
        print("Job " + job_id + " running...")
    elif (status == 2 or status == "Failed"):
        print("Job " + job_id + " failed!")
        print("Error details: " + result["Details"])
        break
    elif (status == 3 or status == "Cancelled"):
        print("Job " + job_id + " cancelled!")
        break
    elif (status == 4 or status == "Finished"):
        print("Job " + job_id + " finished!")
        processResults(result)
        break
    time.sleep(1) # wait one second
return
invokeBatchExecutionService()
```
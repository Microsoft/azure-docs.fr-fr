---
author: PatrickFarley
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 11/14/2019
ms.author: pafarley
ms.openlocfilehash: 6a6b0d9740d19270f8daa3608bc125edd0fbec37
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005082"
---
## <a name="analyze-forms-for-key-value-pairs-and-tables"></a>Analyser des formulaires afin d’en extraire des tableaux et des paires clé-valeur

À présent, vous allez utiliser le modèle nouvellement entraîné pour analyser un document et en extraire des tableaux et des paires clé-valeur. Appelez l’API **[Analyze Form](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm)** (Analyser un formulaire) en exécutant le code suivant dans un nouveau script Python. Avant d’exécuter le script, apportez les modifications suivantes :

1. Remplacez `<file path>` par le chemin d’accès du fichier de votre formulaire (par exemple, C:\temp\file.pdf). Il peut également s’agir de l’URL d’un fichier distant. Dans le cadre de ce guide de démarrage rapide, vous pouvez utiliser les fichiers disponibles dans le dossier **Test** de l’[exemple de jeu de données](https://go.microsoft.com/fwlink/?linkid=2090451) (téléchargez et extrayez *sample_data.zip*).
1. Remplacez `<model_id>` par l’ID de modèle que vous avez reçu à la section précédente.
1. Remplacez `<endpoint>` par le point de terminaison que vous avez obtenu avec votre clé d’abonnement Form Recognizer. Vous la trouverez sous l’onglet **Vue d’ensemble** de la ressource Form Recognizer.
1. Remplacez `<file type>` par le type de fichier. Types pris en charge : `application/pdf`, `image/jpeg`, `image/png`, `image/tiff`.
1. Remplacez `<subscription key>` par votre clé d’abonnement.

    # <a name="v20"></a>[v2.0](#tab/v2-0)
    ```python
    ########### Python Form Recognizer Async Analyze #############
    import json
    import time
    from requests import get, post
    
    # Endpoint URL
    endpoint = r"<endpoint>"
    apim_key = "<subsription key>"
    model_id = "<model_id>"
    post_url = endpoint + "/formrecognizer/v2.0/custom/models/%s/analyze" % model_id
    source = r"<file path>"
    params = {
        "includeTextDetails": True
    }
    
    headers = {
        # Request headers
        'Content-Type': '<file type>',
        'Ocp-Apim-Subscription-Key': apim_key,
    }
    with open(source, "rb") as f:
        data_bytes = f.read()
    
    try:
        resp = post(url = post_url, data = data_bytes, headers = headers, params = params)
        if resp.status_code != 202:
            print("POST analyze failed:\n%s" % json.dumps(resp.json()))
            quit()
        print("POST analyze succeeded:\n%s" % resp.headers)
        get_url = resp.headers["operation-location"]
    except Exception as e:
        print("POST analyze failed:\n%s" % str(e))
        quit() 
    ```    
    # <a name="v21-preview"></a>[v2.1 (préversion)](#tab/v2-1)
    ```python
    ########### Python Form Recognizer Async Analyze #############
    import json
    import time
    from requests import get, post
    
    # Endpoint URL
    endpoint = r"<endpoint>"
    apim_key = "<subsription key>"
    model_id = "<model_id>"
    post_url = endpoint + "/formrecognizer/v2.1-preview.2/custom/models/%s/analyze" % model_id
    source = r"<file path>"
    params = {
        "includeTextDetails": True
    }
    
    headers = {
        # Request headers
        'Content-Type': '<file type>',
        'Ocp-Apim-Subscription-Key': apim_key,
    }
    with open(source, "rb") as f:
        data_bytes = f.read()
    
    try:
        resp = post(url = post_url, data = data_bytes, headers = headers, params = params)
        if resp.status_code != 202:
            print("POST analyze failed:\n%s" % json.dumps(resp.json()))
            quit()
        print("POST analyze succeeded:\n%s" % resp.headers)
        get_url = resp.headers["operation-location"]
    except Exception as e:
        print("POST analyze failed:\n%s" % str(e))
        quit() 
    ```    


    ---



1. Enregistrez le code dans un fichier avec une extension .py. Par exemple, *form-recognizer-analyze.py*.
1. Ouvrir une fenêtre d’invite de commandes.
1. À l’invite, utilisez la commande `python` pour exécuter l’exemple. Par exemple : `python form-recognizer-analyze.py`.

Quand vous appelez l’API **Analyze Form** (Analyser un formulaire), vous recevez une réponse `201 (Success)` avec un en-tête **Operation-Location**. La valeur de cet en-tête est un ID que vous allez utiliser pour suivre les résultats de l’opération d’analyse. Le script ci-dessus affiche la valeur de cet en-tête dans la console.

## <a name="get-the-analyze-results"></a>Obtenir les résultats de l’analyse

Ajoutez le code suivant au bas de votre script Python. Ce code utilise la valeur d’ID de l’appel précédent dans un nouvel appel d’API pour récupérer les résultats de l’analyse. L’opération **Analyze Form** (Analyser un formulaire) étant asynchrone, ce script appelle l’API à intervalles réguliers jusqu’à ce que les résultats soient disponibles. Nous recommandons un intervalle d’au moins une seconde.

```python 
n_tries = 15
n_try = 0
wait_sec = 5
max_wait_sec = 60
while n_try < n_tries:
    try:
        resp = get(url = get_url, headers = {"Ocp-Apim-Subscription-Key": apim_key})
        resp_json = resp.json()
        if resp.status_code != 200:
            print("GET analyze results failed:\n%s" % json.dumps(resp_json))
            quit()
        status = resp_json["status"]
        if status == "succeeded":
            print("Analysis succeeded:\n%s" % json.dumps(resp_json))
            quit()
        if status == "failed":
            print("Analysis failed:\n%s" % json.dumps(resp_json))
            quit()
        # Analysis still running. Wait and retry.
        time.sleep(wait_sec)
        n_try += 1
        wait_sec = min(2*wait_sec, max_wait_sec)     
    except Exception as e:
        msg = "GET analyze results failed:\n%s" % str(e)
        print(msg)
        quit()
print("Analyze operation did not complete within the allocated time.")
```

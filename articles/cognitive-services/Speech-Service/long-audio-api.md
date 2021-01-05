---
title: API Audio long (préversion) - Service Speech
titleSuffix: Azure Cognitive Services
description: Découvrez comment l’API Audio long est conçue pour la synthèse asynchrone de texte de long en parole.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/11/2020
ms.author: trbye
ms.openlocfilehash: 255cfe11f8601abc89a1d96f702f453c2af1ccbd
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533058"
---
# <a name="long-audio-api-preview"></a>API Audio long (préversion)

L’API Audio long est conçue pour la synthèse asynchrone de texte long en parole (par exemple, livres audio, articles de presse et documents). Cette API ne retourne pas d’audio synthétisé en temps réel. Au lieu de cela, vous êtes supposé interroger la ou les réponses et utiliser la ou les sorties à mesure que le service les rend disponibles. Contrairement à l’API de synthèse vocale utilisée par le Kit de développement logiciel (SDK) Speech, l’API Audio long peut créer de l’audio synthétisé de plus de 10 minutes, ce qui la rend idéale pour les éditeurs et les plateformes de contenu audio.

Autres avantages de l’API Audio long  :

* La synthèse vocale renvoyée par le service utilise les meilleures voix neuronales.
* Il n’est pas nécessaire de déployer de point de terminaison vocal, car celui-ci synthétise les voix dans un mode batch qui n’est pas en temps réel.

> [!NOTE]
> L’API Audio long prend désormais en charge les [voix neuronales publiques](./language-support.md#neural-voices) et les [voix neuronales personnalisées](./how-to-custom-voice.md#custom-neural-voices).

## <a name="workflow"></a>Workflow

En règle générale, lorsque vous utilisez l’API Audio long, vous envoyez un ou plusieurs fichiers texte à synthétiser, interrogez l’état, puis, si celui-ci indique une réussite, vous pouvez télécharger la sortie audio.

Le diagramme suivant donne une vue d’ensemble du workflow.

![Diagramme de workflow de l’API Audio long](media/long-audio-api/long-audio-api-workflow.png)

## <a name="prepare-content-for-synthesis"></a>Préparer du contenu pour la synthèse

Lors de la préparation de votre fichier texte, vérifiez les points suivants :

* Il s’agit d’un texte brut (.txt) ou d’un texte SSML (.txt)
* Le texte est encodé au format [UTF-8 avec marque d’ordre d’octet (BOM, Byte Order Mark)](https://www.w3.org/International/questions/qa-utf8-bom.en#bom)
* Il s’agit d’un fichier unique, non d’un fichier zip
* Contient plus de 400 caractères pour le texte brut ou 400 [caractères facturables](./text-to-speech.md#pricing-note) pour du texte SSML, et moins de 10 000 paragraphes
  * Pour du texte brut, chaque paragraphe est séparé en appuyant sur la touche **Entrée/Retour** ([exemple d’entrée de texte brut](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/en-US.txt))
  * Pour du texte SSML, chaque élément SSML est considéré comme un paragraphe. Les éléments SSML sont séparés par différents paragraphes - Consultez [Exemple d’entrée de texte SSML](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/SSMLTextInputSample.txt)
> [!NOTE]
> Pour le chinois (Continent), le chinois (Hong Kong, R.A.S.), le chinois (Taïwan), le japonais et le coréen, un mot sera compté comme deux caractères. 

## <a name="python-example"></a>Exemple Python

Cette section contient des exemples Python qui illustrent l’utilisation de base de l’API Audio long. Créez un projet Python dans votre IDE ou votre éditeur favori. Copiez cet extrait de code dans un fichier nommé `voice_synthesis_client.py`.

```python
import argparse
import json
import ntpath
import urllib3
import requests
import time
from json import dumps, loads, JSONEncoder, JSONDecoder
import pickle

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
```

Ces bibliothèques permettent d’analyser les arguments, de construire la requête HTTP et d’appeler l’API REST de synthèse vocale d’audio long.

### <a name="get-a-list-of-supported-voices"></a>Obtenir une liste des voix prises en charge

Ce code vous permet d’obtenir une liste complète de voix que vous pouvez utiliser pour une région/point de terminaison spécifique. Ajoutez le code à `voice_synthesis_client.py` :

```python
parser = argparse.ArgumentParser(description='Text-to-speech client tool to submit voice synthesis requests.')
parser.add_argument('--voices', action="store_true", default=False, help='print voice list')
parser.add_argument('-key', action="store", dest="key", required=True, help='the speech subscription key, like fg1f763i01d94768bda32u7a******** ')
parser.add_argument('-region', action="store", dest="region", required=True, help='the region information, could be centralindia, canadacentral or uksouth')
args = parser.parse_args()
baseAddress = 'https://%s.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0-beta1/' % args.region

def getVoices():
    response=requests.get(baseAddress+"voicesynthesis/voices", headers={"Ocp-Apim-Subscription-Key":args.key}, verify=False)
    voices = json.loads(response.text)
    return voices

if args.voices:
    voices = getVoices()
    print("There are %d voices available:" % len(voices))
    for voice in voices:
        print ("Name: %s, Description: %s, Id: %s, Locale: %s, Gender: %s, PublicVoice: %s, Created: %s" % (voice['name'], voice['description'], voice['id'], voice['locale'], voice['gender'], voice['isPublicVoice'], voice['created']))
```

Exécutez le script à l’aide de la commande `python voice_synthesis_client.py --voices -key <your_key> -region <region>` et remplacez les valeurs suivantes :

* Remplacez `<your_key>` par votre clé d’abonnement de service de reconnaissance vocale. Ces informations sont disponibles dans l’onglet **Vue d’ensemble** de votre ressource dans le [portail Azure](https://aka.ms/azureportal).
* Remplacez `<region>` par la région dans laquelle votre ressource Speech a été créée (par exemple `eastus` ou `westus`). Ces informations sont disponibles dans l’onglet **Vue d’ensemble** de votre ressource dans le [portail Azure](https://aka.ms/azureportal).

Vous devez obtenir une sortie similaire à celle-ci :

```console
There are xx voices available:

Name: Microsoft Server Speech Text to Speech Voice (en-US, xxx), Description: xxx , Id: xxx, Locale: en-US, Gender: Male, PublicVoice: xxx, Created: 2019-07-22T09:38:14Z
Name: Microsoft Server Speech Text to Speech Voice (zh-CN, xxx), Description: xxx , Id: xxx, Locale: zh-CN, Gender: Female, PublicVoice: xxx, Created: 2019-08-26T04:55:39Z
```

Si le paramètre **PublicVoice** est défini sur **True**, la voix est une voix neuronale publique. Dans le cas contraire, il s’agit d’une voix neuronale personnalisée.

### <a name="convert-text-to-speech"></a>Convertir du texte en parole

Préparez un fichier texte d’entrée, soit en texte brut, soit en texte SSML, puis ajoutez le code suivant à `voice_synthesis_client.py` :

> [!NOTE]
> « concatenateResult » est un paramètre facultatif. S’il n’est pas défini, les sorties audio sont générées par paragraphe. Vous pouvez également concaténer les données audio en une sortie en définissant le paramètre. Par défaut, la sortie audio est définie sur riff-16khz-16bit-mono-pcm. Pour plus d’informations sur les sorties audio prises en charge, consultez [Formats de sortie audio](#audio-output-formats).

```python
parser.add_argument('--submit', action="store_true", default=False, help='submit a synthesis request')
parser.add_argument('--concatenateResult', action="store_true", default=False, help='If concatenate result in a single wave file')
parser.add_argument('-file', action="store", dest="file", help='the input text script file path')
parser.add_argument('-voiceId', action="store", nargs='+', dest="voiceId", help='the id of the voice which used to synthesis')
parser.add_argument('-locale', action="store", dest="locale", help='the locale information like zh-CN/en-US')
parser.add_argument('-format', action="store", dest="format", default='riff-16khz-16bit-mono-pcm', help='the output audio format')

def submitSynthesis():
    modelList = args.voiceId
    data={'name': 'simple test', 'description': 'desc...', 'models': json.dumps(modelList), 'locale': args.locale, 'outputformat': args.format}
    if args.concatenateResult:
        properties={'ConcatenateResult': 'true'}
        data['properties'] = json.dumps(properties)
    if args.file is not None:
        scriptfilename=ntpath.basename(args.file)
        files = {'script': (scriptfilename, open(args.file, 'rb'), 'text/plain')}
    response = requests.post(baseAddress+"voicesynthesis", data, headers={"Ocp-Apim-Subscription-Key":args.key}, files=files, verify=False)
    if response.status_code == 202:
        location = response.headers['Location']
        id = location.split("/")[-1]
        print("Submit synthesis request successful")
        return id
    else:
        print("Submit synthesis request failed")
        print("response.status_code: %d" % response.status_code)
        print("response.text: %s" % response.text)
        return 0

def getSubmittedSynthesis(id):
    response=requests.get(baseAddress+"voicesynthesis/"+id, headers={"Ocp-Apim-Subscription-Key":args.key}, verify=False)
    synthesis = json.loads(response.text)
    return synthesis

if args.submit:
    id = submitSynthesis()
    if (id == 0):
        exit(1)

    while(1):
        print("\r\nChecking status")
        synthesis=getSubmittedSynthesis(id)
        if synthesis['status'] == "Succeeded":
            r = requests.get(synthesis['resultsUrl'])
            filename=id + ".zip"
            with open(filename, 'wb') as f:  
                f.write(r.content)
                print("Succeeded... Result file downloaded : " + filename)
            break
        elif synthesis['status'] == "Failed":
            print("Failed...")
            break
        elif synthesis['status'] == "Running":
            print("Running...")
        elif synthesis['status'] == "NotStarted":
            print("NotStarted...")
        time.sleep(10)
```

Exécutez le script à l’aide de la commande `python voice_synthesis_client.py --submit -key <your_key> -region <region> -file <input> -locale <locale> -voiceId <voice_guid>` et remplacez les valeurs suivantes :

* Remplacez `<your_key>` par votre clé d’abonnement de service de reconnaissance vocale. Ces informations sont disponibles dans l’onglet **Vue d’ensemble** de votre ressource dans le [portail Azure](https://aka.ms/azureportal).
* Remplacez `<region>` par la région dans laquelle votre ressource Speech a été créée (par exemple `eastus` ou `westus`). Ces informations sont disponibles dans l’onglet **Vue d’ensemble** de votre ressource dans le [portail Azure](https://aka.ms/azureportal).
* Remplacez `<input>` par le chemin au fichier texte que vous avez préparé pour la conversion de texte par synthèse vocale.
* Remplacez `<locale>` par les paramètres régionaux de sortie souhaités. Consultez [Prise en charge linguistique](language-support.md#neural-voices) pour plus d'informations.
* Remplacez `<voice_guid>` par la voix de sortie souhaitée. Utilisez l’une des voix retournées par votre appel précédent au point de terminaison `/voicesynthesis/voices`.

Vous devez obtenir une sortie similaire à celle-ci :

```console
Submit synthesis request successful

Checking status
NotStarted...

Checking status
Running...

Checking status
Running...

Checking status
Succeeded... Result file downloaded : xxxx.zip
```

Le résultat contient le texte d’entrée et les fichiers de sortie audio générés par le service. Vous pouvez télécharger ces fichiers dans un fichier zip.

> [!NOTE]
> Si vous avez plusieurs fichiers d’entrée, vous devrez envoyer plusieurs demandes. Il existe certaines limites à connaître. 
> * Le client est autorisé à envoyer jusqu’à **5** demandes par seconde au serveur pour chaque compte d’abonnement Azure. Si la limite est dépassée, le client reçoit un code d’erreur 429 (trop de demandes). Réduisez la quantité de demandes par seconde.
> * Le serveur est autorisé à exécuter et à mettre en file d’attente jusqu’à **120** demandes pour chaque compte d’abonnement Azure. Si la limite est dépassée, le serveur retourne un code d’erreur 429 (trop de demandes). Attendez et évitez d’envoyer une nouvelle demande avant que quelques demandes ne soient traitées.

### <a name="remove-previous-requests"></a>Supprimer les requêtes précédentes

Le service conserve jusqu’à **20 000** requêtes pour chaque compte d’abonnement Azure. Si votre nombre de demandes dépasse cette limite, supprimez des demandes précédentes avant d’en créer d’autres. Si vous ne supprimez pas des demandes existantes, vous recevez une notification d’erreur.

Ajoutez le code suivant à `voice_synthesis_client.py` :

```python
parser.add_argument('--syntheses', action="store_true", default=False, help='print synthesis list')
parser.add_argument('--delete', action="store_true", default=False, help='delete a synthesis request')
parser.add_argument('-synthesisId', action="store", nargs='+', dest="synthesisId", help='the id of the voice synthesis which need to be deleted')

def getSubmittedSyntheses():
    response=requests.get(baseAddress+"voicesynthesis", headers={"Ocp-Apim-Subscription-Key":args.key}, verify=False)
    syntheses = json.loads(response.text)
    return syntheses

def deleteSynthesis(ids):
    for id in ids:
        print("delete voice synthesis %s " % id)
        response = requests.delete(baseAddress+"voicesynthesis/"+id, headers={"Ocp-Apim-Subscription-Key":args.key}, verify=False)
        if (response.status_code == 204):
            print("delete successful")
        else:
            print("delete failed, response.status_code: %d, response.text: %s " % (response.status_code, response.text))

if args.syntheses:
    synthese = getSubmittedSyntheses()
    print("There are %d synthesis requests submitted:" % len(synthese))
    for synthesis in synthese:
        print ("ID : %s , Name : %s, Status : %s " % (synthesis['id'], synthesis['name'], synthesis['status']))

if args.delete:
    deleteSynthesis(args.synthesisId)
```

Exécutez `python voice_synthesis_client.py --syntheses -key <your_key> -region <region>` pour obtenir une liste des requêtes de synthèse que vous avez effectuées. Vous devriez obtenir un résultat similaire à ceci :

```console
There are <number> synthesis requests submitted:
ID : xxx , Name : xxx, Status : Succeeded
ID : xxx , Name : xxx, Status : Running
ID : xxx , Name : xxx : Succeeded
```

Pour supprimer une requête, exécutez `python voice_synthesis_client.py --delete -key <your_key> -region <Region> -synthesisId <synthesis_id>` et remplacez `<synthesis_id>` par une valeur d’ID de requête renvoyée par la requête précédente.

> [!NOTE]
> Les requêtes dont l’état est « En cours »/« En attente » ne peuvent pas être supprimées.

La `voice_synthesis_client.py` complète est disponible sur [GitHub](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Python/voiceclient.py).

## <a name="http-status-codes"></a>Codes d’état HTTP

Le tableau suivant détaille les codes de réponse HTTP et les messages de l’API REST.

| API | Code d'état HTTP | Description | Solution |
|-----|------------------|-------------|----------|
| Créer | 400 | La synthèse vocale n’est pas activée dans cette région. | Modifiez la clé d’abonnement au service Speech de façon à utiliser une région prise en charge. |
|        | 400 | Seuls sont valides les abonnements de niveau **Standard** au service Speech pour cette région. | Modifiez la clé d’abonnement au service Speech de façon à utiliser le niveau de tarification « Standard ». |
|        | 400 | Dépassement de la limite de 20 000 demandes pour le compte Azure. Supprimez des demandes pour pouvoir en envoyer de nouvelles. | Le serveur conserve un maximum de 20 000 demandes pour chaque compte Azure. Supprimez des demandes pour pouvoir en envoyer de nouvelles. |
|        | 400 | Ce modèle ne peut pas être utilisé dans la synthèse vocale : {modelID}. | Vérifiez que l’état de {modelID} est correct. |
|        | 400 | La région de la demande ne correspond pas à celle du modèle : {modelID}. | Vérifiez que la région de {modelID} correspond à celle de la demande. |
|        | 400 | La synthèse vocale ne prend en charge que le fichier texte au format d’encodage UTF-8 avec le marqueur d’ordre d’octet. | Vérifiez que les fichiers d’entrée sont au format d’encodage UTF-8 avec le marqueur d’ordre d’octet. |
|        | 400 | Seules les entrées SSML valides sont autorisées dans la demande de synthèse vocale. | Vérifiez que les expressions SSML d’entrée sont correctes. |
|        | 400 | Le nom de voix {voiceName} est introuvable dans le fichier d’entrée. | Le nom de voix SSML d’entrée n’est pas aligné avec l’ID de modèle. |
|        | 400 | Le nombre de paragraphes dans le fichier d’entrée ne doit pas excéder 10 000. | Vérifiez que le nombre de paragraphes dans le fichier est inférieur à 10 000. |
|        | 400 | Le fichier d’entrée doit comprendre plus de 400 caractères. | Vérifiez que votre fichier d’entrée comporte plus de 400 caractères. |
|        | 404 | Le modèle déclaré dans la définition de synthèse vocale est introuvable : {modelID}. | Vérifiez que le {modelID} est correct. |
|        | 429 | Dépassement de la limite de synthèse vocale active. Veuillez attendre que des demandes soient traitées. | Le serveur est autorisé à exécuter et à mettre en file d’attente un maximum de 120 demandes pour chaque compte Azure. Attendez et évitez d’envoyer de nouvelles demandes avant que quelques demandes ne soient traitées. |
| Tous       | 429 | Il y a trop de demandes. | Le client est autorisé à envoyer un maximum de 5 demandes par seconde au serveur pour chaque compte Azure. Réduisez la quantité de demandes par seconde. |
| DELETE    | 400 | La tâche de synthèse vocale est toujours en cours d’utilisation. | Vous ne pouvez supprimer que les demandes ayant l’état **Terminé** ou **Échec**. |
| GetById   | 404 | L’entité spécifiée est introuvable. | Vérifiez que l’ID de synthèse est correct. |

## <a name="regions-and-endpoints"></a>Régions et points de terminaison

L’API Audio long est disponible dans plusieurs régions avec des points de terminaison uniques.

| Région | Point de terminaison |
|--------|----------|
| USA Est | `https://eastus.customvoice.api.speech.microsoft.com` |
| Inde Centre | `https://centralindia.customvoice.api.speech.microsoft.com` |
| Asie Sud-Est | `https://southeastasia.customvoice.api.speech.microsoft.com` |
| Sud du Royaume-Uni | `https://uksouth.customvoice.api.speech.microsoft.com` |
| Europe Ouest | `https://westeurope.customvoice.api.speech.microsoft.com` |

## <a name="audio-output-formats"></a>Formats de sortie aduio

Nous prenons en charge différents formats de sortie audio. Vous pouvez générer des sorties audio par paragraphe ou concaténer les sorties audio en une seule sortie en définissant le paramètre « concatenateResult ». Les formats de sortie audio suivants sont pris en charge par l’API Audio long :

> [!NOTE]
> Le format audio par défaut est riff-16 khz-16 bits-mono-pcm.

* riff-8 khz-16 bits-mono-pcm
* riff-16 khz-16 bits-mono-pcm
* riff-24 khz-16 bits-mono-pcm
* riff-48 khz-16 bits-mono-pcm
* audio-16 khz-32 kbitrate-mono-mp3
* audio-16 khz-64 kbitrate-mono-mp3
* audio-16 khz-128 kbitrate-mono-mp3
* audio-24 khz-48 kbitrate-mono-mp3
* audio-24 khz-96 kbitrate-mono-mp3
* audio-24 khz-160 kbitrate-mono-mp3

## <a name="sample-code"></a>Exemple de code
Un exemple de code pour l’API Audio long est disponible sur GitHub.

* [Exemple de code : Python](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/Python)
* [Exemple de code : C#](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/CSharp)
* [Exemple de code : Java](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/)
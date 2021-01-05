---
title: 'Tutoriel : Créer une application Flask pour traduire, synthétiser et analyser du texte - Translator'
titleSuffix: Azure Cognitive Services
description: Dans ce tutoriel, vous allez créer une application web Flask pour traduire du texte, analyser les sentiments et synthétiser du texte traduit en voix.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: tutorial
ms.date: 05/26/2020
ms.author: swmachan
ms.custom: devx-track-python, devx-track-js
ms.openlocfilehash: 1cbe4d44f5e1c8b34a3d7bb9d05b9546f320b81c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95023465"
---
# <a name="tutorial-build-a-flask-app-with-azure-cognitive-services"></a>Tutoriel : Créer une application Flask avec Azure Cognitive Services

Dans ce tutoriel, vous allez créer une application web Flask qui utilise Azure Cognitive Services pour traduire du texte, analyser les sentiments et synthétiser du texte traduit en voix. Nous nous concentrons sur le code Python et les routes Flask qui servent de base à notre application ; nous vous aiderons cependant pour le code HTML et Javascript qui lui permet de fonctionner. Si vous rencontrez des problèmes, faites-le nous savoir via le bouton de commentaires ci-dessous.

Voici ce qui est couvert par ce tutoriel :

> [!div class="checklist"]
> * Obtenir des clés d’abonnement Azure
> * Configurer votre environnement de développement et installer les dépendances
> * Créer une application Flask
> * Utiliser Translator pour traduire du texte
> * Utiliser Analyse de texte pour analyser les sentiments positifs/négatifs de textes et de traductions en entrée
> * Utiliser les services Speech pour convertir du texte traduit en synthèse vocale
> * Exécuter votre application Flask localement

> [!TIP]
> Si vous voulez avancer et voir tout le code à la fois, l’exemple complet et les instructions de génération sont disponibles sur [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Flask-App-Tutorial).

## <a name="what-is-flask"></a>Qu’est-ce que Flask ?

Flask est un microframework pour la création d’applications web. Cela signifie que Flask vous fournit des outils, des bibliothèques et des technologies qui vous permettent de créer une application web. Cette application web peut être constituée de quelques pages web, être un blog ou un wiki, une application de calendrier web ou encore un site web marchand.

Pour ceux d’entre vous qui souhaitent plus d’informations après ce tutoriel, voici quelques liens utiles :

* [Documentation de Flask](http://flask.pocoo.org/)
* [Flask for Dummies - A Beginner’s Guide to Flask](https://codeburst.io/flask-for-dummies-a-beginners-guide-to-flask-part-uno-53aec6afc5b1)

## <a name="prerequisites"></a>Prérequis

Passons en revue les logiciels et les clés d’abonnement dont vous avez besoin pour ce tutoriel.

* [Python version 3.5.2 ou ultérieure](https://www.python.org/downloads/)
* [Outils Git](https://git-scm.com/downloads)
* Un IDE ou un éditeur de texte, comme [Visual Studio Code](https://code.visualstudio.com/) ou [Atom](https://atom.io/)  
* [Chrome](https://www.google.com/chrome/browser/) ou [Firefox](https://www.mozilla.org/firefox)
* Une clé d’abonnement **Translator** (notez que vous n’êtes pas obligé de sélectionner une région.)
* Une clé d’abonnement **Analyse de texte** dans la région **USA Ouest**.
* Une clé d’abonnement **Services Speech** dans la région **USA Ouest**.

## <a name="create-an-account-and-subscribe-to-resources"></a>Créer un compte et s’abonner à des ressources

Comme mentionné précédemment, vous aurez besoin de trois clés d’abonnement pour ce tutoriel. Cela signifie que vous devez créer une ressource dans votre compte Azure pour :
* Convertisseur
* Analyse de texte
* Services Speech

Utilisez [Créer un compte Cognitive Services dans le portail Azure](../cognitive-services-apis-create-account.md) pour obtenir des instructions pas sur la création de ressources.

> [!IMPORTANT]
> Pour ce tutoriel, créez vos ressources dans la région USA Ouest. Si vous utilisez une autre région, vous devrez ajuster l’URL de base dans chacun de vos fichiers Python.

## <a name="set-up-your-dev-environment"></a>Configurer votre environnement de développement

Avant de créer votre application web Flask, vous devez créer un répertoire de travail pour votre projet et installer quelques packages Python.

### <a name="create-a-working-directory"></a>Créer un répertoire de travail

1. Ouvrez la ligne de commande (Windows) ou le terminal (macOS/Linux). Ensuite, créez un répertoire de travail et des sous-répertoires pour votre projet :  

   ```
   mkdir -p flask-cog-services/static/scripts && mkdir flask-cog-services/templates
   ```
2. Accédez au répertoire de travail de votre projet :  

   ```
   cd flask-cog-services
   ```

### <a name="create-and-activate-your-virtual-environment-with-virtualenv"></a>Créer et activer votre environnement virtuel avec `virtualenv`

Créons un environnement virtuel pour notre application Flask avec `virtualenv`. L’utilisation d’un environnement virtuel garantit un environnement propre à partir duquel vous pouvez travailler.

1. Dans votre répertoire de travail, exécutez cette commande pour créer un environnement virtuel : **macOS/Linux :**
   ```
   virtualenv venv --python=python3
   ```
   Nous avons déclaré explicitement que l’environnement virtuel devait utiliser Python 3. Ceci garantit que les utilisateurs ayant plusieurs installations de Python utilisent la version correcte.

   **Windows CMD / Windows Bash :**
   ```
   virtualenv venv
   ```
   Pour simplifier les choses, nous nommons « venv » votre environnement virtuel.

2. Les commandes pour activer votre environnement virtuel dépendent de votre plateforme/shell :   

   | Plateforme | Shell | Commande |
   |----------|-------|---------|
   | macOS/Linux | bash/zsh | `source venv/bin/activate` |
   | Windows | bash | `source venv/Scripts/activate` |
   | | Ligne de commande | `venv\Scripts\activate.bat` |
   | | PowerShell | `venv\Scripts\Activate.ps1` |

   Après avoir exécuté cette commande, votre ligne de commande ou votre session du terminal doit être précédée de `venv`.

3. Vous pouvez désactiver la session à tout moment en tapant ceci dans la ligne de commande ou le terminal : `deactivate`.

> [!NOTE]
> Python a une documentation complète sur la création et la gestion des environnements virtuels : consultez [virtualenv](https://virtualenv.pypa.io/en/latest/).

### <a name="install-requests"></a>Installer Requests

Requests est un module répandu qui est utilisé pour envoyer des requêtes HTTP 1.1. Vous n’avez pas besoin d’ajouter manuellement des chaînes de requête à vos URL, ni d’encoder vos données POST dans un formulaire.

1. Pour installer Requests, exécutez :

   ```
   pip install requests
   ```

> [!NOTE]
> Si vous voulez en savoir plus sur Requests, consultez [: HTTP for Humans](https://2.python-requests.org/en/master/).

### <a name="install-and-configure-flask"></a>Installer et configurer Flask

Nous devons ensuite installer Flask. Flask gère le routage pour notre application web et nous permet d’effectuer des appels de serveur à serveur qui masquent nos clés d’abonnement à l’utilisateur final.

1. Pour installer Flask, exécutez :
   ```
   pip install Flask
   ```
   Vérifions que Flask a été installé. Exécutez :
   ```
   flask --version
   ```
   La version doit s’afficher dans le terminal. L’affichage d’autres informations signifie que quelque chose s’est mal passé.

2. Pour exécuter l’application Flask, vous pouvez utiliser la commande flask ou le commutateur -m de Python avec Flask. Avant de pouvoir faire cela, vous devez indiquer à votre terminal avec quelle application travailler en exportant la variable d’environnement `FLASK_APP` :

   **macOS/Linux** :
   ```
   export FLASK_APP=app.py
   ```

   **Windows** :
   ```
   set FLASK_APP=app.py
   ```

## <a name="create-your-flask-app"></a>Créer votre application Flask

Dans cette section, vous allez créer une application Flask pour poste de travail qui retourne un fichier HTML quand les utilisateurs atteignent la racine de votre application. Ne passez pas trop de temps à essayer de décrypter le code : nous reviendrons mettre à jour ce fichier plus tard.

### <a name="what-is-a-flask-route"></a>Qu’est-ce qu’une route Flask ?

Prenons une minute pour parler des « [routes](http://flask.pocoo.org/docs/1.0/api/#flask.Flask.route) ». Le routage est utilisé pour lier une URL à une fonction spécifique. Flask utilise des décorateurs de route pour inscrire des fonctions auprès d’URL spécifiques. Par exemple, quand un utilisateur accède à la racine (`/`) de notre application web, `index.html` est affiché.  

```python
@app.route('/')
def index():
    return render_template('index.html')
```

Considérons un autre exemple pour bien comprendre.

```python
@app.route('/about')
def about():
    return render_template('about.html')
```

Ce code garantit que quand un utilisateur accède à `http://your-web-app.com/about`, le fichier `about.html` est affiché.

Si ces exemples montrent comment effectuer le rendu de pages HTML pour un utilisateur, les routes peuvent également être utilisées pour appeler des API quand un bouton est enfoncé ou pour lancer différentes actions sans devoir à quitter la page d’accueil. Vous verrez ceci en action lors de la création de routes pour la traduction, l’analyse des sentiments et la synthèse vocale.

### <a name="get-started"></a>Bien démarrer

1. Ouvrez le projet dans votre IDE, puis créez un fichier nommé `app.py` à la racine de votre répertoire de travail. Ensuite, copiez ce code dans `app.py` et enregistrez-le :

   ```python
   from flask import Flask, render_template, url_for, jsonify, request

   app = Flask(__name__)
   app.config['JSON_AS_ASCII'] = False

   @app.route('/')
   def index():
       return render_template('index.html')
   ```

   Ce bloc de code indique à l’application d’afficher `index.html` chaque fois qu’un utilisateur accède à la racine de votre application web (`/`).

2. Ensuite, créons le front-end pour notre application web. Créez un fichier nommé `index.html` dans le répertoire `templates`. Copiez ensuite le code suivant dans `templates/index.html`.

   ```html
   <!doctype html>
   <html lang="en">
     <head>
       <!-- Required metadata tags -->
       <meta charset="utf-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
       <meta name="description" content="Translate and analyze text with Azure Cognitive Services.">
       <!-- Bootstrap CSS -->
       <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
       <title>Translate and analyze text with Azure Cognitive Services</title>
     </head>
     <body>
       <div class="container">
         <h1>Translate, synthesize, and analyze text with Azure</h1>
         <p>This simple web app uses Azure for text translation, text-to-speech conversion, and sentiment analysis of input text and translations. Learn more about <a href="https://docs.microsoft.com/azure/cognitive-services/">Azure Cognitive Services</a>.
        </p>
        <!-- HTML provided in the following sections goes here. -->

        <!-- End -->
       </div>

       <!-- Required Javascript for this tutorial -->
       <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
       <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
       <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
       <script type = "text/javascript" src ="static/scripts/main.js"></script>
     </body>
   </html>
   ```

3. Testons l’application Flask. À partir du terminal, exécutez :

   ```
   flask run
   ```

4. Ouvrez un navigateur et accédez à l’URL fournie. Vous voyez normalement votre application monopage. Appuyez sur **Ctrl+C** pour mettre fin à l’application.

## <a name="translate-text"></a>Traduire le texte

Maintenant que vous avez une idée de la façon dont une application Flask simple fonctionne, nous allons :

* Écrire du code Python pour appeler Translator et retourner une réponse
* Créer une route Flask pour appeler votre code Python
* Mettre à jour le code HTML avec une zone de saisie pour le texte et la traduction, un sélecteur de langue et un bouton Translate (Traduire)
* Écrire du code JavaScript qui permet aux utilisateurs d’interagir avec votre application Flask à partir du HTML

### <a name="call-the-translator"></a>Appeler Translator

La première chose à faire est d’écrire une fonction pour appeler Translator. Cette fonction prend deux arguments : `text_input` et `language_output`. Cette fonction est appelée chaque fois qu’un utilisateur appuie sur le bouton Translate dans votre application. La zone de texte dans le HTML est envoyée en tant que `text_input`, et la valeur de la sélection de la langue dans le HTML est envoyée en tant que `language_output`.

1. Commençons par créer un fichier appelé `translate.py` à la racine de votre répertoire de travail.
2. Ajoutez ensuite ce code à `translate.py`. Cette fonction prend deux arguments : `text_input` et `language_output`.
   ```python
   import os, requests, uuid, json

   # Don't forget to replace with your Cog Services subscription key!
   # If you prefer to use environment variables, see Extra Credit for more info.
   subscription_key = 'YOUR_TRANSLATOR_TEXT_SUBSCRIPTION_KEY'
   
   # Don't forget to replace with your Cog Services location!
   # Our Flask route will supply two arguments: text_input and language_output.
   # When the translate text button is pressed in our Flask app, the Ajax request
   # will grab these values from our web app, and use them in the request.
   # See main.js for Ajax calls.
   def get_translation(text_input, language_output):
       base_url = 'https://api.cognitive.microsofttranslator.com'
       path = '/translate?api-version=3.0'
       params = '&to=' + language_output
       constructed_url = base_url + path + params

       headers = {
           'Ocp-Apim-Subscription-Key': subscription_key,
           'Ocp-Apim-Subscription-Region': 'location',
           'Content-type': 'application/json',
           'X-ClientTraceId': str(uuid.uuid4())
       }

       # You can pass more than one object in body.
       body = [{
           'text' : text_input
       }]
       response = requests.post(constructed_url, headers=headers, json=body)
       return response.json()
   ```
3. Ajoutez votre clé d’abonnement Translator et enregistrez.

### <a name="add-a-route-to-apppy"></a>Ajouter une route à `app.py`

Ensuite, vous devez créer une route dans votre application Flask qui appelle `translate.py`. Cette route sera appelée chaque fois qu’un utilisateur appuie sur le bouton Translate dans votre application.

Pour cette application, votre route va accepter des requêtes `POST`. La raison en est que la fonction attend le texte à traduire et une langue en sortie pour la traduction.

Flask fournit des fonctions helper pour vous aider à analyser et à gérer chaque requête. Dans le code fourni, `get_json()` retourne les données de la requête `POST` au format JSON. Ensuite, en utilisant `data['text']` et `data['to']`, les valeurs pour le texte et la langue en sortie sont passées à la fonction `get_translation()` disponible depuis `translate.py`. La dernière étape consiste à retourner la réponse au format JSON, car vous devez afficher ces données dans votre application web.

Dans les sections suivantes, vous allez répéter ce processus lors de la création de routes pour l’analyse des sentiments et pour la synthèse vocale.

1. Ouvrez `app.py`, recherchez l’instruction import en haut de `app.py` et ajoutez la ligne suivante :

   ```python
   import translate
   ```
   Notre application Flask peut maintenant utiliser la méthode disponible via `translate.py`.

2. Copiez ce code à la fin de `app.py` et enregistrez-le :

   ```python
   @app.route('/translate-text', methods=['POST'])
   def translate_text():
       data = request.get_json()
       text_input = data['text']
       translation_output = data['to']
       response = translate.get_translation(text_input, translation_output)
       return jsonify(response)
   ```

### <a name="update-indexhtml"></a>Mettre à jour `index.html`

Maintenant que vous avez une fonction pour traduire du texte et une route dans votre application Flask pour l’appeler, l’étape suivante consiste à créer le code HTML pour votre application. Le HTML ci-dessous fait un certain nombre de choses :

* Il fournit une zone de texte où les utilisateurs peuvent entrer du texte à traduire.
* Il inclut un sélecteur de langue.
* Il inclut des éléments HTML pour afficher la langue détectée et les scores de confiance retournés lors de la traduction.
* Il fournit une zone de texte en lecture seule où le résultat de la traduction est affiché.
* Il inclut des espaces réservés pour le code de l’analyse des sentiments et de la synthèse vocale, que vous ajouterez à ce fichier plus loin dans le tutoriel.

Mettons à jour `index.html`.

1. Ouvrez `index.html` et recherchez ces commentaires dans le code :
   ```html
   <!-- HTML provided in the following sections goes here. -->

   <!-- End -->
   ```

2. Remplacez les commentaires du code par ce bloc HTML :
   ```html
   <div class="row">
     <div class="col">
       <form>
         <!-- Enter text to translate. -->
         <div class="form-group">
           <label for="text-to-translate"><strong>Enter the text you'd like to translate:</strong></label>
           <textarea class="form-control" id="text-to-translate" rows="5"></textarea>
         </div>
         <!-- Select output language. -->
         <div class="form-group">
           <label for="select-language"><strong>Translate to:</strong></label>
           <select class="form-control" id="select-language">
             <option value="ar">Arabic</option>
             <option value="ca">Catalan</option>
             <option value="zh-Hans">Chinese (Simplified)</option>
             <option value="zh-Hant">Chinese (Traditional)</option>
             <option value="hr">Croatian</option>
             <option value="en">English</option>
             <option value="fr">French</option>
             <option value="de">German</option>
             <option value="el">Greek</option>
             <option value="he">Hebrew</option>
             <option value="hi">Hindi</option>
             <option value="it">Italian</option>
             <option value="ja">Japanese</option>
             <option value="ko">Korean</option>
             <option value="pt">Portuguese</option>
             <option value="ru">Russian</option>
             <option value="es">Spanish</option>
             <option value="th">Thai</option>
             <option value="tr">Turkish</option>
             <option value="vi">Vietnamese</option>
           </select>
         </div>
         <button type="submit" class="btn btn-primary mb-2" id="translate">Translate text</button></br>
         <div id="detected-language" style="display: none">
           <strong>Detected language:</strong> <span id="detected-language-result"></span><br />
           <strong>Detection confidence:</strong> <span id="confidence"></span><br /><br />
         </div>

         <!-- Start sentiment code-->

         <!-- End sentiment code -->

       </form>
     </div>
     <div class="col">
       <!-- Translated text returned by the Translate API is rendered here. -->
       <form>
         <div class="form-group" id="translator-text-response">
           <label for="translation-result"><strong>Translated text:</strong></label>
           <textarea readonly class="form-control" id="translation-result" rows="5"></textarea>
         </div>

         <!-- Start voice font selection code -->

         <!-- End voice font selection code -->

       </form>

       <!-- Add Speech Synthesis button and audio element -->

       <!-- End Speech Synthesis button -->

     </div>
   </div>
   ```

L’étape suivante consiste à écrire du code Javascript. Il s’agit du pont entre votre code HTML et la route Flask.

### <a name="create-mainjs"></a>Créez `main.js`.  

Le fichier `main.js` est le pont entre votre code HTML et la route Flask. Votre application utilisera une combinaison de jQuery, Ajax et XMLHttpRequest pour afficher le contenu, et pour faire des requêtes `POST` à vos routes Flask.

Dans le code ci-dessous, le contenu du HTML est utilisé pour construire une requête adressée à votre route Flask. Plus spécifiquement, le contenu de la zone de texte et du sélecteur de langue est affecté à des variables, puis passé dans la requête à `translate-text`.

Le code effectue une itération dans la réponse, puis met à jour le HTML avec la traduction, la langue détectée et le score de confiance.

1. Dans votre IDE, créez un fichier nommé `main.js` dans le répertoire `static/scripts`.
2. Copiez ce code dans `static/scripts/main.js` :
   ```javascript
   //Initiate jQuery on load.
   $(function() {
     //Translate text with flask route
     $("#translate").on("click", function(e) {
       e.preventDefault();
       var translateVal = document.getElementById("text-to-translate").value;
       var languageVal = document.getElementById("select-language").value;
       var translateRequest = { 'text': translateVal, 'to': languageVal }

       if (translateVal !== "") {
         $.ajax({
           url: '/translate-text',
           method: 'POST',
           headers: {
               'Content-Type':'application/json'
           },
           dataType: 'json',
           data: JSON.stringify(translateRequest),
           success: function(data) {
             for (var i = 0; i < data.length; i++) {
               document.getElementById("translation-result").textContent = data[i].translations[0].text;
               document.getElementById("detected-language-result").textContent = data[i].detectedLanguage.language;
               if (document.getElementById("detected-language-result").textContent !== ""){
                 document.getElementById("detected-language").style.display = "block";
               }
               document.getElementById("confidence").textContent = data[i].detectedLanguage.score;
             }
           }
         });
       };
     });
     // In the following sections, you'll add code for sentiment analysis and
     // speech synthesis here.
   })
   ```

### <a name="test-translation"></a>Tester la traduction

Testons la traduction dans l’application.

```
flask run
```

Accédez à l’adresse du serveur fournie. Tapez du texte dans la zone d’entrée, sélectionnez une langue, puis appuyez sur Translate. Vous devez normalement obtenir une traduction. Si cela ne fonctionne pas, vérifiez que vous avez ajouté votre clé d’abonnement.

> [!TIP]
> Si les modifications que vous avez apportées n’apparaissent pas ou si l’application ne fonctionne pas comme prévu, essayez en vidant votre cache ou en ouvrant une fenêtre privée/incognito.

Appuyez sur **Ctrl+C** pour arrêter l’application, puis passez à la section suivante.

## <a name="analyze-sentiment"></a>Analyser les sentiments

L’[API Analyse de texte](../text-analytics/overview.md) peut être utilisée pour effectuer l’analyse des sentiments, extraire des expressions clés du texte ou détecter la langue source. Dans cette application, nous allons utiliser l’analyse des sentiments pour déterminer si le texte fourni est positif, neutre ou négatif. L’API retourne un score numérique compris entre 0 et 1. Un score proche de 1 indique un sentiment positif, et un score proche de 0 un sentiment négatif.

Dans cette section, vous allez effectuer quelques opérations :

* Écrire du code Python pour appeler l’API Analyse de texte pour effectuer une analyse des sentiments et retourner une réponse
* Créer une route Flask pour appeler votre code Python
* Mettre à jour le code HTML avec une zone pour les scores des sentiments et un bouton pour effectuer l’analyse
* Écrire du code JavaScript qui permet aux utilisateurs d’interagir avec votre application Flask à partir du HTML

### <a name="call-the-text-analytics-api"></a>Appeler l’API Text Analytics

Écrivons une fonction pour appeler l’API Analyse de texte. Cette fonction prend quatre arguments : `input_text`,`input_language`,`output_text` et `output_language`. Cette fonction est appelée chaque fois qu’un utilisateur appuie sur le bouton Run sentiment analysis (Exécuter l’analyse des sentiments) dans votre application. Les données fournies par l’utilisateur dans la zone de texte et le sélecteur de langue, ainsi que la langue détectée et le résultat de la traduction, sont fournis avec chaque requête. L’objet en réponse comprend des scores de sentiment pour la source et pour la traduction. Dans les sections suivantes, vous allez écrire du code Javascript pour analyser la réponse et l’utiliser dans votre application. Pour l’instant, concentrons-nous sur l’appel de l’API Analyse de texte.

1. Créons un fichier appelé `sentiment.py` à la racine de votre répertoire de travail.
2. Ajoutez ensuite ce code à `sentiment.py`.
   ```python
   import os, requests, uuid, json

   # Don't forget to replace with your Cog Services subscription key!
   subscription_key = 'YOUR_TEXT_ANALYTICS_SUBSCRIPTION_KEY'

   # Our Flask route will supply four arguments: input_text, input_language,
   # output_text, output_language.
   # When the run sentiment analysis button is pressed in our Flask app,
   # the Ajax request will grab these values from our web app, and use them
   # in the request. See main.js for Ajax calls.

   def get_sentiment(input_text, input_language, output_text, output_language):
       base_url = 'https://westus.api.cognitive.microsoft.com/text/analytics'
       path = '/v2.0/sentiment'
       constructed_url = base_url + path

       headers = {
           'Ocp-Apim-Subscription-Key': subscription_key,
           'Content-type': 'application/json',
           'X-ClientTraceId': str(uuid.uuid4())
       }

       # You can pass more than one object in body.
       body = {
           'documents': [
               {
                   'language': input_language,
                   'id': '1',
                   'text': input_text
               },
               {
                   'language': output_language,
                   'id': '2',
                   'text': output_text
               }
           ]
       }
       response = requests.post(constructed_url, headers=headers, json=body)
       return response.json()
   ```
3. Ajoutez votre clé d’abonnement Analyse de texte et enregistrez.

### <a name="add-a-route-to-apppy"></a>Ajouter une route à `app.py`

Créons ensuite une route dans votre application Flask qui appelle `sentiment.py`. Cette route sera appelée chaque fois qu’un utilisateur appuie sur le bouton Run sentiment analysis dans votre application. Tout comme la route pour la traduction, cette route va accepter des requêtes `POST`, car la fonction attend les arguments.

1. Ouvrez `app.py`, recherchez l’instruction import en haut de `app.py` et mettez-la à jour :

   ```python
   import translate, sentiment
   ```
   Notre application Flask peut maintenant utiliser la méthode disponible via `sentiment.py`.

2. Copiez ce code à la fin de `app.py` et enregistrez-le :
   ```python
   @app.route('/sentiment-analysis', methods=['POST'])
   def sentiment_analysis():
       data = request.get_json()
       input_text = data['inputText']
       input_lang = data['inputLanguage']
       output_text = data['outputText']
       output_lang =  data['outputLanguage']
       response = sentiment.get_sentiment(input_text, input_lang, output_text, output_lang)
       return jsonify(response)
   ```

### <a name="update-indexhtml"></a>Mettre à jour `index.html`

Maintenant que vous avez une fonction pour effectuer une analyse des sentiments et une route dans votre application Flask pour l’appeler, l’étape suivante consiste à écrire le code HTML pour votre application. Le HTML ci-dessous fait un certain nombre de choses :

* Ajoute un bouton à votre application pour effectuer l’analyse des sentiments
* Ajoute un élément qui explique les scores de sentiment
* Ajoute un élément pour afficher les scores de sentiment

1. Ouvrez `index.html` et recherchez ces commentaires dans le code :
   ```html
   <!-- Start sentiment code-->

   <!-- End sentiment code -->
   ```

2. Remplacez les commentaires du code par ce bloc HTML :
   ```html
   <button type="submit" class="btn btn-primary mb-2" id="sentiment-analysis">Run sentiment analysis</button></br>
   <div id="sentiment" style="display: none">
      <p>Sentiment scores are provided on a 1 point scale. The closer the sentiment score is to 1, indicates positive sentiment. The closer it is to 0, indicates negative sentiment.</p>
      <strong>Sentiment score for input:</strong> <span id="input-sentiment"></span><br />
      <strong>Sentiment score for translation:</strong> <span id="translation-sentiment"></span>
   </div>
   ```

### <a name="update-mainjs"></a>Mettre à jour `main.js`

Dans le code ci-dessous, le contenu du HTML est utilisé pour construire une requête adressée à votre route Flask. Plus spécifiquement, le contenu de la zone de texte et du sélecteur de langue est affecté à des variables, puis passé dans la requête à la route `sentiment-analysis`.

Le code effectue ensuite une itération dans la réponse, puis met à jour le code HTML avec les scores de sentiment.

1. Dans votre IDE, créez un fichier nommé `main.js` dans le répertoire `static`.

2. Copiez ce code dans `static/scripts/main.js` :
   ```javascript
   //Run sentinment analysis on input and translation.
   $("#sentiment-analysis").on("click", function(e) {
     e.preventDefault();
     var inputText = document.getElementById("text-to-translate").value;
     var inputLanguage = document.getElementById("detected-language-result").innerHTML;
     var outputText = document.getElementById("translation-result").value;
     var outputLanguage = document.getElementById("select-language").value;

     var sentimentRequest = { "inputText": inputText, "inputLanguage": inputLanguage, "outputText": outputText,  "outputLanguage": outputLanguage };

     if (inputText !== "") {
       $.ajax({
         url: "/sentiment-analysis",
         method: "POST",
         headers: {
             "Content-Type":"application/json"
         },
         dataType: "json",
         data: JSON.stringify(sentimentRequest),
         success: function(data) {
           for (var i = 0; i < data.documents.length; i++) {
             if (typeof data.documents[i] !== "undefined"){
               if (data.documents[i].id === "1") {
                 document.getElementById("input-sentiment").textContent = data.documents[i].score;
               }
               if (data.documents[i].id === "2") {
                 document.getElementById("translation-sentiment").textContent = data.documents[i].score;
               }
             }
           }
           for (var i = 0; i < data.errors.length; i++) {
             if (typeof data.errors[i] !== "undefined"){
               if (data.errors[i].id === "1") {
                 document.getElementById("input-sentiment").textContent = data.errors[i].message;
               }
               if (data.errors[i].id === "2") {
                 document.getElementById("translation-sentiment").textContent = data.errors[i].message;
               }
             }
           }
           if (document.getElementById("input-sentiment").textContent !== '' && document.getElementById("translation-sentiment").textContent !== ""){
             document.getElementById("sentiment").style.display = "block";
           }
         }
       });
     }
   });
   // In the next section, you'll add code for speech synthesis here.
   ```

### <a name="test-sentiment-analysis"></a>Tester l’analyse des sentiments

Testons l’analyse des sentiments dans l’application.

```
flask run
```

Accédez à l’adresse du serveur fournie. Tapez du texte dans la zone d’entrée, sélectionnez une langue, puis appuyez sur Translate. Vous devez normalement obtenir une traduction. Ensuite, appuyez sur le bouton Run sentiment analysis. Vous devez normalement voir deux scores. Si cela ne fonctionne pas, vérifiez que vous avez ajouté votre clé d’abonnement.

> [!TIP]
> Si les modifications que vous avez apportées n’apparaissent pas ou si l’application ne fonctionne pas comme prévu, essayez en vidant votre cache ou en ouvrant une fenêtre privée/incognito.

Appuyez sur **Ctrl+C** pour arrêter l’application, puis passez à la section suivante.

## <a name="convert-text-to-speech"></a>Convertir la synthèse vocale

L’[API Synthèse vocale](../speech-service/text-to-speech.md) permet à votre application de convertir du texte en synthèse vocale quasi humaine. Le service prend en charge des voix standard, neuronales et personnalisées. Notre exemple d’application utilise quelques-unes des voix disponibles. Pour en obtenir la liste complète, consultez [Langues prises en charge](../speech-service/language-support.md#text-to-speech).

Dans cette section, vous allez effectuer quelques opérations :

* Écrire du code Python pour convertir du texte en voix avec l’API Synthèse vocale
* Créer une route Flask pour appeler votre code Python
* Mettre à jour le code HTML avec un bouton pour convertir du texte en voix et un élément pour la lecture de l’audio
* Écrire du code JavaScript qui permet aux utilisateurs d’interagir avec votre application Flask

### <a name="call-the-text-to-speech-api"></a>Appeler l’API Synthèse vocale

Écrivons une fonction pour convertir du texte en voix. Cette fonction prend deux arguments : `input_text` et `voice_font`. Cette fonction est appelée chaque fois qu’un utilisateur appuie sur le bouton Convert text-to-speech (Convertir du texte en voix) dans votre application. `input_text` est le résultat de la traduction retournée par l’appel pour traduire du texte ; `voice_font` est la valeur du sélecteur de police de la voix dans le code HTML.

1. Créons un fichier appelé `synthesize.py` à la racine de votre répertoire de travail.

2. Ajoutez ensuite ce code à `synthesize.py`.
   ```Python
   import os, requests, time
   from xml.etree import ElementTree

   class TextToSpeech(object):
       def __init__(self, input_text, voice_font):
           subscription_key = 'YOUR_SPEECH_SERVICES_SUBSCRIPTION_KEY'
           self.subscription_key = subscription_key
           self.input_text = input_text
           self.voice_font = voice_font
           self.timestr = time.strftime('%Y%m%d-%H%M')
           self.access_token = None

       # This function performs the token exchange.
       def get_token(self):
           fetch_token_url = 'https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken'
           headers = {
               'Ocp-Apim-Subscription-Key': self.subscription_key
           }
           response = requests.post(fetch_token_url, headers=headers)
           self.access_token = str(response.text)

       # This function calls the TTS endpoint with the access token.
       def save_audio(self):
           base_url = 'https://westus.tts.speech.microsoft.com/'
           path = 'cognitiveservices/v1'
           constructed_url = base_url + path
           headers = {
               'Authorization': 'Bearer ' + self.access_token,
               'Content-Type': 'application/ssml+xml',
               'X-Microsoft-OutputFormat': 'riff-24khz-16bit-mono-pcm',
               'User-Agent': 'YOUR_RESOURCE_NAME',
           }
           # Build the SSML request with ElementTree
           xml_body = ElementTree.Element('speak', version='1.0')
           xml_body.set('{http://www.w3.org/XML/1998/namespace}lang', 'en-us')
           voice = ElementTree.SubElement(xml_body, 'voice')
           voice.set('{http://www.w3.org/XML/1998/namespace}lang', 'en-US')
           voice.set('name', 'Microsoft Server Speech Text to Speech Voice {}'.format(self.voice_font))
           voice.text = self.input_text
           # The body must be encoded as UTF-8 to handle non-ascii characters.
           body = ElementTree.tostring(xml_body, encoding="utf-8")

           #Send the request
           response = requests.post(constructed_url, headers=headers, data=body)

           # Write the response as a wav file for playback. The file is located
           # in the same directory where this sample is run.
           return response.content
   ```
3. Ajoutez votre clé d’abonnement Services Speech et enregistrez.

### <a name="add-a-route-to-apppy"></a>Ajouter une route à `app.py`

Créons ensuite une route dans votre application Flask qui appelle `synthesize.py`. Cette route sera appelée chaque fois qu’un utilisateur appuie sur le bouton Convert text-to-speech dans votre application. Tout comme les routes pour la traduction et l’analyse des sentiments, cette route va accepter des requêtes `POST`, car la fonction attend deux arguments : le texte à synthétiser et la police de la voix pour la lecture.

1. Ouvrez `app.py`, recherchez l’instruction import en haut de `app.py` et mettez-la à jour :

   ```python
   import translate, sentiment, synthesize
   ```
   Notre application Flask peut maintenant utiliser la méthode disponible via `synthesize.py`.

2. Copiez ce code à la fin de `app.py` et enregistrez-le :

   ```Python
   @app.route('/text-to-speech', methods=['POST'])
   def text_to_speech():
       data = request.get_json()
       text_input = data['text']
       voice_font = data['voice']
       tts = synthesize.TextToSpeech(text_input, voice_font)
       tts.get_token()
       audio_response = tts.save_audio()
       return audio_response
   ```

### <a name="update-indexhtml"></a>Mettre à jour `index.html`

Maintenant que vous avez une fonction pour convertir du texte en voix et une route dans votre application Flask pour l’appeler, l’étape suivante consiste à écrire le code HTML pour votre application. Le HTML ci-dessous fait un certain nombre de choses :

* Il fournit une liste déroulante de sélection de la voix
* Il ajoute un bouton pour convertir le texte en voix
* Il ajoute un élément audio, qui est utilisé pour lire la synthèse vocale

1. Ouvrez `index.html` et recherchez ces commentaires dans le code :
   ```html
   <!-- Start voice font selection code -->

   <!-- End voice font selection code -->
   ```

2. Remplacez les commentaires du code par ce bloc HTML :
   ```html
   <div class="form-group">
     <label for="select-voice"><strong>Select voice font:</strong></label>
     <select class="form-control" id="select-voice">
       <option value="(ar-SA, Naayf)">Arabic | Male | Naayf</option>
       <option value="(ca-ES, HerenaRUS)">Catalan | Female | HerenaRUS</option>
       <option value="(zh-CN, HuihuiRUS)">Chinese (Mainland) | Female | HuihuiRUS</option>
       <option value="(zh-CN, Kangkang, Apollo)">Chinese (Mainland) | Male | Kangkang, Apollo</option>
       <option value="(zh-HK, Tracy, Apollo)">Chinese (Hong Kong)| Female | Tracy, Apollo</option>
       <option value="(zh-HK, Danny, Apollo)">Chinese (Hong Kong) | Male | Danny, Apollo</option>
       <option value="(zh-TW, Yating, Apollo)">Chinese (Taiwan)| Female | Yating, Apollo</option>
       <option value="(zh-TW, Zhiwei, Apollo)">Chinese (Taiwan) | Male | Zhiwei, Apollo</option>
       <option value="(hr-HR, Matej)">Croatian | Male | Matej</option>
       <option value="(en-US, AriaRUS)">English (US) | Female | AriaRUS</option>
       <option value="(en-US, Guy24kRUS)">English (US) | Male | Guy24kRUS</option>
       <option value="(en-IE, Sean)">English (IE) | Male | Sean</option>
       <option value="(fr-FR, Julie, Apollo)">French | Female | Julie, Apollo</option>
       <option value="(fr-FR, HortenseRUS)">French | Female | Julie, HortenseRUS</option>
       <option value="(fr-FR, Paul, Apollo)">French | Male | Paul, Apollo</option>
       <option value="(de-DE, Hedda)">German | Female | Hedda</option>
       <option value="(de-DE, HeddaRUS)">German | Female | HeddaRUS</option>
       <option value="(de-DE, Stefan, Apollo)">German | Male | Apollo</option>
       <option value="(el-GR, Stefanos)">Greek | Male | Stefanos</option>
       <option value="(he-IL, Asaf)">Hebrew (Isreal) | Male | Asaf</option>
       <option value="(hi-IN, Kalpana, Apollo)">Hindi | Female | Kalpana, Apollo</option>
       <option value="(hi-IN, Hemant)">Hindi | Male | Hemant</option>
       <option value="(it-IT, LuciaRUS)">Italian | Female | LuciaRUS</option>
       <option value="(it-IT, Cosimo, Apollo)">Italian | Male | Cosimo, Apollo</option>
       <option value="(ja-JP, Ichiro, Apollo)">Japanese | Male | Ichiro</option>
       <option value="(ja-JP, HarukaRUS)">Japanese | Female | HarukaRUS</option>
       <option value="(ko-KR, HeamiRUS)">Korean | Female | Heami</option>
       <option value="(pt-BR, HeloisaRUS)">Portuguese (Brazil) | Female | HeloisaRUS</option>
       <option value="(pt-BR, Daniel, Apollo)">Portuguese (Brazil) | Male | Daniel, Apollo</option>
       <option value="(pt-PT, HeliaRUS)">Portuguese (Portugal) | Female | HeliaRUS</option>
       <option value="(ru-RU, Irina, Apollo)">Russian | Female | Irina, Apollo</option>
       <option value="(ru-RU, Pavel, Apollo)">Russian | Male | Pavel, Apollo</option>
       <option value="(ru-RU, EkaterinaRUS)">Russian | Female | EkaterinaRUS</option>
       <option value="(es-ES, Laura, Apollo)">Spanish | Female | Laura, Apollo</option>
       <option value="(es-ES, HelenaRUS)">Spanish | Female | HelenaRUS</option>
       <option value="(es-ES, Pablo, Apollo)">Spanish | Male | Pablo, Apollo</option>
       <option value="(th-TH, Pattara)">Thai | Male | Pattara</option>
       <option value="(tr-TR, SedaRUS)">Turkish | Female | SedaRUS</option>
       <option value="(vi-VN, An)">Vietnamese | Male | An</option>
     </select>
   </div>
   ```

3. Recherchez ensuite ces commentaires dans le code :
   ```html
   <!-- Add Speech Synthesis button and audio element -->

   <!-- End Speech Synthesis button -->
   ```

4. Remplacez les commentaires du code par ce bloc HTML :

```html
<button type="submit" class="btn btn-primary mb-2" id="text-to-speech">Convert text-to-speech</button>
<div id="audio-playback">
  <audio id="audio" controls>
    <source id="audio-source" type="audio/mpeg" />
  </audio>
</div>
```

5. N’oubliez pas d’enregistrer votre travail.

### <a name="update-mainjs"></a>Mettre à jour `main.js`

Dans le code ci-dessous, le contenu du HTML est utilisé pour construire une requête adressée à votre route Flask. Plus spécifiquement, la traduction et la police de la voix sont affectées à des variables, puis passées dans la requête à la route `text-to-speech`.

Le code effectue ensuite une itération dans la réponse, puis met à jour le code HTML avec les scores de sentiment.

1. Dans votre IDE, créez un fichier nommé `main.js` dans le répertoire `static`.
2. Copiez ce code dans `static/scripts/main.js` :
   ```javascript
   // Convert text-to-speech
   $("#text-to-speech").on("click", function(e) {
     e.preventDefault();
     var ttsInput = document.getElementById("translation-result").value;
     var ttsVoice = document.getElementById("select-voice").value;
     var ttsRequest = { 'text': ttsInput, 'voice': ttsVoice }

     var xhr = new XMLHttpRequest();
     xhr.open("post", "/text-to-speech", true);
     xhr.setRequestHeader("Content-Type", "application/json");
     xhr.responseType = "blob";
     xhr.onload = function(evt){
       if (xhr.status === 200) {
         audioBlob = new Blob([xhr.response], {type: "audio/mpeg"});
         audioURL = URL.createObjectURL(audioBlob);
         if (audioURL.length > 5){
           var audio = document.getElementById("audio");
           var source = document.getElementById("audio-source");
           source.src = audioURL;
           audio.load();
           audio.play();
         }else{
           console.log("An error occurred getting and playing the audio.")
         }
       }
     }
     xhr.send(JSON.stringify(ttsRequest));
   });
   // Code for automatic language selection goes here.
   ```
3. Vous avez presque terminé. La dernière chose que vous allez faire est d’ajouter du code à `main.js` pour sélectionner automatiquement une police de la voix en fonction de la langue sélectionnée pour la traduction. Ajoutez ce bloc de code à `main.js` :
   ```javascript
   // Automatic voice font selection based on translation output.
   $('select[id="select-language"]').change(function(e) {
     if ($(this).val() == "ar"){
       document.getElementById("select-voice").value = "(ar-SA, Naayf)";
     }
     if ($(this).val() == "ca"){
       document.getElementById("select-voice").value = "(ca-ES, HerenaRUS)";
     }
     if ($(this).val() == "zh-Hans"){
       document.getElementById("select-voice").value = "(zh-HK, Tracy, Apollo)";
     }
     if ($(this).val() == "zh-Hant"){
       document.getElementById("select-voice").value = "(zh-HK, Tracy, Apollo)";
     }
     if ($(this).val() == "hr"){
       document.getElementById("select-voice").value = "(hr-HR, Matej)";
     }
     if ($(this).val() == "en"){
       document.getElementById("select-voice").value = "(en-US, Jessa24kRUS)";
     }
     if ($(this).val() == "fr"){
       document.getElementById("select-voice").value = "(fr-FR, HortenseRUS)";
     }
     if ($(this).val() == "de"){
       document.getElementById("select-voice").value = "(de-DE, HeddaRUS)";
     }
     if ($(this).val() == "el"){
       document.getElementById("select-voice").value = "(el-GR, Stefanos)";
     }
     if ($(this).val() == "he"){
       document.getElementById("select-voice").value = "(he-IL, Asaf)";
     }
     if ($(this).val() == "hi"){
       document.getElementById("select-voice").value = "(hi-IN, Kalpana, Apollo)";
     }
     if ($(this).val() == "it"){
       document.getElementById("select-voice").value = "(it-IT, LuciaRUS)";
     }
     if ($(this).val() == "ja"){
       document.getElementById("select-voice").value = "(ja-JP, HarukaRUS)";
     }
     if ($(this).val() == "ko"){
       document.getElementById("select-voice").value = "(ko-KR, HeamiRUS)";
     }
     if ($(this).val() == "pt"){
       document.getElementById("select-voice").value = "(pt-BR, HeloisaRUS)";
     }
     if ($(this).val() == "ru"){
       document.getElementById("select-voice").value = "(ru-RU, EkaterinaRUS)";
     }
     if ($(this).val() == "es"){
       document.getElementById("select-voice").value = "(es-ES, HelenaRUS)";
     }
     if ($(this).val() == "th"){
       document.getElementById("select-voice").value = "(th-TH, Pattara)";
     }
     if ($(this).val() == "tr"){
       document.getElementById("select-voice").value = "(tr-TR, SedaRUS)";
     }
     if ($(this).val() == "vi"){
       document.getElementById("select-voice").value = "(vi-VN, An)";
     }
   });
   ```

### <a name="test-your-app"></a>Test de l'application

Testons la synthèse vocale dans l’application.

```
flask run
```

Accédez à l’adresse du serveur fournie. Tapez du texte dans la zone d’entrée, sélectionnez une langue, puis appuyez sur Translate. Vous devez normalement obtenir une traduction. Ensuite, sélectionnez une voix, puis appuyez sur le bouton Convert text-to-speech. La traduction doit normalement être lue en synthèse vocale. Si cela ne fonctionne pas, vérifiez que vous avez ajouté votre clé d’abonnement.

> [!TIP]
> Si les modifications que vous avez apportées n’apparaissent pas ou si l’application ne fonctionne pas comme prévu, essayez en vidant votre cache ou en ouvrant une fenêtre privée/incognito.

Voilà, vous avez une application qui effectue des traductions, analyse des sentiments et produit une synthèse vocale. Appuyez sur **Ctrl+C** pour mettre fin à l’application. Veillez à consulter les autres services d’[Azure Cognitive Services](../index.yml).

## <a name="get-the-source-code"></a>Obtenir le code source

Le code source de ce projet est disponible sur [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Flask-App-Tutorial).

## <a name="next-steps"></a>Étapes suivantes

* [Informations de référence Translator](./reference/v3-0-reference.md)
* [Informations de référence sur l’API Analyse de texte](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c7)
* [Référence sur l’API conversion de texte par synthèse vocale](../speech-service/rest-text-to-speech.md)
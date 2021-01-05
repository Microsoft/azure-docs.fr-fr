---
title: 'Tutoriel : Intégrer plusieurs ressources Lecteur immersif'
titleSuffix: Azure Cognitive Services
description: Dans ce tutoriel, vous créez une application Node.js qui lance le Lecteur immersif avec plusieurs ressources Lecteur immersif.
author: skamal
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: tutorial
ms.date: 01/14/2020
ms.author: skamal
ms.custom: devx-track-js
ms.openlocfilehash: eba2b53c20f3b0edb79cc32f3c8fb3d82d5433b8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91309283"
---
# <a name="tutorial-integrate-multiple-immersive-reader-resources"></a>Tutoriel : Intégrer plusieurs ressources Lecteur immersif

Dans la [présentation](./overview.md), vous avez appris ce qu’est le Lecteur immersif et comment il implémente des techniques éprouvées pour améliorer la compréhension de la lecture pour les apprenants en langue, les lecteurs émergents et les étudiants présentant des difficultés d’apprentissage. Dans le [guide de démarrage rapide Node.js](./quickstarts/client-libraries.md?pivots=programming-language-nodejs), vous avez appris à utiliser le Lecteur immersif avec une seule ressource. Ce tutoriel explique comment intégrer plusieurs ressources du Lecteur immersif dans la même application. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer plusieurs ressources Lecteur immersif dans un groupe de ressources existant
> * Lancer le Lecteur immersif avec plusieurs ressources

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Suivez les étapes du [guide de démarrage rapide](./quickstarts/client-libraries.md?pivots=programming-language-nodejs) pour créer une application web qui lance le Lecteur immersif avec Node.js. Dans ce guide de démarrage rapide, vous configurez une seule ressource Lecteur immersif. Nous partirons de là pour effectuer le présent tutoriel.

## <a name="create-the-immersive-reader-resources"></a>Créer des ressources Lecteur immersif

Suivez [ces instructions](./how-to-create-immersive-reader.md) pour créer chaque ressource Lecteur immersif. Le script **Create-ImmersiveReaderResource** utilise les paramètres `ResourceName`, `ResourceSubdomain` et `ResourceLocation`. Ces paramètres doivent être uniques pour chaque ressource créée. Les autres paramètres doivent être les mêmes que ceux que vous avez utilisés pour configurer votre première ressource Lecteur immersif. De cette façon, chaque ressource peut être liée au même groupe de ressources Azure et à la même application Azure AD.

L’exemple ci-dessous montre comment créer deux ressources, l’une dans WestUS et l’autre dans EastUS. Notez que les valeurs de `ResourceName`, `ResourceSubdomain` et `ResourceLocation` sont propres à chaque ressource.

```azurepowershell-interactive
Create-ImmersiveReaderResource
  -SubscriptionName <SUBSCRIPTION_NAME> `
  -ResourceName Resource_name_wus `
  -ResourceSubdomain resource-subdomain-wus `
  -ResourceSKU <RESOURCE_SKU> `
  -ResourceLocation westus `
  -ResourceGroupName <RESOURCE_GROUP_NAME> `
  -ResourceGroupLocation <RESOURCE_GROUP_LOCATION> `
  -AADAppDisplayName <AAD_APP_DISPLAY_NAME> `
  -AADAppIdentifierUri <AAD_APP_IDENTIFIER_URI> `
  -AADAppClientSecret <AAD_APP_CLIENT_SECRET>

Create-ImmersiveReaderResource
  -SubscriptionName <SUBSCRIPTION_NAME> `
  -ResourceName Resource_name_eus `
  -ResourceSubdomain resource-subdomain-eus `
  -ResourceSKU <RESOURCE_SKU> `
  -ResourceLocation eastus `
  -ResourceGroupName <RESOURCE_GROUP_NAME> `
  -ResourceGroupLocation <RESOURCE_GROUP_LOCATION> `
  -AADAppDisplayName <AAD_APP_DISPLAY_NAME> `
  -AADAppIdentifierUri <AAD_APP_IDENTIFIER_URI> `
  -AADAppClientSecret <AAD_APP_CLIENT_SECRET>
```

## <a name="add-resources-to-environment-configuration"></a>Ajouter des ressources à la configuration de l’environnement

Dans le guide de démarrage rapide, vous avez créé un fichier de configuration de l’environnement contenant les paramètres `TenantId`, `ClientId`, `ClientSecret` et `Subdomain`. Dans la mesure où toutes vos ressources utilisent la même application Azure AD, nous pouvons utiliser les mêmes valeurs pour `TenantId`, `ClientId` et `ClientSecret`. Le seul changement à faire est de lister chaque sous-domaine pour chaque ressource.

Votre nouveau fichier __.env__ doit maintenant ressembler à ceci :

```text
TENANT_ID={YOUR_TENANT_ID}
CLIENT_ID={YOUR_CLIENT_ID}
CLIENT_SECRET={YOUR_CLIENT_SECRET}
SUBDOMAIN_WUS={YOUR_WESTUS_SUBDOMAIN}
SUBDOMAIN_EUS={YOUR_EASTUS_SUBDOMAIN}
```

Veillez à ne pas valider ce fichier dans le contrôle de code source, car il contient des secrets qui ne doivent pas être rendus publics.

Nous allons maintenant modifier le fichier _routes\index.js_ que nous avons créé pour prendre en charge les ressources multiples. Remplacez le contenu de ce fichier par le code ci-dessous.

Comme auparavant, ce code crée un point de terminaison d’API qui obtient un jeton d’authentification Azure AD en utilisant le mot de passe du principal de service. Cette fois-ci, il permet à l’utilisateur de spécifier un emplacement de ressource et de le passer en tant que paramètre de requête. Le code retourne ensuite un objet contenant le jeton et le sous-domaine correspondant.

```javascript
var express = require('express');
var router = express.Router();
var request = require('request');

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

router.get('/GetTokenAndSubdomain', function(req, res) {
    try {
        request.post({
            headers: {
                'content-type': 'application/x-www-form-urlencoded'
            },
            url: `https://login.windows.net/${process.env.TENANT_ID}/oauth2/token`,
            form: {
                grant_type: 'client_credentials',
                client_id: process.env.CLIENT_ID,
                client_secret: process.env.CLIENT_SECRET,
                resource: 'https://cognitiveservices.azure.com/'
            }
        },
        function(err, resp, tokenResult) {
            if (err) {
                console.log(err);
                return res.status(500).send('CogSvcs IssueToken error');
            }

            var tokenResultParsed = JSON.parse(tokenResult);

            if (tokenResultParsed.error) {
                console.log(tokenResult);
                return res.send({error :  "Unable to acquire Azure AD token. Check the debugger for more information."})
            }

            var token = tokenResultParsed.access_token;

            var subdomain = "";
            var region = req.query && req.query.region;
            switch (region) {
                case "eus":
                    subdomain = process.env.SUBDOMAIN_EUS
                    break;
                case "wus":
                default:
                    subdomain = process.env.SUBDOMAIN_WUS
            }

            return res.send({token, subdomain});
        });
    } catch (err) {
        console.log(err);
        return res.status(500).send('CogSvcs IssueToken error');
    }
});

module.exports = router;
```

Le point de terminaison d’API **getimmersivereaderlaunchparams** doit être sécurisé derrière une certaine forme d’authentification (par exemple [OAuth](https://oauth.net/2/)) pour empêcher les utilisateurs non autorisés d’obtenir des jetons à utiliser auprès de vos services Facturation et Lecteur immersif ; ce travail dépasse le cadre de ce tutoriel.

## <a name="launch-the-immersive-reader-with-sample-content"></a>Lancer le Lecteur immersif avec un exemple de contenu

1. Ouvrez _views\index.pug_, puis remplacez son contenu par le code suivant. Ce code remplit la page avec un exemple de contenu et ajoute deux boutons qui lancent le Lecteur immersif pour la ressource EastUS et la ressource WestUS, respectivement.

    ```pug
    doctype html
    html
        head
            title Immersive Reader Quickstart Node.js

            link(rel='stylesheet', href='https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css')

            // A polyfill for Promise is needed for IE11 support.
            script(src='https://cdn.jsdelivr.net/npm/promise-polyfill@8/dist/polyfill.min.js')

            script(src='https://contentstorage.onenote.office.net/onenoteltir/immersivereadersdk/immersive-reader-sdk.1.0.0.js')
            script(src='https://code.jquery.com/jquery-3.3.1.min.js')

            style(type="text/css").
                .immersive-reader-button {
                background-color: white;
                margin-top: 5px;
                border: 1px solid black;
                float: right;
                }
        body
            div(class="container")
                button(class="immersive-reader-button" data-button-style="icon" data-locale="en" onclick='handleLaunchImmersiveReader("wus")') WestUS Immersive Reader
                button(class="immersive-reader-button" data-button-style="icon" data-locale="en" onclick='handleLaunchImmersiveReader("eus")') EastUS Immersive Reader

                h1(id="ir-title") About Immersive Reader
                div(id="ir-content" lang="en-us")
                p Immersive Reader is a tool that implements proven techniques to improve reading comprehension for emerging readers, language learners, and people with learning differences. The Immersive Reader is designed to make reading more accessible for everyone. The Immersive Reader

                    ul
                        li Shows content in a minimal reading view
                        li Displays pictures of commonly used words
                        li Highlights nouns, verbs, adjectives, and adverbs
                        li Reads your content out loud to you
                        li Translates your content into another language
                        li Breaks down words into syllables

                h3 The Immersive Reader is available in many languages.

                p(lang="es-es") El Lector inmersivo está disponible en varios idiomas.
                p(lang="zh-cn") 沉浸式阅读器支持许多语言
                p(lang="de-de") Der plastische Reader ist in vielen Sprachen verfügbar.
                p(lang="ar-eg" dir="rtl" style="text-align:right") يتوفر \"القارئ الشامل\" في العديد من اللغات.

    script(type="text/javascript").
    function getTokenAndSubdomainAsync(region) {
            return new Promise(function (resolve, reject) {
                $.ajax({
                    url: "/GetTokenAndSubdomain",
                    type: "GET",
                    data: {
                        region: region
                    },
                    success: function (data) {
                        if (data.error) {
                            reject(data.error);
                        } else {
                            resolve(data);
                        }
                    },
                    error: function (err) {
                        reject(err);
                    }
                });
            });
        }

        function handleLaunchImmersiveReader(region) {
            getTokenAndSubdomainAsync(region)
                .then(function (response) {
                    const token = response["token"];
                    const subdomain = response["subdomain"];
                    // Learn more about chunk usage and supported MIME types https://docs.microsoft.com/azure/cognitive-services/immersive-reader/reference#chunk
                    const data = {
                        title: $("#ir-title").text(),
                        chunks: [{
                            content: $("#ir-content").html(),
                            mimeType: "text/html"
                        }]
                    };
                    // Learn more about options https://docs.microsoft.com/azure/cognitive-services/immersive-reader/reference#options
                    const options = {
                        "onExit": exitCallback,
                        "uiZIndex": 2000
                    };
                    ImmersiveReader.launchAsync(token, subdomain, data, options)
                        .catch(function (error) {
                            alert("Error in launching the Immersive Reader. Check the console.");
                            console.log(error);
                        });
                })
                .catch(function (error) {
                    alert("Error in getting the Immersive Reader token and subdomain. Check the console.");
                    console.log(error);
                });
        }

        function exitCallback() {
            console.log("This is the callback function. It is executed when the Immersive Reader closes.");
        }
    ```

3. Notre application web est maintenant prête. Démarrez l’application en exécutant :

    ```bash
    npm start
    ```

4. Ouvrez votre navigateur et accédez à `http://localhost:3000`. Vous devez voir le contenu ci-dessus dans la page. Cliquez sur le bouton **EastUS Immersive Reader** ou sur le bouton **WestUS Immersive Reader** pour lancer le Lecteur immersif en utilisant ces ressources respectives.

## <a name="next-steps"></a>Étapes suivantes

* Explorer le [SDK Lecteur Immersif](https://github.com/microsoft/immersive-reader-sdk) et la [référence du SDK Lecteur immersif](./reference.md)
* Voir des exemples de code sur [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/advanced-csharp)
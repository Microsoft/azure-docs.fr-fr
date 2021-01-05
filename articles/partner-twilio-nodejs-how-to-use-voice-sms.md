---
title: Utilisation de Twilio pour les fonctionnalités vocales, VoIP et de messagerie SMS dans Azure
description: Découvrez comment passer un appel téléphonique et envoyer un SMS avec le service d'API Twilio sur Azure. Exemples de code écrits en Node.js.
documentationcenter: nodejs
author: georgewallace
ms.assetid: f558cbbd-13d2-416f-b9b1-33a99c426af9
ms.service: multiple
ms.devlang: nodejs
ms.topic: article
ms.date: 11/25/2014
ms.author: gwallace
ms.openlocfilehash: c4b1c506ff0e78e5e304ceb1682c73f9cd784c95
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88080539"
---
# <a name="using-twilio-for-voice-voip-and-sms-messaging-in-azure"></a>Utilisation de Twilio pour les fonctionnalités vocales, VoIP et de messagerie SMS dans Azure
Ce guide montre comment générer des applications qui communiquent avec Twilio et node.js sur Azure.

<a name="whatis"></a>

## <a name="what-is-twilio"></a>Présentation de Twilio
Twilio est une plateforme API qui facilite pour les développeurs le passage et la réception d'appels téléphoniques, l'envoi et la réception de SMS. Elle incorpore l'appel VoIP dans des applications basées sur un navigateur et mobiles natives. Survolons rapidement le fonctionnement de Twilio avant de l'étudier plus en détail.

### <a name="receiving-calls-and-text-messages"></a>Réception d'appels et de SMS
Twilio permet aux développeurs [d’acheter des numéros de téléphone programmables][purchase_phone] qui peuvent être utilisés à la fois pour envoyer et recevoir des appels et des SMS. Lorsqu'un numéro Twilio reçoit un appel ou un SMS, il envoie à votre application web une requête HTTP POST ou GET, vous demandant des instructions sur la gestion de l'appel ou du SMS. Votre serveur répond à la requête HTTP de Twilio avec [TwiML][twiml], jeu simple de balises XML qui contient des instructions sur la gestion d’un appel ou d’un SMS. Nous vous présenterons des exemples de TwiML un peu plus loin.

### <a name="making-calls-and-sending-text-messages"></a>Passage d'appels et envoi de SMS
En émettant des requêtes HTTP vers l'API de service Web Twilio, les développeurs peuvent envoyer des SMS et passer des appels téléphoniques. Pour les appels, le développeur doit également spécifier une URL qui renvoie des instructions TwiML relatives à la gestion de l'appel une fois qu'il est connecté.

### <a name="embedding-voip-capabilities-in-ui-code-javascript-ios-or-android"></a>Incorporation des fonctionnalités VoIP dans un code de l'interface utilisateur (JavaScript, iOS ou Android)
Twilio offre un kit SDK côté client qui peut transformer n'importe quel navigateur web, application iOS ou Android en un téléphone VoIP. Dans cet article, nous nous concentrerons sur l'utilisation de l'appel VoIP dans le navigateur. En plus du kit *SDK JavaScript Twilio* s’exécutant dans le navigateur, vous devez utiliser une application côté serveur (notre application node.js) pour émettre un « jeton de fonctionnalité » pour le client JavaScript. Vous trouverez plus d’informations sur l’utilisation de VoIP avec node.js [sur le blog des développeurs Twilio][voipnode].

<a name="signup"></a>

## <a name="sign-up-for-twilio-microsoft-discount"></a>Inscription à Twilio (remise Microsoft)
Avant d’utiliser les services Twilio, vous devez [vous inscrire pour obtenir un compte][signup]. Les clients Microsoft Azure reçoivent une remise spéciale. Pour y avoir droit, [veillez à vous inscrire ici][signup] !

<a name="azuresite"></a>

## <a name="create-and-deploy-a-nodejs-azure-website"></a>Création et déploiement d'un site web Azure node.js
Ensuite, vous allez devoir créer un site web node.js s'exécutant sur Azure. [La documentation officielle relative à cette création est située ici][azure_new_site]. À un niveau supérieur, vous réaliserez les tâches suivantes :

* Création d'un compte Azure si vous n'en possédez pas déjà un
* Utilisation de la console d'administration pour créer un site web
* Ajout d'une prise en charge du contrôle de code source (nous présumons que vous avez utilisé git)
* Création d’un fichier `server.js` avec une application web node.js simple
* Déploiement de cette application simple sur Azure

<a name="twiliomodule"></a>

## <a name="configure-the-twilio-module"></a>Configuration du module Twilio
Ensuite, nous commencerons à rédiger une simple application node.js qui utilise l'API Twilio. Avant de commencer, nous devons configurer les informations d'identification de notre compte Twilio.

### <a name="configuring-twilio-credentials-in-system-environment-variables"></a>Configuration des informations d'identification Twilio dans des variables d'environnement système
Afin d'émettre des requêtes authentifiées au serveur principal Twilio, nous avons besoin de notre SID de compte et de notre jeton d'authentification, qui fonctionnent en tant que nom d'utilisateur et mot de passe pour notre compte Twilio. Le moyen le plus sécurisé de configurer ces deux éléments pour une utilisation avec le module Node dans Azure est d'utiliser les variables d'environnement système, que vous pouvez définir directement dans la console d'administration d'Azure.

Sélectionnez votre site web node.js et cliquez sur le lien « CONFIGURER ».  Si vous faites un peu défiler vers le bas, vous verrez une zone dans laquelle vous pouvez définir les propriétés de configuration de votre application.  Entrez les informations d’identification de votre compte Twilio ([disponibles dans la console Twilio][twilio_console]) comme illustré : assurez-vous de les nommer `TWILIO_ACCOUNT_SID` et `TWILIO_AUTH_TOKEN`, respectivement :

![Console d'administration Azure][azure-admin-console]

Une fois que vous avez configuré ces variables, redémarrez votre application dans la console Azure.

### <a name="declaring-the-twilio-module-in-packagejson"></a>Déclaration du module Twilio dans package.json
Ensuite, nous devons créer un fichier package.json pour gérer nos dépendances de module Node via [npm]. Au même niveau que le fichier `server.js` que vous avez créé dans le didacticiel *Azure/node.js*, créez un fichier nommé `package.json`.  Placez-y les éléments suivants :

```json
{
  "name": "application-name",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node server"
  },
  "dependencies": {
    "body-parser": "^1.16.1",
    "ejs": "^2.5.5",
    "errorhandler": "^1.5.0",
    "express": "^4.14.1",
    "morgan": "^1.8.1",
    "twilio": "^2.11.1"
  }
}
```

Cette instruction déclare le module Twilio en tant que dépendance ainsi que la [structure web Express][express] courante et le moteur de modèles EJS.  Tout est maintenant prêt et nous pouvons passer à l'écriture du code.

<a name="makecall"></a>

## <a name="make-an-outbound-call"></a>Exécution d'un appel téléphonique
Créons un simple formulaire qui passera un appel vers un numéro que nous choisissons. Ouvrez `server.js` et saisissez le code suivant. Lorsque le code indique "CHANGE_ME", placez le nom de votre site web Azure à cet endroit :

```javascript
// Module dependencies
const express = require('express');
const path = require('path');
const http = require('http');
const twilio = require('twilio');
const logger = require('morgan');
const bodyParser = require('body-parser');
const errorHandler = require('errorhandler');
const accountSid = process.env.TWILIO_ACCOUNT_SID;
const authToken = process.env.TWILIO_AUTH_TOKEN;
// Create Express web application
const app = express();

// Express configuration
app.set('port', process.env.PORT || 3000);
app.set('views', __dirname + '/views');
app.set('view engine', 'ejs');
app.use(logger('tiny'));
app.use(bodyParser.urlencoded({ extended: false }))
app.use(bodyParser.json())
app.use(express.static(path.join(__dirname, 'public')));

if (app.get('env') !== 'production') {
  app.use(errorHandler());
}

// Render an HTML user interface for the application's home page
app.get('/', (request, response) => response.render('index'));

// Handle the form POST to place a call
app.post('/call', (request, response) => {
  var client = twilio(accountSid, authToken);

  client.makeCall({
    // make a call to this number
    to:request.body.number,

    // Change to a Twilio number you bought - see:
    // https://www.twilio.com/console/phone-numbers/incoming
    from:'+15558675309',

    // A URL in our app which generates TwiML
    // Change "CHANGE_ME" to your app's name
    url:'https://CHANGE_ME.azurewebsites.net/outbound_call'
  }, () => {
      // Go back to the home page
      response.redirect('/');
  });
});

// Generate TwiML to handle an outbound call
app.post('/outbound_call', (request, response) => {
  var twiml = new twilio.TwimlResponse();

  // Say a message to the call's receiver
  twiml.say('hello - thanks for checking out Twilio and Azure', {
      voice:'woman'
  });

  response.set('Content-Type', 'text/xml');
  response.send(twiml.toString());
});

// Start server
app.listen(app.get('port'), function(){
  console.log(`Express server listening on port ${app.get('port')}`);
});
```

Ensuite, créez un répertoire appelé `views` et créez-y un fichier nommé `index.ejs` avec le contenu suivant :

```html
<!DOCTYPE html>
<html>
<head>
  <title>Twilio Test</title>
  <style>
    input { height:20px; width:300px; font-size:18px; margin:5px; padding:5px; }
  </style>
</head>
<body>
  <h1>Twilio Test</h1>
  <form action="/call" method="POST">
      <input placeholder="Enter a phone number" name="number"/>
      <br/>
      <input type="submit" value="Call the number above"/>
  </form>
</body>
</html>
```

Maintenant, déployez votre site web sur Azure et ouvrez votre page d'accueil. Vous devez être en mesure de saisir votre numéro de téléphone dans le champ de texte et de recevoir un appel depuis votre numéro Twilio !

<a name="sendmessage"></a>

## <a name="send-an-sms-message"></a>Envoi d'un SMS
À présent, configurons une interface utilisateur et une logique de gestion pour envoyer un SMS. Ouvrez `server.js` et ajoutez le code suivant après le dernier appel à `app.post` :

```javascript
app.post('/sms', (request, response) => {
  const client = twilio(accountSid, authToken);

  client.sendSms({
      // send a text to this number
      to:request.body.number,

      // A Twilio number you bought - see:
      // https://www.twilio.com/console/phone-numbers/incoming
      from:'+15558675309',

      // The body of the text message
      body: request.body.message

  }, () => {
      // Go back to the home page
      response.redirect('/');
  });
});
```

Dans `views/index.ejs`, ajoutez un autre formulaire sous le premier afin de soumettre un numéro et un SMS :

```html
<form action="/sms" method="POST">
  <input placeholder="Enter a phone number" name="number"/>
  <br/>
  <input placeholder="Enter a message to send" name="message"/>
  <br/>
  <input type="submit" value="Send text to the number above"/>
</form>
```

Redéployez votre application sur Azure. Vous devriez désormais être en mesure d’envoyer ce formulaire et de vous envoyer (ou d’envoyer à un ami proche) un SMS !

<a name="nextsteps"></a>

## <a name="next-steps"></a>Étapes suivantes
Vous avez appris les bases d'utilisation de node.js et de Twilio pour générer des applications qui communiquent. Mais ces exemples ne représentent d'une infime partie des possibilités offertes par Twilio et node.js. Pour plus d'informations sur l'utilisation de Twilio avec node.js, consultez les ressources suivantes :

* [Documentations officielles sur le module][docs]
* [Didacticiel sur VoIP avec des applications node.js][voipnode]
* [Votr – application de vote par SMS en temps réel avec node.js et CouchDB (trois parties)][votr]
* [Programmation par paire dans le navigateur avec node.js][pair]

Nous espérons que vous aimerez travailler sur node.js et Twilio sur Azure !

[purchase_phone]: https://www.twilio.com/console/phone-numbers/search
[twiml]: https://www.twilio.com/docs/api/twiml
[signup]: https://ahoy.twilio.com/azure
[azure_new_site]: app-service/quickstart-nodejs.md
[twilio_console]: https://www.twilio.com/console
[npm]: https://npmjs.org
[express]: https://expressjs.com
[voipnode]: https://www.twilio.com/blog/2013/04/introduction-to-twilio-client-with-node-js.html
[docs]: https://www.twilio.com/docs/libraries/reference/twilio-node/
[votr]: https://www.twilio.com/blog/2012/09/building-a-real-time-sms-voting-app-part-1-node-js-couchdb.html
[pair]: https://www.twilio.com/blog/2013/06/pair-programming-in-the-browser-with-twilio.html
[azure-admin-console]: ./media/partner-twilio-nodejs-how-to-use-voice-sms/twilio_1.png

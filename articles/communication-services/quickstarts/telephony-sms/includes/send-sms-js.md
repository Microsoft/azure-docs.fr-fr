---
title: Fichier include
description: Fichier include
services: azure-communication-services
author: dademath
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 07/28/2020
ms.topic: include
ms.custom: include file
ms.author: dademath
ms.openlocfilehash: ad8266d936c272ee2f6bad254738622c3f81bf03
ms.sourcegitcommit: 6a4687b86b7aabaeb6aacdfa6c2a1229073254de
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/06/2020
ms.locfileid: "91757129"
---
Commencez avec Azure Communication Services en utilisant la bibliothèque de client Communication Services SMS JavaScript pour envoyer des SMS.

Le fait de suivre ce guide de démarrage rapide entraîne une petite dépense de quelques cents USD tout au plus dans votre compte Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](https://github.com/Azure/azure-sdk-for-js-pr/tree/feature/communication/sdk/communication/communication-sms) | [Package (NPM)](https://www.npmjs.com/package/@azure/communication-sms) | [Samples](#todo-samples)-->

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Node.js](https://nodejs.org/) versions Active LTS et Maintenance LTS (8.11.1 et 10.14.1 recommandées).
- Une ressource Communication Services active et une chaîne de connexion. [Créez une ressource Communication Services](../../create-communication-resource.md).
- Un numéro de téléphone permettant de recevoir des SMS. [Obtenez un numéro de téléphone](../get-phone-number.md).

### <a name="prerequisite-check"></a>Vérification du prérequis

- Dans une fenêtre de terminal ou de commande, exécutez `node --version` pour vérifier que Node.js est installé.
- Pour voir les numéros de téléphone associés à votre ressource Communication Services, connectez-vous au [portail Azure](https://portal.azure.com/), localisez votre ressource Communication Services, puis ouvrez l’onglet **Phone Numbers** (Numéros de téléphone) dans le volet de navigation gauche.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-nodejs-application"></a>Créer une application Node.js

Pour commencer, ouvrez votre fenêtre de terminal ou de commande, créez un répertoire pour votre application, puis accédez-y.

```console
mkdir sms-quickstart && cd sms-quickstart
```

Exécutez `npm init -y` pour créer un fichier **package.json** avec les paramètres par défaut.

```console
npm init -y
```

Utilisez un éditeur de texte pour créer un fichier appelé **send-sms.js** dans le répertoire racine du projet. Dans les sections suivantes, vous ajouterez l’ensemble du code source de ce démarrage rapide dans ce fichier.

### <a name="install-the-package"></a>Installer le package

Utilisez la commande `npm install` pour installer la bibliothèque de client Azure Communication Services SMS pour JavaScript.

```console
npm install @azure/communication-sms --save
```

L’option `--save` liste la bibliothèque comme dépendance dans votre fichier **package.json**.

## <a name="object-model"></a>Modèle objet

Les classes et les interfaces suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client Azure Communication Services SMS pour Node.js.

| Nom                                  | Description                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| SmsClient | Cette classe est nécessaire pour toutes les fonctionnalités SMS. Vous l’instanciez avec vos informations d’abonnement et vous l’utilisez pour envoyer des SMS. |
| SendSmsOptions | Cette interface fournit des options permettant de configurer la création de rapports de remise. Si `enable_delivery_report` est défini sur `true`, un événement est émis quand la remise a réussi. |
| SendMessageRequest | Cette interface est le modèle de génération de la demande par SMS (par exemple, configurer les numéros de téléphone d’origine et de destination, ainsi que le contenu du SMS). |

## <a name="authenticate-the-client"></a>Authentifier le client

Importez le **SmsClient** à partir de la bibliothèque de client et instanciez-le avec votre chaîne de connexion. Le code ci-dessous récupère la chaîne de connexion pour la ressource à partir d’une variable d’environnement nommée `COMMUNICATION_SERVICES_CONNECTION_STRING`. Découvrez comment [gérer la chaîne de connexion de votre ressource](../../create-communication-resource.md#store-your-connection-string).

Ajoutez le code suivant dans **send-sms.js** :

```javascript
const { SmsClient } = require('@azure/communication-sms');

// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the SMS client
const smsClient = new SmsClient(connectionString);
```

## <a name="send-an-sms-message"></a>Envoyer un message SMS

Envoyez un SMS en appelant la méthode `send`. Ajoutez ce code à la fin du fichier **send-sms.js** :

```javascript
async function main() {
  await smsClient.send({
    from: "<leased-phone-number>",
    to: ["<to-phone-number>"],
    message: "Hello World 👋🏻 via Sms"
  }, {
    enableDeliveryReport: true //Optional parameter
  });
}

main();
```

Vous devez remplacer `<leased-phone-number>` par un numéro de téléphone permettant de recevoir des SMS associé à votre ressource Communication Services et `<to-phone-number>` par le numéro de téléphone auquel vous souhaitez envoyer un message.

Le paramètre `enableDeliveryReport` est un paramètre facultatif que vous pouvez utiliser pour configurer la création de rapports de remise. C’est utile pour les scénarios où vous souhaitez émettre des événements quand des SMS sont remis. Consultez le guide de démarrage rapide [Gérer les événements SMS](../handle-sms-events.md) pour configurer la création de rapports de remise pour vos SMS.

## <a name="run-the-code"></a>Exécuter le code

Utilisez la commande `node` pour exécuter le code que vous avez ajouté au fichier **send-sms.js**.

```console

node ./send-sms.js

```

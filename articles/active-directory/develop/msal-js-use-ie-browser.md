---
title: Problèmes avec Internet Explorer (MSAL.js) | Azure
titleSuffix: Microsoft identity platform
description: Utilisez la bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js) avec le navigateur Internet Explorer.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/16/2019
ms.author: nacanuma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 633166f3bb46212991920d6720737f8268b3f401
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87026852"
---
# <a name="known-issues-on-internet-explorer-browsers-msaljs"></a>Problèmes connus sur les navigateurs Internet Explorer (MSAL.js)

La bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js) est générée pour [JavaScript ES5](https://fr.wikipedia.org/wiki/ECMAScript#ECMAScript_Edition_5_.28ES5.29) de manière à pouvoir s’exécuter dans Internet Explorer. Toutefois, il existe quelques éléments à connaître.

## <a name="run-an-app-in-internet-explorer"></a>Exécuter une application dans Internet Explorer
Si vous avez l’intention d’utiliser MSAL.js dans des applications qui peuvent s’exécuter dans Internet Explorer, vous devez ajouter une référence à un polyfill promis avant de référencer le script MSAL.js.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/bluebird/3.3.4/bluebird.min.js" class="pre"></script>
```

En effet, Internet Explorer ne prend pas en charge les promesses JavaScript en mode natif.

## <a name="debugging-an-application-running-in-internet-explorer"></a>Débogage d’une application en cours d’exécution dans Internet Explorer

### <a name="running-in-production"></a>Exécution en production
Le déploiement de votre application en production (par exemple, dans les applications web Azure) doit fonctionner correctement, à condition que l’utilisateur final ait accepté les messages apparus dans les fenêtres contextuelles. Nous l’avons testé avec Internet Explorer 11.

### <a name="running-locally"></a>Exécution locale
Si vous souhaitez exécuter et déboguer localement votre application en cours d’exécution dans Internet Explorer, vous devez savoir ce qui suit (en supposant que vous souhaitiez exécuter votre application en tant que *http://localhost:1234* ) :

- Internet Explorer possède un mécanisme de sécurité nommé « mode protégé » qui empêche le bon fonctionnement de MSAL.js. Par exemple, après connexion, la page peut être redirigée vers http://localhost:1234/null.

- Pour exécuter et déboguer votre application localement, vous devez désactiver ce « mode protégé ». Pour ce faire, procédez comme suit :

    1. Cliquez sur Internet Explorer **Outils** (icône d’engrenage).
    1. Sélectionnez **Options Internet**, puis l’onglet **Sécurité**.
    1. Cliquez sur la zone **Internet**, puis décochez l’option **Activer le mode protégé (redémarrage d’Internet Explorer requis)** . Internet Explorer vous avertit que votre ordinateur n’est plus protégé. Cliquez sur **OK**.
    1. Redémarrez Internet Explorer.
    1. Exécutez et déboguez votre application.

Lorsque vous avez terminé, restaurez les paramètres de sécurité d’Internet Explorer.  Sélectionnez **Paramètres** -> **Options Internet** -> **Sécurité** -> **Rétablir toutes les zones au niveau par défaut**.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les [problèmes connus d’utilisation de MSAL.js dans Internet Explorer](msal-js-use-ie-browser.md).
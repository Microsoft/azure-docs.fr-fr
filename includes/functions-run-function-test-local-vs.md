---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/06/2020
ms.author: glenga
ms.openlocfilehash: b4b2409928b6a4196738c7cc6c7040e781d34686
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "80056622"
---
1. Pour exécuter votre fonction, appuyez sur F5 dans Visual Studio. Vous devez peut-être activer une exception de pare-feu afin de permettre aux outils de prendre en charge les requêtes HTTP. Les niveaux d’autorisation ne sont jamais appliqués quand vous exécutez une fonction localement.

2. Copiez l’URL de votre fonction à partir de la sortie runtime Azure Functions.

    ![Azure runtime local](./media/functions-run-function-test-local-vs/functions-debug-local-vs.png)

3. Collez l’URL de la demande HTTP dans la barre d’adresses de votre navigateur. Ajoutez la chaîne de requête `?name=<YOUR_NAME>` à cette URL et exécutez la demande. L’image suivante montre dans le navigateur la réponse de la fonction à la requête GET locale : 

    ![Réponse de la fonction localhost dans le navigateur](./media/functions-run-function-test-local-vs/functions-run-browser-local-vs.png)

4. Pour arrêter le débogage, appuyez sur Maj+F5 dans Visual Studio.
---
author: burkeholland
ms.service: static-web-apps
ms.topic: include
ms.date: 05/08/2020
ms.author: buhollan
ms.openlocfilehash: 879d0c2e8c4cd66ce04c0298d00f7d6a1987acf5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83594398"
---
#### <a name="local-proxy"></a>Proxy local

Vous pouvez configurer un proxy pour l’extension Live Server Visual Studio Code qui achemine toutes les requêtes à `/api` vers le point de terminaison de l’API en cours d’exécution sur `http://127.0.0.1:7071/api`.

1. Ouvrez le fichier _.vscode/settings.json_.

1. Ajoutez les paramètres suivants pour spécifier un proxy pour Live Server.

   ```json
   "liveServer.settings.proxy": {
      "enable": true,
      "baseUri": "/api",
      "proxyUri": "http://127.0.0.1:7071/api"
   }
   ```

   Cette configuration est à enregistrer de préférence dans le fichier de paramètres du projet, par opposition au fichier de paramètres utilisateur.

   Le fait d’utiliser les paramètres de projet garantit que le proxy n’est pas appliqué à tous les autres projets ouverts dans Visual Studio Code.

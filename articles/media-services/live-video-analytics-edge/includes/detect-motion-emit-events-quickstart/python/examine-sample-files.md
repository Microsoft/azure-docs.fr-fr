---
ms.openlocfilehash: 46e8609be2ec57600e666ef9dab33eae2900e1ce
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88690989"
---
1. Dans Visual Studio Code, accédez à *src/edge*. Le fichier  *.env* et quelques fichiers de modèle de déploiement s’affichent.

    Le modèle de déploiement fait référence au manifeste de déploiement de l’appareil de périphérie, où des variables sont utilisées pour certaines propriétés. Le fichier  *.env* contient les valeurs de ces variables.
1. Accédez au dossier *src/cloud-to-device-console-app*. Ce dernier contient le fichier *appsettings.json* et quelques autres fichiers :

    * ***operations.json*** : liste des opérations que vous voulez que le programme exécute.
    * **main.py** : exemple de code de programme. Ce code :
    
      * Il charge les paramètres de l’application.
      * Invoque les méthodes directes qui sont exposées par le module Live Video Analytics sur IoT Edge. Vous pouvez utiliser le module pour analyser des flux vidéo en direct en invoquant ses [méthodes directes](../../../direct-methods.md).
      * S’interrompt pour vous permettre d’examiner la sortie du programme dans la fenêtre **TERMINAL** et d’examiner les événements générés par le module dans la fenêtre **SORTIE**.
      * Invoque des méthodes directes pour nettoyer des ressources.   


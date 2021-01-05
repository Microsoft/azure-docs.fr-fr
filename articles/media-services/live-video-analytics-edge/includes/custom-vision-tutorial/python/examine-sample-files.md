---
author: russell-cooks
ms.service: media-services
ms.topic: include
ms.date: 10/05/2020
ms.author: russellcooks
ms.openlocfilehash: 5dd823c618a0d49bef29c473e6c293762b859149
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358243"
---
1. Dans Visual Studio Code, accédez à src/edge. Vous verrez le fichier .env que vous avez créé, ainsi que quelques fichiers de modèle de déploiement.

    Le modèle de déploiement fait référence au manifeste de déploiement du périphérique avec des valeurs d’espace réservé. Le fichier .env contient les valeurs de ces variables.
1. Ensuite, accédez au dossier src/cloud-to-device-console-app. Vous y voyez le fichier appsettings.json que vous avez créé, ainsi que quelques autres fichiers :

   * operations.json : répertorie les différentes opérations que vous souhaitez que le programme exécute.
   * main.py - Exemple de code de programme, qui effectue les opérations suivantes :
    
        * Il charge les paramètres de l’application.
        * Utilise les méthodes directes du module Live Video Analytics sur IoT Edge pour créer une topologie, ainsi que pour instancier le graphe et l’activer.
        * S’interrompt pour vous permettre d’examiner la sortie du graphe dans la fenêtre **TERMINAL** et les événements envoyés au hub IoT dans la fenêtre **SORTIE**.
        * Désactive puis supprime l’instance de graphe, et supprime la topologie du graphe.

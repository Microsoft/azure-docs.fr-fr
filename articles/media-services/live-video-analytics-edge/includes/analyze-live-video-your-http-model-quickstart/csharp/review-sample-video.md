---
ms.openlocfilehash: eb6551a9ec9accd4a7ee270ec47794fdaed91a8a
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88684243"
---
Quand vous configurez les ressources Azure, une courte vidéo du trafic d’une autoroute est copiée dans Azure sur la machine virtuelle Linux que vous utilisez comme appareil IoT Edge. Ce guide de démarrage rapide utilise le fichier vidéo pour simuler un flux en direct.

Ouvrez une application comme le [lecteur multimédia VLC](https://www.videolan.org/vlc/). Sélectionnez `Ctrl+N`, puis collez un lien vers [l’exemple de vidéo d’intersection d’autoroute](https://lvamedia.blob.core.windows.net/public/camera-300s.mkv) pour démarrer la lecture. Vous voyez la séquence vidéo de nombreux véhicules se déplaçant dans le trafic d’une autoroute.

Dans ce guide de démarrage rapide, vous allez utiliser Live Video Analytics sur IoT Edge pour détecter des objets tels que des véhicules et des personnes. Vous allez publier les événements d’inférence associés sur IoT Edge Hub.

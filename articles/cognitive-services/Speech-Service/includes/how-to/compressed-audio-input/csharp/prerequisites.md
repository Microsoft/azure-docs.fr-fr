---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/09/2020
ms.author: trbye
ms.openlocfilehash: 64eab53d24a14bb51633be3f979db20963677d93
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81422076"
---
La gestion d'un contenu audio compressé est implémentée à l’aide de [GStreamer](https://gstreamer.freedesktop.org). Pour une raison liée à la gestion des licences, les fichiers binaires GStreamer ne sont pas compilés et liés avec le Kit de développement logiciel (SDK) Speech. Les développeurs doivent installer plusieurs dépendances et plug-ins. Voir [installation sur Windows](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html?gi-language=c). Les binaires GStreamer doivent se trouver dans le chemin d’accès système afin que le Kit de développement logiciel (SDK) puisse charger les binaires GStreamer lors de l’exécution. Si le Kit de développement logiciel (SDK) Speech est capable de trouver le fichier libgstreamer-1.0-0.dll lors de l’exécution, cela signifie que les fichiers binaires GStreamer se trouvent dans le chemin d’accès système.


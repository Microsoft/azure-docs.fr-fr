---
author: amitkumarshukla
ms.service: cognitive-services
ms.topic: include
ms.date: 03/09/2020
ms.author: amishu
ms.openlocfilehash: a715d0ece88cf2caf1cb3d20d703f550353094c5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87282729"
---
La gestion de contenu audio compressé est implémentée à l’aide de [`GStreamer`](https://gstreamer.freedesktop.org). Pour une raison liée à la gestion des licences, les fichiers binaires `GStreamer` ne sont pas compilés et liés avec le Kit de développement logiciel (SDK) Speech. Les développeurs doivent installer plusieurs dépendances et plug-ins. Voir [installation sur Windows](https://gstreamer.freedesktop.org/documentation/installing/on-windows.html?gi-language=c). Les fichiers binaires `GStreamer` doivent se trouver dans le chemin d’accès système afin que le Kit de développement logiciel (SDK) Speech puisse les charger les fichiers binaires lors de l’exécution du runtime. Si le kit de développement logiciel (SDK) Speech est en mesure de trouver `libgstreamer-1.0-0.dll` lors de l’exécution du runtime, cela signifie que les fichiers binaires se trouvent dans le chemin d’accès système.


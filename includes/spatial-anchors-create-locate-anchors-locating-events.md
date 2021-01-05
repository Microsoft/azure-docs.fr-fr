---
author: msftradford
ms.service: azure-spatial-anchors
ms.topic: include
ms.date: 11/20/2020
ms.author: parkerra
ms.openlocfilehash: 5360cbb7bdfbdc59e87366e73a891b5c2583672e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993056"
---
Une fois votre ressource Watcher créée, l’événement `AnchorLocated` se déclenche pour chaque ancre demandée. Cet événement se déclenche lorsqu’une ancre est localisée, ou si l’ancre ne peut pas être localisée. Si cette situation se produit, la raison est indiquée dans l’état. Après le traitement de toutes les ancres d’une ressource Watcher, trouvées ou non, l’événement `LocateAnchorsCompleted` se déclenche. Il y a une limite de 35 identificateurs par ressource Watcher. 

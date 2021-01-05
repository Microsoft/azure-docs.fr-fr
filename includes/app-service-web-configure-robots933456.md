---
title: Fichier Include
description: Fichier Include
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 03/02/2020
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 6fc1f4152b2e16e1597c018e5af6e0245b075c3b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78255821"
---
## <a name="robots933456-in-logs"></a>robots933456 dans les journaux

Le message suivant peut s'afficher dans les journaux du conteneur :

```
2019-04-08T14:07:56.641002476Z "-" - - [08/Apr/2019:14:07:56 +0000] "GET /robots933456.txt HTTP/1.1" 404 415 "-" "-"
```

Vous pouvez sans risque ignorer ce message. `/robots933456.txt` est un chemin d’URL factice utilisé par App Service pour vérifier si le conteneur est capable de traiter des requêtes. Une réponse 404 indique que le chemin d’accès n’existe pas, mais informe App Service que le conteneur est intègre et prêt à répondre aux requêtes.


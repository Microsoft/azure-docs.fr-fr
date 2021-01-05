---
author: Juliako
ms.service: media-services
ms.topic: include
ms.date: 11/13/2020
ms.author: juliako
ms.openlocfilehash: fbc77b960cac0db2d077345c74d64485bd7ad8bb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994489"
---
URL qui est utilisée pour notifier le client (à l’aide d’une requête POST) sur les événements suivants :

- Changement de l’état d’indexation : 
    - Propriétés :    
    
        |Nom|Description|
        |---|---|
        |id|L’ID de la vidéo|
        |state|État de la vidéo|  
    - Exemple : https:\//test.com/notifyme?projectName=MyProject&id=1234abcd&state=Processed
- Personne identifiée dans la vidéo :
  - Propriétés
    
      |Nom|Description|
      |---|---|
      |id| L’ID de la vidéo|
      |faceId|ID de visage qui apparaît dans l’index de la vidéo|
      |knownPersonId|ID de la personne qui est unique au sein d’un modèle de visage|
      |personName|Nom de la personne|
        
    - Exemple : https:\//test.com/notifyme?projectName=MyProject&id=1234abcd&faceid=12&knownPersonId=CCA84350-89B7-4262-861C-3CAC796542A5&personName=Inigo_Montoya 

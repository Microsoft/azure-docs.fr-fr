---
title: Guide de dépannage de la vidéo en flux continu | Microsoft Docs
description: Cet article fournit des suggestions sur la résolution de certains problèmes de streaming en direct dans Azure Media Services.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 97ad3048fccc4f8ed6baf0f1d5304f1c8d81c3f2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89269231"
---
# <a name="troubleshooting-guide-for-live-streaming"></a>Guide de dépannage de la vidéo en flux continu

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

Cet article fournit des suggestions sur la résolution de certains problèmes de streaming en direct.

## <a name="issues-related-to-on-premises-encoders"></a>Problèmes liés aux encodeurs locaux
Cette section fournit des suggestions sur la façon de résoudre les problèmes liés aux encodeurs locaux qui sont configurés pour envoyer un flux à débit binaire unique à des canaux AMS activés pour l’encodage live.

### <a name="problem-would-like-to-see-logs"></a>Problème : vous aimeriez voir les journaux d’activité
* **Problème potentiel**: impossible de trouver des journaux d’activité de l’encodeur qui pourraient aider à déboguer des problèmes.
  
  * **Telestream Wirecast** : les journaux d’activité se trouvent en général sous C:\Users\{nom_utilisateur}\AppData\Roaming\Wirecast\ 
  * **Elemental Live** : vous pouvez trouver des liens vers les journaux d’activité sur le portail de gestion. Cliquez sur **Statistiques**, puis **Journaux d’activité**. Dans la page **Fichiers journaux d’activité**, vous trouvez une liste des journaux d’activité pour tous les éléments LiveEvent. Sélectionnez celui qui correspond à votre session active. 
  * **Flash Media Live Encoder** : vous pouvez trouver le **Répertoire des journaux...** en accédant à l’onglet **Journal d’encodage**.

### <a name="problem-there-is-no-option-for-outputting-a-progressive-stream"></a>Problème : il n’existe aucune option pour générer un flux progressif
* **Problème potentiel**: l’encodeur utilisé n’effectue pas de désentrelacement automatique. 
  
    **Étapes de dépannage**: recherchez une option de désentrelacement dans l’interface de l’encodeur. Une fois le désentrelacement activé, revérifiez les paramètres de sortie progressive. 

### <a name="problem-tried-several-encoder-output-settings-and-still-unable-to-connect"></a>Problème : vous avez essayé plusieurs paramètres de sortie d’encodeur et la connexion échoue encore.
* **Problème potentiel**: le canal d’encodage Azure n’a pas été réinitialisé correctement. 
  
    **Étapes de dépannage** : vérifiez que l’encodeur ne transmet plus les données à AMS, arrêtez puis réinitialisez le canal. Une fois le canal redémarré, essayez de connecter votre encodeur avec les nouveaux paramètres. Si cela ne résout toujours pas le problème, essayez de créer un canal. Parfois, les canaux peuvent être endommagés après plusieurs tentatives infructueuses.  
* **Problème potentiel**: la taille de GOP ou les paramètres d’image clé ne sont pas optimaux. 
  
    **Étapes de dépannage**: la taille de GOP ou l’intervalle d’image clé recommandé(e) est de deux secondes. Certains encodeurs calculent ce paramètre en nombre d’images, tandis que d’autres utilisent des secondes. Par exemple : lors de la sortie de 30 i/s, la taille de GOP serait de 60 images, ce qui équivaut à 2 secondes.  
* **Problème potentiel**: des ports fermés bloquent le flux de données. 
  
    **Étapes de dépannage**: lors de la diffusion en flux continu via RTMP, vérifiez les paramètres de pare-feu et/ou de proxy pour confirmer que les ports sortants 1935 et 1936 sont ouverts. 

> [!NOTE]
> Si, après avoir suivi la procédure de dépannage, vous ne pouvez toujours pas diffuser en continu avec succès, envoyez un ticket de support en utilisant le portail Azure.
> 
> 

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]


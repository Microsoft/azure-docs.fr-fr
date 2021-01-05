---
title: Préchargement d’éléments multimédias sur un point de terminaison CDN Azure | Microsoft Docs
description: Découvrez comment précharger du contenu mis en cache sur un point de terminaison Azure Content Delivery Network. Cette fonctionnalité est disponible sur certaines versions du produit.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: 5ea3eba5-1335-413e-9af3-3918ce608a83
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/12/2018
ms.author: allensu
ms.openlocfilehash: 7fb414680b8ff64d057f9f68d7b97380baeaa551
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92778196"
---
# <a name="pre-load-assets-on-an-azure-cdn-endpoint"></a>Préchargement d’éléments multimédias sur un point de terminaison CDN Azure
[!INCLUDE [cdn-verizon-only](../../includes/cdn-verizon-only.md)]

Par défaut, les ressources sont mises en cache uniquement lorsqu’elles sont demandées. Dans la mesure où le contenu des serveurs Edge n’a pas encore été mis en cache et où vous devez transmettre la requête au serveur d’origine, la première requête de chaque région peut prendre plus de temps que les requêtes suivantes. Pour éviter cette latence du premier accès, préchargez vos ressources. En plus d’offrir une meilleure expérience client, le préchargement de vos ressources mises en cache peut également réduire le trafic réseau sur le serveur d’origine.

> [!NOTE]
> Le préchargement des ressources est utile lors de grands événements ou lorsque le contenu est disponible simultanément pour de nombreux utilisateurs, par exemple lors de la sortie d’un film ou d’une mise à jour logicielle.
> 
> 

Ce didacticiel vous guide tout au long du préchargement de contenu mis en cache sur tous les nœuds de périphérie CDN Azure.

## <a name="to-pre-load-assets"></a>Pour précharger des ressources
1. Dans le [portail Azure](https://portal.azure.com), recherchez le profil CDN qui contient le point de terminaison que vous souhaitez précharger. Le volet du profil s’ouvre.
    
2. Cliquez sur le point de terminaison dans la liste. Le volet du point de terminaison s’ouvre.
3. Dans le volet du point de terminaison CDN, sélectionnez **Charger** .
   
    ![Volet du point de terminaison CDN](./media/cdn-preload-endpoint/cdn-endpoint-blade.png)
   
    Le volet **Charger** s’ouvre.
   
    ![Volet de chargement CDN](./media/cdn-preload-endpoint/cdn-load-blade.png)
4. Sous **Chemin du contenu** , entrez le chemin complet de chaque ressource que vous souhaitez charger (par exemple, `/pictures/kitten.png`).
   
   > [!TIP]
   > Après avoir commencé à saisir du texte, d’autres zones de texte **Chemin du contenu** s’afficheront pour vous permettre de créer une liste de plusieurs ressources. Pour supprimer des ressources de la liste, sélectionnez le bouton avec les points de suspension (...), puis sélectionnez **Supprimer** .
   > 
   > Chaque chemin du contenu doit être une URL relative qui satisfait aux [expressions régulières](/dotnet/standard/base-types/regular-expression-language-quick-reference) suivantes :  
   > - Chargement d’un fichier unique : `^(?:\/[a-zA-Z0-9-_.%=\u0020]+)+$`  
   > - Chargement d’un fichier unique avec chaîne de requête : `^(?:\?[-_a-zA-Z0-9\/%:;=!,.\+'&\u0020]*)?$` 
   > 
   > Étant donné que chaque ressource doit avoir son propre chemin, il n’existe aucune fonctionnalité générique pour le préchargement des ressources.
   > 
   > 
   
    ![Bouton Charger](./media/cdn-preload-endpoint/cdn-load-paths.png)
5. Lorsque vous avez terminé d’entrer des chemins de contenu, sélectionnez **Charger** .
   

> [!NOTE]
> Vous êtes limité à 10 requêtes de chargement par minute par profil CDN et 50 chemins peuvent être traités simultanément. Chaque chemin a une longueur limitée à 1 024 caractères.
> 
> 

## <a name="see-also"></a>Voir aussi
* [Purger un point de terminaison CDN Azure](cdn-purge-endpoint.md)
* [Référence API REST du CDN Azure : précharger le contenu d’un point de terminaison](/rest/api/cdn/endpoints/loadcontent)
* [Référence API REST du CDN Azure : vider le contenu d’un point de terminaison](/rest/api/cdn/endpoints/purgecontent)
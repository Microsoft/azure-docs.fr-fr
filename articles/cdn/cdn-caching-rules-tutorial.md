---
title: 'Didacticiel : Définir des règles de mise en cache d’Azure CDN | Microsoft Docs'
description: Dans ce didacticiel, vous allez définir une règle de mise en cache Azure CDN globale et une règle de mise en cache personnalisée.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/20/2018
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: ae217352b83617600fd983a747d578f8f28e7ddd
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92779235"
---
# <a name="tutorial-set-azure-cdn-caching-rules"></a>Tutoriel : Définir les règles de mise en cache d’Azure CDN

> [!NOTE] 
> Les règles de mise en cache sont disponibles uniquement pour les profils **CDN Azure Standard fourni par Verizon** et **CDN Azure Standard fourni par Akamai** . Pour des profils **Azure CDN de Microsoft** , vous devez utiliser le [moteur de règles standard](cdn-standard-rules-engine-reference.md) Pour les profils **Azure CDN Premium de Verizon** , vous devez utiliser le [moteur de règles Premium Verizon](./cdn-verizon-premium-rules-engine.md) dans le portail **Gérer** pour une fonctionnalité similaire.
 

Ce didacticiel explique comment vous servir des règles de mise en cache du réseau de distribution de contenu (CDN) Azure pour définir ou modifier le comportement d’expiration du cache par défaut, globalement et avec des conditions personnalisées, telles qu’un chemin URL et une extension de fichier. Azure CDN fournit deux types de règles de mise en cache :
- Règles de mise en cache globales : vous pouvez définir une règle de mise en cache globale pour chaque point de terminaison dans votre profil, ce qui affecte toutes les requêtes au point de terminaison. La règle de mise en cache globale se substitue à tous les en-têtes à directive de cache HTTP, s’ils sont définis.

- Règles de mise en cache personnalisées : vous pouvez définir une ou plusieurs règles de mise en cache personnalisées pour chaque point de terminaison dans votre profil. Les règles de mise en cache personnalisées correspondent à des chemins et extensions de fichier spécifiques, elles sont traitées dans l’ordre et remplacent la règle de mise en cache globale, si elle est définie. 

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - ouvrir la page des règles de mise en cache ;
> - créer une règle de mise en cache générale ;
> - créer une règle de mise en cache personnalisée.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Avant d’effectuer les étapes de ce tutoriel, vous devez d’abord créer un profil CDN et au moins un point de terminaison CDN. Pour plus d’informations, consultez [Démarrage rapide : Créer un point de terminaison et un profil de réseau de distribution de contenu Azure](cdn-create-new-endpoint.md).

## <a name="open-the-azure-cdn-caching-rules-page"></a>Ouvrir la page des règles de mise en cache d’Azure CDN

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez un profil CDN, puis un point de terminaison.

2. Dans le volet gauche, sous Paramètres, sélectionnez **Règles de mise en cache** .

   ![Bouton Règles de mise en cache CDN](./media/cdn-caching-rules/cdn-caching-rules-btn.png)

   La page **Règles de mise en cache** s’affiche.

   ![Page des règles de mise en cache de CDN](./media/cdn-caching-rules/cdn-caching-rules-page.png)


## <a name="set-global-caching-rules"></a>Définir des règles de mise en cache générales

Créez une règle de mise en cache globale comme suit :

1. Sous **Règles de mise en cache générales** , définissez **Comportement de mise en cache des chaînes de requête** sur **Ignorer les chaînes de requête** .

2. Définissez **Comportement de mise en cache** sur **Définir en cas d’absence** .
       
3. Pour **Durée d’expiration du cache** , entrez 10 dans le champ **Jours** .

    La règle de mise en cache globale affecte toutes les requêtes au point de terminaison. Cette règle respecte les en-têtes à directive de cache d’origine, s’ils existent (`Cache-Control` ou `Expires`) ; s’ils ne sont pas spécifiés, elle définit le cache sur 10 jours. 

    ![Règles de mise en cache générales](./media/cdn-caching-rules/cdn-global-caching-rules.png)

## <a name="set-custom-caching-rules"></a>Définir des règles de mise en cache personnalisées

Créez une règle de mise en cache personnalisée comme suit :

1. Sous **Règles de mise en cache personnalisées** , définissez **Condition de correspondance** sur **Chemin** et **Valeur(s) de correspondance** sur `/images/*.jpg`.
    
2. Définissez **Comportement de mise en cache** sur **Remplacer** et entrez 30 dans le champ **Jours** .
       
    Cette règle de mise en cache personnalisée définit une durée de cache de 30 jours sur tous les fichiers image `.jpg` présents dans le dossier `/images` de votre point de terminaison. Elle se substitue à tout en-tête HTTP `Cache-Control` ou `Expires` qui sont envoyés par le serveur d’origine.

    ![Règles de mise en cache personnalisées](./media/cdn-caching-rules/cdn-custom-caching-rules.png)

    
## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans les étapes précédentes, vous avez créé des règles de mise en cache. Si vous ne souhaitez plus les utiliser, vous pouvez les supprimer en procédant comme suit :
 
1. Sélectionnez un profil CDN, puis le point de terminaison avec les règles de mise en cache que vous souhaitez supprimer.

2. Dans le volet gauche, sous Paramètres, sélectionnez **Règles de mise en cache** .

3. Sous **Règles de mise en cache générales** , définissez **Comportement de mise en cache** sur **Non défini** .
 
4. Sous **Règles de mise en cache personnalisées** , cochez la case en regard de la règle que vous souhaitez supprimer.

5. Sélectionnez **Supprimer** .

6. En haut de la page, sélectionnez **Enregistrer** .


## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> - ouvrir la page des règles de mise en cache ;
> - créer une règle de mise en cache générale ;
> - créer une règle de mise en cache personnalisée.

Passez à l’article suivant pour découvrir comment configurer d’autres paramètres de règles de mise en cache.

> [!div class="nextstepaction"]
> [Contrôler le comportement de mise en cache d’Azure CDN avec des règles de mise en cache](cdn-caching-rules.md)
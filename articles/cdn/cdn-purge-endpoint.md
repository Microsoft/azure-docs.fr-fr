---
title: Purger un point de terminaison CDN Azure | Microsoft Docs
description: Découvrez comment vider tout le contenu mis en cache à partir d’un point de terminaison Content Delivery Network (CDN) Azure. Les nœuds de périphérie mettent en cache les ressources jusqu’à expiration de leur durée de vie.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: sohamnchatterjee
ms.assetid: 0b50230b-fe82-4740-90aa-95d4dde8bd4f
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/17/2019
ms.author: allensu
ms.openlocfilehash: e90086be18e344e3061b9e683780f4427b3c15a1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018628"
---
# <a name="purge-an-azure-cdn-endpoint"></a>Purger un point de terminaison CDN Azure
## <a name="overview"></a>Vue d’ensemble
Les nœuds de périmètre CDN Azure mettent en cache les éléments multimédias jusqu’à l’expiration de leur durée de vie.  Une fois la durée de vie de l’élément multimédia expirée, lorsqu’un client demande l’élément multimédia à partir du nœud de périmètre, ce dernier récupère une nouvelle copie mise à jour de l’élément pour répondre à la demande du client et actualiser le cache.

La bonne pratique pour vous assurer que vos utilisateurs obtiennent toujours la dernière copie de vos éléments multimédia consiste à établir une version de vos ressources pour chaque mise à jour et à les publier en tant que nouvelles URL.  CDN récupère immédiatement les nouveaux éléments multimédia pour les demandes suivantes du client.  Il est parfois souhaitable de vider le contenu mis en cache sur tous les nœuds de périmètre et de tous les forcer à récupérer de nouveaux éléments multimédias mis à jour.  Ce besoin peut être dû à des mises à jour de votre application web, ou à la nécessité de mettre à jour rapidement les éléments multimédias qui contiennent des informations incorrectes.

> [!TIP]
> Notez que le vidage efface uniquement le contenu mis en cache sur les serveurs de périmètre CDN.  Les caches en aval, tels que les serveurs proxy et les caches de navigateurs locaux, peuvent toujours contenir une copie mise en cache du fichier.  Il est important de s’en rappeler au moment où vous définissez la durée de vie d’un fichier.  Vous pouvez forcer un client en aval à demander la dernière version de votre fichier en lui donnant un nom unique chaque fois que vous le mettez à jour ou en tirant parti de la [mise en cache de chaîne de requête](cdn-query-string.md).  
> 
> 

Ce didacticiel vous guide dans le processus de vidage des éléments multimédias de tous les nœuds d’un point de terminaison.

## <a name="walkthrough"></a>Procédure pas à pas
1. Dans le [portail Azure](https://portal.azure.com), recherchez le profil CDN qui contient le point de terminaison que vous souhaitez vider.
2. Dans le panneau du profil CDN, cliquez sur le bouton de vidage.
   
    ![Panneau du profil CDN](./media/cdn-purge-endpoint/cdn-profile-blade.png)
   
    Le panneau correspondant s’ouvre.
   
    ![Panneau de vidage CDN](./media/cdn-purge-endpoint/cdn-purge-blade.png)
3. Dans le panneau de vidage, sélectionnez l’adresse de service que vous souhaitez vider dans la liste déroulante des URL.
   
    ![Formulaire de vidage](./media/cdn-purge-endpoint/cdn-purge-form.png)
   
   > [!NOTE]
   > Vous pouvez également accéder au panneau de vidage en cliquant sur le bouton **Vider** dans le panneau des points de terminaison CDN.  Dans ce cas, le champ **URL** sera déjà renseigné avec l’adresse de service de ce point de terminaison spécifique.
   > 
   > 
4. Sélectionnez les éléments multimédias que vous souhaitez vider sur les nœuds de périmètre.  Si vous souhaitez effacer tous les éléments multimédias, cochez la case **Vider tout** .  Sinon, tapez le chemin de chaque élément multimédia que vous souhaitez vider dans la zone de texte **Chemin d’accès**. Les formats suivants sont pris en charge dans le chemin d’accès.
    1. **Vidage d’URL unique** : vidage d’un élément multimédia individuel en spécifiant l’URL complète, avec ou sans l’extension de fichier, par exemple,`/pictures/strasbourg.png` ; `/pictures/strasbourg`
    2. **Vidage de caractère générique** : l’astérisque (\*) peut être utilisé comme caractère générique. Videz tous les dossiers, sous-dossiers et fichiers dans un point de terminaison avec `/*` dans le chemin d’accès ou videz tous les sous-dossiers et fichiers dans un dossier spécifique en spécifiant le dossier suivi de `/*`, par exemple,`/pictures/*`.  Notez que le vidage de caractère générique n’est pas compatible avec Azure CDN par Akamai. 
    3. **Vidage du domaine racine** : videz la racine du point de terminaison avec « / » dans le chemin d’accès.
   
   > [!TIP]
   > Les chemins doivent être spécifiés pour le vidage et doivent être une URL relative qui satisfait à [l’expression régulière](/dotnet/standard/base-types/regular-expression-language-quick-reference) suivante. Le **vidage totale** et le **vidage de caractère générique** ne sont pas pris en charge par compatible avec **Azure CDN par Akamai**.
   > > Vidage d’URL unique `@"^\/(?>(?:[a-zA-Z0-9-_.%=\(\)\u0020]+\/?)*)$";`  
   > > Chaîne de requête `@"^(?:\?[-\@_a-zA-Z0-9\/%:;=!,.\+'&\(\)\u0020]*)?$";`  
   > > Vidage de caractère générique `@"^\/(?:[a-zA-Z0-9-_.%=\(\)\u0020]+\/)*\*$";`. 
   > 
   > Après avoir saisi du texte, d’autres zones de texte **Chemin d’accès** s’afficheront pour vous permettre de créer une liste de plusieurs éléments multimédias.  Vous pouvez supprimer des éléments multimédias de la liste en cliquant sur le bouton points de suspension (...).
   > 
5. Cliquez sur le bouton **Vider** .
   
    ![Bouton Vider](./media/cdn-purge-endpoint/cdn-purge-button.png)

> [!IMPORTANT]
> Le traitement des demandes de vidage prend environ 10 minutes avec le service **Azure CDN de Microsoft**, environ 2 minutes avec le service **Azure CDN de Verizon** (Standard et Premium) et environ 10 secondes avec le service **Azure CDN d’Akamai**.  Le CDN Azure impose une limite de 100 demandes de vidage simultanées à un moment donné au niveau du profil. 
> 
> 

## <a name="see-also"></a>Voir aussi
* [Préchargement d’éléments multimédias sur un point de terminaison CDN Azure](cdn-preload-endpoint.md)
* [Référence API REST du CDN Azure - Vider ou pré-charger un point de terminaison](/rest/api/cdn/endpoints)


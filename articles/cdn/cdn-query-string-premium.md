---
title: Contrôler le comportement de mise en cache du CDN Azure avec des chaînes de requête - niveau premium
description: La mise en cache des chaînes de requête du CDN Azure contrôle la manière dont les fichiers sont mis en cache lorsqu’une requête Web contient une chaîne de requête. Cet article décrit la chaîne de requête mise en cache dans les produits CDN Azure Premium fournis par Verizon.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: 99db4a85-4f5f-431f-ac3a-69e05518c997
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/11/2018
ms.author: allensu
ms.openlocfilehash: a799309b6e5d00db3b6c206187eec7097c9dc11a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018594"
---
# <a name="control-azure-cdn-caching-behavior-with-query-strings---premium-tier"></a>Contrôler le comportement de mise en cache du CDN Azure avec des chaînes de requête - niveau premium
> [!div class="op_single_selector"]
> * [Niveau standard](cdn-query-string.md)
> * [Niveau Premium](cdn-query-string-premium.md)
> 

## <a name="overview"></a>Vue d’ensemble
Avec Azure Content Delivery Network (CDN), vous pouvez contrôler la manière dont les fichiers sont mis en cache pour une requête web qui contient une chaîne de requête. Dans une requête web contenant une chaîne de requête, la chaîne de requête représente la partie de la demande qui apparaît après le point d’interrogation (?). Une chaîne de requête peut contenir une ou plusieurs paires clé-valeur où le nom du champ et sa valeur sont séparés par un signe égal (=). Chaque paire clé-valeur est séparée par une esperluette (&). Par exemple, http:\//www.contoso.com/content.mov?field1=value1&field2=value2. S’il existe plusieurs paires clé-valeur dans la chaîne de requête d’une demande, leur ordre n’a pas d’importance. 

> [!IMPORTANT]
> Les produits CDN standard et premium proposent les mêmes fonctionnalités de mise en cache des chaînes de requête, mais l’interface utilisateur est différente. Cet article décrit l’interface d’**Azure CDN Premium fourni par Verizon**. Pour la mise en cache des chaînes de requête avec des produits Azure CDN standard, consultez l’article [Contrôler le comportement de mise en cache du CDN Azure avec des chaînes de requête – niveau standard](cdn-query-string.md).
>


Trois modes de chaîne de requête sont disponibles :

- **cache standard** : mode par défaut. Dans ce mode, le nœud POP (point de présence) du CDN transmet les chaînes de requête, du demandeur au serveur d’origine de la première requête et met en cache la ressource. Toutes les requêtes suivantes pour la ressource, qui sont traitées à partir du serveur POP, ignorent les chaînes de requête jusqu’à l’arrivée à expiration de la ressource mise en cache.

    >[!IMPORTANT] 
    > Si l’autorisation de jeton est activée pour n’importe quel chemin d’accès sur ce compte, le mode de cache standard est le seul mode qui peut être utilisé. 

- **no-cache** : dans ce mode, les requêtes avec des chaînes de requête ne sont pas mises en cache au niveau du nœud POP CDN. Le nœud POP récupère la ressource directement à partir du serveur d’origine et le transmet au demandeur avec chaque requête.

- **cache unique** : dans ce mode, chaque demande contenant une URL unique, y compris la chaîne de requête, est traitée comme une ressource unique avec son propre cache. Par exemple, la réponse du serveur d’origine à une requête pour example.ashx?q=test1 est mise en cache au niveau du nœud POP et retournée pour les caches suivants avec la même chaîne de requête. Une requête pour example.ashx?q=test2 est mise en cache en tant que ressource distincte avec son propre paramètre de durée de vie.
   
    >[!IMPORTANT] 
    > N’utilisez pas ce mode lorsque la chaîne de requête contient des paramètres qui varient à chaque requête, par exemple un ID de session ou un nom d’utilisateur, car cela entraînerait un faible taux d’accès au cache.

## <a name="changing-query-string-caching-settings-for-premium-cdn-profiles"></a>Modification des paramètres de mise en cache des chaînes de requête pour les profils CDN Premium
1. Ouvrez un profil CDN, puis cliquez sur **Gérer**.
   
    ![Bouton Gérer du profil CDN](./media/cdn-query-string-premium/cdn-manage-btn.png)
   
    Le portail de gestion CDN s'ouvre.
2. Pointez sur l’onglet **HTTP volumineux**, puis pointez sur le menu volant **Paramètres de cache**. Cliquez sur **Mise en cache des chaînes de requête**.
   
    Les options de mise en cache des chaînes de requête s'affichent.
   
    ![Options de mise en cache des chaînes de requête CDN](./media/cdn-query-string-premium/cdn-query-string.png)
3. Sélectionnez un mode de chaîne de requête, puis cliquez sur **Mettre à jour**.

> [!IMPORTANT]
> La propagation de l’inscription dans CDN prenant un certain temps, la modification des paramètres des chaînes mises en cache peut ne pas être visible immédiatement. En général, la propagation dure 10 minutes.
 


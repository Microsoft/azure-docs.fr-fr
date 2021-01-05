---
title: Contrôler le comportement de mise en cache du CDN Azure avec des chaînes de requête - niveau standard
description: La mise en cache des chaînes de requête du CDN Azure contrôle la manière dont les fichiers sont mis en cache lorsqu’une requête Web contient une chaîne de requête. Cet article décrit la chaîne de requête mise en cache dans les produits standard du CDN Azure.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: 17410e4f-130e-489c-834e-7ca6d6f9778d
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/11/2018
ms.author: allensu
ms.openlocfilehash: 1521d08ef9d431bbe8b3fd3a578297d440ed56b3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018577"
---
# <a name="control-azure-cdn-caching-behavior-with-query-strings---standard-tier"></a>Contrôler le comportement de mise en cache du CDN Azure avec des chaînes de requête - niveau standard
> [!div class="op_single_selector"]
> * [Niveau standard](cdn-query-string.md)
> * [Niveau Premium](cdn-query-string-premium.md)
> 

## <a name="overview"></a>Vue d’ensemble
Avec Azure Content Delivery Network (CDN), vous pouvez contrôler la manière dont les fichiers sont mis en cache pour une requête web qui contient une chaîne de requête. Dans une requête web contenant une chaîne de requête, la chaîne de requête représente la partie de la demande qui apparaît après le point d’interrogation (?). Une chaîne de requête peut contenir une ou plusieurs paires clé-valeur où le nom du champ et sa valeur sont séparés par un signe égal (=). Chaque paire clé-valeur est séparée par une esperluette (&). Par exemple, http:\//www.contoso.com/content.mov?field1=value1&field2=value2. S’il existe plusieurs paires clé-valeur dans la chaîne de requête d’une demande, leur ordre n’a pas d’importance. 

> [!IMPORTANT]
> Les produits CDN standard et premium Azure proposent la même fonctionnalité de mise en cache des chaînes de requête, mais l’interface utilisateur est différente. Cet article décrit l’interface **CDN Azure Standard fourni par Microsoft**, **CDN Azure Standard fourni par Akamai** et **CDN Azure Standard fourni par Verizon**. Pour la mise en cache des chaînes de requête avec le **CDN Azure Premium fourni par Verizon**, consultez [Contrôler le comportement de mise en cache avec des chaînes de requête – niveau Premium](cdn-query-string-premium.md).

Trois modes de chaîne de requête sont disponibles :

- **Ignorer les chaînes de requête** : mode par défaut. Dans ce mode, le nœud POP (point de présence) du CDN transmet les chaînes de requête, du demandeur au serveur d’origine de la première requête et met en cache la ressource. Toutes les requêtes ultérieures pour la ressource, qui sont traitées à partir du nœud POP, ignorent les chaînes de requête jusqu’à l’arrivée à expiration de la ressource mise en cache.

- **Contourner la mise en cache des chaînes de requête** : dans ce mode, les requêtes avec des chaînes de requête ne sont pas mises en cache au niveau du nœud POP du CDN. Le nœud POP récupère la ressource directement à partir du serveur d’origine et le transmet au demandeur avec chaque requête.

- **Mettre en cache chaque URL unique** : dans ce mode, chaque demande contenant une URL unique, y compris la chaîne de requête, est traitée comme une ressource unique avec son propre cache. Par exemple, la réponse du serveur d’origine à une requête pour example.ashx?q=test1 est mise en cache au niveau du nœud POP et retournée pour les caches suivants avec la même chaîne de requête. Une requête pour example.ashx?q=test2 est mise en cache en tant que ressource distincte avec son propre paramètre de durée de vie.
   
    >[!IMPORTANT] 
    > N’utilisez pas ce mode lorsque la chaîne de requête contient des paramètres qui varient à chaque requête, par exemple un ID de session ou un nom d’utilisateur, car cela entraînerait un faible taux d’accès au cache.

## <a name="changing-query-string-caching-settings-for-standard-cdn-profiles"></a>Modification des paramètres de mise en cache des chaînes de requête pour les profils CDN Standard
1. Ouvrez un profil CDN, puis sélectionnez le point de terminaison CDN que vous souhaitez gérer.
   
   ![Points de terminaison du profil CDN](./media/cdn-query-string/cdn-endpoints.png)
   
2. Dans le volet gauche, sous Paramètres, cliquez sur **Règles de mise en cache**.
   
    ![Bouton Règles de mise en cache CDN](./media/cdn-query-string/cdn-caching-rules-btn.png)
   
3. Dans la liste **Comportement de mise en cache des chaînes de requête**, sélectionnez un mode de chaîne de requête, puis cliquez sur **Enregistrer**.
   
   ![Options de mise en cache des chaînes de requête CDN](./media/cdn-query-string/cdn-query-string.png)

> [!IMPORTANT]
> La propagation de l’inscription dans le CDN Azure prenant un certain temps, la modification des paramètres des chaînes mises en cache peut ne pas être visible immédiatement :
> - Pour les profils du **CDN Azure Standard fourni par Microsoft**, la propagation s’effectue généralement dans un délai de 10 minutes. 
> - Pour les profils du **CDN Azure Standard fourni par Akamai**, la propagation s’effectue généralement dans un délai d’une minute. 
> - Dans le cas des profils **Azure CDN Standard fourni par Verizon** et **Azure CDN Premium fourni par Verizon**, la propagation s’effectue généralement dans un délai de 10 minutes. 




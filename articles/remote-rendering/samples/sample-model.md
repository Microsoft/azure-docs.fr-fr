---
title: Exemples de modèles
description: Liste les sources d’exemples de modèles.
author: florianborn71
ms.author: flborn
ms.date: 01/29/2020
ms.topic: sample
ms.openlocfilehash: 8e5dcb6c9dfa08efc0889fcab779d6cb333d2330
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88507533"
---
# <a name="sample-models"></a>Exemples de modèles

Cet article liste certaines ressources d’exemples de données qui peuvent être utilisés pour tester le service Azure Remote Rendering.

## <a name="built-in-sample-model"></a>Exemple de modèle intégré

Nous vous proposons un exemple de modèle intégré qui peut toujours être chargé à l’aide de l’URL **builtin://Engine**.

![Exemple de modèle](./media/sample-model.png "Exemple de modèle")

Statistiques du modèle :

| Nom | Valeur |
|-----------|:-----------|
| [Taille de serveur requise](../how-tos/session-rest-api.md#create-a-session) | standard |
| Nombre de triangles | 18,7 millions |
| Nombre de pièces mobiles | 2 073 |
| Nombre de matériaux | 94 |

## <a name="third-party-data"></a>Données tierces

Le groupe Khronos gère un ensemble d’exemples de modèles glTF à des fins de test. ARR prend en charge le format glTF pour le format texte ( *.gltf*) et pour le format binaire ( *.glb*). Nous vous suggérons d’utiliser les modèles PBR pour obtenir de meilleurs résultats visuels :

* [Exemples de modèles glTF](https://github.com/KhronosGroup/glTF-Sample-Models)

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Afficher un modèle avec Unity](../quickstarts/render-model.md)
* [Démarrage rapide : Convertir un modèle pour le rendu](../quickstarts/convert-model.md)

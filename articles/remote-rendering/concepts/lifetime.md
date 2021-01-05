---
title: Durée de vie des objets et ressources
description: Explique la gestion de la durée de vie pour différents types
author: jakrams
ms.author: jakras
ms.date: 02/06/2020
ms.topic: conceptual
ms.openlocfilehash: d031ff4a6ee86da2843f0f18ac428c50f7cfc121
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80679407"
---
# <a name="object-and-resource-lifetime"></a>Durée de vie des objets et ressources

Azure Remote Rendering fait la distinction entre deux types : les **objets** et les **ressources**.

## <a name="object-lifetime"></a>Durée de vie d’objet

*Les objets* sont considérés comme des éléments que l’utilisateur peut créer, modifier et détruire à sa guise. Ils peuvent être dupliqués librement et chaque instance peut muter au fil du temps. Par conséquent les [entités](entities.md) et les [composants](components.md) sont des objets.

La durée de vie des objets est entièrement sous contrôle de l’utilisateur. Toutefois, elle n’est pas liée à la durée de vie de la représentation côté client. Des classes comme `Entity` et `Component` ont une fonction `Destroy` qui doit être appelée pour libérer l’objet sur l’hôte de rendu à distance. En outre, `Entity.Destroy()` détruit l’entité, ses enfants et tous les composants dans cette hiérarchie.

## <a name="resource-lifetime"></a>Durée de vie des ressources

Les *ressources* sont des éléments dont la durée de vie est entièrement gérée par l’hôte de rendu à distance. Les ressources font l’objet d’un décompte de références en interne. Elles sont libérées quand plus personne ne les référence.

Il n’est possible de créer la plupart des ressources qu’indirectement, généralement en les chargeant à partir d’un fichier. Quand le même fichier est chargé plusieurs fois, Azure Remote Rendering retourne la même référence au lieu de recharger les données.

De nombreuses ressources sont immuables, par exemple, les [maillages](meshes.md) et les [textures](textures.md). Toutefois, certaines ressources sont mutables, par exemple, les [matériaux](materials.md). Les ressources étant souvent partagées, la modification de l’une d’elles peut affecter plusieurs objets. Par exemple, la modification de la couleur d’un matériau a pour effet de modifier la couleur de tous les objets utilisant des maillages référençant ce matériau.

### <a name="built-in-resources"></a>Ressources intégrées

Azure Remote Rendering intègre des ressources qu’il est possible de charger en préfixant leur identificateur avec `builtin://` lors de l’appel de la commande `AzureSession.Actions.LoadXYZAsync()`. Les ressources intégrées disponibles sont répertoriées dans la documentation de chaque fonctionnalité. Par exemple, le chapitre [Ciel](../overview/features/sky.md) répertorie les textures de ciel intégrées.

## <a name="general-lifetime"></a>Durée de vie générale

La durée de vie de tous les objets et ressources est liée à la connexion. Lors de la déconnexion, tous les éléments sont écartés. Lors de la reconnexion à la même session, le graphe de scène est vide et toutes les ressources sont purgées.

Dans la pratique, le chargement de la même ressource dans une session après une déconnexion est généralement plus rapide que la première fois. Cela résulte du fait que la plupart des ressources doivent être téléchargées à partir du service Stockage Azure la première fois, mais pas la deuxième fois, ce qui constitue un gain de temps considérable.

## <a name="next-steps"></a>Étapes suivantes

* [Entités](entities.md)
* [Composants](components.md)
* [Modèles](models.md)

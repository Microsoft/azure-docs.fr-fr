---
title: Disposition des fichiers pour la conversion
description: Recommandations sur la meilleure façon de placer les fichiers dans le conteneur d’entrée.
author: MalcolmTyrrell
ms.author: matyrr
ms.date: 09/03/2020
ms.topic: how-to
ms.openlocfilehash: 13c4b841fe2d0b62472628d9382b5f6ee3d1fa6c
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91318073"
---
# <a name="laying-out-files-for-conversion"></a>Disposition des fichiers pour la conversion

Pour pouvoir traiter correctement un une ressource, le service de conversion doit être en mesure de trouver tous les fichiers d’entrée.
Il s’agit du fichier de ressource principal en cours de conversion et généralement d’autres fichiers référencés par des chemins d’accès à l’intérieur du fichier de ressource.
La demande de conversion d’une ressource reçoit deux paramètres qui déterminent la façon dont le service de conversion trouve ces fichiers, à savoir : le chemin d’accès `input.folderPath` (facultatif) et le chemin d’accès `input.inputAssetPath`.
Tout cela est entièrement documenté dans la page [API REST Conversion](conversion-rest-api.md).
En ce qui concerne la disposition des fichiers, il est important de noter que le chemin d’accès `folderPath` détermine l’ensemble complet des fichiers qui sont à la disposition du service de conversion lors du traitement de la ressource.

## <a name="placing-files-so-they-can-be-found"></a>Placement des fichiers pour faciliter leur localisation

Quand une ressource source utilise des fichiers externes, les chemins d’accès à ceux-ci sont stockés dans la ressource.
Le service de conversion doit interpréter ces chemins d’accès dans un système de fichiers différent du système de fichiers d’origine de la ressource.
Si les chemins d’accès sont stockés sous forme de chemins d’accès relatifs, et que l’emplacement relatif de la ressource source et du fichier auquel elle fait référence est inchangé, le service de conversion peut facilement trouver le fichier référencé.

> [!Note]
> Nous vous recommandons de placer les fichiers dans le conteneur d’entrée de façon à ce que les emplacements relatifs des fichiers soient les mêmes que lors de la création de la ressource.

> [!Note]
> Préférez créer des ressources contenant des chemins d’accès relatifs.
> Le tutoriel sur la [configuration des ressources pour 3ds Max](../../tutorials/modeling/3dsmax-material-setup.md) fournit un exemple 3ds Max illustrant la manière de veiller à ce qu’une ressource utilise des chemins d’accès relatifs.

## <a name="finding-textures"></a>Recherche de textures

En raison des nombreuses façons de générer des ressources, le service de conversion doit être flexible.
En particulier, il doit gérer les situations dans lesquelles les chemins figurant dans la ressource et l’emplacement des textures ne correspondent pas précisément.
C’est le cas, par exemple, lors de la génération de ressources contenant des chemins absolus, car ceux-ci ne correspondent jamais au système de fichiers qu’utilise le service de conversion.
Pour gérer cette situation particulière, nous utilisons une approche optimale pour la recherche de textures.

L’algorithme utilisé pour localiser les textures est le suivant : À partir d’un chemin d’accès stocké dans une ressource, recherchez le suffixe de sous-chemin d’accès le plus long qui, utilisé comme chemin d’accès relatif à partir de l’emplacement de la ressource source, cible un fichier existant.
Si aucun sous-chemin d’accès (y compris le chemin d’accès complet) ne cible un fichier, la texture est considérée comme manquante.

Prenons le système de fichiers fictif suivant : 
```
G:\CONVERSION
├───Assets
│   │   myAsset.fbx
│   │   myTexture.png            <- A
│   │
│   └───Textures
│       │   myTexture.png        <- B
│       │
│       └───MyAssetTextures
│               myTexture.png    <- C
│
└───Textures
    │   myTexture.png            <- D
    │
    └───MyAssetTextures
            myTexture.png        <- E
```
Si le fichier myAsset.fbx fait référence à une texture dont le chemin d’accès relatif est `..\Textures\MyAssetTextures\myTexture.png`, le service de conversion utilise le fichier E. En fait, il peut utiliser n’importe lequel des fichiers A, C ou E s’ils existent, le fichier E étant privilégié parce que son suffixe est le plus long.
Les fichiers B et D ne sont jamais utilisés, car `Textures\myTexture.png` ne fait partie d’aucun suffixe du chemin d’accès stocké.
Si la ressource contient à la place les chemins d’accès `H:\Foo\Bar\Textures\MyAssetTextures\myTexture.png` ou `..\..\..\Foo\Bar\Textures\MyAssetTextures\myTexture.png`, le service de conversion est en mesure de trouver les fichiers A et C s’ils existent (en préférant C à A). En revanche, le fichier E est introuvable de cette façon et doit être déplacé.
Vous pouvez résoudre ce problème en plaçant le dossier Textures à côté de la ressource.

> [!Note]
> Si aucune texture n’est trouvée, une solution possible consiste à s’assurer que la ressource est sœur d’une sous-arborescence contenant les textures.

## <a name="next-steps"></a>Étapes suivantes

- [Conversion de modèle](model-conversion.md)

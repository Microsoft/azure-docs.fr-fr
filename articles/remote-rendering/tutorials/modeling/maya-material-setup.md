---
title: Configurer des matériaux PBR dans Maya
description: Explique comment configurer des matériaux au rendu physique réaliste (matériaux PBR) dans Maya et comment les exporter au format FBX.
author: FlorianBorn71
ms.author: flborn
ms.date: 06/16/2020
ms.topic: tutorial
ms.openlocfilehash: 56aa0d91372ac2d21a20f28b1044f0811c716b0c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91358030"
---
# <a name="tutorial-set-up-physically-based-rendering-materials-in-maya"></a>Tutoriel : Configurer des matériaux PBR dans Maya

## <a name="overview"></a>Vue d’ensemble
Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Affecter des matériaux dotés d’un éclairage avancé à des objets dans la scène.
> * Gérer l’instanciation des objets et des matériaux.
> * Exporter une scène au format FBX et sélectionner les options importantes.

La création de [matériaux au rendu physique réaliste (matériaux PBR)](../../overview/features/pbr-materials.md) dans Maya est une tâche relativement simple. Elle est semblable en de nombreux points à la configuration du rendu PBR dans d’autres applications de création de contenu comme 3DS Max. Ce tutoriel explique les étapes de base de la configuration du nuanceur PBR et de l’exportation au format FBX dans le cadre de projets Azure Remote Rendering. 

L’exemple de scène de ce tutoriel contient plusieurs boîtes polygonales. Différents matériaux leur sont attribués, tels que du bois, du métal, du métal peint, du plastique et du caoutchouc. Globalement, chaque matériau comporte la totalité ou la plupart des textures suivantes :

* **Albedo**, qui est la table des couleurs du matériau, également appelée **Diffuse** ou **BaseColor**.
* **Metalness** (Aspect métallique), qui détermine si un matériau est métallique et quelles parties sont métalliques. 
* **Roughness** (Rugosité), qui détermine dans quelle mesure la surface est rugueuse ou lisse, ce qui rend les réflexions et les surbrillances sur une surface plus ou moins nettes ou floues.
* **Normal**, qui ajoute des détails à une surface sans imposer l’ajout de polygones. Des exemples de tels détails sont des piqûres et des bosses sur une surface métallique ou un grain de bois.
* **Ambient Occlusion** (Occlusion ambiante), qui ajoute un léger ombrage et des ombres de contact à un modèle. Il s’agit d’une carte de nuances de gris qui détermine quelles zones sont totalement éclairées (blanches) ou ombrées (noires). 

## <a name="prerequisites"></a>Prérequis
* Autodesk Maya 2017 ou version ultérieure

## <a name="set-up-materials-in-the-scene"></a>Configurer des matériaux dans la scène
Voici comment configurer un matériau PBR dans Maya.

Comme vous le verrez dans l’exemple de scène, nous avons créé plusieurs objets de boîte. Chaque objet représente un type de matériau différent. Comme illustré dans l’image, chacun de ces objets a un nom approprié qui lui est propre.

Azure Remote Rendering utilise le mètre comme unité de mesure et l’axe Y correspond à la direction vers le haut. Avant de commencer à créer des ressources, nous vous recommandons de définir le mètre comme unité de scène dans Maya. Pour l’exportation, définissez le mètre comme unité dans les paramètres d’exportation FBX.

> [!TIP]
> Donnez des noms appropriés à vos ressources de modèle en fonction de la pièce ou du type de matériau en question. Choisir des noms pertinents facilite la navigation dans les scènes contenant beaucoup d’objets.

![les noms d'objets ;](media/object-names.jpg)

Une fois que vous avez créé ou acquis des textures, vous pouvez également créer des textures uniques. Vous pouvez utiliser des applications de texturing, comme Quixel Suite, PhotoShop ou Substance Suite, ou bien obtenir des textures de mosaïque génériques à partir d’autres sources.

Pour appliquer des textures à votre modèle :

1. Dans la fenêtre d’affichage (le viewport) de votre scène, sélectionnez le modèle ou la géométrie, puis cliquez dessus avec le bouton droit. Dans le menu qui s’affiche, sélectionnez **Assign New Material** (Affecter un nouveau matériau).
1. Dans la boîte de dialogue **Assign New Material** (Affecter un nouveau matériau), accédez à **Maya** > **Stingray PBS**. Cette action affecte un matériau PBR à votre modèle. 

Dans Maya 2020, plusieurs nuanceurs PBR sont disponibles. Ils incluent **Maya Standard Surface**, **Arnold Standard Surface** et **Stingray PBR**. Le nuanceur **Maya Standard Surface Shader** n’est pas encore exportable via le plug-in FBX 2020. Le nuanceur **Arnold Standard Surface Shader** peut être exporté avec des fichiers FBX. Sur la plupart des autres aspects, il est identique au nuanceur **Maya Standard Surface Shader**. Il est analogue à **Physical Material** dans 3D Studio Max.

Le nuanceur **Stingray PBR Shader** est compatible avec de nombreuses autres applications et satisfait au mieux les exigences d’Azure Remote Rendering. Il est pris en charge depuis Maya 2017. Lorsque ce type de matériau est visualisé dans la fenêtre d’affichage, il est semblable à ce qui est visualisé ultérieurement dans Azure Remote Rendering.

![Matériau Stingray](media/stingray-material.jpg)

Maintenant que vous avez affecté le matériau à votre ressource et que vous l’avez nommé de manière appropriée, vous pouvez affecter vos différentes textures. Les images suivantes montrent à quoi correspond chaque type de texture sur le matériau PBR. Le matériau PBR Stingray vous permet de sélectionner les attributs que vous pouvez activer. Avant d’insérer vos cartes de texture, vous devez activer les attributs appropriés.

![Configuration des matériaux](media/material-setup.jpg)

Nommez vos matériaux de manière pertinente en prenant en compte leur usage ou leur type. Un matériau utilisé sur une pièce unique peut être nommé en fonction de cette pièce. Un matériau utilisé sur un plus grand nombre de zones peut être nommé en fonction de ses propriétés ou de son type.

Affectez vos textures comme indiqué dans l’image.

![Configuration des textures](media/texture-setup.jpg)

Une fois que vous avez créé et configuré vos matériaux PBR, envisagez d’[instancier les objets](../../how-tos/conversion/configure-model-conversion.md#instancing) dans votre scène. L’instanciation d’objets similaires dans votre scène, tels que des écrous, des boulons, des vis et des rondelles, entraîne une économie substantielle de taille de fichier. Les instances d’un objet principal peuvent avoir leurs propres transformations, rotation et échelle, ce qui vous permet de les placer où vous le souhaitez dans votre scène. 

Dans Maya, le processus d’instanciation est simple.

1. Dans le menu **Edit** (Edition), accédez à **Duplicate Special** (Duplication spéciale) pour ouvrir les options.
1. Dans la boîte de dialogue **Duplicate Special Options** (Options de duplication spéciale), pour **Geometry type** (Type de géométrie), sélectionnez l’option **Instance**. 
1. Sélectionnez **Duplicate Special** (Duplication spéciale).

   ![Capture d’écran montrant une fenêtre Maya avec la boîte de dialogue Options de duplication spéciale ouverte et Duplication spéciale sélectionnée.](media/instancing.jpg)

Cette action crée une instance de votre objet. Vous pouvez la déplacer, la faire pivoter ou la mettre à l’échelle indépendamment de son parent et des autres instances de ce parent. 

Toutes les modifications que vous apportez à une instance en mode composant sont transmises à toutes les instances de votre objet. Par exemple, vous pouvez utiliser les composants d’un objet instancié, tels que les sommets et les faces de polygone. Veillez à ce que toutes les modifications que vous avez apportées affectent toutes les instances. 

Dans l’exemple de scène, chaque boîte individuelle a été instanciée. Nous verrons en quoi cela est utile quand nous exporterons la scène au format FBX.

![Vue d’ensemble de la scène](media/scene-overview.jpg)

> [!TIP]
> Créez des instances dans votre scène au fur et à mesure. Le remplacement ultérieur de copies par des objets instanciés est extrêmement difficile. 

## <a name="fbx-export-process"></a>Processus d’exportation au format FBX

Procédons maintenant à l’exportation au format FBX de votre scène ou de ses ressources. Lorsque vous exportez des ressources, il est judicieux de sélectionner uniquement les objets ou les ressources de votre scène que vous souhaitez exporter. Par exemple, vous pouvez avoir 100 objets dans une scène. Si vous souhaitez utiliser uniquement 30 d’entre eux, il n’est pas judicieux d’exporter la scène entière. 

Pour effectuer votre sélection :

1. Accédez à **File** (Fichier) > **Export Selection** (Exporter la sélection) pour ouvrir la boîte de dialogue **Export Selection** (Exporter la sélection).
1. Dans la zone **Files of type** (Fichiers de type), sélectionnez **FBX export** (Exportation FBX) pour afficher les paramètres d’exportation FBX. Les principaux paramètres d’exportation au format FBX sont encadrés en rouge dans l’image.

   ![Exportation au format FBX](media/FBX-exporting.jpg)

Selon vos besoins, par exemple, vous pouvez envoyer une ressource à un client. Vous ne souhaitez peut-être pas envoyer un grand nombre de fichiers de texture avec la ressource. Vous pouvez choisir d’incorporer les textures dans le fichier FBX exporté. Cette option signifie que vous avez un seul fichier à inclure dans le package, mais la taille de cette ressource FBX augmente considérablement. Pour activer l’incorporation des textures, sélectionnez l’option **Embed Media** (Incorporer des médias) comme illustré ci-dessous.

> [!TIP]
> Dans cet exemple, le fichier a été nommé pour refléter cette condition. Ce style de nommage est pratique pour effectuer le suivi de vos ressources. 

Une fois que vous avez défini votre configuration pour l’exportation, sélectionnez **Export Selection** (Exporter la sélection) dans le coin inférieur droit.

![Incorporation de ressource](media/embedding-media.jpg)

## <a name="conclusion"></a>Conclusion

En général, ce type de matériau semble plus réaliste, car il est basé sur les propriétés physiques réelles de la lumière. Il ajoute un effet immersif qui donne un aspect réel à la scène.

## <a name="next-steps"></a>Étapes suivantes

Vous savez maintenant configurer des matériaux avec un éclairage avancé pour les objets d’une scène. Vous savez également exporter les objets au format FBX pris en charge par Azure Remote Rendering. L’étape suivante consiste à convertir le fichier FBX et à le visualiser dans Azure Remote Rendering.

> [!div class="nextstepaction"]
> [Démarrage rapide : Convertir un modèle pour le rendu](../../quickstarts\convert-model.md)
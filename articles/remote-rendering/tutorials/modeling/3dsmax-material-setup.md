---
title: Configurer des matériaux au rendu physique réaliste dans 3ds Max
description: Explique comment configurer des matériaux au rendu physique réaliste (matériaux PBR) dans 3ds Max et comment les exporter au format FBX.
author: FlorianBorn71
ms.author: flborn
ms.date: 06/16/2020
ms.topic: tutorial
ms.openlocfilehash: 12407d6344c69c747230e9db6fa4d53b4520dc82
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020277"
---
# <a name="tutorial-set-up-physically-based-rendering-materials-in-3ds-max"></a>Tutoriel : Configurer des matériaux PBR dans 3ds Max

## <a name="overview"></a>Vue d’ensemble
Ce didacticiel vous montre comment effectuer les opérations suivantes :

>[!div class="checklist"]
>
> * Attribuer des matériaux dotés d’un éclairage avancé à des objets dans une scène.
> * Gérer l’instanciation des objets et des matériaux.
> * Exporter une scène au format FBX et sélectionner les options importantes.

> [!Note]
> La procédure décrite dans ce tutoriel fonctionne dans 3ds Max 2019 et 3ds Max 2020.
> Un changement dans la façon dont 3ds Max 2021 exporte les placages de texture fait que les placages normaux ne seront pas trouvés par le service de conversion si cette version est utilisée.

La création de [matériaux au rendu physique réaliste (matériaux PBR)](../../overview/features/pbr-materials.md) dans 3ds Max est une tâche simple. Elle est semblable en de nombreux points à la configuration du rendu PBR dans d’autres applications de création de contenu comme Maya. Ce tutoriel explique les étapes de base de la configuration du nuanceur PBR et de l’exportation au format FBX dans le cadre de projets Azure Remote Rendering.

L’exemple de scène de ce tutoriel contient plusieurs boîtes polygonales. Différents matériaux leur sont attribués, tels que du bois, du métal, du métal peint, du plastique et du caoutchouc. Globalement, chaque matériau comporte la totalité ou la plupart des textures suivantes :

* **Albedo**, qui est la table des couleurs du matériau, également appelée **Diffuse** et **BaseColor**.
* **Metalness** (Aspect métallique), qui détermine si un matériau est métallique et quelles parties sont métalliques. 
* **Roughness** (Rugosité), qui détermine dans quelle mesure une surface est rugueuse ou lisse.
Cela rend également les réflexions et les surbrillances de la surface plus ou moins nettes ou floues.
* **Normal**, qui ajoute des détails à une surface sans ajout de polygones. Des exemples de tels détails sont des piqûres et des bosses sur une surface métallique ou un grain de bois.
* **Ambient Occlusion** (Occlusion ambiante), qui ajoute un léger ombrage et des ombres de contact à un modèle. Il s’agit d’une carte de nuances de gris qui détermine quelles zones sont totalement éclairées (blanches) ou ombrées (noires).

## <a name="prepare-the-scene"></a>Préparer la scène
Dans 3ds Max, le processus de configuration d’un matériau PBR est le suivant.

Pour commencer, vous allez créer un certain nombre d’objets Boîte, chacun représentant un type de matériau différent.

>[!TIP]
>Avant de commencer à créer des ressources pour Remote Rendering, il est important de noter que l’unité de mesure que le logiciel utilise est le mètre.  
>
>Il est donc judicieux de définir les unités système de votre scène sur cette même unité de mesure. Il est également judicieux de définir les **unités** sur des mètres dans les paramètres d’exportation au format FBX lorsque vous exportez une scène.

La capture d’écran suivante montre les étapes permettant de définir le mètre comme unité système dans 3ds max. 

1. Dans le menu principal, accédez à **Customize (Personnaliser)**  > **Units Setup (Configuration des unités)**  > **System Units Setup (Configuration des unités système)** . Dans **System Unit Scale** (Échelle de l’unité système), sélectionnez **Meters**  (Mètres) : ![Capture d’écran montrant comment définir des unités système.](media/3dsmax/system-units.jpg)

1. Vous pouvez maintenant commencer à créer les modèles. Dans l’exemple de scène, vous allez créer plusieurs objets Box, chacun représentant un type de matériau différent. Par exemple, métal, caoutchouc et plastique. 

   >[!TIP]
   >Lorsque vous créez des ressources, il est recommandé de les nommer de façon appropriée. Cela vous aidera à les retrouver facilement plus tard si la scène comporte un grand nombre d’objets.

1. Renommez les objets, comme indiqué dans la capture d'écran suivante : 

   ![Capture d’écran montrant comment renommer des objets.](media/3dsmax/rename-objects.jpg)

## <a name="assign-materials"></a>Affecter des matériaux

Maintenant que vous avez créé quelques objets dans votre scène (dans ce cas, plusieurs cubes), vous pouvez commencer la configuration du rendu PBR :

1. Dans la barre d’outils principale, sélectionnez l’icône **Material Editor** (Éditeur de matériaux), comme illustré dans la capture d’écran suivante. Vous pouvez également sélectionner la touche **M** du clavier pour ouvrir l’éditeur. L’éditeur de matériaux comporte deux modes que vous pouvez sélectionner dans la liste **Modes** : **Compact Material Editor (Éditeur de matériau compact)** et **Slate Material Editor (Éditeur de matériau ardoise)** . Comme cette scène est relativement simple, nous allons utiliser le mode compact.

1. Dans l’éditeur de matériaux, un certain nombre de sphères s’affichent. Ces sphères correspondent aux matériaux. Vous allez attribuer l’un de ces matériaux à chaque objet (boîte) dans la scène. Pour attribuer les matériaux, commencez par sélectionner l’un des objets dans la fenêtre d’affichage principale. Sélectionnez ensuite la première sphère dans l’éditeur de matériaux. Une fois qu’il est attribué à un objet, le matériau sélectionné est mis en évidence comme dans l’image suivante.

1. Sélectionnez **Assign Material to Selection** (Attribuer le matériau à la sélection) comme illustré. Le matériau est maintenant attribué à l’objet sélectionné.

   ![Capture d’écran montrant comment attribuer des matériaux.](media/3dsmax/assign-material.jpg)

    Dans l’éditeur de matériaux, vous pouvez sélectionner des types de matériaux parmi une vaste sélection, en fonction de vos besoins. En règle générale, le type de matériau est défini sur **Standard** par défaut. Ce matériau est un matériau basique qui ne convient pas à la configuration du rendu PBR. Vous devez donc changer le type de matériau en matériau PBR. Le matériau physique est le matériau 3ds Max préféré pour les projets Azure Remote Rendering.

1. Dans l’éditeur de matériaux, sélectionnez l’onglet **Standard**. Dans **Material/Map Browser** (Explorateur de matériaux/cartes), sélectionnez **Physical Material** (Matériau physique). Cette action permet de convertir le matériau **Standard** attribué en matériau physique PBR.

   ![Capture d’écran montrant comment modifier le matériau.](media/3dsmax/physical-material.jpg)

    Dans l’éditeur de matériaux, vous voyez maintenant les propriétés du matériau physique, comme illustré dans la capture d’écran suivante. Vous pouvez maintenant commencer à attribuer des textures à la ressource.

   ![Capture d’écran montrant la liste des textures.](media/3dsmax/textures-list.jpg)

Comme vous pouvez le voir, il existe un large éventail de cartes et de textures que vous pouvez ajouter au matériau. Pour ce tutoriel, vous allez utiliser seulement cinq emplacements de texture dans le matériau.

>[!TIP]
>Une bonne pratique consiste à nommer vos matériaux de manière pertinente, comme illustré dans la capture d’écran ci-dessus.

La façon dont vous générez vos textures peut varier en fonction de vos préférences ou de votre utilisation. Par exemple, vous pouvez utiliser des textures en mosaïque qui peuvent être appliquées à n’importe quelle ressource. Ou vous voulez que des parties spécifiques d’un projet ou d’une ressource aient leurs propres ensembles personnalisés de textures. Vous voulez utiliser des textures de mosaïque génériques que vous pouvez obtenir en ligne. Vous pouvez également les créer vous-même dans des applications telles que Photoshop, Quixel Suite et Substance Suite.

Avant de commencer à attribuer des textures, vous devez tenir compte des coordonnées de texture de la ressource (UVW). Lorsque vous appliquez des textures à un modèle, il est recommandé de s’assurer que le modèle est déplié. (Les textures ne s’affichent pas correctement sans dépliage UV approprié.) C’est particulièrement important dans le cadre de ce tutoriel, car vous souhaitez utiliser une carte d’occlusion ambiante (AO) sur votre modèle. Contrairement à Stingray Shader dans Maya, le matériau physique dans 3ds Max n’a pas d’emplacement de texture AO dédié. Par conséquent, nous allons appliquer la carte AO dans un autre emplacement. Pour lui permettre d’être utilisée séparément des autres textures (par exemple, des textures de mosaïque), vous allez lui attribuer un canal de mappage UVW distinct. 

Vous allez commencer par attribuer un modificateur UVW de dépliage au modèle, comme illustré dans la capture d’écran suivante. 

- Dans l’éditeur de propriétés des objets sélectionnés, sélectionnez la liste de modificateurs. Dans la liste déroulante qui s’affiche, faites défiler et sélectionnez **Unwrap UVW**. Cette action applique un modificateur UVW de dépliage à la ressource.
![Capture d’écran montrant comment sélectionner Unwrap UVW.](media/3dsmax/unwrap-modifier.jpg)

  Le canal de mappage est défini sur 1. En général, vous effectuez le dépliage principal dans le canal de mappage 1. Dans le cas présent, l’objet a été déplié sans coordonnées de texture superposées (UV).
![Capture d’écran montrant les coordonnées de texture dépliées (UVW).](media/3dsmax/unwrapped-uvw.jpg)

L’étape suivante consiste à créer un deuxième canal de mappage UV.

1. Fermez l’éditeur UV s’il est ouvert. Dans la section **Channel** (Canal) du menu **Edit UVs** (Modifier les UV), remplacez le numéro de canal en le définissant sur **2**. Le canal de mappage 2 est le canal attendu pour les cartes d’occlusion ambiante. 

1. Dans la boîte de dialogue **Channel Change Warning** (Avertissement de modification de canal), vous pouvez déplacer (**Move**) les UV existants du canal 1 vers le nouveau canal 2, ou abandonner (**Abandon**) les UV existants, ce qui crée automatiquement un nouveau dépliage UV (UV Unwrap). Sélectionnez **Abandon** (Abandonner) uniquement si vous envisagez de créer un nouveau dépliage UV pour la carte d’occlusion ambiante qui diffère des UV du canal de mappage 1. (Par exemple, si vous souhaitez utiliser des textures de mosaïque dans le canal 1.) Dans ce tutoriel, vous allez déplacer les UV du canal 1 vers le canal 2 car vous n’avez pas besoin de modifier le nouveau canal UV.

   >[!NOTE]
   >Même si vous avez copié (déplacé) le dépliage UV du canal de mappage 1 vers le canal de mappage 2, vous pouvez apporter toutes les modifications nécessaires aux UV du nouveau canal sans que cela ait un impact sur le canal de mappage d’origine.

   ![Capture d’écran qui montre l’avertissement de modification du canal.](media/3dsmax/channel-change.jpg)

Maintenant que vous avez créé le nouveau canal de mappage, vous pouvez revenir au matériau physique dans l’éditeur de matériaux et commencer à lui ajouter des textures. Tout d’abord, vous allez ajouter la carte d’occlusion ambiante, car il existe une autre étape pour lui permettre de fonctionner correctement. Une fois que la carte d’occlusion ambiante est insérée dans votre matériau, vous devez la configurer de sorte qu’elle utilise le canal de mappage 2.

Comme indiqué précédemment, il n’existe pas d’emplacement dédié pour les cartes AO dans le matériau physique 3ds Max. Au lieu de cela, vous appliquez la carte AO à l’emplacement **Diffuse Roughness** (Rugosité diffuse).

1. Dans la liste **Generic Maps** (Cartes génériques) du matériau physique, sélectionnez l’emplacement **No Map** (Aucune carte) en regard de **Diffuse Roughness** (Rugosité diffuse) et chargez votre carte AO.

1. Dans les propriétés des textures AO, le canal de mappage est défini sur **1** par défaut. Remplacez cette valeur par **2**. Cette action est la dernière des étapes nécessaires pour ajouter votre carte AO.

   >[!IMPORTANT]
   >Il s’agit d’une étape importante, en particulier si vos UV du canal 2 sont différents de ceux du canal 1, car l’occlusion ambiante n’effectue pas de mappage correct si le mauvais canal est sélectionné.

   ![Capture d’écran montrant comment attribuer une carte AO.](media/3dsmax/assign-ao-map.jpg)

Vous allez maintenant attribuer la map Normales au matériau PBR. Cette action diffère quelque peu du processus dans Maya. La map Normales n’est pas appliquée directement à l’emplacement de la carte de relief. (Il n’y a pas d’emplacement de map Normales dans le matériau physique 3ds max.) À la place, vous ajoutez la map Normales à un modificateur de map Normales, qui est lui-même branché dans l’emplacement de la normale.

1. Dans la section **Special Maps** (Cartes spéciales) des propriétés du matériau physique (dans l’éditeur de matériaux), sélectionnez l’emplacement **No Map** (Aucune carte) en regard de **Bump Map** (Carte de relief). 

1. Dans le **navigateur de matériaux/cartes**, recherchez et sélectionnez **Normal Bump** (Relief normal). Cette action ajoute un modificateur **Relief normal** au matériau.

1. Dans le modificateur **Normal Bump** (Relief normal), sélectionnez **No map** (Aucune carte) en regard de **Normal**. Recherchez et chargez votre map Normales.

1. Vérifiez que la méthode est définie sur **Tangent** (Tangente) (elle doit l’être par défaut). Si nécessaire, activez/désactivez **Flip Green (Y)** (Inverser le vert (Y)).

   ![Capture d’écran montrant comment sélectionner un relief normal.](media/3dsmax/normal-bump.jpg)
   ![Capture d’écran qui montre le chargement de la map Normales.](media/3dsmax/load-normal-map.jpg)

Une fois que la map Normales a été correctement attribuée, vous pouvez attribuer les textures restantes pour terminer la configuration du matériau physique. Le processus est simple. Il n’y a aucun paramètre spécial à prendre en compte. La capture d’écran suivante montre le jeu complet de textures attribuées au matériau : 

![Capture d’écran qui montre le jeu complet de textures attribuées au matériau.](media/3dsmax/all-textures.jpg)

Maintenant que vous avez créé et configuré vos matériaux PBR, il est maintenant temps de réfléchir à l’instanciation des objets dans la scène. Instanciez les objets similaires dans la scène, tels que les écrous, les boulons, les vis et les rondelles. Les objets identiques peuvent générer des gains significatifs en termes de taille de fichier. Les instances d’un objet principal peuvent avoir leurs propres transformations, rotation et échelle, ce qui vous permet de les placer où vous le souhaitez dans votre scène. Dans 3ds Max, le processus d’instanciation est simple.

1. Dans la fenêtre d’affichage principale, sélectionnez le ou les objets que vous souhaitez exporter.

1. Maintenez la touche **Maj** et faites glisser les ressources vers le haut à l’aide de l’outil de transformation (déplacement). 

1. Dans la boîte de dialogue **Clone Options** (Options de clonage), définissez **Object** (Objet) sur **Instance** et sélectionnez **OK** :

   ![Capture d’écran de la boîte de dialogue Options de clonage.](media/3dsmax/instance-object.jpg)

Cette action crée une instance de votre objet qui peut être déplacée, pivotée ou mise à l’échelle indépendamment de son parent et des autres instances de ce parent.

>[!IMPORTANT]
>Toutes les modifications que vous apportez à une instance en mode sous-objet sont transmises à toutes les instances de l’objet. Par conséquent, si vous travaillez avec les composants d’un objet instancié, tels que les sommets et les faces de polygones, les modifications que vous apportez affectent toutes les instances. N’oubliez pas que tout objet instancié peut être transformé en objet unique à tout moment. 

>[!TIP]
>Lorsque vous instanciez votre scène, il est judicieux de créer des instances au fur et à mesure. Le remplacement ultérieur de copies par des objets instanciés est difficile. 

Une dernière chose à prendre en compte avant de passer au processus d’exportation est la façon dont vous pouvez empaqueter votre scène/ressource pour le partage. Dans l’idéal, si vous transmettez la ressource à un client ou à un membre d’équipe, vous souhaitez qu’il puisse l’ouvrir et l’afficher avec le moins de difficultés possible. Il est donc important de conserver les chemins des textures des ressources relatifs au fichier de scène. Si les chemins des textures de votre ressource pointent vers un lecteur local ou un chemin/emplacement absolu, elles ne sont pas chargées dans la scène si celle-ci est ouverte sur un autre ordinateur, même si le fichier .max se trouve dans le même dossier que les textures. Le fait de rendre les chemins des textures relatifs dans 3ds Max résout ce problème de manière relativement simple.

1. Dans la barre d’outils principale, accédez à **File** (Fichier) > **Reference** (Reférence) > **Asset Tracking Toggle** (Activer/désactiver le suivi des ressources). 

1. Dans la fenêtre de suivi des ressources, la totalité ou la plupart des textures que vous avez appliquées à vos matériaux PBR sont listées sous la colonne **Maps/Shaders** (Cartes/nuanceurs).

1. En regard de celles-ci, dans la colonne **Full Path** (Chemin complet), vous voyez le chemin d’accès à l’emplacement de vos textures, qui correspond probablement à un emplacement sur votre ordinateur local.

1. Enfin, vous voyez une colonne appelée **Status** (État). Cette colonne indique si une texture donnée a été localisée et appliquée à votre scène. Elle accompagne la texture de l’un des termes suivants : **OK**, **Found** ou **File Missing**. Les deux premiers indiquent que le fichier a été trouvé et chargé. Le dernier signifie évidemment que l’outil de suivi n’a pas réussi à localiser le fichier.
 
   ![Capture d’écran montrant la fenêtre de suivi des ressources.](media/3dsmax/texture-paths.jpg)

Vous remarquerez peut-être que les textures ne sont pas toutes listées dans la fenêtre de suivi des ressources lorsque vous l’ouvrez pour la première fois. Cela n’est pas un problème. Le fait d’exécuter le processus de recherche de chemin d’accès une ou deux fois permet généralement de trouver toutes les textures d’une scène. Le processus de recherche de chemin est le suivant : 

1. Dans la fenêtre de suivi des ressources, maintenez enfoncée la touche **Maj** et sélectionnez la première texture de la liste **Maps/Shaders** (Cartes/nuanceurs) et, tout en maintenant la touche **Maj** enfoncée, cliquez sur la dernière texture de la liste. Cette action permet de sélectionner toutes les textures de la liste. Les textures sélectionnées sont mises en surbrillance en bleu. (Voir la capture d’écran précédente.)

1. Cliquez avec le bouton droit sur la sélection, puis sélectionnez **Set Path** (Définir le chemin).

1. Dans la zone **Specify Asset Path** (Spécifier le chemin de la ressource), sélectionnez le chemin d’accès local de vos textures et remplacez-le par `.\`.  Sélectionnez **OK**. 

    La fenêtre Suivi des ressources est mise à jour comme indiqué dans la capture d’écran suivante. Cette mise à jour peut prendre un certain temps, en fonction du nombre de textures et de la taille de votre scène.
![Capture d’écran qui affiche la fenêtre de suivi des ressources mise à jour.](media/3dsmax/resolve-textures.jpg)

Remarquez que la colonne **Full Path** (Chemin complet) est maintenant vide. Cela signifie que la scène ne recherche plus les textures appropriées dans un emplacement spécifique (absolu). Elle les trouve toujours tant que le fichier .max ou le fichier FBX associé se trouve dans le même dossier que les textures. 

>[!NOTE]
>Vous devrez peut-être répéter ce processus plusieurs fois pour rechercher et résoudre l’ensemble des textures et des chemins. Cela n’est pas un problème. Il vous suffit de répéter le processus jusqu’à ce que toutes les ressources pertinentes soient comptabilisées. Il se peut que certains fichiers restent introuvables. Dans ce cas, sélectionnez simplement toutes les ressources de la liste, puis sélectionnez **Remove Missing Paths** (Supprimer les chemins manquants). (Voir l’image précédente.)

## <a name="fbx-export"></a>Exportation au format FBX

Maintenant que vous avez défini les chemins de texture sur des chemins relatifs, vous pouvez passer à l’exportation au format FBX. Là encore, le processus est simple et peut être effectué de plusieurs manières. 

>[!TIP]
>Sauf si vous souhaitez exporter l’intégralité de votre scène, il est judicieux de sélectionner pour l’exportation uniquement les ressources dont vous avez besoin. Dans les scènes nécessitant beaucoup de ressources, l’exportation peut prendre beaucoup de temps.
>
>Si vous avez utilisé des modificateurs comme Turbosmooth ou Open SubDiv, il est conseillé de les réduire avant l’exportation, car ils peuvent provoquer des problèmes lors de l’exportation. Veillez à enregistrer votre scène avant de les réduire. 

1. Dans la scène, sélectionnez les ressources que vous souhaitez exporter. Dans la barre d’outils principale, accédez à **File (Fichier)**  > **Export (Exporter)**  > **Export Selected (Exporter la sélection)** .

1. Dans la boîte de dialogue **Select File to Export** (Sélectionner le fichier à exporter), tapez ou sélectionnez un nom de fichier de sortie. Dans la liste **Save as type** (Enregistrer sous), sélectionnez **Autodesk (*.fbx)** . Cette action ouvre la fenêtre d’exportation au format FBX.

  >[!IMPORTANT] 
  >Si vous avez créé des instances dans votre scène, il est important de sélectionner **Preserve Instances** (Conserver les instances) dans les paramètres d’exportation au format FBX. 

  ![Capture d’écran montrant comment exporter au format FBX.](media/3dsmax/fbx-export.jpg)

  Rappelez-vous qu’il existe deux façons d’exporter le fichier. Si l’objectif est de partager le FBX avec ses fichiers de texture dans un dossier/répertoire, les paramètres affichés dans la capture d’écran suivante doivent fonctionner correctement. 

   Si vous préférez ne pas partager de grands dossiers/répertoires de textures avec le fichier FBX, vous pouvez choisir d’incorporer les textures dans le fichier FBX. Si vous incorporez les textures, la totalité de la ressource, notamment les textures, est ajoutée à un seul FBX. Dans ce cas, l’exportation s’effectue dans une seule ressource, mais le fichier FBX est alors considérablement plus volumineux.

   >[!IMPORTANT]
   >Si la taille du fichier FBX résultant est supérieure à 2,4 Go, la version minimale spécifiée dans les paramètres d’exportation au format FBX doit être 2016 ou ultérieure. (Voir la capture d’écran précédente.) Les versions plus récentes disposent de la prise en charge 64 bits et prennent donc en charge des fichiers plus volumineux.

1. Si vous souhaitez exporter la scène en incluant les textures, dans la fenêtre d’exportation *FBX, sélectionnez **Embed Media** (Incorporer le média). 

1. Sélectionnez le reste de vos paramètres, puis sélectionnez **OK** :

    ![Capture d’écran montrant les paramètres d’exportation au format FBX.](media/3dsmax/fbx-settings.jpg)


   Lorsque vous exportez au format FBX en utilisant un matériau physique, l’avertissement suivant s’affiche probablement une fois que vous avez sélectionné **OK** dans la fenêtre d’exportation au format FBX : 

   ![Capture d’écran montrant l’échec de l’exportation de matériel.](media/3dsmax/export-warnings.jpg)

   Cet avertissement vous informe que les matériaux exportés peuvent ne pas être compatibles avec d’autres packages logiciels. Comme le matériau physique est compatible avec Azure Remote Rendering, vous n’avez pas à vous en soucier. 

1. Sélectionnez simplement **OK** pour terminer le processus et fermer la fenêtre.

## <a name="conclusion"></a>Conclusion

En général, ce type de matériau semble plus réaliste, car il est basé sur les propriétés physiques réelles de la lumière. Il ajoute un effet immersif qui donne un aspect réel à la scène.

## <a name="next-steps"></a>Étapes suivantes

Vous savez maintenant configurer des matériaux avec un éclairage avancé pour les objets d’une scène. Vous savez également exporter les objets au format FBX pris en charge par Azure Remote Rendering. L’étape suivante consiste à convertir le fichier FBX et à le visualiser dans Azure Remote Rendering.

>[!div class="nextstepaction"]
>[Démarrage rapide : Convertir un modèle pour le rendu](../../quickstarts\convert-model.md)

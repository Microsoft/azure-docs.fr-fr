---
title: Dépanner
description: Informations sur la résolution des problèmes concernant Azure Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 02/25/2020
ms.topic: troubleshooting
ms.openlocfilehash: 34a947a2a0f6d8c87c0580f273130b671b4f17fc
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97722230"
---
# <a name="troubleshoot"></a>Dépanner

Cette page liste les problèmes courants qui interfèrent avec Azure Remote Rendering et les façons de les résoudre.

## <a name="cant-link-storage-account-to-arr-account"></a>Impossibilité de lier un compte de stockage à un compte ARR

Parfois, lors de la [liaison d’un compte de stockage](../how-tos/create-an-account.md#link-storage-accounts), le compte de rendu à distance (ARR, Remote Rendering Account) n’est pas répertorié. Pour résoudre ce problème, accédez au compte ARR dans le portail Azure, puis sélectionnez **Identité** dans le groupe **Paramètres** sur la gauche. Assurez-vous que l’**État** est **Activé**.
![Débogueur de frames Unity](./media/troubleshoot-portal-identity.png)

## <a name="client-cant-connect-to-server"></a>Le client ne peut pas se connecter au serveur

Assurez-vous que vos pare-feu (sur l’appareil, les routeurs internes, etc.) ne bloquent pas les ports mentionnés dans la [Configuration requise](../overview/system-requirements.md#network-firewall).

## <a name="error-disconnected-videoformatnotavailable"></a>Erreur : `Disconnected: VideoFormatNotAvailable`

Vérifiez que votre GPU prend en charge le décodage vidéo matériel. Consultez [PC de développement](../overview/system-requirements.md#development-pc).

Si vous travaillez sur un ordinateur portable avec deux GPU, il est possible que le GPU que vous utilisez par défaut n’offre pas de fonctionnalité de décodage vidéo matériel. Si c’est le cas, essayez de forcer votre application à utiliser l’autre GPU. Cela est souvent possible dans les paramètres du pilote GPU.

## <a name="retrieve-sessionconversion-status-fails"></a>La récupération de l’état de la session/conversion échoue

Une fréquence trop élevée d’envoi de commandes d’API REST entraîne une limitation du serveur, et finit par retourner un échec. Le code d’état http dans le cas de limitation est 429 (« trop de demandes »). En règle générale, il doit y avoir un délai de **5 à 10 secondes entre les appels successifs**.

Notez que cette limite affecte non seulement les appels d’API REST effectués directement, mais aussi leurs équivalents C#/C++, tels que `Session.GetPropertiesAsync`, `Session.RenewAsync` ou `Frontend.GetAssetConversionStatusAsync`.

Si vous rencontrez une limitation côté serveur, modifiez le code pour effectuer les appels moins fréquemment. Le serveur réinitialisant l’état de limitation à chaque minutes, vous pouvez sans problème réexécuter le code au bout d’une minute.

## <a name="h265-codec-not-available"></a>Codec H265 non disponible

Le serveur peut refuser de se connecter avec une erreur `codec not available` pour deux raisons.

**Le codec H265 n’est pas installé :**

Tout d’abord, veillez à installer les **extensions vidéo HEVC** comme indiqué dans la section [Logiciels](../overview/system-requirements.md#software) de la configuration système requise.

Si les problèmes persistent, vérifiez que votre carte graphique prend en charge le codec H265 et que le pilote graphique le plus récent est installé. Pour connaître les informations spécifiques au fournisseur, consultez la section [PC de développement](../overview/system-requirements.md#development-pc) de la configuration requise.

**Le codec est installé, mais ne peut pas être utilisé :**

La raison de ce problème est un paramètre de sécurité incorrect sur les DLL. Ce problème ne se manifeste pas quand vous tentez de regarder des vidéos encodées avec le codec H265. La réinstallation du codec ne résout pas non plus le problème. Effectuez plutôt les étapes suivantes :

1. Ouvrez **PowerShell** avec des droits d’administrateur, puis exécutez la commande suivante

    ```PowerShell
    Get-AppxPackage -Name Microsoft.HEVCVideoExtension
    ```
  
    Cette commande doit générer la valeur `InstallLocation` du codec, similaire à ce qui suit :
  
    ```cmd
    InstallLocation   : C:\Program Files\WindowsApps\Microsoft.HEVCVideoExtension_1.0.23254.0_x64__5wasdgertewe
    ```

1. Ouvrez le dossier dans l’Explorateur Windows
1. Il doit y avoir un sous-dossier **x86** et un sous-dossier **x64**. Cliquez avec le bouton droit sur l’un des dossiers, puis choisissez **Propriétés**
    1. Sélectionnez l’onglet **Sécurité**, puis cliquez sur le bouton **Paramètres avancés**
    1. Cliquez sur **Modifier** pour le **Propriétaire**
    1. Tapez **Administrateurs** dans le champ de texte
    1. Cliquez sur **Vérifier les noms**, puis sur **OK**
1. Répétez les étapes ci-dessus pour l’autre dossier
1. Répétez également les étapes ci-dessus sur chaque fichier DLL présent dans les deux dossiers. Il doit y avoir quatre DLL en tout.

Pour vérifier que les paramètres sont désormais corrects, effectuez les étapes suivantes pour chacune des quatre DLL :

1. Sélectionnez **Propriétés > Sécurité > Modifier**
1. Parcourez la liste de tous les **Groupes/Utilisateurs**, puis vérifiez que le droit **Lire et exécuter** est défini pour chacun d’eux (la colonne **Autoriser** doit contenir une coche)

## <a name="low-video-quality"></a>Vidéo de mauvaise qualité

La qualité de la vidéo peut être compromise par la qualité du réseau ou par l’absence du codec vidéo H265.

* Consultez les étapes indiquées pour [identifier les problèmes réseau](#unstable-holograms).
* Consultez la [configuration système requise](../overview/system-requirements.md#development-pc) pour savoir comment installer le pilote graphique le plus récent.

## <a name="video-recorded-with-mrc-does-not-reflect-the-quality-of-the-live-experience"></a>La vidéo enregistrée avec la fonction MRC ne reflète pas la qualité de l’expérience en direct

Une vidéo peut être enregistrée sur HoloLens par le biais de la fonction [Mixed Reality Capture (MRC)](/windows/mixed-reality/mixed-reality-capture-for-developers). Toutefois, la vidéo qui en résulte est de moins bonne qualité que l’expérience en direct pour deux raisons :
* La fréquence d’images vidéo est limitée à 30 Hz au lieu de 60 Hz.
* Les images vidéo ne passent pas par l’étape de traitement [Reprojection en phase tardive](../overview/features/late-stage-reprojection.md), de sorte que la vidéo semble plus hachée.

Les deux sont des limitations inhérentes à la technique d’enregistrement.

## <a name="black-screen-after-successful-model-loading"></a>Écran noir après le chargement réussi d’un modèle

Si vous êtes connecté au runtime de rendu et que vous avez correctement chargé un modèle, mais que vous ne voyez ensuite qu’un écran noir, cela peut avoir plusieurs causes distinctes.

Nous vous recommandons de tester les opérations suivantes avant de procéder à une analyse plus approfondie :

* Le codec H265 est-il installé? Même s’il y existe un mécanisme de secours pour le codec H264, nous avons observé des cas où ce mécanisme de secours ne fonctionnait pas correctement. Consultez la [configuration système requise](../overview/system-requirements.md#development-pc) pour savoir comment installer le pilote graphique le plus récent.
* Quand vous utilisez un projet Unity, fermez Unity, supprimez les dossiers *library* et *obj* temporaires dans le répertoire du projet, puis chargez/générez à nouveau le projet. Dans certains cas, les données mises en cache entraînaient un mauvais fonctionnement de l’exemple sans aucune raison évidente.

Si ces deux étapes n’ont pas permis de résoudre le problème, il est nécessaire de déterminer si les trames vidéo sont reçues, ou non, par le client. Cela peut être déterminé par programmation, comme expliqué dans le chapitre [Requêtes de performances côté serveur](../overview/features/performance-queries.md). Le `FrameStatistics struct` a un membre qui indique le nombre de trames vidéo reçues. Si ce nombre est supérieur à 0 et qu’il augmente dans le temps, le client reçoit des trames vidéo réelles du serveur. Par conséquent, il doit s’agir d’un problème côté client.

### <a name="common-client-side-issues"></a>Problèmes courants côté client

**Le modèle dépasse les limites de la machine virtuelle sélectionnée, notamment le nombre maximal de polygones :**

Consultez les [limites de taille de serveur](../reference/limits.md#overall-number-of-polygons) spécifiques.

**Le modèle ne se trouve pas dans le frustum de l’appareil photo :**

Dans de nombreux cas, le modèle s’affiche correctement, mais il se trouve en dehors du frustum de l’appareil photo. Une raison courante à cela est que le modèle a été exporté avec un tableau croisé dynamique trop décentré, de sorte qu’il est coupé par le plan arrière de découpage de l’appareil photo. Cela permet d’interroger par programmation le cadre englobant du modèle et de visualiser la zone avec Unity sous la forme d’une zone de ligne ou d’imprimer ses valeurs dans le journal de débogage.

En outre, le processus de conversion génère un [fichier JSON de sortie](../how-tos/conversion/get-information.md) avec le modèle converti. Pour déboguer des problèmes de positionnement du modèle, il est utile de regarder l’entrée `boundingBox` dans la [section outputStatistics](../how-tos/conversion/get-information.md#the-outputstatistics-section) :

```JSON
{
    ...
    "outputStatistics": {
        ...
        "boundingBox": {
            "min": [
                -43.52,
                -61.775,
                -79.6416
            ],
            "max": [
                43.52,
                61.775,
                79.6416
            ]
        }
    }
}
```

Le cadre englobant est décrit comme une position `min` et `max` dans l’espace 3D, en mètres. Par conséquent, une coordonnée de 1000,0 signifie qu’elle est à 1 kilomètre du point d’origine.

Ce cadre englobant peut présenter deux problèmes qui mènent à une géométrie invisible :
* **Le cadre peut être très décentré**, de sorte que l’objet est coupé entièrement en raison du plan arrière de découpage. Dans ce cas, les valeurs de `boundingBox` ressemblent à ceci : `min = [-2000, -5,-5], max = [-1990, 5,5]`, en utilisant un décalage important sur l’axe X comme illustré en exemple ici. Pour résoudre ce type de problème, activez l’option `recenterToOrigin` dans la [configuration de conversion du modèle](../how-tos/conversion/configure-model-conversion.md).
* **Le cadre peut être centré, mais être d’un ordre de grandeur trop important**. Cela signifie que, bien que l’appareil photo démarre au centre du modèle, sa géométrie est découpée dans toutes les directions. Dans ce cas, les valeurs typiques de `boundingBox` se présentent comme suit : `min = [-1000,-1000,-1000], max = [1000,1000,1000]`. La raison de ce type de problème est généralement une incompatibilité de l’échelle d’unités. Pour compenser, spécifiez une [valeur de mise à l’échelle pendant la conversion](../how-tos/conversion/configure-model-conversion.md#geometry-parameters) ou marquez le modèle source avec les unités appropriées. La mise à l’échelle peut également être appliquée au nœud racine lors du chargement du modèle au moment de l’exécution.

**Le pipeline de rendu Unity n’inclut pas les crochets de rendu :**

Azure Remote Rendering se raccroche au pipeline de rendu Unity pour créer la composition du cadre avec la vidéo et effectuer la reprojection. Pour vérifier que ces crochets, ouvrez le menu *:::no-loc text="Window > Analysis > Frame debugger":::* . Activez-le et vérifiez qu’il existe deux entrées pour `HolographicRemotingCallbackPass` dans le pipeline :

![Pipeline de rendu Unity](./media/troubleshoot-unity-pipeline.png)

## <a name="checkerboard-pattern-is-rendered-after-model-loading"></a>Le modèle de damier est rendu après le chargement du modèle

Si l’image rendue se présente comme suit : ![La capture d’écran présente une grille de carrés noirs et blancs avec un menu Outils.](../reference/media/checkerboard.png)
alors le renderer atteint les [limites de polygones pour la taille de configuration standard](../reference/vm-sizes.md). Pour atténuer ce problème, basculez vers une taille de configuration **Premium** ou réduisez le nombre de polygones visibles.

## <a name="the-rendered-image-in-unity-is-upside-down"></a>L’image rendue dans Unity est à l’envers

Veillez à suivre à la lettre le [Tutoriel Unity : Afficher des modèles distants](../tutorials/unity/view-remote-models/view-remote-models.md). Une image à l’envers indique que Unity est nécessaire pour créer une cible de rendu hors écran. Ce comportement n’est pas pris en charge actuellement, et crée un impact énorme sur les performances sur HoloLens 2.

Ce problème peut être lié à MSAA, à HDR ou à l’activation du post-traitement. Vérifiez que le profil de qualité inférieure est sélectionné et défini comme valeur par défaut dans Unity. Pour ce faire, accédez à *Edit > Project Settings... > Quality* (Modifier > Paramètres du projet... > Qualité).

## <a name="unity-code-using-the-remote-rendering-api-doesnt-compile"></a>Le code Unity utilisant l’API Remote Rendering ne se compile pas

### <a name="use-debug-when-compiling-for-unity-editor"></a>Utiliser Debug lors de la compilation dans l’éditeur Unity

Basculez le *type de build* de la solution Unity sur **Déboguer**. Lors du test de ARR dans l’éditeur Unity, le `UNITY_EDITOR` défini est disponible uniquement dans les builds « Debug ». Notez que cela n’est pas lié au type de build utilisé pour les [applications déployées](../quickstarts/deploy-to-hololens.md), dans lesquelles vous devez préférer les builds « Release ».

### <a name="compile-failures-when-compiling-unity-samples-for-hololens-2"></a>Échecs de compilation lors de la compilation d’exemples Unity pour HoloLens 2

Nous avons constaté de faux échecs lors de tentative de compilation d’exemples Unity (démarrage rapide, ShowCaseApp, etc.) pour HoloLens 2. Visual Studio signale qu’il ne peut pas copier certains fichiers qui sont pourtant présents. Si vous avez rencontré ce problème :
* Supprimez tous les fichiers Unity temporaires du projet, puis réessayez. C’est-à-dire, fermez Unity, supprimez les dossiers *library* et *obj* temporaires dans le répertoire du projet, puis chargez/générez à nouveau le projet.
* Vérifiez que les projets se trouvent dans un répertoire sur disque dont le chemin est relativement court, car l’étape de copie échoue parfois en présence de noms de fichiers longs.
* Si cela n’est pas la cause, il est possible que MS Sense interfère avec l’étape de copie. Pour configurer une exception, exécutez cette commande de Registre à partir de la ligne de commande (nécessite des droits d’administrateur) :
    ```cmd
    reg.exe ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection" /v groupIds /t REG_SZ /d "Unity”
    ```
    
### <a name="arm64-builds-for-unity-projects-fail-because-audiopluginmshrtfdll-is-missing"></a>Les builds Arm64 pour les projets Unity échouent car AudioPluginMsHRTF.dll est manquant

`AudioPluginMsHRTF.dll` pour Arm64 a été ajouté au package *Windows Mixed Reality* *(com.unity.xr.windowsmr.metro)* dans la version 3.0.1. Vérifiez que la version 3.0.1 ou ultérieure est installée par le biais de Unity Package Manager. Dans la barre de menus Unity, accédez à *Window > Package Manager* (Fenêtre > Gestionnaire de package) et recherchez le package *Windows Mixed Reality*.

## <a name="native-c-based-application-does-not-compile"></a>L’application basée sur C++ native ne se compile pas

### <a name="library-not-found-error-for-uwp-application-or-dll"></a>Erreur « bibliothèque introuvable » pour une application ou dll UWP

Le package NuGet C++ contient un fichier `microsoft.azure.remoterendering.Cpp.targets` qui définit la version de fichier binaire à utiliser. Pour identifier `UWP`, les conditions dans le fichier vérifient la présence de `ApplicationType == 'Windows Store'`. Il faut donc vérifier que ce type est défini dans le projet. Cela devrait être le cas lors de la création d’une application ou dll UWP via l’Assistant Projet de Visual Studio.

## <a name="unstable-holograms"></a>Hologrammes instables

Si les objets rendus semblent se déplacer avec les mouvements des têtes, vous rencontrez peut-être des problèmes avec la *Reprojection en phase tardive* (LSR). Pour obtenir des conseils sur la façon d’aborder une telle situation, consultez la section relative à la [reprojection en phase tardive](../overview/features/late-stage-reprojection.md).

Les hologrammes instables (hologrammes qui oscillent, déformés, qui vacillent ou qui sautent) peuvent également s’expliquer par une mauvaise connectivité réseau, en particulier une bande passante réseau insuffisante ou une latence trop élevée. La valeur de [statistiques de performances](../overview/features/performance-queries.md) `ARRServiceStats.VideoFramesReused` constitue un bon indicateur de la qualité de votre connexion réseau. Les trames réutilisées indiquent les situations où une ancienne trame vidéo devait être réutilisée côté client car aucune nouvelle trame vidéo n’était disponible, par exemple en raison d’une perte de paquets ou de variations de la latence du réseau. Si `ARRServiceStats.VideoFramesReused` est souvent supérieur à zéro, cela indique un problème réseau.

Une autre valeur à examiner est `ARRServiceStats.LatencyPoseToReceiveAvg`. Elle doit toujours être inférieure à 100 ms. Des valeurs plus élevées pourraient indiquer que vous êtes connecté à un centre de données qui est trop éloigné.

Pour obtenir la liste des atténuations potentielles, consultez les [instructions pour la connectivité réseau](../reference/network-requirements.md#guidelines-for-network-connectivity).

## <a name="z-fighting"></a>Z-fighting

Bien qu’ARR propose une [fonctionnalité d’atténuation du Z-fighting](../overview/features/z-fighting-mitigation.md), il est possible que le Z-fighting soit encore visible dans la scène. Ce guide vise à résoudre ce problème.

### <a name="recommended-steps"></a>Étapes recommandées

Utilisez le workflow suivant pour atténuer le Z-fighting  :

1. Testez la scène avec les paramètres par défaut d’ARR (atténuation du Z-fighting activée).

1. Désactivez l’atténuation du Z-fighting par le biais de son [API](../overview/features/z-fighting-mitigation.md). 

1. Changez les plans proche et lointain de la caméra de façon à obtenir une plage plus proche.

1. Résolvez les problèmes de la scène à l’aide de la section suivante.

### <a name="investigating-remaining-z-fighting"></a>Examen du Z-fighting restant

Si vous avez effectué les étapes ci-dessus et que le Z-fighting restant est inacceptable, vous devez en rechercher la cause sous-jacente. Comme indiqué dans la [page de fonctionnalité d’atténuation du Z-fighting](../overview/features/z-fighting-mitigation.md), il existe deux raisons principales pour la présence de Z-fighting : la perte de précision de profondeur à l’extrémité lointaine de la plage de profondeur, et des surfaces qui se croisent tout en étant coplanaires. La perte de précision de profondeur est une éventualité mathématique qui ne peut être atténuée qu’en effectuant l’étape 3 ci-dessus. Des surfaces coplanaires traduisent un défaut d’élément multimédia source, et il est préférable de les corriger dans les données sources.

ARR offre une fonctionnalité permettant de déterminer si des surfaces peuvent présenter un Z-fighting : la [mise en surbrillance du damier](../overview/features/z-fighting-mitigation.md). Vous pouvez également déterminer visuellement ce qui provoque le Z-fighting. La première animation ci-dessous montre un exemple de perte de précision de profondeur dans la distance, tandis que la deuxième montre un exemple de surfaces presque coplanaires :

![L’animation montre un exemple de perte de précision de profondeur sur la distance.](./media/depth-precision-z-fighting.gif)  ![L’animation montre un exemple de surfaces presque coplanaires.](./media/coplanar-z-fighting.gif)

Comparez ces exemples à votre Z-fighting pour en déterminer la cause, ou suivez éventuellement ce workflow pas à pas :

1. Placez la caméra au-dessus des surfaces Z-fighting pour regarder directement la surface.
1. Reculez lentement la caméra afin de l’éloigner des surfaces.
1. Si le Z-fighting est visible à tout moment, les surfaces sont parfaitement coplanaires. 
1. Si le Z-fighting est visible la plupart du temps, les surfaces sont presque coplanaires.
1. Si le Z-fighting est uniquement visible de loin, la cause est un manque de précision de profondeur.

Les surfaces coplanaires peuvent avoir plusieurs causes :

* Un objet a été dupliqué par l’application d’exportation en raison d’une erreur ou d’approches de workflow différentes.

    Vérifiez ces problèmes avec l’application concernée.

* Les surfaces sont dupliquées et retournées de façon à apparaître recto verso dans les convertisseurs qui utilisent le front-face ou back-face culling.

    L’importation par le biais de la [conversion de modèle](../how-tos/conversion/model-conversion.md) détermine le côté principal du modèle. Le caractère bilatéral est considéré comme la valeur par défaut. La surface sera rendue sous la forme d’un mur fin avec un éclairage physiquement correct des deux côtés. Le caractère unilatéral peut être impliqué par des indicateurs dans l’élément multimédia source, ou forcé explicitement pendant la [conversion de modèle](../how-tos/conversion/model-conversion.md). En outre, mais de manière facultative, le [mode unilatéral](../overview/features/single-sided-rendering.md) peut être défini sur « normal ».

* Des objets se croisent dans les éléments multimédias sources.

     Un Z-fighting peut aussi être créé quand des objets sont transformés de telle manière que certaines de leurs surfaces se chevauchent. La transformation de certaines parties de l’arborescence de la scène dans la scène importée dans ARR peut également créer ce problème.

* Des surfaces sont intentionnellement créées pour se toucher, comme des décalques ou du texte sur des murs.

## <a name="graphics-artifacts-using-multi-pass-stereo-rendering-in-native-c-apps"></a>Artefacts graphiques utilisant le rendu stéréo à plusieurs passes dans les applications C++ natives

Dans certains cas, les applications C++ natives qui utilisent un mode de rendu stéréo à plusieurs passes pour le contenu local (rendu gauche et droite dans des passes séparées) après l’appel de [**BlitRemoteFrame**](../concepts/graphics-bindings.md#render-remote-image) peuvent déclencher un bogue du pilote. Le bogue entraîne des problèmes de pixellisation non déterministes, provoquant la disparition aléatoire de triangles individuels ou de parties de triangles du contenu local. Pour des raisons de performances, il est de toute manière recommandé de rendre le contenu local avec une technique de rendu stéréo en une seule passe plus moderne, par exemple utilisant **SV_RenderTargetArrayIndex**.

## <a name="next-steps"></a>Étapes suivantes

* [Configuration système requise](../overview/system-requirements.md)
* [Configuration requise pour le réseau](../reference/network-requirements.md)
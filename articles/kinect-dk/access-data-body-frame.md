---
title: Accéder aux données d’Azure Kinect DK dans une image de corps
description: Apprenez-en davantage sur les données d’une image de corps et les fonctions permettant d’accéder à celles-ci dans Azure Kinect DK.
author: qm13
ms.author: quentinm
ms.reviewer: yijwan
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: how-to
keywords: corps, image, azure, kinect, corps, suivi, conseils
ms.openlocfilehash: be44f59cb84e99129bf526575293eee69ca64598
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276762"
---
# <a name="access-data-in-body-frame"></a>Accéder aux données de l’ossature du corps

Cet article décrit les données contenues dans une image de corps et les fonctions permettant d’accéder à celles-ci.

Voici les fonctions qui sont abordées ici :

- [k4abt_frame_get_body_id()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga1d612404d133a279af847974e9359a92.html#ga1d612404d133a279af847974e9359a92)
- [k4abt_frame_get_body_index_map()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga0e4f2d0d7e330d444de7070fb1fee4f6.html#ga0e4f2d0d7e330d444de7070fb1fee4f6)
- [k4abt_frame_get_body_skeleton()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gac7032ab06268253538556750775064fb.html#gac7032ab06268253538556750775064fb)
- [k4abt_frame_get_capture()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gad9eef11f6496bbfe997536c374217d9a.html#gad9eef11f6496bbfe997536c374217d9a)
- [k4abt_frame_get_num_bodies()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga29ab088b1a0d1a246bdb5542e21aa3c3.html#ga29ab088b1a0d1a246bdb5542e21aa3c3)
- [k4abt_frame_get_device_timestamp_usec()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga04be7b814b40296cd6b97044ed7283e4.html#ga04be7b814b40296cd6b97044ed7283e4)

## <a name="key-components-of-a-body-frame"></a>Composants clés d’une image de corps

Chaque image de corps se compose d’une collection de structures de corps, d’un mappage d’index de corps 2D et de la capture d’entrée ayant produit ce résultat.

![Composants d’image du corps](./media/how-to-guides/body-frame.png)

## <a name="access-the-collection-of-body-structs"></a>Accéder à la collection de structures de corps

Plusieurs corps peuvent être détectés dans une seule capture. Vous pouvez interroger le nombre de corps en appelant la fonction [k4abt_frame_get_num_bodies()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga29ab088b1a0d1a246bdb5542e21aa3c3.html#ga29ab088b1a0d1a246bdb5542e21aa3c3).

```C
size_t num_bodies = k4abt_frame_get_num_bodies(body_frame);
```

Vous utilisez les fonctions [k4abt_frame_get_body_id()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga1d612404d133a279af847974e9359a92.html#ga1d612404d133a279af847974e9359a92) et [k4abt_frame_get_body_skeleton()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gac7032ab06268253538556750775064fb.html#gac7032ab06268253538556750775064fb) pour opérer une itération dans chaque index de corps afin de rechercher l’ID de corps et les informations de position/orientation jointes.

```C
for (size_t i = 0; i < num_bodies; i++)
{
    k4abt_skeleton_t skeleton;
    k4abt_frame_get_body_skeleton(body_frame, i, &skeleton);
    uint32_t id = k4abt_frame_get_body_id(body_frame, i);
}
```

## <a name="access-the-body-index-map"></a>Accéder au mappage d’index de corps

Vous utilisez la fonction [k4abt_frame_get_body_index_map()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga0e4f2d0d7e330d444de7070fb1fee4f6.html#ga0e4f2d0d7e330d444de7070fb1fee4f6) pour accéder au mappage d’index de corps. Pour une explication détaillée du mappage d’index de corps, consultez [Mappage d’index de corps](body-index-map.md). Veillez à libérer le mappage d’index de corps quand il n’est plus nécessaire.

```C
k4a_image_t body_index_map = k4abt_frame_get_body_index_map(body_frame);
...  // Do your work with the body index map
k4a_image_release(body_index_map);
```

## <a name="access-the-input-capture"></a>Accéder à la capture d’entrée

Le dispositif de suivi du corps est une API asynchrone. La capture d’origine a peut-être déjà été libérée au moment où le résultat s’affiche. Utilisez la fonction [k4abt_frame_get_capture()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gad9eef11f6496bbfe997536c374217d9a.html#gad9eef11f6496bbfe997536c374217d9a) pour interroger la capture d’entrée utilisée pour générer ce résultat de suivi du corps. Le nombre de références pour k4a_capture_t est augmenté à chaque appel de cette fonction. Utilisez la fonction [k4a_capture_release()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0ed6f74ec403c3eac1b8ef3afb32cee6.html#ga0ed6f74ec403c3eac1b8ef3afb32cee6) lorsque la capture n’est plus nécessaire.

```C
k4a_capture_t input_capture = k4abt_frame_get_capture(body_frame);
... // Do your work with the input capture
k4a_capture_release(input_capture);
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Kit de développement logiciel (SDK) de suivi du corps d’Azure Kinect](https://microsoft.github.io/Azure-Kinect-Body-Tracking/)

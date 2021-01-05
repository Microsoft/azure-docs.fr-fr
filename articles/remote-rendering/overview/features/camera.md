---
title: Appareil photo
description: Décrit les paramètres de caméra et les cas d’usage
author: christophermanthei
ms.author: chmant
ms.date: 03/07/2020
ms.topic: article
ms.openlocfilehash: fc82d046caa3663cffcda585258642813ab3a7d8
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207255"
---
# <a name="camera"></a>Appareil photo

Pour permettre une composition sans interruption du contenu rendu localement et à distance, plusieurs propriétés de caméra doivent rester synchronisées entre le serveur et le client. C’est tout particulièrement le cas des propriétés de transformation et de projection de caméra. Par exemple, le contenu rendu localement doit utiliser les propriétés de transformation et de projection de caméra avec lesquelles les dernières images distantes ont été rendues.

![Caméra locale et distante](./media/camera.png)

Dans l’image ci-dessus, la caméra locale a été déplacée par rapport à l’image distante envoyée par le serveur. De ce fait, le contenu local doit être rendu avec la même perspective que l’image distante ; l’image obtenue est enfin reprojetée dans l’espace de la caméra locale, ce qui est expliqué en détail dans [Reprojection en phase tardive](late-stage-reprojection.md).

Le délai observé entre le rendu distant et le rendu local signifie que toute modification de ces paramètres est appliquée avec un décalage. Par conséquent, une nouvelle valeur ne peut pas être utilisée immédiatement dans la même image.

Les distances des plans rapproché et éloigné peuvent être définies dans les paramètres de caméra. Sur HoloLens 2, les données de transformation et de projection restantes sont définies par le matériel. Lors de l’utilisation de la [simulation de bureau](../../concepts/graphics-bindings.md), elles sont définies via l’entrée de sa fonction `Update`.

Les valeurs modifiées peuvent être utilisées localement dès l’arrivée de la première image distante qui a été rendue avec ces valeurs. Sur HoloLens 2, les données à utiliser pour le rendu sont fournies par *HolographicFrame* comme dans n’importe quelle autre application holographique. Dans la [simulation de bureau](../../concepts/graphics-bindings.md), elles sont récupérées via la sortie de la fonction `Update`.

## <a name="camera-settings"></a>Paramètres de caméra

Les propriétés suivantes peuvent être modifiées dans les paramètres de caméra :

**Plan rapproché et éloigné :**

Pour empêcher la définition de plages non valides, les propriétés **NearPlane** et **FarPlane** sont en lecture seule et une fonction distincte **SetNearAndFarPlane** permet de modifier la plage. Ces données sont envoyées au serveur à la fin de l’image.

> [!IMPORTANT]
> Dans Unity, cela est gérée automatiquement quand les plans rapproché et éloigné de la caméra principale sont modifiés.

**EnableDepth** :

Il est parfois utile de désactiver l’écriture de la mémoire tampon de profondeur de l’image distante à des fins de débogage. La désactivation de la profondeur fait également arrêter l’envoi de données de profondeur par la serveur, ce qui réduit la bande passante.

> [!TIP]
> Dans Unity, le composant de débogage fourni appelé **EnableDepthComponent** permet d’activer ou de désactiver cette fonctionnalité dans l’interface utilisateur de l’éditeur.

La modification des paramètres de caméra peut être effectuée comme suit :

```cs
void ChangeCameraSetting(AzureSession session)
{
    CameraSettings settings = session.Actions.CameraSettings;

    settings.SetNearAndFarPlane(0.1f, 20.0f);
    settings.EnableDepth = false;
}
```

```cpp
void ChangeStageSpace(ApiHandle<AzureSession> session)
{
    ApiHandle<CameraSettings> settings = session->Actions()->GetCameraSettings();

    settings->SetNearAndFarPlane(0.1f, 20.0f);
    settings->SetEnableDepth(false);
}
```

## <a name="api-documentation"></a>Documentation de l’API

* [CameraSettings, C#](/dotnet/api/microsoft.azure.remoterendering.camerasettings)
* [CameraSettings, C++](/cpp/api/remote-rendering/camerasettings)
* [GraphicsBindingSimD3d11.Update, fonction C#](/dotnet/api/microsoft.azure.remoterendering.graphicsbindingsimd3d11.update)
* [GraphicsBindingSimD3d11::Update, fonction C++](/cpp/api/remote-rendering/graphicsbindingsimd3d11#update)

## <a name="next-steps"></a>Étapes suivantes

* [Liaison graphique](../../concepts/graphics-bindings.md)
* [Reprojection en phase tardive](late-stage-reprojection.md)
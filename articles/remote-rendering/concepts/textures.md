---
title: Textures
description: Workflow relatif aux ressources de texture
author: florianborn71
ms.author: flborn
ms.date: 02/05/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: b951dab1ad01187c7612fad047bc52eb6aa9700e
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701872"
---
# <a name="textures"></a>Textures

Les textures sont une [ressource partagée](../concepts/lifetime.md) immuable. Les textures peuvent être chargées à partir d’une [stockage Blob](../how-tos/conversion/blob-storage.md) et appliquées directement aux modèles comme illustré dans le [Tutoriel : Changement d’environnement et de matériaux](../tutorials/unity/materials-lighting-effects/materials-lighting-effects.md). Cependant, le plus souvent, les textures font partie d’un [modèle converti](../how-tos/conversion/model-conversion.md), dans lequel elles sont référencées par ses [matériaux](materials.md).

## <a name="texture-types"></a>Types de texture

À cas d’usage correspondent des types de texture spécifiques :

* Les **textures 2D** sont principalement utilisées dans les [matériaux](materials.md).
* Les **cubemaps** peuvent être utilisés pour le [ciel](../overview/features/sky.md).

## <a name="supported-texture-formats"></a>Formats de texture pris en charge

Toutes les textures fournies à ARR doivent être au [format DDS](https://en.wikipedia.org/wiki/DirectDraw_Surface), de préférence avec des mipmaps et la compression de texture.

## <a name="loading-textures"></a>Chargement des textures

Quand vous chargez une texture, vous devez spécifier son type attendu. Si le type ne correspond pas, le chargement de la texture échoue.
Le chargement d’une texture avec le même URI à deux reprises retourne le même objet de texture, car il s’agit d’une [ressource partagée](../concepts/lifetime.md).

Comme pour le chargement de modèles, il existe deux méthodes pour envoyer une ressource de texture dans un stockage Blob source :

* La ressource de texture peut être envoyée par le biais de son URI SAS. La fonction de chargement appropriée est `LoadTextureFromSASAsync` avec le paramètre `LoadTextureFromSASParams`. Utilisez également cette méthode pour le chargement de [textures intégrées](../overview/features/sky.md#built-in-environment-maps).
* La texture peut être envoyée directement par le biais des paramètres de stockage Blob si le [stockage Blob est lié au compte](../how-tos/create-an-account.md#link-storage-accounts). Dans ce cas, la fonction de chargement appropriée est `LoadTextureAsync` avec le paramètre `LoadTextureParams`.

L’exemple de code suivant montre comment charger une texture (ou une texture intégrée) par le biais de son URI SAS. Notez que seul le paramètre/la fonction de chargement diffère de l’autre cas :

```cs
LoadTextureAsync _textureLoad = null;
void LoadMyTexture(AzureSession session, string textureUri)
{
    _textureLoad = session.Actions.LoadTextureFromSASAsync(new LoadTextureFromSASParams(textureUri, TextureType.Texture2D));
    _textureLoad.Completed +=
        (LoadTextureAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                //use res.Result
            }
            else
            {
                System.Console.WriteLine("Texture loading failed!");
            }
            _textureLoad = null;
        };
}
```

```cpp
void LoadMyTexture(ApiHandle<AzureSession> session, std::string textureUri)
{
    LoadTextureFromSASParams params;
    params.TextureType = TextureType::Texture2D;
    params.TextureUrl = std::move(textureUri);
    ApiHandle<LoadTextureAsync> textureLoad = *session->Actions()->LoadTextureFromSASAsync(params);
    textureLoad->Completed([](ApiHandle<LoadTextureAsync> res)
    {
        if (res->GetIsRanToCompletion())
        {
            //use res->Result()
        }
        else
        {
            printf("Texture loading failed!");
        }
    });
}
```

Selon la façon dont vous envisagez d’utiliser la texture, son contenu et son type peuvent être soumis à des restrictions. Par exemple, la carte de rugosité d’un [matériau PBR](../overview/features/pbr-materials.md) doit être en nuances de gris.

> [!CAUTION]
> Toutes les fonctions *Async* dans ARR retournent des objets d’opérations asynchrones. Vous devez stocker une référence à ces objets jusqu’à ce que l’opération soit terminée. Sinon, le récupérateur de mémoire C# peut supprimer l’opération de façon précoce et ne jamais se terminer. Dans l’exemple de code ci-dessus, la variable membre « _textureLoad » est utilisée pour stocker une référence jusqu’à ce que l’événement *Completed* se produise.

## <a name="api-documentation"></a>Documentation de l’API

* [Texture, classe C#](/dotnet/api/microsoft.azure.remoterendering.texture)
* [RemoteManager.LoadTextureAsync(), C#](/dotnet/api/microsoft.azure.remoterendering.remotemanager.loadtextureasync)
* [RemoteManager.LoadTextureFromSASAsync(), C#](/dotnet/api/microsoft.azure.remoterendering.remotemanager.loadtexturefromsasasync)
* [Texture, classe C++](/cpp/api/remote-rendering/texture)
* [RemoteManager::LoadTextureAsync(), C++](/cpp/api/remote-rendering/remotemanager#loadtextureasync)
* [RemoteManager::LoadTextureFromSASAsync(), C++](/cpp/api/remote-rendering/remotemanager#loadtexturefromsasasync)

## <a name="next-steps"></a>Étapes suivantes

* [Matériaux](materials.md)
* [Lumière ambiante](../overview/features/sky.md)
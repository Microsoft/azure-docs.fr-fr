---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: 5a06a0663601c221dd456b9cf4437cb9f32a18f8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81399851"
---
:::row:::
    :::column span="3":::
        Le Kit de développement logiciel (SDK) Speech C++ est disponible sur Windows, Linux et macOS. Pour plus d’informations, consultez <a href="https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech" target="_blank">Microsoft.CognitiveServices.Speech <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="C++" src="https://docs.microsoft.com/media/logos/logo_Cplusplus.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

#### <a name="c-nuget-package"></a>Package NuGet C++

Le Kit de développement logiciel (SDK) Speech C++ peut être installé à partir du **Gestionnaire de package** avec la commande `Install-Package` suivante.

```powershell
Install-Package Microsoft.CognitiveServices.Speech
```

#### <a name="c-binaries-and-header-files"></a>Binaires et fichiers d’en-tête C++

Vous pouvez également installer le Kit de développement logiciel (SDK) Speech à partir de binaires. Téléchargez le SDK sous forme de <a href="https://aka.ms/csspeech/linuxbinary" target="_blank">package .tar <span class="docon docon-navigate-external x-hidden-focus"></span></a> et décompressez les fichiers dans le répertoire de votre choix. Le contenu de ce package (qui inclut les fichiers d’en-tête pour les architectures cible x86 et x64) est structuré comme suit :

  | Path                   | Description                                          |
  |------------------------|------------------------------------------------------|
  | `license.md`           | Licence                                              |
  | `ThirdPartyNotices.md` | Mentions tierces                                  |
  | `include`              | Les fichiers d’en-tête pour C++                                 |
  | `lib/x64`              | Bibliothèque x64 native pour la liaison avec votre application |
  | `lib/x86`              | Bibliothèque x86 native pour la liaison avec votre application |

  Pour créer une application, copiez ou déplacez les fichiers binaires (et les bibliothèques) exigés dans votre environnement de développement. Incluez-les dans votre processus de génération en fonction de vos besoins.

#### <a name="additional-resources"></a>Ressources supplémentaires

- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp" target="_blank">Code source C++ de démarrage rapide pour Windows, Linux et macOS <span class="docon docon-navigate-external x-hidden-focus"></span></a>
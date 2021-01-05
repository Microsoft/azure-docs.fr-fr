---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: 33cc9b52c4b687ca8147867b99dc70e8d71a9223
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88944390"
---
:::row:::
    :::column span="3":::
        Le Kit de développement logiciel (SDK) Speech prend en charge Windows 10 et Windows Server 2016, ou versions ultérieures. Les versions antérieures **ne sont pas** prises en charge. Il est possible d’utiliser certaines parties du Kit de développement logiciel (SDK) Speech avec des versions antérieures de Windows, mais ce n’est pas conseillé.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Windows" src="https://docs.microsoft.com/media/logos/logo_Windows.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

### <a name="system-requirements"></a>Configuration système requise

Le Kit de développement logiciel (SDK) Speech sur Windows nécessite que <a href="https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads" target="_blank">Microsoft Visual C++ Redistributable pour Visual Studio 2019<span class="docon docon-navigate-external x-hidden-focus"></span></a> soit installé sur le système.

- <a href="https://aka.ms/vs/16/release/vc_redist.x86.exe" target="_blank">Installer pour x86 <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="https://aka.ms/vs/16/release/vc_redist.x64.exe" target="_blank">Installer pour x64 <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="https://aka.ms/vs/16/release/vc_redist.arm64.exe" target="_blank">Installer pour ARMx64<span class="docon docon-navigate-external x-hidden-focus"></span></a>

### <a name="c"></a>C#

[!INCLUDE [Get .NET Speech SDK](get-speech-sdk-dotnet.md)]

Pour l’entrée du microphone, les bibliothèques Media Foundation doivent être installées. Ces bibliothèques font partie de Windows 10 et de Windows Server 2016. Il est possible d’utiliser le SDK Speech sans ces bibliothèques, tant qu’un microphone n’est pas utilisé comme périphérique d’entrée audio.

Les fichiers du kit de développement logiciel (SDK) Speech requis peuvent être déployés dans le même répertoire que votre application. De cette façon, votre application peut accéder directement aux bibliothèques. Veillez à sélectionner la version appropriée (x86/x64) correspondant à votre application.

| Nom                                            | Fonction                                             |
|-------------------------------------------------|------------------------------------------------------|
| `Microsoft.CognitiveServices.Speech.core.dll`   | SDK principal, requis pour le déploiement natif et managé |
| `Microsoft.CognitiveServices.Speech.csharp.dll` | Requis pour le déploiement managé                      |

> [!NOTE]
> Depuis la version 1.3.0, le fichier `Microsoft.CognitiveServices.Speech.csharp.bindings.dll` (fourni dans les versions précédentes) n’est plus nécessaire. La fonctionnalité est désormais intégrée dans le kit SDK principal.

> [!IMPORTANT]
> Pour le projet C# Windows Forms App (.NET Framework), vérifiez que les bibliothèques sont incluses dans les paramètres de déploiement de votre projet. Vous pouvez vérifier cela dans `Properties -> Publish Section`. Cliquez sur le bouton `Application Files` et recherchez les bibliothèques correspondantes dans la liste déroulante. Assurez-vous que la valeur est définie sur `Included`. Visual Studio inclut le fichier lorsque le projet est publié/déployé.

### <a name="c"></a>C++

[!INCLUDE [Get C++ Speech SDK](get-speech-sdk-cpp.md)]

### <a name="python"></a>Python

[!INCLUDE [Get Python Speech SDK](get-speech-sdk-python.md)]

### <a name="java"></a>Java

[!INCLUDE [Get Java Speech SDK](get-speech-sdk-java.md)]

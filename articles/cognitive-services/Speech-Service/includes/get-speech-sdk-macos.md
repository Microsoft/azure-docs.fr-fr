---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: 963405f0874e23553a118081b080a0f30e1942eb
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93135661"
---
Trois SDK Speech sont disponibles pour les développements destinés à macOS.

- Le SDK Speech Objective-C est disponible en natif sous forme de package CocoaPod
- Le SDK Speech .NET peut être utilisé avec **Xamarin.Mac** , car il implémente .NET Standard 2.0
- Le SDK Speech Python est disponible sous forme de module PyPI

> [!TIP]
> Pour plus d’informations sur l’utilisation du SDK Speech Objective-C avec Swift, consultez <a href="https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift" target="_blank">Importation d’Objective-C dans Swift<span class="docon docon-navigate-external x-hidden-focus"></span></a>.

### <a name="system-requirements"></a>Configuration système requise

- macOS version 10.13 ou ultérieure

# <a name="xcode"></a>[Xcode](#tab/mac-xcode)

:::row:::
    :::column span="3":::
        Le package CocoaPod macOS peut être téléchargé et utilisé avec l’IDE <a href="https://apps.apple.com/us/app/xcode/id497799835" target="_blank">Xcode 9.4.1 (ou ultérieur) <span class="docon docon-navigate-external x-hidden-focus"></span></a>. Tout d’abord, <a href="https://aka.ms/csspeech/macosbinary" target="_blank">téléchargez le fichier binaire de CocoaPod<span class="docon docon-navigate-external x-hidden-focus"></span></a>. Extrayez le pod dans le répertoire où vous prévoyez de l’utiliser, créez un *Podfile* et listez le `pod` en tant que `target`.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Xcode" src="https://docs.microsoft.com/media/logos/logo_xcode.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

```
platform :ios, '9.3'
use_frameworks!

target 'MyApp' do
  pod 'MicrosoftCognitiveServicesSpeech', '~> 1.14.0'
end
```

# <a name="xamarinmac"></a>[Xamarin.Mac](#tab/mac-xamarin)

:::row:::
    :::column span="3":::
        Xamarin.Mac expose le SDK macOS complet pour permettre aux développeurs .NET de créer des applications Mac natives en utilisant C#. Pour plus d’informations, consultez <a href="https://docs.microsoft.com/xamarin/mac/" target="_blank">Xamarin.Mac <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Xamarin" src="https://docs.microsoft.com/media/logos/logo_xamarin.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

[!INCLUDE [Get .NET Speech SDK](get-speech-sdk-dotnet.md)]

# <a name="python"></a>[Python](#tab/mac-python)

[!INCLUDE [Get Python Speech SDK](get-speech-sdk-python.md)]

---

#### <a name="additional-resources"></a>Ressources supplémentaires

- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/objectivec/macos" target="_blank">Code source Objective-C du guide de démarrage rapide du SDK Speech pour macOS<span class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/swift/macos" target="_blank">Code source Swift du guide de démarrage rapide du SDK Speech pour macOS<span class="docon docon-navigate-external x-hidden-focus"></span></a>
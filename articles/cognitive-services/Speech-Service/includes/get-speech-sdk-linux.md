---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/03/2020
ms.author: trbye
ms.openlocfilehash: d0298bcd675b1b94999dab3a1ad1c40a6feb7438
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93135561"
---
:::row:::
    :::column span="3":::
        Le SDK Speech prend en charge **Ubuntu 16.04/18.04/20.04** , **Debian 9/10** , **Red Hat Enterprise Linux (RHEL) 7/8** et **CentOS 7/8** uniquement sur les architectures cibles suivantes en cas d’utilisation avec Linux :
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Linux" src="https://docs.microsoft.com/media/logos/logo_linux-color.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

- x86 (Debian/Ubuntu), x64, ARM32 (Debian/Ubuntu) et ARM64 (Debian/Ubuntu) pour le développement de C++
- x64, ARM32 (Debian/Ubuntu) et ARM64 (Debian/Ubuntu) pour Java
- x64, ARM32 (Debian/Ubuntu) et ARM64 (Debian/Ubuntu) pour .NET Core
- x64 pour Python

> [!IMPORTANT]
> Pour C# sur Linux ARM64, le .NET Core 3.x (package dotnet-sdk-3.x) est requis.

### <a name="system-requirements"></a>Configuration système requise

Pour une application native, le Kit de développement logiciel (SDK) Speech s’appuie sur `libMicrosoft.CognitiveServices.Speech.core.so`. Assurez-vous que l’architecture cible (x86, x64) correspond à l’application. Selon la version de Linux, des dépendances supplémentaires peuvent être nécessaires.

- Les bibliothèques partagées de la bibliothèque C de GNU (notamment la bibliothèque de programmation POSIX Threads, `libpthreads`)
- La bibliothèque OpenSSL (`libssl.so.1.0.0` ou `libssl.so.1.0.2`)
- La bibliothèque partagée pour les applications ALSA (`libasound.so.2`)

# <a name="ubuntu-160418042004"></a>[Ubuntu 16.04/18.04/20.04](#tab/ubuntu)

```Bash
sudo apt-get update
sudo apt-get install build-essential libssl1.0.0 libasound2
```

> [!NOTE]
> Si libssl1.0.x n’est pas disponible, installez libssl1.1 à la place.

# <a name="debian-910"></a>[Debian 9/10](#tab/debian)

```Bash
sudo apt-get update
sudo apt-get install build-essential libssl1.0.2 libasound2
```

> [!NOTE]
> Si libssl1.0.x n’est pas disponible, installez libssl1.1 à la place.

# <a name="rhel-78-and-centos-78"></a>[RHEL 7/8 and CentOS 7/8](#tab/rhel-centos)

```Bash
sudo yum update
sudo yum install alsa-lib openssl
```

> [!IMPORTANT]
> - Sur RHEL/CentOS 7, suivez les instructions décrivant [comment configurer RHEL/CentOS 7 pour le Kit de développement logiciel (SDK) Speech](~/articles/cognitive-services/speech-service/how-to-configure-rhel-centos-7.md).
> - Sur RHEL/CentOS 8, suivez les instructions du [Guide pratique pour configurer OpenSSL pour Linux](~/articles/cognitive-services/speech-service/how-to-configure-openssl-linux.md).

---

### <a name="c"></a>C#

[!INCLUDE [Get .NET Speech SDK](get-speech-sdk-dotnet.md)]

### <a name="c"></a>C++

[!INCLUDE [Get C++ Speech SDK](get-speech-sdk-cpp.md)]

### <a name="python"></a>Python

[!INCLUDE [Get Python Speech SDK](get-speech-sdk-python.md)]

### <a name="java"></a>Java

[!INCLUDE [Get Java Speech SDK](get-speech-sdk-java.md)]

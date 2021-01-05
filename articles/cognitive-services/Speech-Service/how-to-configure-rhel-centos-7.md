---
title: Configurer RHEL/CentOS 7 - Service Speech
titleSuffix: Azure Cognitive Services
description: Découvrez comment configurer RHEL/CentOS 7 de manière à utiliser le kit de développement logiciel (SDK) Speech.
services: cognitive-services
author: pankopon
manager: jhakulin
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/02/2020
ms.author: pankopon
ms.openlocfilehash: ba531164e024f96d3bdd23912f3f6e90275edda4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83589735"
---
# <a name="configure-rhelcentos-7-for-speech-sdk"></a>Configurer RHEL/CentOS 7 pour le kit de développement logiciel (SDK) Speech

Red Hat Enterprise Linux (RHEL) 8 x64 et CentOS 8 x64 sont officiellement pris en charge par le kit de développement logiciel (SDK) Speech 1.10.0 et versions ultérieures. Il est également possible d’utiliser le kit de développement logiciel (SDK) Speech sur RHEL/CentOS 7 x64, mais cela implique la mise à jour du compilateur C++ (développement C++) et de la bibliothèque runtime C++ partagée sur votre système.

Pour vérifier la version du compilateur C++, exécutez :

```bash
g++ --version
```

Si le compilateur est installé, la sortie doit se présenter comme suit :

```bash
g++ (GCC) 4.8.5 20150623 (Red Hat 4.8.5-39)
```

Ce message vous indique que la version principale, GCC 4, est installée. Cette version ne prend pas pleinement en charge la version standard C++ 11 utilisée par le kit de développement logiciel (SDK) Speech. Tenter de compiler un programme C++ avec cette version de GCC et les en-têtes du kit de développement logiciel (SDK) Speech se soldera par des erreurs de compilation.

Il est également important de vérifier la version de la bibliothèque runtime C++ partagée (libstdc++). Pour l'essentiel, le kit de développement logiciel (SDK) Speech est implémenté sous forme de bibliothèques C++ natives, ce qui signifie qu’il dépend de libstdc++ quel que soit le langage utilisé pour développer des applications.

Pour rechercher l’emplacement de libstdc++ sur votre système, exécutez :

```bash
ldconfig -p | grep libstdc++
```

La sortie RHEL/CentOS 7 (x64) vanille est la suivante :

```bash
libstdc++.so.6 (libc6,x86-64) => /lib64/libstdc++.so.6
```

Sur la base de ce message, vous vérifierez les définitions de version à l’aide de cette commande :

```bash
strings /lib64/libstdc++.so.6 | egrep "GLIBCXX_|CXXABI_"
```

La sortie doit être :

```bash
...
GLIBCXX_3.4.19
...
CXXABI_1.3.7
...
```

Le kit de développement logiciel (SDK) Speech requiert **CXXABI_1.3.9** et **GLIBCXX_3.4.21**. Pour rechercher ces informations, exécutez `ldd libMicrosoft.CognitiveServices.Speech.core.so` sur les bibliothèques du kit de développement logiciel (SDK) Speech à partir du package Linux.

> [!NOTE]
> Il est recommandé d'installer la version GCC minimale **5.4.0**, avec les bibliothèques runtime correspondantes.

## <a name="example"></a>Exemple

Il s’agit d’un exemple de jeu de commandes qui illustre la configuration de RHEL/CentOS 7 x64 à des fins de développement (C++, C#, Java, Python) avec le kit de développement logiciel (SDK) Speech 1.10.0 ou version ultérieure :

### <a name="1-general-setup"></a>1. Configuration générale

Commencez par installer toutes les dépendances générales :

```bash
# Only run ONE of the following two commands
# - for CentOS 7:
sudo rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm
# - for RHEL 7:
sudo rpm -Uvh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm

# Install development tools and libraries
sudo yum update -y
sudo yum groupinstall -y "Development tools"
sudo yum install -y alsa-lib dotnet-sdk-2.1 java-1.8.0-openjdk-devel openssl python3
sudo yum install -y gstreamer1 gstreamer1-plugins-base gstreamer1-plugins-good gstreamer1-plugins-bad-free gstreamer1-plugins-ugly-free
```

### <a name="2-cc-compiler-and-runtime-libraries"></a>2. Compilateur C/C++ et bibliothèques du runtime

Installez les packages requis à l’aide de la commande suivante :

```bash
sudo yum install -y gmp-devel mpfr-devel libmpc-devel
```

> [!NOTE]
> Le package libmpc-devel est déconseillé depuis la mise à jour RHEL 7.8. Si la sortie de la commande précédente contient un message
>
> ```bash
> No package libmpc-devel available.
> ```
>
> les fichiers nécessaires doivent être installés à partir des sources d’origine. Exécutez les commandes suivantes :
>
> ```bash
> curl https://ftp.gnu.org/gnu/mpc/mpc-1.1.0.tar.gz -O
> tar zxf mpc-1.1.0.tar.gz
> mkdir mpc-1.1.0-build && cd mpc-1.1.0-build
> ../mpc-1.1.0/configure --prefix=/usr/local --libdir=/usr/local/lib64
> make -j$(nproc)
> sudo make install-strip
> ```

Ensuite, mettez à jour le compilateur et les bibliothèques du runtime :

```bash
# Build GCC 5.4.0 and runtimes and install them under /usr/local
curl https://ftp.gnu.org/gnu/gcc/gcc-5.4.0/gcc-5.4.0.tar.bz2 -O
tar jxf gcc-5.4.0.tar.bz2
mkdir gcc-5.4.0-build && cd gcc-5.4.0-build
../gcc-5.4.0/configure --enable-languages=c,c++ --disable-bootstrap --disable-multilib --prefix=/usr/local
make -j$(nproc)
sudo make install-strip
```

Si le compilateur et les bibliothèques mis à jour doivent être déployés sur plusieurs machines, vous pouvez simplement les copier à partir de `/usr/local` vers d’autres machines. Si seules les bibliothèques du runtime sont nécessaires, les fichiers figurant dans `/usr/local/lib64` seront suffisants.

### <a name="3-environment-settings"></a>3. Paramètres d'environnement

Exécutez les commandes suivantes pour achever la configuration :

```bash
# Set SSL cert file location
# (this is required for any development/testing with Speech SDK)
export SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt

# Add updated C/C++ runtimes to the library path
# (this is required for any development/testing with Speech SDK)
export LD_LIBRARY_PATH=/usr/local/lib64:$LD_LIBRARY_PATH

# For C++ development only:
# - add the updated compiler to PATH
#   (note, /usr/local/bin should be already first in PATH on vanilla systems)
# - add Speech SDK libraries from the Linux tar package to LD_LIBRARY_PATH
#   (note, use the actual path to extracted files!)
export PATH=/usr/local/bin:$PATH
hash -r # reset cached paths in the current shell session just in case
export LD_LIBRARY_PATH=/path/to/extracted/SpeechSDK-Linux-1.10.0/lib/x64:$LD_LIBRARY_PATH

# For Python: install the Speech SDK module
python3 -m pip install azure-cognitiveservices-speech --user
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [À propos du kit SDK Speech](speech-sdk.md)

---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 10/15/2020
ms.author: trbye
ms.openlocfilehash: d50d4c554f47629f6e04adf957e02f8ffcc48fe5
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96509197"
---
Ce guide explique comment installer le [Kit de développement logiciel (SDK) Speech](~/articles/cognitive-services/speech-service/speech-sdk.md) pour Python. Si vous voulez simplement le nom du package pour commencer seul, exécutez `pip install azure-cognitiveservices-speech`.

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

- Le package Python du kit SDK Speech est disponible pour les systèmes d’exploitation suivants :
  - Windows : x64 et x86
  - Mac : macOS X version 10.12 ou ultérieure
  - Linux : consultez la liste des [architectures cibles et distributions Linux prises en charge](~/articles/cognitive-services/speech-service/speech-sdk.md).

## <a name="prerequisites"></a>Prérequis

- Les plateformes Linux prises en charge nécessitent l’installation de certaines bibliothèques (`libssl` pour la prise en charge du protocole SSL et `libasound2` pour la prise en charge du son). Reportez-vous à votre distribution ci-dessous pour connaître les commandes nécessaires à l’installation des versions appropriées de ces bibliothèques.

  - Sur Ubuntu/Debian, exécutez les commandes suivantes pour installer les packages requis :

    ```sh
    sudo apt-get update
    sudo apt-get install build-essential libssl1.0.0 libasound2
    ```

    Si libssl1.0.0 n’est pas disponible, installez libssl1.0.x (où x est supérieur à 0) ou libssl1.1 à la place.

  - Sur RHEL/CentOS, exécutez les commandes suivantes pour installer les packages nécessaires :

    ```sh
    sudo yum update
    sudo yum install alsa-lib openssl python3
    ```

> [!NOTE]
> - Sur RHEL/CentOS 7, suivez les instructions décrivant [comment configurer RHEL/CentOS 7 pour le Kit de développement logiciel (SDK) Speech](~/articles/cognitive-services/speech-service/how-to-configure-rhel-centos-7.md).
> - Sur RHEL/CentOS 8, suivez les instructions du [Guide pratique pour configurer OpenSSL pour Linux](~/articles/cognitive-services/speech-service/how-to-configure-openssl-linux.md).

- Sur Windows, vous avez besoin de [Microsoft Redistributable Visual C++ pour Visual Studio 2019](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads) pour votre plateforme. Notez que la première installation peut nécessiter le redémarrage de Windows avant de continuer à suivre ce guide.
- Enfin, vous aurez besoin de [Python 3.5 à 3.8](https://www.python.org/downloads/). Pour vérifier votre installation, ouvrez une invite de commandes, tapez la commande `python --version` et vérifiez le résultat. Si elle est correctement installée, vous obtenez une réponse « Python 3.5.1 » ou similaire.

## <a name="install-the-speech-sdk-from-pypi"></a>Installer le SDK Speech à partir de PyPI

Si vous utilisez votre propre environnement ou des outils de génération, exécutez la commande suivante pour installer le SDK Speech à partir de [PyPI](https://pypi.org/). Pour les utilisateurs de Visual Studio Code, passez à la sous-section suivante pour suivre une installation guidée.

```sh
pip install azure-cognitiveservices-speech
```

Si vous utilisez macOS, il se peut que vous deviez exécuter la commande suivante pour faire fonctionner la commande `pip` ci-dessus :

```sh
python3 -m pip install --upgrade pip
```

Une fois que vous avez correctement utilisé `pip` pour installer `azure-cognitiveservices-speech`, vous pouvez utiliser le Kit de développement logiciel (SDK) Speech en important l’espace de noms dans vos projets Python.

```py
import azure.cognitiveservices.speech as speechsdk
```

## <a name="install-the-speech-sdk-using-visual-studio-code"></a>Installer le Kit de développement logiciel (SDK) Speech avec Visual Studio Code

1. Téléchargez et installez la dernière version prise en charge de [Python](https://www.python.org/downloads/) pour votre plateforme (3.5 à 3.8).
   - Les utilisateurs de Windows veillent à sélectionner « Ajouter Python à votre chemin d’accès » au cours du processus d’installation.
1. Téléchargez et installez [Visual Studio Code](https://code.visualstudio.com/Download).
1. Ouvrez Visual Studio Code et installez l’extension Python. Sélectionnez **Fichier** > **Préférences** > **Extensions** dans le menu. Recherchez **Python**, puis cliquez sur **Installer**.

   ![Installer l’extension Python](~/articles/cognitive-services/speech-service/media/sdk/qs-python-vscode-python-extension.png)

1. À partir de Visual Studio Code, installez le package Python du Kit de développement logiciel (SDK) Speech à partir de la ligne de commande intégrée :
   1. Ouvrez un terminal (dans les menus déroulants, **Terminal** > **Nouveau terminal**)
   1. Dans le terminal qui s’ouvre, entrez la commande `python -m pip install azure-cognitiveservices-speech`

Si vous débutez avec Visual Studio Code, reportez-vous à la [Documentation Visual Studio Code](https://code.visualstudio.com/docs) plus complète. Pour en savoir plus sur Visual Studio Code et Python, voir le [Didacticiel Python pour Visual Studio Code](https://code.visualstudio.com/docs/python/python-tutorial).

## <a name="support-and-updates"></a>Mises à jour et support technique

Les mises à jour du package Python du kit SDK Speech sont distribuées via PyPI et annoncées dans les [notes de publication](~/articles/cognitive-services/speech-service/releasenotes.md).
Si une nouvelle version est disponible, vous pouvez effectuer la mise à jour avec la commande `pip install --upgrade azure-cognitiveservices-speech`.
Vérifiez la version actuellement installée en inspectant la variable `azure.cognitiveservices.speech.__version__`.

Si vous avez un problème ou si une fonctionnalité est manquante, consultez [Options d’aide et de support](../../../../cognitive-services-support-options.md?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext%253fcontext%253d%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext).

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [windows](../quickstart-list.md)]
---
title: Guide de démarrage rapide sur la création de mot clé - Service de reconnaissance vocale
titleSuffix: Azure Cognitive Services
description: Votre appareil est toujours à l’écoute d’un mot (ou d’une phrase) clé. Quand l’utilisateur prononce le mot clé, l’appareil envoie tous les sons suivants vers le cloud, jusqu’à ce que l’utilisateur arrête de parler. Pour différencier votre appareil et renforcer votre marque, vous pouvez personnaliser votre mot clé.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/03/2020
ms.author: trbye
ms.custom: devx-track-csharp
zone_pivot_groups: keyword-quickstart
ms.openlocfilehash: 49ac70b6881085f48c8bc3a12e31e4a1aa220c6a
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021947"
---
# <a name="get-started-with-custom-keyword"></a>Bien démarrer avec Mot clé personnalisé

Dans ce guide de démarrage rapide, vous allez découvrir les principes de base de l’utilisation des mots clés personnalisés, en utilisant Speech Studio et le kit SDK Speech. Un mot clé est un mot ou une expression courte qui permet d’activer la fonction vocale de votre produit. Vous créez des modèles de mots clés dans Speech Studio, puis vous exportez un fichier de modèle que vous utilisez avec le kit SDK Speech dans vos applications.

## <a name="prerequisites"></a>Prérequis

Les étapes décrites dans cet article nécessitent un abonnement Speech et le kit SDK Speech. Si vous n’avez pas encore d’abonnement, [essayez gratuitement le service Speech](overview.md#try-the-speech-service-for-free). Pour obtenir le kit SDK, consultez le [guide d’installation](quickstarts/setup-platform.md) pour votre plateforme.

## <a name="create-a-keyword-in-speech-studio"></a>Créer un mot clé dans Speech Studio

Avant de pouvoir utiliser un mot clé personnalisé, vous devez créer ce mot clé dans la page [Mot clé personnalisé](https://aka.ms/sdsdk-wakewordportal), dans [Speech Studio](https://aka.ms/sdsdk-speechportal). Une fois que vous avez fourni un mot clé, le logiciel génère un fichier `.table` que vous pouvez utiliser avec le kit SDK Speech.

> [!IMPORTANT]
> Les modèles de mots clés personnalisés et les fichiers `.table` résultants peuvent **uniquement** être créés dans Speech Studio.
> Vous ne pouvez pas créer de mots clés personnalisés à partir du kit SDK ni avec des appels REST.

1. Accédez au [Speech Studio](https://aka.ms/sdsdk-speechportal) et **connectez-vous**, ou, si vous n’avez pas d’abonnement vocal, choisissez [**Créer un abonnement**](https://go.microsoft.com/fwlink/?linkid=2086754).

1. Sur la page [Mot clé personnalisé](https://aka.ms/sdsdk-wakewordportal), créez un **Nouveau projet**. 

1. Entrez un **Nom** et une **Description** facultative, puis sélectionnez la langue. Vous avez besoin d’un projet par langue et la prise en charge est actuellement limitée à la langue `en-US` (anglais des États-Unis).

    ![Décrire votre projet de mot clé](media/custom-keyword/custom-kws-portal-new-project.png)

1. Sélectionnez votre projet dans la liste. 

    ![Sélectionner votre projet de mot clé](media/custom-keyword/custom-kws-portal-project-list.png)

1. Pour créer un nouveau modèle de mot clé, cliquez sur **Entraîner le modèle**.

1. Entrez un **nom** pour le modèle, une **description** facultative et le **mot clé** de votre choix, puis cliquez sur **Suivant**. Consultez les [instructions](./custom-keyword-overview.md#choose-an-effective-keyword) sur le choix d’un mot clé efficace.

    ![Entrez votre mot clé](media/custom-keyword/custom-kws-portal-new-model.png)

1. Le portail crée des propositions de prononciation pour votre mot clé. Écoutez chaque prononciation en cliquant sur les boutons de lecture, supprimez les coches en regard des prononciations incorrectes. Une fois que seules les bonnes prononciations sont cochées, cliquez sur **Entraîner** pour commencer à générer le modèle de mot clé. 

    ![Capture d’écran montrant où vous choisissez les bonnes prononciations.](media/custom-keyword/custom-kws-portal-choose-prons.png)

1. La génération du modèle peut prendre jusqu’à trente minutes. La liste de mots clés passe de **Traitement en cours** à **Terminé** lorsque le modèle est terminé. Vous pouvez ensuite télécharger le fichier.

    ![Vérifier votre mot clé](media/custom-keyword/custom-kws-portal-download-model.png)

1. Le fichier téléchargé est une archive `.zip`. Extrayez l’archive et un fichier avec l’extension `.table` apparaît. Il s’agit du fichier que vous allez utiliser avec le kit SDK dans la section suivante. Veillez donc à noter son chemin d’accès. Le nom de fichier reflète votre nom de mot clé. Par exemple, un mot clé **Activate device** (Activer l’appareil) a le nom de fichier `Activate_device.table`.

## <a name="use-a-keyword-model-with-the-sdk"></a>Utiliser un modèle de mot clé avec le kit SDK

::: zone pivot="programming-language-csharp"
[!INCLUDE [C# Basics include](includes/how-to/keyword-recognition/keyword-basics-csharp.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python Basics include](includes/how-to/keyword-recognition/keyword-basics-python.md)]
::: zone-end

::: zone pivot="programming-languages-objectivec-swift"
[!INCLUDE [ObjectiveC/Swift Basics include](includes/how-to/keyword-recognition/keyword-basics-objc.md)]
::: zone-end

## <a name="next-steps"></a>Étapes suivantes

Testez votre mot clé personnalisé avec le [Guide de démarrage rapide du kit SDK Speech Devices](./speech-devices-sdk-quickstart.md?pivots=platform-android).
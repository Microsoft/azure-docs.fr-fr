---
title: Améliorer la synthèse avec Custom Voice – Service Speech
titleSuffix: Azure Cognitive Services
description: Custom Voice est un ensemble d’outils en ligne qui vous permet de créer une voix unique reconnaissable entre toutes pour votre marque. Pour commencer, vous n’avez besoin que de quelques fichiers audio et des transcriptions associées. Suivez les liens ci-dessous pour commencer à créer une expérience de reconnaissance vocale personnalisée.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/17/2020
ms.author: trbye
ms.openlocfilehash: 6ae164b1300119481e495e647dd491beafdd5241
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025076"
---
# <a name="get-started-with-custom-voice"></a>Bien démarrer avec Custom Voice

[Custom Voice](https://aka.ms/customvoice) est un ensemble d’outils en ligne qui vous permettent de créer une voix unique reconnaissable entre toutes pour votre marque. Pour commencer, vous n’avez besoin que de quelques fichiers audio et des transcriptions associées. Suivez les liens ci-dessous pour commencer à créer une expérience de synthèse vocale personnalisée.

## <a name="whats-in-custom-voice"></a>Qu’y a-t-il dans Custom Voice ?

Avant de commencer avec Custom Voice, vous aurez besoin d’un compte Azure et d’un abonnement au service Speech. Une fois que vous aurez créé un compte, vous pourrez préparer vos données, entraîner et tester vos modèles, évaluer la qualité de la voix et en dernier lieu déployer votre modèle vocal personnalisé.

Le diagramme ci-dessous présente les étapes à suivre pour créer un modèle vocal personnalisé à partir du [portail Custom Voice](https://aka.ms/customvoice). Suivez les liens pour en savoir plus.

![Schéma de l’architecture Custom Voice](media/custom-voice/custom-voice-diagram.png)

1. [S’abonner et créer un projet](#set-up-your-azure-account) – Créez un compte Azure ainsi qu’un abonnement au service Speech. Cet abonnement unifié vous donne accès à la reconnaissance vocale, à la synthèse vocale, à la traduction vocale et au portail Custom Voice. Ensuite, servez-vous de votre abonnement Speech Services pour créer votre premier projet Custom Voice.

2. [Charger des données](how-to-custom-voice-create-voice.md#upload-your-datasets) – Chargez des données (audio et texte) via le portail Custom Voice ou l’API Custom Voice. Sur le portail, vous pouvez examiner et évaluer les scores de prononciation et les rapports signal/bruit. Pour plus d’informations, consultez [Guide pratique pour préparer des données pour Custom Voice](how-to-custom-voice-prepare-data.md).

3. [Entraîner votre modèle](how-to-custom-voice-create-voice.md#build-your-custom-voice-model) – Servez-vous de vos données pour créer un modèle vocal personnalisé pour la synthèse vocale. Vous pouvez entraîner un modèle dans différentes langues. Après l’entraînement, testez votre modèle. Si vous êtes satisfait du résultat, vous pouvez le déployer.

4. [Déployer votre modèle](how-to-custom-voice-create-voice.md#create-and-use-a-custom-voice-endpoint) – Créez un point de terminaison personnalisé pour votre modèle vocal et utilisez-le pour la synthèse vocale dans vos produits, outils et applications.

## <a name="custom-neural-voices"></a>Voix neurales personnalisées

La fonctionnalité de personnalisation de la voix neurale est actuellement en version préliminaire publique, limitée aux clients sélectionnés. Renseignez ce [formulaire d’application](https://go.microsoft.com/fwlink/?linkid=2108737) pour commencer.

> [!NOTE]
> Dans le cadre de l’engagement de Microsoft à concevoir une IA responsable, notre intention est de protéger les droits des individus et de la société, et de favoriser des interactions transparentes entre humains et ordinateurs. C’est la raison pour laquelle la voix neuronale personnalisée n’est généralement pas disponible pour tous les clients. Vous pouvez accéder à la technologie uniquement après que vos applications ont été examinées et que vous vous êtes engagé à l’utiliser en conformité avec nos principes déontologiques. En savoir plus sur notre [processus de vérification des applications](./concepts-gating-overview.md).

## <a name="set-up-your-azure-account"></a>Configurer votre compte Azure

Vous avez besoin d’un abonnement au service Speech pour pouvoir créer un modèle personnalisé à partir du portail Custom Speech. Suivez ces instructions pour créer un abonnement Speech standard dans Azure. Si vous n’avez pas de compte Azure, vous pouvez vous inscrire pour en créer un.  

Une fois que vous avez créé un compte Azure et un abonnement Speech, vous devez vous connecter au portail Custom Voice et connecter votre abonnement.

1. Obtenez votre clé d’abonnement Speech sur le portail Azure.
2. Connectez-vous au [portail Custom Voice](https://aka.ms/custom-voice).
3. Sélectionnez votre abonnement et créez un projet Speech.
4. Si vous voulez basculer vers un autre abonnement Speech, utilisez l’icône représentant une roue dentée dans le volet de navigation du haut.

> [!NOTE]
> Pour pouvoir utiliser le service, vous devez disposer d’une clé F0 ou S0 créée dans Azure.

## <a name="how-to-create-a-project"></a>Comment créer un projet

Le contenu que représentent les données, les modèles, les tests et les points de terminaison sont organisées en **projets** sur le portail Custom Voice. Chaque projet est spécifique à un pays/langue et au genre de la voix que vous voulez créer. Par exemple, vous pouvez créer un projet de voix féminine pour les bots conversationnels de votre centre d'appels en anglais des États-Unis.

Pour créer votre premier projet, sélectionnez l’onglet **Text-to-Speech/Custom Voice**, puis cliquez sur **New Project**. Suivez les instructions fournies par l’Assistant pour créer votre projet. Une fois le projet créé, quatre onglets sont affichés : **Data**, **Training**, **Testing** et **Deployment**. Utilisez les liens fournis dans [Étapes suivantes](#next-steps) pour savoir comment utiliser chaque onglet.

> [!IMPORTANT]
> Le [portail Custom Voice](https://aka.ms/custom-voice) a été récemment mis à jour. Si vous avez créé antérieurement des données, modèles, tests et points de terminaison publiés sur le portail CRIS.ai ou à l’aide d’API, vous devez créer un projet sur le nouveau portail pour vous connecter à ces anciennes entités.

## <a name="next-steps"></a>Étapes suivantes

- [Préparer des données pour Custom Voice](how-to-custom-voice-prepare-data.md)
- [Créer une voix personnalisée Custom Voice](how-to-custom-voice-create-voice.md)
- [Guide : Enregistrer des échantillons vocaux](record-custom-voice-samples.md)
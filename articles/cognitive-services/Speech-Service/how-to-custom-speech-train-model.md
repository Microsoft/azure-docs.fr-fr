---
title: Entraîner et déployer un modèle Custom Speech - Service Speech
titleSuffix: Azure Cognitive Services
description: Découvrez comment effectuer l’apprentissage de modèles Custom Speech et déployer ceux-ci. L’apprentissage d’un modèle de reconnaissance vocale peut améliorer l’exactitude de reconnaissance du modèle de référence Microsoft ou d’un modèle personnalisé.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: trbye
ms.openlocfilehash: 130cd643856b38471eac6d6869cdc1ed8b0bcd2e
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499150"
---
# <a name="train-and-deploy-a-custom-speech-model"></a>Entraîner et déployer un modèle Custom Speech

Cet article explique comment effectuer l’apprentissage de modèles Custom Speech et déployer ceux-ci. L’apprentissage d’un modèle de reconnaissance vocale peut améliorer l’exactitude de reconnaissance du modèle de référence Microsoft. Vous utilisez des transcriptions étiquetées à la main et un texte associé pour effectuer l’apprentissage d’un modèle. Ces jeux de données, associés à des données audio chargées précédemment, permettent d’affiner le modèle de reconnaissance vocale et d’en effectuer l’apprentissage.

## <a name="use-training-to-resolve-accuracy-problems"></a>Utiliser un apprentissage pour résoudre les problèmes d’exactitude

Si vous rencontrez des problèmes de reconnaissance avec un modèle de base, vous pouvez utiliser des transcriptions étiquetées à la main et des données associées pour effectuer l’apprentissage d’un modèle personnalisé afin d’améliorer l’exactitude. Servez-vous de ce tableau pour déterminer le jeu de données à utiliser pour résoudre vos problèmes :

| Cas d’utilisation | Type de données |
| -------- | --------- |
| Améliorez l’exactitude de reconnaissance du vocabulaire ou de la grammaire spécifiques d’un secteur, par exemple, de la terminologie médicale ou du jargon informatique. | Texte associé (phrases/énoncés) |
| Définissez les formes phonétique et affichée d’un mot ou d’un terme dont la prononciation n’est pas standard, par exemple, de noms de produits ou d’acronymes. | Texte associé (prononciation) |
| Améliorez la précision de la reconnaissance des styles de discours, des accents ou des bruits de fond spécifiques | Transcriptions audio + étiquetées à la main |

## <a name="train-and-evaluate-a-model"></a>Entraîner et évaluer un modèle

La première étape pour entraîner un modèle consiste à charger des données d’entraînement. Consultez [Préparer et tester les données](./how-to-custom-speech-test-and-train.md) pour obtenir des instructions pas à pas sur la manière de préparer des transcriptions étiquetées à la main et du texte associé (énoncés et prononciations). Après avoir chargé des données d’apprentissage, suivez les instructions suivantes pour commencer à effectuer l’apprentissage de votre modèle :

1. Connectez-vous au [portail Custom Speech](https://speech.microsoft.com/customspeech).
2. Accédez à **Synthèse vocale** > **Custom Speech** >  **[nom du projet]**  > **Formation**.
3. Sélectionnez **Effectuer l’apprentissage du modèle**.
4. Attribuez à votre apprentissage un **Nom** et une **Description**.
5. Dans la liste **Scénario et modèle de référence**, sélectionnez le scénario qui correspond le mieux à votre domaine. Si hésitez sur le choix du scénario, sélectionnez **Général**. Le modèle de référence est le point de départ de votre entraînement. Le modèle le plus récent est généralement le meilleur choix.
6. Dans la page **Sélectionner les données d’apprentissage**, choisissez un ou plusieurs jeux de données de transcription étiquetée à la main à utiliser pour l’apprentissage.
7. Une fois l’apprentissage du modèle terminé, vous pouvez effectuer des tests d’exactitude sur celui-ci. Cette étape est facultative.
8. Sélectionnez **Create** pour générer votre modèle personnalisé.

La table **Formation** affiche une nouvelle entrée correspondant au nouveau modèle. Elle indique également l’état : **En cours de traitement**, **Réussite** ou **Échec**.

Consultez le [Guide pratique](how-to-custom-speech-evaluate-data.md) sur l’évaluation et l’amélioration de la justesse du modèle Custom Speech. Si vous choisissez de tester l’exactitude, il est important de sélectionner un jeu de données acoustique différent de celui que vous avez utilisé avec votre modèle pour vous faire une idée réaliste des performances du modèle.

## <a name="deploy-a-custom-model"></a>Déployer un modèle personnalisé

Après avoir chargé et inspecté les données, évalué l’exactitude et effectué l’apprentissage d’un modèle personnalisé, vous pouvez déployer un point de terminaison personnalisé à utiliser avec vos applications, outils et produits. 

Pour créer un point de terminaison personnalisé, connectez-vous au [portail Custom Speech](https://speech.microsoft.com/customspeech). Sélectionnez **Déploiement** dans le menu **Custom Speech** en haut de la page. S’il s’agit de votre première exécution, vous remarquerez que la table ne contient aucun point de terminaison. Une fois que vous avez créé un point de terminaison, vous utilisez cette page pour effectuer le suivi de chaque point de terminaison déployé.

Ensuite, sélectionnez **Ajouter un point de terminaison** et entrez un **nom** et une **description** pour votre point de terminaison personnalisé. Sélectionnez ensuite le modèle personnalisé que vous souhaitez associer au point de terminaison.  Vous pouvez également activer la journalisation à partir de cette page. La journalisation vous permet de surveiller le trafic du point de terminaison. Si elle est désactivée, le trafic n’est pas enregistré.

![Capture d’écran montrant la page Nouveau point de terminaison.](./media/custom-speech/custom-speech-deploy-model.png)

> [!NOTE]
> N’oubliez pas d’accepter les conditions d’utilisation et la tarification.

Ensuite, sélectionnez **Créer**. Cette action vous renvoie à la page **Déploiement**. La table inclut désormais une entrée qui correspond à votre point de terminaison personnalisé. L’état du point de terminaison indique son état actuel. L’instanciation d’un nouveau point de terminaison avec vos modèles personnalisés peut prendre jusqu’à 30 minutes. Lorsque l’état du déploiement est **Complete**, le point de terminaison est prêt à être utilisé.

Une fois le point de terminaison déployé, son nom s’affiche sous forme de lien. Sélectionnez ce lien pour afficher les informations spécifiques de votre point de terminaison, telles que sa clé, son URL et un exemple de code.

## <a name="view-logging-data"></a>Afficher les données de journalisation

Les données de journalisation sont disponibles en téléchargement sous **Point de terminaison** > **Détails**.
> [!NOTE]
>Les données de journalisation sont disponibles pendant 30 jours sur le stockage détenu par Microsoft. Ensuite, elles seront supprimées. Si un compte de stockage appartenant à un client est lié à l’abonnement Cognitive Services, les données de journalisation ne seront pas automatiquement supprimées.

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment utiliser votre modèle personnalisé](how-to-specify-source-language.md)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Préparer et tester les données](./how-to-custom-speech-test-and-train.md)
- [Inspecter les données](how-to-custom-speech-inspect-data.md)
- [Évaluer les données](how-to-custom-speech-evaluate-data.md)

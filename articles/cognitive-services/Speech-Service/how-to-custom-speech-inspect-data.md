---
title: Inspecter la qualité des données pour Custom Speech - Service Speech
titleSuffix: Azure Cognitive Services
description: Custom Speech propose des outils qui vous permettent d’inspecter visuellement la qualité de la reconnaissance d’un modèle en comparant les données audio au résultat de la reconnaissance correspondante. Vous pouvez lire le contenu audio chargé pour déterminer si le résultat proposé de la reconnaissance est correct.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/23/2020
ms.author: erhopf
ms.openlocfilehash: 87bc411903604a7068f346604853a2a61c9a6f1c
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499184"
---
# <a name="inspect-custom-speech-data"></a>Inspecter des données Custom Speech

> [!NOTE]
> Cette page suppose que vous avez lu [Préparer des données de test pour Custom Speech](./how-to-custom-speech-test-and-train.md) et que vous avez chargé un jeu de données pour l’inspecter.

Custom Speech propose des outils qui vous permettent d’inspecter visuellement la qualité de la reconnaissance d’un modèle en comparant les données audio au résultat de la reconnaissance correspondante. À partir du [portail Custom Speech](https://speech.microsoft.com/customspeech), vous pouvez lire le contenu audio chargé et déterminer si le résultat proposé de la reconnaissance est correct. Cet outil vous permet d’inspecter la qualité du modèle de reconnaissance vocale de base de Microsoft, d’inspecter un modèle personnalisé entraîné ou de comparer la transcription selon deux modèles.

Dans ce document, vous allez apprendre à inspecter visuellement la qualité du modèle de reconnaissance vocale de référence de Microsoft et/ou de modèles personnalisés que vous avez formés. Vous allez également apprendre à utiliser l’éditeur de transcription en ligne pour créer et affiner les jeux de données audio étiquetés.

## <a name="create-a-test"></a>Créer un test

Pour créer un test, suivez ces instructions :

1. Connectez-vous au [portail Custom Speech](https://speech.microsoft.com/customspeech).
2. Accédez à **Synthèse vocale > Custom Speech > [nom du projet] > Test en cours**.
3. Cliquez sur **Add Test** (Ajouter un test).
4. Sélectionnez **Inspect quality (Audio-only data)** [Inspecter la qualité (données audio uniquement)]. Nommez et décrivez le test et sélectionnez votre jeu de données audio.
5. Sélectionnez les modèles que vous voulez tester (deux au maximum).
6. Cliquez sur **Créer**.

Une fois qu’un test a été créé avec succès, vous pouvez voir comment un modèle transcrit le jeu de données audio que vous avez spécifié ou comparer les résultats de deux modèles côte à côte.

[!INCLUDE [service-pricing-advisory](includes/service-pricing-advisory.md)]

## <a name="side-by-side-model-comparisons"></a>Comparaisons côte à côte de modèles

Quand l’état du test est _Succeeded_ (Opération réussie), cliquez sur le nom du test pour en voir les détails. Cette page de détails liste tous les énoncés de votre jeu de données et affiche les résultats de reconnaissance des deux modèles que vous comparez.

Pour inspecter plus facilement la comparaison côte à côte, vous pouvez voir les différents types d’erreurs (insertion, suppression et substitution). En écoutant le contenu audio et en comparant les résultats de la reconnaissance dans chaque colonne (transcription étiquetée à la main et résultats des deux modèles de reconnaissance vocale), vous pouvez identifier le modèle qui répond à vos besoins et à quel niveau il est nécessaire d’apporter des améliorations.

Le test de modèles côte à côte est utile pour valider le modèle de reconnaissance vocale le mieux adapté à une application. Pour obtenir une mesure objective de la précision, ce qui nécessite que le contenu audio soit transcrit, suivez les instructions fournies dans [Évaluer la précision](how-to-custom-speech-evaluate-data.md).

## <a name="online-transcription-editor"></a>Éditeur de transcription en ligne

L’éditeur de transcription en ligne vous permet de travailler facilement avec des transcriptions audio dans Custom Speech. Les principaux cas d’usage de l’éditeur sont les suivants : 

* Vous n’avez que des données audio, mais vous souhaitez créer de toutes pièces des jeux de données audio et étiquetés à la main précis afin de les utiliser dans la formation des modèles.
* Vous avez déjà des jeux de données audio et étiquetés à la main, mais il existe des erreurs ou des défauts dans la transcription. L’éditeur vous permet de modifier rapidement les transcriptions pour obtenir la meilleure exactitude de formation.

La seule condition pour utiliser l’éditeur de transcription est d’avoir des données audio chargées (soit audio uniquement, soit audio + transcription).

### <a name="import-datasets-to-editor"></a>Importer des jeux de donnés dans l’éditeur

Pour importer des données dans l’éditeur, accédez d’abord à **Custom Speech > [votre projet] > Éditeur**.

![Onglet Modifier](media/custom-speech/custom-speech-editor-detail.png)

Ensuite, procédez comme suit pour importer des données.

1. Cliquez sur **Importer des données**.
1. Créez un ou plusieurs jeux de données et attribuez-leur une description.
1. Sélectionnez des jeux de données. Les sélections multiples sont prises en charge, et vous pouvez sélectionner des données audio uniquement ainsi que des données audio + étiquetées à la main.
1. Pour les données audio uniquement, vous pouvez éventuellement utiliser les modèles par défaut pour générer systématiquement une transcription automatique après l’importation dans l’éditeur.
1. Cliquez sur **Importer**.

Une fois les données importées, vous pouvez cliquer sur les jeux de données et commencer leur modification.

> [!TIP]
> Vous pouvez également importer des jeux de données directement dans l’éditeur en sélectionnant des jeux de données, puis en cliquant sur **Exporter vers l’éditeur**.

### <a name="edit-transcription-by-listening-to-audio"></a>Modifier la transcription en écoutant l’audio

Une fois le chargement des données réussi, cliquez sur le nom de chaque élément pour afficher les détails des données. Vous pouvez également utiliser **Précédent** et **Suivant** pour vous déplacer entre chaque fichier.

La page de détails répertorie tous les segments de chaque fichier audio, et vous pouvez cliquer sur l’énoncé souhaité. Pour chaque énoncé, vous pouvez lire l’audio et examiner les transcriptions, puis modifier les transcriptions si vous trouvez des erreurs d’insertion, de suppression ou de substitution. Pour plus d’informations sur les types d’erreurs, consultez le [guide pratique relatif à l’évaluation des données](how-to-custom-speech-evaluate-data.md).

![Page de l’éditeur](media/custom-speech/custom-speech-editor.png)

Une fois les modifications effectuées, cliquez sur le bouton **Enregistrer**.

### <a name="export-datasets-from-the-editor"></a>Exporter des jeux de données à partir de l’éditeur

Pour exporter des jeux de données vers l’onglet **Données**, accédez à la page de détails des données, puis cliquez sur le bouton **Exporter** pour exporter tous les fichiers en tant que nouveau jeu de données. Vous pouvez également filtrer les fichiers en fonction de l’heure de la dernière modification, des durées audio, etc. pour sélectionner partiellement les fichiers souhaités. 

![Exporter des données](media/custom-speech/custom-speech-editor-export.png)

Les fichiers exportés vers Données seront utilisés comme un tout nouveau jeu de données et ne modifieront aucune des entités de données/formation/test existantes.

## <a name="next-steps"></a>Étapes suivantes

- [Évaluer les données](how-to-custom-speech-evaluate-data.md)
- [Entraîner un modèle](how-to-custom-speech-train-model.md)
- [Améliorer un modèle](./how-to-custom-speech-evaluate-data.md)
- [Déployer un modèle](./how-to-custom-speech-train-model.md)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Préparer les données de test pour Custom Speech](./how-to-custom-speech-test-and-train.md)
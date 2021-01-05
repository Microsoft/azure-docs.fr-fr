---
title: Créer une voix personnalisée Custom Voice - Service Speech
titleSuffix: Azure Cognitive Services
description: Pour charger vos données, accédez au portail Custom Voice. Créez ou sélectionnez un projet Custom Voice. Il doit avoir des paramètres régionaux et des propriétés de langue et de sexe adaptés aux données qui seront utilisées pour l’apprentissage de votre voix.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: erhopf
ms.openlocfilehash: 3747033fcaf65e0c6da07e9f1bb625771958bb4f
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91319059"
---
# <a name="create-a-custom-voice"></a>Créer une voix personnalisée Custom Voice

Dans [Préparer des données pour Custom Voice](how-to-custom-voice-prepare-data.md), nous avons décrit les différents types de données permettant d’entraîner une voix personnalisée, ainsi que les exigences de ces formats. Dès que vous avez préparé vos données, vous pouvez commencer à les charger sur le [portail Custom Voice](https://aka.ms/custom-voice-portal) ou avec l’API d’apprentissage Custom Voice. Nous décrivons ici les étapes de l’apprentissage d’une voix personnalisée sur le portail.

> [!NOTE]
> Cette page suppose d’avoir lu [Bien démarrer avec Custom Voice](how-to-custom-voice.md) et [Préparer les données pour Custom Voice](how-to-custom-voice-prepare-data.md) et d’avoir créé un projet Custom Voice.

Vérifier les langues prises en charge par Custom Voice : [langues de personnalisation](language-support.md#customization).

## <a name="upload-your-datasets"></a>Charger vos jeux de données

Pour charger vos données, accédez au [portail Custom Voice](https://aka.ms/custom-voice-portal). Créez ou sélectionnez un projet Custom Voice. Il doit avoir des paramètres régionaux et des propriétés de langue et de sexe adaptés aux données qui seront utilisées pour l’apprentissage de votre voix. Par exemple, sélectionnez `en-GB` si les enregistrements audio que vous avez effectués sont en anglais avec un accent britannique.

Accédez à l’onglet **Données** et cliquez sur **Charger des données**. Dans l’Assistant, sélectionnez le type de données correspondant à ce que vous avez préparé.

Chaque jeu de données que vous chargez doit respecter les exigences associé au type de données choisi. Il est important de bien formater les données avant de les charger pour qu’elles soient correctement traitées par le service Custom Voice. Accédez à [Préparer des données pour Custom Voice](how-to-custom-voice-prepare-data.md) pour vérifier que vos données sont bien mises en forme.

> [!NOTE]
> Les utilisateurs disposant d’un abonnement gratuit (F0) peuvent charger simultanément deux jeux de données. Ceux qui disposent d’un abonnement standard (S0) peuvent, quant à eux, en charger cinq simultanément. Si vous atteignez la limite, attendez au moins la fin de l’importation de l’un de vos jeux de données, puis réessayez.

> [!NOTE]
> Le nombre maximal de jeux de données qu’il est autorisé d’importer par abonnement est de 10 fichiers .zip pour les utilisateurs disposant d’un abonnement gratuit (F0) et de 500 pour ceux qui disposent d’un abonnement standard (S0).

Les jeux de données sont validés automatiquement après avoir appuyé sur le bouton de chargement. La validation des données passe par une série de vérifications des fichiers audio visant à contrôler leur format, leur taille et leur taux d’échantillonnage. Corrigez les erreurs éventuelles et soumettez-les à nouveau. Une fois la demande d’importation de données lancée, une entrée correspondant au jeu de données chargé apparaît dans la table de données.

Le tableau suivant présente les états de traitement des jeux de données importés :

| State | Signification |
| ----- | ------- |
| Traitement en cours | Votre jeu de données est bien reçu et est en cours de traitement. |
| Opération réussie | Votre jeu de données a été validé et est maintenant utilisable pour créer un modèle vocal. |
| Échec | Le traitement de votre jeu de données a échoué pour plusieurs raisons possibles, par exemple des erreurs dans les fichiers, des problèmes de données ou des problèmes de réseau. |

Une fois la validation terminée, le nombre total d’énoncés correspondants apparaît pour chacun des jeux de données dans la colonne **Énoncés**. Si le type de données que vous avez sélectionné implique une longue segmentation audio, cette colonne ne reflètera que les énoncés segmentés automatiquement à partir de vos transcriptions ou avec le service de transcription vocale. Vous pouvez télécharger la suite du jeu de données validé pour voir les résultats détaillés des énoncés importés et les transcriptions associées. Remarque : la segmentation audio longue durée peut prendre plus d’une heure dans le cadre du traitement des données.

Avec des jeux de données en-US et zh-CN, vous pouvez télécharger la suite d’un rapport pour vérifier les scores de prononciation et le niveau de bruit de chacun de vos enregistrements. Le score de prononciation est compris entre 0 et 100. Normalement, un score inférieur à 70 indique une erreur de prononciation ou un problème de correspondance du script. Un accent marqué peut réduire votre score de prononciation et avoir un impact sur la voix numérique générée.

Un rapport signal/bruit (SNR) supérieur indique un bruit plus faible dans votre contenu audio. Vous pouvez généralement atteindre un SNR supérieur à 50 en effectuant les enregistrements dans un studio professionnel. Un contenu audio dont le SNR est inférieur à 20 peut entraîner un niveau de bruit conséquent dans la voix générée.

N’hésitez pas à réenregistrer les énoncés affichant un faible score de prononciation ou un mauvais ratio signal / bruit. Si vous ne pouvez pas réenregistrer, vous pouvez exclure ces énoncés de votre jeu de données.

## <a name="build-your-custom-voice-model"></a>Créer un modèle vocal personnalisé

Une fois votre jeu de données validé, vous pouvez l’utiliser pour créer votre modèle vocal personnalisé.

1.  Accédez à **Synthèse vocale > Custom Voice > [nom du projet] > Entraînement en cours**.

2.  Cliquez sur **Train model**.

3.  Ensuite, entrez un **Nom** et une **Description** pour identifier plus facilement ce modèle.

    Choisissez un nom avec soin. Le nom que vous entrez ici est celui que vous utiliserez pour spécifier la voix dans votre requête de synthèse vocale dans le cadre de l’entrée SSML. Seuls les lettres, les chiffres et quelques caractères de ponctuation comme -, \_ et (', ') sont autorisés. Utilisez un nom différent par modèle vocal.

    Le champ **Description** sert souvent à enregistrer les noms des jeux de données qui ont été utilisés pour créer le modèle.

4.  Sur la page **Sélectionner les données d’apprentissage**, choisissez le ou les jeux de données à utiliser pour l’apprentissage. Vérifiez le nombre d’énoncés avant de les envoyer. Le nombre d’énoncés de départ est indifférent pour les modèles vocaux en-US et zh-CN. Pour les autres paramètres régionaux, il faut sélectionner plus de 2 000 énoncés pour pouvoir entraîner une voix.

    > [!NOTE]
    > Les noms audio en double seront retirés de l’apprentissage. Veillez à ce que les jeux de données sélectionnés ne comportent pas les mêmes noms audio dans plusieurs fichiers .zip.

    > [!TIP]
    > Il est nécessaire d’utiliser des jeux de données provenant du même orateur pour obtenir des résultats de qualité. Si les jeux de données soumis pour l’apprentissage contiennent au total moins de 6 000 énoncés distincts, le modèle vocal sera entraîné suivant la technique de synthèse paramétrique statistique. Dans le cas où les données d’apprentissage dépassent 6 000 énoncés distincts, le processus d’apprentissage qui se déclenchera utilisera la technique de synthèse concaténative. La technologie de concaténation donne normalement des résultats vocaux plus naturels et plus fidèles. [Contactez l’équipe Custom Voice](https://go.microsoft.com/fwlink/?linkid=2108737) si vous souhaitez entraîner un modèle avec la dernière technologie de synthèse vocale neuronale, capable de produire une voix numérique équivalente aux [voix neuronales](language-support.md#neural-voices) proposées au public.

5.  Cliquez sur **Entraîner** pour commencer à créer votre modèle vocal.

La table Training (Entraînement) comporte une nouvelle entrée correspondant à ce nouveau modèle. Elle indique également l’état : Processing, Succeeded, Failed (En cours de traitement, Réussite, Échec).

L’état indiqué reflète le processus de conversion du jeu de données en modèle vocal :

| State | Signification |
| ----- | ------- |
| Traitement en cours | Le modèle vocal est en cours de création. |
| Opération réussie | Le modèle vocal a été créé et peut être déployé. |
| Échec | L’apprentissage du modèle vocal a échoué pour plusieurs raisons possibles, par exemple des problèmes inédits de données ou de réseau. |

Le temps d’apprentissage varie selon le volume de données audio traitées. Il varie généralement d’environ 30 minutes pour plusieurs centaines d’énoncés à 40 heures pour 20 000 énoncés. Une fois l’apprentissage de votre modèle réussi, vous pouvez commencer à le tester.

> [!NOTE]
> Les utilisateurs disposant d’un abonnement gratuit (F0) peuvent entraîner une seule police de la voix à la fois. Ceux qui disposent d’un abonnement standard (S0) peuvent, quant à eux, en entraîner trois simultanément. Si vous atteignez la limite, attendez au moins la fin de la formation de l’une de vos polices de la voix, puis réessayez.

> [!NOTE]
> Le nombre maximal de modèles vocaux qu’il est autorisé d’entraîner par abonnement est de 10 pour les utilisateurs disposant d’un abonnement gratuit (F0) et de 100 pour ceux qui disposent d’un abonnement standard (S0).

Si vous utilisez la fonctionnalité d’entraînement de voix neuronale, vous pouvez choisir d’entraîner un modèle optimisé pour les scénarios de diffusion en continu en temps réel, ou un modèle neuronal HD optimisé pour la [synthèse d’audio long](long-audio-api.md).  

## <a name="test-your-voice-model"></a>Tester un modèle vocal

Une fois votre police de la voix créée, vous pouvez la tester avant de procéder à son déploiement.

1.  Accédez à **Synthèse vocale > Custom Voice > [nom du projet] > Test en cours**.

2.  Cliquez sur **Ajouter un test**.

3.  Sélectionnez un ou plusieurs modèles à tester.

4.  Indiquez le texte qui devra être prononcé. Si vous avez choisi de tester plusieurs modèles en même temps, le même texte sera utilisé.

    > [!NOTE]
    > La langue de votre texte doit être identique à celle de votre police vocale. Seuls les modèles dont l’apprentissage a réussi peuvent être testés. Seul le texte brut est pris en charge à cette étape.

5.  Cliquez sur **Créer**.

Après avoir envoyé votre demande de test, vous revenez à la page de test. Le tableau inclut désormais une entrée qui correspond à votre nouvelle demande, ainsi que la colonne d’état. La synthétisation vocale peut prendre plusieurs minutes. Dès que la colonne d’état indique **Réussite**, vous pouvez lire la sortie audio ou télécharger l’entrée de texte (un fichier .txt) et la sortie audio (un fichier .wav) pour tester la qualité de cette dernière.

Vous trouverez également les résultats des tests sur la page de détails de chacun des modèles sélectionnés pour le test. Accédez à l’onglet **Apprentissage** et cliquez sur le nom du modèle pour vous rendre sur la page de détails correspondante.

## <a name="create-and-use-a-custom-voice-endpoint"></a>Créer et utiliser un point de terminaison Custom Voice

Une fois que vous avez créé et testé votre modèle vocal, déployez-le sur un point de terminaison de synthèse vocale personnalisé. Vous pouvez ensuite utiliser ce point de terminaison à la place du point de terminaison habituel quand vous effectuez des requêtes de synthèse vocale par le biais de l’API REST. Votre point de terminaison personnalisé ne peut être appelé que par l’abonnement ayant servi à déployer la police.

Pour créer un nouveau point de terminaison Custom Voice, accédez à **Synthèse vocale > Custom Voice > Déploiement**. Sélectionnez **Ajouter un point de terminaison** et entrez le **Nom** et la **Description** de votre point de terminaison personnalisé. Ensuite, sélectionnez le modèle vocal personnalisé que vous souhaitez associer à ce point de terminaison.

Dès que vous avez cliqué sur le bouton **Ajouter** dans la table du point de terminaison, l’entrée de votre nouveau point de terminaison apparaît. L’instanciation d’un nouveau point de terminaison peut prendre quelques minutes. Quand l’état du déploiement affiche **Succeeded** (Réussite), le point de terminaison est prêt à être utilisé.

> [!NOTE]
> Les utilisateurs disposant d’un abonnement gratuit (F0) ne peuvent déployer qu’un seul modèle à la fois. Ceux qui disposent d’un abonnement standard (S0) peuvent, quant à eux, créer jusqu’à 50 points de terminaison, chacun ayant sa propre voix personnalisée.

> [!NOTE]
> Pour utiliser votre voix personnalisée, vous devez spécifier le nom du modèle vocal, utiliser directement l’URI personnalisé dans une requête HTTP et vous servir du même abonnement pour effectuer l’authentification du service de synthèse vocale.

Une fois le point de terminaison déployé, son nom s’affiche sous forme de lien. Cliquez sur ce lien pour afficher les informations qui concernent spécifiquement votre point de terminaison, comme sa clé, son URL et l’exemple de code.

Il est également possible d’effectuer un test en ligne du point de terminaison via le portail Custom Voice. Pour tester votre point de terminaison, choisissez **Vérifier le point de terminaison** sur la page **Détails du point de terminaison**. La page de test des points de terminaison s’affiche. Entrez le texte à prononcer (au format texte brut ou [SSML](speech-synthesis-markup.md)) dans la zone de texte. Pour écouter le texte parlé dans votre police de la voix personnalisée, sélectionnez **Play** (Lecture). Cette fonctionnalité de test est facturée selon l’utilisation de la synthèse vocale Custom Speech.

D’un point de vue fonctionnel, le point de terminaison personnalisé est identique au point de terminaison standard utilisé pour les demandes de synthèse vocale. Consultez [API REST](rest-text-to-speech.md) pour en savoir plus.

## <a name="next-steps"></a>Étapes suivantes

* [Guide : Enregistrer des échantillons vocaux](record-custom-voice-samples.md)
* [Référence API Synthèse vocale](rest-text-to-speech.md)
* [API Audio long](long-audio-api.md)

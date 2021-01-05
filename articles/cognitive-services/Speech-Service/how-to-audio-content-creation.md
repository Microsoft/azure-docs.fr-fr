---
title: Création de contenu audio - Service Speech
titleSuffix: Azure Cognitive Services
description: Création de contenu audio est un outil en ligne qui vous permet de personnaliser et d’affiner le résultat de la conversion de texte par synthèse vocale de Microsoft pour vos applications et vos produits.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/31/2020
ms.author: trbye
ms.openlocfilehash: 7d4d1acac591bfe6ce40efcf6166357d43718363
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024957"
---
# <a name="improve-synthesis-with-the-audio-content-creation-tool"></a>Améliorer la synthèse avec l’outil de création de contenu audio

[Création de contenu audio](https://aka.ms/audiocontentcreation) est un outil simple d’utilisation et puissant qui vous permet de créer du contenu audio très naturel pour un large éventail de scénarios, comme les livres audio, les diffusions d’actualités, les narrations vidéo et les robots de conversation. Grâce à Création de contenu audio, vous pouvez ajuster les voix de synthèse vocale et concevoir des expériences audio personnalisées de façon efficace et économique.

L’outil est basé sur le [SSML (Speech Synthesis Markup Language)](speech-synthesis-markup.md). Elle vous permet d’ajuster les attributs de sortie de conversion de texte par synthèse vocale en temps réel ou en synthèse par lot, comme les caractères vocaux, les styles vocaux, la vitesse d’élocution, la prononciation et la prosodie.

Vous pouvez accéder facilement à plus de 150 voix prégénérées dans près de 50 différentes langues, y compris des voix de synthèse vocale de pointe et votre voix personnalisée si vous en avez créé une. 

Pour la création de contenu audio, consultez le [tutoriel vidéo](https://www.youtube.com/watch?v=O1wIJ7mts_w).

## <a name="how-to-get-started"></a>Bien démarrer

Création de contenu audio est un outil gratuit, mais vous payez pour le service Azure Speech que vous consommez. Pour utiliser l’outil, vous devez vous connecter avec un compte Azure et créer une ressource de reconnaissance Speech. Pour chaque compte Azure, vous disposez d’un quota Speech mensuel gratuit qui inclut 500 000 caractères pour les voix TTS neuronales (par mois), 5 millions de caractères pour les voix standard et personnalisées (par mois) et 1 service d’hébergement de point de terminaison vocal personnalisé (par mois). Le montant alloué mensuellement est généralement suffisant pour une petite équipe de contenu d’environ 3-5 personnes. Voici les étapes de création d’un compte Azure et d’obtention d’une ressource Speech. 

### <a name="step-1---create-an-azure-account"></a>Étape 1 : Créer un compte Azure

Pour utiliser Création de contenu audio, vous devez disposer d’un [compte Microsoft](https://account.microsoft.com/account) et d’un [compte Azure](https://azure.microsoft.com/free/ai/). Suivez ces instructions pour [configurer le compte](./overview.md#try-the-speech-service-for-free). 

Le [portail Azure](https://portal.azure.com/) est l’emplacement centralisé pour gérer votre compte Azure. Vous pouvez créer la ressource Speech, gérer l’accès aux produits et tout surveiller, des applications web simples aux déploiements cloud complexes. 

### <a name="step-2---create-a-speech-resource"></a>Étape 2 : Créer une ressource Speech

Après vous être inscrit au compte Azure, vous devez créer une ressource Speech sous votre compte Azure pour accéder aux services Speech. Consultez les instructions pour [Créer une ressource Speech](./overview.md#create-the-azure-resource). 

Le déploiement de votre nouvelle ressource Speech prend quelques instants. Une fois le déploiement terminé, vous pouvez démarrer le parcours Création de contenu audio. 

 >[!NOTE]
   > Si vous envisagez d’utiliser des voix neuronales, assurez-vous de créer votre ressource dans [une région qui prend en charge les voix neuronales](regions.md#standard-and-neural-voices).
 
### <a name="step-3---log-into-the-audio-content-creation-with-your-azure-account-and-speech-resource"></a>Étape 3 : Se connecter à Création de contenu audio avec votre compte Azure et votre ressource Speech

1. Après avoir obtenu le compte Azure et la ressource Speech, vous pouvez vous connecter à [Création de contenu audio](https://aka.ms/audiocontentcreation) en cliquant sur **Démarrage**.
2. La page de la **ressource Speech** s’affiche. Sélectionnez la ressource Speech sur laquelle vous voulez travailler. Cliquez sur **Accéder à Studio** pour démarrer la création audio. Vous pouvez également créer une ressource Speech ici en cliquant sur **Créer nouveau**. Quand vous vous connectez à l’outil Création de contenu audio la fois suivante, nous vous envoyons directement les fichiers de travail audio sous la ressource Speech actuelle. 
3. Vous pouvez modifier votre ressource Speech à tout moment avec l’option **Paramètres**, située dans le haut de la section.

## <a name="how-to-use-the-tool"></a>Comment utiliser l’outil ?

Ce diagramme vous montre les étapes nécessaires pour régler les sorties de la conversion de texte par synthèse vocale. Utilisez les liens ci-dessous pour en savoir plus sur chaque étape.

:::image source="media/audio-content-creation/audio-content-creation-diagram.jpg" alt-text="Diagramme de la procédure à suivre pour ajuster les sorties de la conversion de texte par synthèse vocale":::


1. Sélectionnez la ressource Speech sur laquelle vous voulez travailler.
2. [Créez un fichier de réglage audio](#create-an-audio-tuning-file) en utilisant des scripts en texte brut ou SSML. Saisissez ou chargez votre contenu dans Création de contenu audio.
3. Choisissez la voix et la langue du contenu de votre script. Création de contenu audio comprend toutes les [voix de synthèse vocale de Microsoft](language-support.md#text-to-speech). Vous pouvez utiliser une voix standard, neuronale ou votre propre voix personnalisée.
   >[!NOTE]
   > L’accès contrôlé est disponible pour les voix neuronales personnalisées, ce qui vous permet de créer des voix haute définition similaires à une voix naturelle. Pour plus d’informations, consultez [Processus de vérification](./text-to-speech.md).

4. Cliquez sur l’icône **Lire** (un triangle) pour afficher un aperçu de la sortie de synthèse par défaut. Améliorez ensuite la sortie en ajustant la prononciation, les pauses, le ton, le débit, l’intonation, le style vocal, etc. Pour obtenir la liste complète des options, consultez [Langage de balisage de synthèse vocale](speech-synthesis-markup.md). Voici une [vidéo](https://www.youtube.com/watch?v=O1wIJ7mts_w) illustrant comment ajuster la sortie vocale avec la Création de contenu audio. 
5. Enregistrez et [exportez vos audio réglés](#export-tuned-audio). Lorsque vous enregistrez la piste de paramétrage dans le système, vous pouvez continuer à travailler et itérer sur la sortie. Lorsque vous êtes satisfait de la sortie, vous pouvez créer une tâche de création audio avec la fonctionnalité d’exportation. Vous pouvez observer l’état de la tâche d’exportation et télécharger la sortie à utiliser avec vos applications et produits.

## <a name="create-an-audio-tuning-file"></a>Créer un fichier de réglage audio

Il existe deux façons d’intégrer votre contenu dans l’outil Création de contenu audio.

**Option 1 :**

1. Cliquez sur l’icône **Nouveau fichier** en haut à droite pour créer un fichier de réglage audio.
2. Tapez ou collez votre contenu dans la fenêtre d’édition. Chaque fichier ne doit pas excéder 20 000 caractères. Si votre script dépasse cette limite, vous pouvez utiliser l’option 2 pour fractionner automatiquement votre contenu en plusieurs fichiers. 
3. N’oubliez pas d’enregistrer.

**Option 2 :**

1. Cliquez sur **Charger** pour importer un ou plusieurs fichiers texte. Le texte brut et le SSML sont pris en charge. Si votre fichier de script comporte plus de 20 000 caractères, divisez le fichier par paragraphes, caractères ou expressions régulières. 
3. Lorsque vous téléchargez vos fichiers texte, assurez-vous que le fichier répond à ces exigences.

   | Propriété | Valeur/remarques |
   |----------|---------------|
   | Format de fichier | Texte brut (.txt)<br/> Texte SSML (.txt)<br/> Les fichiers zip ne sont pas pris en charge. |
   | Format d’encodage | UTF-8 |
   | Nom de fichier | Chaque fichier doit avoir un nom unique. Les doublons ne sont pas pris en charge. |
   | Longueur du texte | La limitation de caractères du fichier texte est de 20 000. Si vos fichiers dépassent cette limite, séparez les fichiers à l’aide des instructions de l’outil. |
   | Restrictions SSML | Chaque fichier SSML ne peut contenir qu’un seul élément SSML. |

**Exemple de texte brut**

```txt
Welcome to use Audio Content Creation to customize audio output for your products.
```
**Exemple de texte SSML**

```xml
<speak xmlns="http://www.w3.org/2001/10/synthesis" xmlns:mstts="http://www.w3.org/2001/mstts" version="1.0" xml:lang="en-US">
    <voice name="Microsoft Server Speech Text to Speech Voice (en-US, AriaNeural)">
    Welcome to use Audio Content Creation <break time="10ms" />to customize audio output for your products.
    </voice>
</speak>
```

## <a name="export-tuned-audio"></a>Exporter un audio réglé

Une fois que vous avez vérifié la sortie audio et que vous êtes satisfait de votre paramétrage et de votre réglage, vous pouvez exporter l’audio.

1. Cliquez sur **Exporter** pour créer une tâche de création d’audio. Il est recommandé d’**exporter vers une bibliothèque audio**, car celle-ci prend en charge la longue sortie audio et l’expérience de sortie audio dans son intégralité. Vous pouvez également télécharger l’audio directement vers votre disque local, mais seules les 10 première minutes sont disponibles.
2. Choisissez le format de sortie de votre audio réglé. Vous trouverez ci-dessous une liste des formats et des taux d’échantillonnage pris en charge.
3. Vous pouvez afficher l’état de la tâche dans l’onglet **Exporter une tâche**. Si la tâche échoue, consultez la page d’informations détaillées pour obtenir un rapport complet.
4. Une fois la tâche terminée, votre audio est disponible en téléchargement dans l’onglet **Bibliothèque audio**.
5. Cliquez sur **Télécharger**. Vous êtes maintenant prêt à utiliser votre audio réglé personnalisé dans vos applications ou vos produits.

**Formats audio pris en charge**

| Format | Taux d’échantillonnage 16 kHz | Taux d’échantillonnage 24 kHz |
|--------|--------------------|--------------------|
| wav | riff-16 khz-16 bits-mono-pcm | riff-24 khz-16 bits-mono-pcm |
| mp3 | audio-16 khz-128 kbitrate-mono-mp3 | audio-24 khz-160 kbitrate-mono-mp3 |

## <a name="how-to-addremove-audio-content-creation-users"></a>Comment ajouter/supprimer des utilisateurs dans Création de contenu audio ?

Si plusieurs utilisateurs souhaitent utiliser Création de contenu audio, vous pouvez soit partager votre compte et votre mot de passe Azure avec l’utilisateur, soit accorder à l’utilisateur l’accès à l’abonnement Azure et à la ressource Speech. Si vous ajoutez un utilisateur à un abonnement Azure, l’utilisateur peut accéder à toutes les ressources sous l’abonnement Azure. Toutefois, si vous ajoutez uniquement un utilisateur à une ressource Speech, l’utilisateur n’aura accès qu’à la ressource Speech et ne pourra pas accéder aux autres ressources sous cet abonnement Azure. Un utilisateur ayant accès à la ressource Speech peut utiliser Création de contenu audio.

### <a name="add-users-to-a-speech-resource"></a>Ajouter des utilisateurs à une ressource Speech

Procédez comme suit pour ajouter un utilisateur à une ressource Speech afin qu’il puisse utiliser Création de contenu audio.

1. Recherchez **Cognitive Services** dans le [portail Azure](https://portal.azure.com/), sélectionnez la ressource Speech à laquelle vous souhaitez ajouter des utilisateurs.
2. Cliquez sur **Contrôle d’accès (IAM)** . Cliquez sur l’onglet **Attributions de rôles** afin d’afficher toutes les attributions de rôles pour cet abonnement.
    :::image source="media/audio-content-creation/access-control-roles.png" alt-text="Onglet d’attribution de rôle":::
1. Cliquez sur **Ajouter** > **Ajouter une attribution de rôle** pour ouvrir le volet Ajouter une attribution de rôle. Dans la liste déroulante Rôle, sélectionnez le rôle **Utilisateur du service cognitif**. Si vous souhaitez attribuer à l’utilisateur la propriété de cette ressource Speech, vous pouvez sélectionner le rôle **Propriétaire**.
1. Sélectionnez un utilisateur dans la liste. Si vous ne voyez pas l’utilisateur dans la liste, vous pouvez taper dans la zone Sélectionner pour rechercher des noms d’affichage et des adresses e-mail dans l’annuaire. Si l’utilisateur ne se trouve pas dans ce répertoire, vous pouvez entrer le [compte Microsoft](https://account.microsoft.com/account) de l’utilisateur (qui est approuvé par Azure Active Directory).
1. Cliquez sur **Enregistrer** pour attribuer le rôle. Après quelques instants, l’utilisateur se voit attribuer le rôle Utilisateur du service cognitif au niveau de l’étendue de la ressource Speech.

    :::image source="media/audio-content-creation/add-role-first.png" alt-text="Boîte de dialogue Ajouter un rôle":::

1. Les utilisateurs que vous ajoutez recevront un e-mail d’invitation. Une fois qu’ils cliquent sur **Accepter l’invitation** > **Accepter de rejoindre Azure**, ils peuvent utiliser [Création de contenu audio](https://aka.ms/audiocontentcreation).

Les utilisateurs qui se trouvent dans la même ressource Speech voient le travail des uns et des autres dans le studio Création de contenu audio. Si vous souhaitez que chaque utilisateur individuel dispose d’un espace de travail unique et privé dans Création de contenu audio, veuillez [créer une ressource Speech](#step-2---create-a-speech-resource) pour chaque utilisateur et octroyer à chaque utilisateur l’accès unique à la ressource Speech. 

### <a name="remove-users-from-a-speech-resource"></a>Supprimer des utilisateurs d’une ressource Speech
1. Recherchez **Cognitive Services** dans le portail Azure, sélectionnez la ressource Speech dont vous souhaitez supprimer des utilisateurs.
2. Cliquez sur **Contrôle d’accès (IAM)** . Cliquez sur l’onglet **Attributions de rôles** afin d’afficher toutes les attributions de rôles pour cette ressource vocale.
3. Sélectionnez les utilisateurs que vous souhaitez supprimer, puis cliquez sur **Supprimer** > **OK**.
    :::image source="media/audio-content-creation/remove-user.png" alt-text="Bouton Supprimer":::

### <a name="enable-users-to-grant-access"></a>Autoriser les utilisateurs à accorder l’accès
Si vous souhaitez qu’un des utilisateurs puisse donner l’accès à d’autres utilisateurs, vous devez lui attribuer le rôle Propriétaire pour la ressource Speech et définir l’utilisateur en tant que lecteur de répertoires Azure. 
1. Ajoutez l’utilisateur en tant que propriétaire de la ressource Speech. Consultez [Comment ajouter des utilisateurs à une ressource Speech](#add-users-to-a-speech-resource).
    :::image source="media/audio-content-creation/add-role.png" alt-text="Champ Propriétaire du rôle":::
1. Sélectionnez le menu réduit dans le coin supérieur gauche. Cliquez sur **Azure Active Directory**, puis sur **Utilisateurs**.
1. Recherchez le compte Microsoft de l’utilisateur et accédez à la page de détails de l’utilisateur. Cliquez sur **Rôles affectés**.
1. Cliquez sur **Ajouter des affectations** -> **Lecteurs de répertoires**.

## <a name="see-also"></a>Voir aussi

* [API Audio long](./long-audio-api.md)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Speech Studio](https://speech.microsoft.com)

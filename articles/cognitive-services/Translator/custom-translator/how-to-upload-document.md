---
title: Pour télécharger un document - Custom Translator
titleSuffix: Azure Cognitive Services
description: La fonctionnalité de chargement de documents charge les documents parallèles (deux documents, l'un correspondant à l'original et l’autre à la traduction) dans le service.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 8597b3e4ffab44672e88374010fa829211fbc18b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88507313"
---
# <a name="upload-a-document"></a>Charger un document

Dans [Custom Translator](https://portal.customtranslator.azure.ai), vous pouvez charger des documents parallèles pour former vos modèles de traduction. Les [documents parallèles](what-are-parallel-documents.md) sont des paires de documents dont un document est la traduction de l'autre. Un document de la paire contient des phrases dans la langue source et l’autre document contient ces phrases traduites dans la langue cible.

Avant de charger vos documents, consultez [l’aide relatives à la convention d’affectation de noms et aux formats des documents](document-formats-naming-convention.md) pour vérifier que votre format de fichier est pris en charge dans Custom Translator.

## <a name="how-to-upload-document"></a>Comment charger un document ?

À partir du portail [Custom Translator](https://portal.customtranslator.azure.ai), cliquez sur l’onglet de « Documents » pour accéder à la page des documents.

![Lien de chargement de documents](media/how-to/how-to-upload-1.png)


1.  Cliquez sur le bouton Charger des fichiers sur la page des documents.

    ![Charger la page des documents](media/how-to/how-to-upload-2.png)

2.  Dans la boîte de dialogue, renseignez les informations suivantes :

    a.  Type de document :

    -  Formation : Ces documents seront utilisés pour un ensemble de formation.
    -  Réglage : Ces documents seront utilisés pour un ensemble de réglage.
    -  Test : Ces documents seront utilisés pour un ensemble de test.
    -  Dictionnaire d’expressions : Ces documents seront utilisés pour le dictionnaire d’expressions.
    -  Dictionnaire de phrases : Ces documents seront utilisés pour le dictionnaire de phrases.

    b.  Paire de langues

    c.  Remplacer un document existant : Cochez cette case pour remplacer des documents existants portant le même nom.

    d.  Remplissez la section appropriée pour les données en parallèle ou les données de liste déroulante.

    -  Copie parallèle :
        -  Fichier source : Sélectionnez un fichier de langue source sur votre ordinateur local.
        -  Fichier cible : Sélectionnez un fichier de langue cible sur votre ordinateur local.
        -  Nom du document : Utilisé uniquement si vous chargez des fichiers parallèles.

    - Données de liste déroulante :
        -  Fichier de liste déroulante : Sélectionnez le fichier de liste déroulante à partir de votre ordinateur local. Votre fichier de liste déroulante contient à la fois les phrases des langues source et cible. La [convention d’affectation de noms](document-formats-naming-convention.md) est importante pour les fichiers de liste déroulante.

    e.  Cliquez sur Charger

    ![Chargez la boîte de dialogue des documents](media/how-to/how-to-upload-dialog.png)

3.  À ce stade, nous traitons vos documents et tentons d’extraire les phrases. Vous pouvez cliquer sur « Afficher la progression du chargement » pour vérifier l’état de vos documents pendant le traitement.

    ![Charger la boîte de dialogue de traitement de documents](media/how-to/how-to-upload-processing-dialog.png)

4.  Cette page affiche l’état et les erreurs pour chaque fichier dans votre chargement. Vous pouvez afficher l’état après le chargement à tout moment en cliquant sur l’onglet « Charger l’historique ».

    ![Charger la boîte de dialogue de l’historique des documents](media/how-to/how-to-upload-document-history.png)


## <a name="view-upload-history"></a>Afficher l’historique du chargement

À la page de l’historique du chargement, vous pouvez afficher l’historique de tous les chargements des documents, tels que le type de document, la paire de langues, l’état du chargement, etc.

1. À partir du portail [Custom Translator](https://portal.customtranslator.azure.ai), cliquez sur l’onglet Historique de chargement pour afficher l’historique.

    ![Onglet Historique du chargement](media/how-to/how-to-upload-history-1.png)

2. Cette page affiche l’état de tous vos chargements passés. Elle affiche les chargements du plus récent au moins récent. Pour chaque chargement, il affiche le nom du document, l’état du chargement, la date du chargement, le nombre de fichiers chargés, le type de fichier chargé et la paire de langues du fichier.

    ![Afficher la page Historique](media/how-to/how-to-document-history-2.png)

3. Cliquez sur n’importe quel enregistrement de l’historique de chargement. À la page des détails de l’historique de chargement, vous pouvez afficher les fichiers chargés dans le cadre du chargement, l’état de chargement du fichier, la langue du fichier et le message d’erreur (en cas d’erreur de chargement).

## <a name="next-steps"></a>Étapes suivantes

- Utilisez la [page de détails du document](how-to-view-document-details.md) pour passer en revue la liste des phrases extraites.
- [Pour effectuer l'apprentissage d'un modèle](how-to-train-model.md).

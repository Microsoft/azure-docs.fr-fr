---
title: Qu’est-ce qu’un espace de travail et un projet ? - Custom Translator
titleSuffix: Azure Cognitive Services
description: Cet article décrit les différences entre un espace de travail et un projet, et entre les catégories et les étiquettes de projet pour le service Custom Translator.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 28bc07cadf9ffea56059755e243cb3e828077588
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368965"
---
# <a name="what-is-a-custom-translator-workspace"></a>Qu’est-ce qu’un espace de travail Custom Translator ?

Un espace de travail est une zone de travail pour la composition et la création de votre système de traduction personnalisé. Un espace de travail peut contenir plusieurs projets, modèles et documents. Toutes les tâches que vous effectuez dans Custom Translator s’effectue dans un espace de travail spécifique.

L’espace de travail est privé et vous est réservé, ainsi qu’aux personnes que vous y invitez. Les personnes non invitées n’ont pas accès au contenu de votre espace de travail. Vous pouvez inviter autant de personnes que vous le souhaitez dans votre espace de travail, et modifier ou supprimer leur accès à tout moment. Vous pouvez également créer un espace de travail. Par défaut, un espace de travail ne contient pas les projets ou les documents qui se trouvent dans vos autres espaces de travail.

## <a name="what-is-a-custom-translator-project"></a>Qu’est-ce qu’un projet Custom Translator ?

Un projet est un wrapper avec un modèle, des documents et des tests. Chaque projet inclut automatiquement tous les documents téléchargés dans cet espace de travail dotés de la paire de langues qui convient. Par exemple, si vous avez un projet anglais vers espagnol et un projet espagnol vers anglais, les mêmes documents figurent dans les deux projets. Chaque projet a un CategoryID associé qui est utilisé lors de l’interrogation de [l’API V3](../reference/v3-0-translate.md?tabs=curl) pour obtenir des traductions. CategoryID est un paramètre utilisé pour obtenir des traductions d’un système personnalisé créé avec Custom Translator.

## <a name="project-categories"></a>Catégories de projet

La catégorie identifie le domaine (la zone de terminologie et de style à utiliser) pour votre projet. Choisissez la catégorie la plus pertinente pour vos documents. Dans certains cas, votre choix de catégorie influence directement le comportement de Custom Translator.

Nous avons deux ensembles de modèles de base. Il s’agit de Général et Technologie. Si la catégorie **Technologie** est sélectionnée, les modèles de base Technologie sont utilisés. Pour toute autre sélection de catégorie, les modèles de base Général sont utilisés. Le modèle de base Technologie convient bien dans le domaine de la technologie, mais s’avère de moindre qualité si les phrases utilisées pour la traduction ne cadrent pas avec le domaine de technologie. Nous suggérons aux clients de sélectionner la catégorie Technologie uniquement si les phrases cadrent strictement avec le domaine de technologie.

Dans le même espace de travail, vous pouvez créer des projets pour la même paire de langues dans différentes catégories. Custom Translator empêche la création d’un projet en double avec la même paire de langues et la même catégorie. Appliquez une étiquette à votre projet pour éviter cette restriction. N’utilisez pas d’étiquettes sauf si vous créez des systèmes de traduction pour plusieurs clients, car l’ajout d’une étiquette unique à votre projet est reflété dans le CategoryID de vos projets.

## <a name="project-labels"></a>Étiquettes de projet

Custom Translator vous permet d’attribuer une étiquette de projet à votre projet. L’étiquette de projet permet de différencier plusieurs projets présentant la même paire de langues et la même catégorie. La meilleure pratique consiste à éviter d’utiliser des étiquettes de projet, sauf si nécessaire.

L’étiquette de projet est utilisée dans le cadre de CategoryID. Si l’étiquette de projet n’est pas définie ou si elle est définie de manière identique entre plusieurs projets, alors les projets avec la même catégorie et des paires de langues *différentes* partagent le même CategoryID. Cette approche est avantageuse, car elle vous permet ou permet à votre client de basculer entre les langues lors de l’utilisation de l’API de traduction de texte sans vous soucier du CategoryID qui est unique pour chaque projet.

Par exemple, si je veux activer les traductions dans le domaine Technologie de l’anglais vers le français et du français vers l’anglais, je dois créer deux projets : un pour anglais -\> français, et l’autre pour français -\> anglais. Je spécifie la même catégorie (Technologie) pour les deux et laisse l’étiquette de projet non renseignée. Le CategoryID pour les deux projets correspond, donc je peux interroger l’API pour les traductions en anglais et en français sans avoir à modifier mon CategoryID.

Si vous êtes un fournisseur de services linguistiques et si vous souhaitez présenter à plusieurs clients des modèles différents qui conservent la même catégorie et la même paire de langues, alors c’est une bonne idée d’utiliser une étiquette de projet pour différencier les clients.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [la formation et le modèle](training-and-model.md) pour savoir comment créer un modèle de traduction de manière efficace.
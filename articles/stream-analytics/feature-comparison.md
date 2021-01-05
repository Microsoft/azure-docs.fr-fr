---
title: Comparaison des fonctionnalités Azure Stream Analytics
description: Cet article compare les fonctionnalités prises en charge pour les tâches cloud et IoT Edge Azure Stream Analytics dans le portail Azure, Visual Studio et Visual Studio Code.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 06/27/2019
ms.openlocfilehash: 2fbaaac0095622f14d5bc0033b7787bec8ec2fa2
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92018285"
---
# <a name="azure-stream-analytics-feature-comparison"></a>Comparaison des fonctionnalités Azure Stream Analytics

Avec Azure Stream Analytics, vous pouvez créer des solutions de diffusion en continu dans le cloud et IoT Edge à l’aide du [portail Azure](stream-analytics-quick-create-portal.md), de [Visual Studio](stream-analytics-quick-create-vs.md), et de [Visual Studio Code](quick-create-visual-studio-code.md). Les tableaux de cet article indiquent les fonctionnalités prises en charge par chaque plateforme pour les deux types de tâches.

> [!NOTE]
> Les outils Visual Studio et Visual Studio Code ne prennent pas en charge les travaux dans les régions Chine Est, Chine Nord, Allemagne Centre et Allemagne Nord-Est.

## <a name="cloud-job-features"></a>Fonctionnalités des tâches du cloud


|Fonctionnalité  |Portail  |Visual Studio  |Visual Studio Code  |
|---------|---------|---------|---------|
|Multiplateforme     |Mac</br>Linux</br> Windows         | Windows        |Mac</br>Linux</br> Windows          |
|Création de script     |Oui         |Oui         |Oui         |
|Script Intellisense     |Mise en surbrillance de la syntaxe         |Mise en surbrillance de la syntaxe</br>Complétion de code</br>Marqueur d’erreur         |Mise en surbrillance de la syntaxe</br>Complétion de code</br>Marqueur d’erreur         |
|Définir tous les types d'entrées, de sorties et de configurations de projet     |Oui         |Oui         |Oui         |
|Contrôle de code source     |Non          |Oui         |Oui         |
|Prise en charge de CI/CD     |Partiel         |Oui         |Oui         |
|Partager des entrées et des sorties entre plusieurs requêtes     |Non          |Oui         |Oui         |
|Test de requête avec un exemple de fichier     |Oui         |Oui        |Oui         |
|Test local des données actives     |Non          |Oui       |Oui      |
|Répertorier les travaux et afficher les entités de travail     |Oui         |Oui        |Oui         |
|Exporter un travail vers un projet local     |Non          |Oui         |Oui         |
|Envoyer, lancer et arrêter des tâches     |Oui         |Oui         |Oui         |
|Afficher les métriques de tâche et le diagramme     |Oui         |Oui         |Ouvrir dans le portail         |
|Afficher les erreurs d’exécution de tâches     |Oui         |Oui         |Non          |
|Journaux d’activité de ressources     |Oui         |Non          |Non          |
|Propriétés de message personnalisées     |Oui         |Oui         |Non        |
|Fonction de code personnalisé C# et désérialiseur|Mode Lecture seule|Oui|Non |
|UDF et UDA JavaScript     |Oui         |Oui         |Windows uniquement         |
|Machine Learning Service     |Oui        |Oui         |Non          |
|Azure Machine Learning Studio (classique)|Oui, mais la requête ne peut pas être testée        |Oui |Non          |
|Niveau de compatibilité     |1.0</br>1.1</br>1.2 (valeur par défaut)         |1.0</br>1.1</br>1.2 (valeur par défaut)           |1.0</br>1.1</br>1.2 (valeur par défaut)           |
|Fonctions de détection d’anomalie ML intégrées     |Oui         |Oui         |Oui         |
|Fonctions géospatiales intégrées     |Oui         |Oui         |Oui         |



## <a name="iot-edge-job-features"></a>Fonctionnalités des tâches IoT Edge

|Fonctionnalité  |Portail  |Visual Studio  |Visual Studio Code  |
|---------|---------|---------|---------|
|Création de tâches     |Oui         |Oui         |Non          |
|Contrôle de code source     |Non          |Oui         |Non          |
|Exporter un travail vers un projet local     |Non          |Oui         |Non          |
|Test de requête avec un exemple de fichier     |Oui         |Oui         |Non          |
|Partager des entrées et des sorties entre plusieurs requêtes     |Non          |Oui         |Non          |
|C# UDF     |Non          |Oui         |Non          |
|Soumettre les travaux     |Oui         |Oui         |Non          |
|Répertorier les travaux et afficher les entités de travail     |Oui         |Oui         |Non          |
|Afficher les métriques de tâche et le diagramme     |Oui         |Partiel         |Non          |
|Afficher les erreurs d’exécution de tâches     |Oui         |Partiel         |Non          |
|Prise en charge de CI/CD     |Non          |Non          |Non          |


## <a name="next-steps"></a>Étapes suivantes

* [Azure Stream Analytics sur IoT Edge](stream-analytics-edge.md)
* [Tutoriel : Écrire une fonction C# définie par l’utilisateur pour une tâche IoT Edge Azure Stream Analytics (préversion)](stream-analytics-edge-csharp-udf.md)
* [Développer des tâches IoT Edge Azure Stream Analytics avec les outils Visual Studio](stream-analytics-tools-for-visual-studio-edge-jobs.md)
* [Utiliser Visual Studio pour afficher les tâches Azure Stream Analytics](stream-analytics-vs-tools.md)
* [Explorer Azure Stream Analytics avec Visual Studio Code (préversion)](visual-studio-code-explore-jobs.md)



---
title: Terminologie – Personalizer
description: Personalizer utilise la terminologie de l’apprentissage par renforcement. Ces termes sont utilisés dans le Portail Azure et les API.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 04/23/2020
ms.openlocfilehash: cd0d85be5447aad0f2a3c37041e7d5d5d047a468
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91777290"
---
# <a name="personalizer-terminology"></a>Terminologie de Personalizer

Personalizer utilise la terminologie de l’apprentissage par renforcement. Ces termes sont utilisés dans le Portail Azure et les API.

## <a name="conceptual-terminology"></a>Terminologie conceptuelle

* **Boucle d’apprentissage** : Vous créez une ressource Personalizer, appelée _boucle d’apprentissage_, pour chaque partie de votre application qui peut tirer parti de la personnalisation. Si vous avez plusieurs expériences à personnaliser, créez une boucle pour chacune d’elles.

* **Modèle** : Un modèle Personalizer capture toutes les données apprises sur le comportement des utilisateurs, obtenant les données d’entraînement de la combinaison des arguments que vous envoyez aux appels de classement et de récompense, et avec un comportement d’entraînement déterminé par la stratégie d’apprentissage.

* **Mode En ligne** : [comportement d'apprentissage](#learning-behavior) par défaut de Personalizer, dans lequel votre boucle d'apprentissage utilise l'apprentissage automatique pour générer le modèle qui prédit la **première action** relative à votre contenu.

* **Mode Apprenti** : [comportement d'apprentissage](#learning-behavior) qui permet de démarrer à chaud un modèle Personalizer pour effectuer l'apprentissage sans affecter les résultats et les actions des applications.

## <a name="learning-behavior"></a>Comportement d'apprentissage :

* **Mode En ligne** : permet de retourner la meilleure action. Votre modèle répond aux appels de l'API Rank en retournant la meilleure action, et utilise les appels de l'API Reward pour apprendre et améliorer ses sélections au fil du temps.
* **[Mode Apprenti](concept-apprentice-mode.md)**  : permet d'apprendre en tant qu'apprenti. Votre modèle apprend en observant le comportement de votre système. Les appels de l'API Rank retournent toujours l'**action par défaut** (référence) de l'application.

## <a name="personalizer-configuration"></a>Configuration de Personalizer

Personalizer est configuré à partir du [Portail Azure](https://portal.azure.com).

* **Récompenses** : configurez les valeurs par défaut pour la durée d’attente de la récompense, la récompense par défaut et la stratégie d’agrégation de récompenses.

* **Exploration** : configurez le pourcentage d’appels de classement à utiliser pour l’exploration.

* **Fréquence de mise à jour du modèle** : Fréquence à laquelle l’apprentissage du modèle est effectué à nouveau.

* **Conservation des données** : Nombre de jours de données à stocker. Cela peut avoir un impact sur les évaluations hors connexion, qui sont utilisées pour améliorer votre boucle d’apprentissage.

## <a name="use-rank-and-reward-apis"></a>Utiliser des API de classement et de récompense

* **Rank** : Étant donné les actions dotées de caractéristiques et les caractéristiques de contexte, utilisez Explorer ou Exploiter pour retourner la première action (élément de contenu).

    * **Actions** : Les actions sont les éléments de contenu, comme des produits ou des promotions, parmi lesquels choisir. Personalizer choisit la première action (ID de l’action récompensée retournée) à montrer à vos utilisateurs via l’API de classement.

    * **Context** : Pour fournir un classement plus précis, fournissez des informations sur votre contexte, par exemple :
        * Votre utilisateur.
        * L’appareil sur lequel il travaille.
        * L’heure actuelle.
        * D’autres données relatives à la situation actuelle.
        * L’historique des données sur l’utilisateur ou sur le contexte.

        Votre application spécifique peut avoir des informations de contexte différentes.

    * **[Caractéristiques](concepts-features.md)**  : Une unité d’informations sur un élément de contenu ou un contexte utilisateur. Veillez à utiliser uniquement les caractéristiques qui sont agrégées. N’utilisez pas d’heures spécifiques, d’identifiants utilisateur ou d’autres données non agrégées comme caractéristiques.

        * Une _caractéristique d’action_ est une métadonnée sur le contenu.
        * Une _caractéristique de contexte_ est une métadonnée sur le contexte dans lequel le contenu est présenté.

* **Exploration** : Le service Personalizer effectue une exploration quand, au lieu de retourner la meilleure action, il choisit une autre action pour l’utilisateur. Le service Personalizer évite les dérives et la stagnation, et peut s’adapter au comportement actuel de l’utilisateur en effectuant une exploration.

* **Exploitation** : Le service Personalizer utilise le modèle actuel pour déterminer la meilleure action en fonction des données antérieures.

* **Durée de l’expérience** : Temps pendant lequel le service Personalizer attend une récompense, à partir du moment où l’appel du classement s’est produit pour cet événement.

* **Événements inactifs** : Un événement inactif est un événement où vous avez appelé le classement, mais où vous n’êtes pas sûr que l’utilisateur verra jamais le résultat, en raison des décisions de l’application cliente. Les événements inactifs vous permettent de créer et de stocker les résultats de la personnalisation, puis de décider de les abandonner plus tard sans affecter le modèle de machine learning.


* **Récompense** : Une mesure de la façon dont l’utilisateur a répondu à l’ID de l’action récompensée retournée de l’API de classement, sous la forme d’un score compris entre 0 et 1. La valeur de 0 à 1 est définie par votre logique métier, en fonction de la façon dont le choix a aidé à atteindre vos objectifs métier de personnalisation. La boucle d’apprentissage ne stocke pas cette récompense en tant qu’historique utilisateur individuel.

## <a name="evaluations"></a>Évaluations

### <a name="offline-evaluations"></a>Évaluations hors connexion

* **Évaluation** : Une évaluation hors connexion détermine la meilleure stratégie d'apprentissage à appliquer à votre boucle en fonction des données de votre application.

* **Stratégie d’apprentissage** : La façon dont Personalizer effectue l’apprentissage d’un modèle sur chaque événement est déterminée par certains paramètres qui influencent le fonctionnement de l’algorithme Machine Learning. Une nouvelle boucle d’apprentissage commence par une **stratégie d’apprentissage** par défaut, ce qui peut donner des performances modérées. Lors de l’exécution des [Évaluations](concepts-offline-evaluation.md), Personalizer crée de nouvelles stratégies d’apprentissage spécifiquement optimisées pour les cas d’utilisation de votre boucle. Personalizer s’exécutera considérablement mieux avec les stratégies optimisées pour chaque boucle spécifique, générées pendant l’évaluation. La politique d’apprentissage est nommée _Paramètres d’apprentissage_ sur la page **Paramètres de modèle et d’apprentissage** de la ressource Personalizer dans le Portail Azure.

### <a name="apprentice-mode-evaluations"></a>Évaluations en mode Apprenti

Le mode Apprenti fournit les **mesures d'évaluation** suivantes :
* **Référence - récompense moyenne** :  Récompenses moyennes de l'action par défaut (référence) de l'application.
* **Personalizer - récompense moyenne** : Moyenne des récompenses totales que Personalizer aurait pu atteindre.
* **Moyenne mobile** : Rapport récompense de référence/récompense Personalizer - normalisé sur les 1 000 événements les plus récents.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [l'éthique et l'utilisation responsable](ethics-responsible-use.md)
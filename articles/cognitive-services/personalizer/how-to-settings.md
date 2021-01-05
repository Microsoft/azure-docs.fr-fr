---
title: Configurer Personalizer
description: La configuration du service porte sur la façon dont le service traite les récompenses, la fréquence à laquelle le service effectue une exploration, la fréquence à laquelle le modèle est réentraîné et la quantité de données stockées.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: how-to
ms.date: 04/29/2020
ms.openlocfilehash: 9b7347cb98bcbf2e1d92f115d404197083acef4d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91292606"
---
# <a name="configure-personalizer-learning-loop"></a>Configurer la boucle d’apprentissage de Personalizer

La configuration du service porte sur la façon dont le service traite les récompenses, la fréquence à laquelle le service effectue une exploration, la fréquence à laquelle le modèle est réentraîné et la quantité de données stockées.

Configurez la boucle d’apprentissage sur la page **Configuration** du Portail Azure pour la ressource Personalizer.

<a name="configure-service-settings-in-the-azure-portal"></a>
<a name="configure-reward-settings-for-the-feedback-loop-based-on-use-case"></a>

## <a name="planning-configuration-changes"></a>Planification des modifications de configuration

Dans la mesure où certaines modifications de configuration [réinitialisent votre modèle](#settings-that-include-resetting-the-model), vous devez planifier vos modifications.

Si vous prévoyez d'utiliser le [mode Apprenti](concept-apprentice-mode.md), commencez par revoir la configuration de Personalizer.

<a name="clear-data-for-your-learning-loop"></a>

## <a name="settings-that-include-resetting-the-model"></a>Paramètres qui incluent la réinitialisation du modèle

Les actions suivantes déclenchent un nouvel apprentissage du modèle à l'aide des données disponibles jusqu'aux 2 derniers jours.

* Récompense
* Exploration

Pour [effacer](how-to-manage-model.md) toutes vos données, utilisez la page **Paramètres de modèle et d'apprentissage**.

## <a name="configure-rewards-for-the-feedback-loop"></a>Configurer les récompenses pour la boucle de rétroaction

Configurez le service pour l’utilisation des récompenses par votre boucle d’apprentissage. Les modifications apportées aux valeurs suivantes réinitialisent le modèle actuel de Personalizer et effectuent de nouveau son apprentissage avec les données des deux derniers jours.

> [!div class="mx-imgBorder"]
> ![Configurer les valeurs de récompense pour la boucle de rétroaction](media/settings/configure-model-reward-settings.png)

|Valeur|Objectif|
|--|--|
|Temps d’attente des récompenses|Définit la durée pendant laquelle Personalizer collecte les valeurs des récompenses pour un appel du classement, en commençant au moment où l’appel du classement se produit. Vous définissez cette valeur en vous posant cette question : « Combien de temps Personalizer doit-il attendre les appels des récompenses ? » Toute récompense arrivant après cette fenêtre de temps est journalisée, mais elle n’est pas utilisée pour l’apprentissage.|
|Récompense par défaut|Si aucun appel de récompense n’est reçu par Personalizer pendant la fenêtre Temps d’attente des récompenses associée à un appel du classement, Personalizer affecte la récompense par défaut. Par défaut, et dans la plupart des scénarios, la récompense par défaut est égale à zéro (0).|
|Agrégation de récompenses|Si plusieurs récompenses sont reçues pour le même appel de l’API de classement, cette méthode d’agrégation est utilisée : **somme** ou **le plus ancien**. « Le plus ancien » sélectionne le score reçu le plus ancien et ignore le reste. Cela est utile si vous voulez une seule récompense parmi des appels éventuellement en double. |

Après avoir changé ces valeurs, veillez à sélectionner **Enregistrer**.

## <a name="configure-exploration-to-allow-the-learning-loop-to-adapt"></a>Configurer l’exploration pour permettre l’adaptation de la boucle d’apprentissage

La personnalisation est en mesure de découvrir de nouveaux modèles et de s’adapter aux changements de comportement des utilisateurs au fil du temps en explorant des alternatives au lieu d’utiliser la prédiction du modèle entraîné. La valeur **Exploration** détermine le pourcentage des appels du classement qui reçoivent une réponse donnée via une exploration.

Les modifications apportées à ce paramètre réinitialisent le modèle actuel de Personalizer et effectuent de nouveau son apprentissage avec les données des 2 derniers jours.

![La valeur Exploration détermine le pourcentage des appels du classement qui reçoivent une réponse donnée via une exploration](media/settings/configure-exploration-setting.png)

Après avoir changé cette valeur, veillez à sélectionner **Enregistrer**.

<a name="model-update-frequency"></a>

## <a name="configure-model-update-frequency-for-model-training"></a>Configurer la fréquence de mise à jour du modèle pour la formation des modèles

La **fréquence de mise à jour des modèles** définit la fréquence à laquelle l’apprentissage du modèle est effectué.

|Paramètre de fréquence|Objectif|
|--|--|
|1 minute|Une mise à jour effectuée toutes les minutes peut se révéler utile lorsque vous **déboguez** le code d’une application à l’aide de Personalizer, lorsque vous effectuez des démonstrations ou lorsque vous testez interactivement certains aspects de Machine Learning.|
|15 minutes|Les fréquences élevées de mise à jour des modèles sont utiles lorsque vous souhaitez **suivre de près les modifications** apportées au comportements des utilisateurs, par exemple, dans le cas de sites d’actualités en direct, de contenu viral ou d’enchères en direct. Dans de tels scénarios, vous pouvez utiliser une fréquence de 15 minutes. |
|1 heure|Pour la plupart des cas d’usage, une fréquence de mise à jour moins élevée restera efficace.|

![La fréquence de mise à jour du modèle définit la fréquence à laquelle un nouveau modèle Personalizer est réentraîné.](media/settings/configure-model-update-frequency-settings-15-minutes.png)

Après avoir changé cette valeur, veillez à sélectionner **Enregistrer**.

## <a name="data-retention"></a>Conservation des données

**Période de conservation des données** définit le nombre de jours que Personalizer conserve les journaux de données. Les journaux de données du passé sont nécessaires pour effectuer des [évaluations hors connexion](concepts-offline-evaluation.md), qui sont utilisées pour mesurer l’efficacité de Personalizer et pour optimiser la stratégie d’apprentissage.

Après avoir changé cette valeur, veillez à sélectionner **Enregistrer**.



## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur la gestion de votre modèle](how-to-manage-model.md)

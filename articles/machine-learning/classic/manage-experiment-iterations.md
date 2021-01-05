---
title: 'ML Studio (classique) : Affichage et réexécution des expériences – Azure'
description: Gérez des exécutions d’expériences dans Azure Machine Learning Studio (classique). Vous pouvez consulter les précédentes exécutions de vos expériences à tout moment pour défier, réexaminer et finalement confirmer ou affiner les hypothèses précédentes.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 03/20/2017
ms.openlocfilehash: c958de5e49fbb7519ae71b13e9deba9af0c698de
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93312869"
---
# <a name="manage-experiment-runs-in-azure-machine-learning-studio-classic"></a>Gérer des exécutions d’expériences dans Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Le développement d'un modèle d'analyse prédictive est un processus itératif : lorsque vous modifiez les diverses fonctions et les paramètres de votre expérience, vos résultats convergent jusqu'à ce que l'efficacité du modèle formé vous donne satisfaction. L'élément clé de ce processus est le suivi des différentes itérations de vos paramètres d'expérience et de vos configurations.

Vous pouvez consulter les précédentes exécutions de vos expériences à tout moment pour défier, réexaminer et finalement confirmer ou affiner les hypothèses précédentes. Quand vous exécutez une expérience, Machine Learning Studio (classique) conserve un historique de l’exécution, y compris l’ensemble de données, le module, les connexions de port et les paramètres. Cet historique capture également les résultats, les informations d'exécution telles que les heures de démarrage et d'arrêt, les messages de journal et l'état d'exécution. Vous pouvez réexaminer, à tout moment, l'une de ces exécutions pour consulter la chronologie de votre expérience et les résultats intermédiaires. Vous pouvez même utiliser une exécution précédente de votre expérience pour entamer une nouvelle phase de recherche et de découverte sur votre chemin d'accès pour la création de solutions de modélisation simples, complexes ou même d'ensemble.

> [!NOTE]
> Lorsque vous affichez une exécution précédente de l'expérience, cette version de l'expérience est verrouillée et ne peut pas être modifiée. Vous pouvez, toutefois, en enregistrer une copie en cliquant sur **ENREGISTRER SOUS** et en fournissant un nouveau nom à cette copie. Machine Learning Studio (classique) ouvre la nouvelle copie, que vous pouvez ensuite modifier et exécuter. Cette copie de votre expérience est disponible dans la liste **EXPÉRIENCES** , avec vos autres expériences.
> 
> 

## <a name="view-the-prior-run"></a>Afficher l’exécution précédente
Lorsqu'une expérience que vous avez exécutée au moins une fois est ouverte, vous pouvez afficher l'exécution précédente de l'expérience en cliquant sur **Exécution précédente** dans le volet Propriétés.

Supposons par exemple que vous créez une expérience et que vous exécutez des versions de celle-ci à 11:23, 11:42 et 11:55. Si vous ouvrez la dernière exécution de l'expérience (11:55) et que vous cliquez sur **Exécution précédente** , la version que vous avez exécutée à 11:42 est ouverte.

## <a name="view-the-run-history"></a>Afficher l’historique des exécutions
Vous pouvez afficher toutes les précédentes exécutions d'une expérience en cliquant sur **Afficher l'historique d'exécution** d'une expérience ouverte.

Supposons par exemple que vous créez une expérience à l’aide du module [Régression linéaire][linear-regression] et que vous souhaitez observer l’effet de la modification de valeur du **Taux d’apprentissage** sur les résultats de votre expérience. Vous exécutez l'expérience à plusieurs reprises en utilisant différentes valeurs pour ce paramètre, comme suit :

| Valeur du taux d'apprentissage | Heure de début de l'exécution |
| --- | --- |
| 0.1 |11/9/2014 16:18:58 |
| 0.2 |11/9/2014 16:24:33 |
| 0.4 |11/9/2014 16:28:36 |
| 0.5 |11/9/2014 16:33:31 |

Si vous cliquez sur **AFFICHER L'HISTORIQUE D'EXÉCUTION** , une liste de toutes ces exécutions apparaîtra :

![Exemple d'historique d'exécution](./media/manage-experiment-iterations/viewrunhistory.jpg)

Cliquez sur une de ces exécutions pour afficher un instantané de l'expérience lorsque vous l'exécutiez. La configuration, les valeurs de paramètre, les commentaires et les résultats sont tous conservés pour vous fournir un enregistrement complet de l'exécution de votre expérience.

> [!TIP]
> Pour documenter les itérations de votre expérience, vous pouvez modifier le titre à chaque fois que vous l'exécutez, mettre à jour le **Résumé** de l'expérience dans le volet Propriétés et ajouter ou mettre à jour des commentaires sur les modules individuels pour enregistrer vos modifications. Le titre, le résumé et les commentaires du module sont enregistrés chaque fois que l'expérience est exécutée.
> 
> 

La liste des expériences de l’onglet **EXPÉRIENCES** dans Machine Learning Studio (classique) affiche toujours la version la plus récente d’une expérience. Si vous ouvrez une exécution précédente de l'expérience (via **Exécution précédente** ou **AFFICHER L'HISTORIQUE D'EXÉCUTION** ), vous pouvez revenir à la version brouillon en cliquant sur **AFFICHER L'HISTORIQUE D'EXÉCUTION** et en sélectionnant l'itération dont l' **ÉTAT** est défini sur **Modifiable**.

## <a name="run-a-previous-experiment"></a>Exécuter une expérience précédente
Lorsque vous cliquez sur **Exécution précédente** ou **AFFICHER L'HISTORIQUE D'EXÉCUTION** et que vous ouvrez une exécution précédente, vous pouvez afficher une expérience terminée en mode lecture seule.

Si vous souhaitez lancer une itération du début de votre expérience, configurée de la même manière qu'une exécution précédente, ouvrez l'exécution et cliquez sur **ENREGISTRER SOUS**. Ceci créera une nouvelle expérience, avec un nouveau titre, un historique d'exécution vierge ainsi que tous les composants et valeurs de paramètre de l'exécution précédente. Cette nouvelle expérience est répertoriée sous l’onglet **EXPÉRIENCES** de la page d’accueil Machine Learning Studio (classique) et vous pouvez la modifier et l’exécuter, en lançant un nouvel historique d’exécution pour cette itération de votre expérience. 

Supposons par exemple que l'historique d'exécution d'une expérience est indiqué dans la section précédente. Vous souhaitez découvrir ce qui se passe lorsque vous définissez le **Taux d'apprentissage** paramètre sur 0,4, et essayer différentes valeurs pour le paramètre **Nombre d'époques de formation**.

1. Cliquez sur **AFFICHER L'HISTORIQUE D'EXÉCUTION** et ouvrez l'itération de l'expérience que vous avez exécutée à 16:28:36 (pour laquelle vous définissez la valeur de paramètre sur 0,4).
2. Cliquez sur **ENREGISTRER SOUS**.
3. Entrez un nouveau titre et cliquez sur la coche **OK** . Une nouvelle copie de l'expérience est créée.
4. Modifiez le paramètre **Nombre d'époques de formation** .
5. Cliquez sur **EXÉCUTER**.

Vous pouvez maintenant continuer à modifier et exécuter cette version de votre expérience, en créant un nouvel historique d'exécution pour enregistrer votre travail.

<!-- Module References -->
[linear-regression]: /azure/machine-learning/studio-module-reference/linear-regression
---
title: 'Entrée/Sortie du service web : sur le module Modifier les métadonnées'
description: Découvrez comment utiliser les modules de service web dans le concepteur Azure Machine Learning pour gérer les entrées et les sorties.
titleSuffix: Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/13/2020
ms.openlocfilehash: 475ad3de8e0a7636a14949d4fcd8a5ec2812ad5e
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421122"
---
# <a name="web-service-input-and-web-service-output-modules"></a>Modules Entrée du service web et Sortie du service web

Cet article décrit les modules Entrée du service web et Sortie du service web dans le concepteur Azure Machine Learning.

Le module Entrée du service web peut uniquement se connecter à un port d’entrée de type **DataFrameDirectory**. Le module Sortie du service web peut uniquement être connecté à partir d’un port de sortie de type **DataFrameDirectory**. Les deux modules se trouvent dans l’arborescence du module, sous la catégorie **Service web**. 

Le module Entrée du service web indique où les données utilisateur entrent dans le pipeline. Le module Sortie du service web indique où les données utilisateur sont retournées dans un pipeline d’inférence en temps réel.

## <a name="how-to-use-web-service-input-and-output"></a>Comment utiliser les modules Entrée/Sortie du service web

Quand vous [créez un pipeline d’inférence en temps réel](../tutorial-designer-automobile-price-deploy.md#create-a-real-time-inference-pipeline) à partir de votre pipeline d’entraînement, les modules Entrée du service web et Sortie du service web sont automatiquement ajoutés pour indiquer l’emplacement où les données utilisateur entrent dans le pipeline et l’emplacement où elles sont retournées. 

> [!NOTE]
> La génération automatique du pipeline d’inférence en temps réel est un processus de meilleur effort basé sur une règle. Il n’y a aucune garantie d’exactitude. 

Vous pouvez ajouter ou supprimer manuellement les modules Entrée du service web et Sortie du service web pour répondre à vos besoins. Vérifiez qu’il existe au moins un module Entrée du service web et un module Sortie du service web dans votre pipeline d’inférence en temps réel. Si vous avez plusieurs modules Entrée du service web ou Sortie du service web, vérifiez qu’ils ont des noms uniques. Vous pouvez entrer le nom dans le volet droit du module.

Vous pouvez également créer manuellement un pipeline d’inférence en temps réel en ajoutant les modules Entrée du service web et Sortie du service web à votre pipeline avant de l’envoyer.

> [!NOTE]
> Le type de pipeline sera déterminé au premier envoi du pipeline. Veillez donc à ajouter les modules Entrée du service web et Sortie du service web avant le premier envoi.

L’exemple ci-dessous montre comment créer manuellement un pipeline d’inférence en temps réel à partir du module Exécuter un script Python. 

![Exemple](media/module/web-service-input-output-example.png)
   
Une fois le pipeline envoyé et l’exécution terminée, vous pouvez déployer le point de terminaison en temps réel.
   
> [!NOTE]
>  Dans l’exemple précédent, l’option **Entrer des données manuellement** fournit le schéma de données pour l’entrée du service web et elle est nécessaire pour déployer le point de terminaison en temps réel. En règle générale, vous devez toujours connecter un module ou un jeu de données au port où le module **Entrée de service web** est connecté pour fournir le schéma de données.
   
## <a name="next-steps"></a>Étapes suivantes
Découvrez-en plus sur le [déploiement du point de terminaison en temps réel](../tutorial-designer-automobile-price-deploy.md#deploy-the-real-time-endpoint).

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning.
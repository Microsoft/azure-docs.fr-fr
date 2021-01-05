---
title: Évolutivité et performance - Personalizer
titleSuffix: Azure Cognitive Services
description: 'Les sites web et applications à trafic élevé et hautes performances ont deux principaux facteurs à prendre en considération avec Personalizer pour l’évolutivité et les performances : la latence et le rythme d’apprentissage.'
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 10/24/2019
ms.openlocfilehash: 52d22fce64a26267e73cb9a1df8614129bb96c3e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87132685"
---
# <a name="scalability-and-performance"></a>Extensibilité et performance

Les sites web et applications à trafic élevé et hautes performances ont deux principaux facteurs à prendre en considération avec Personalizer pour l’évolutivité et les performances :

* Maintenir une faible latence lors des appels à l’API de classement
* S’assurer que le rythme d’apprentissage s’adapte à l’entrée des événements

La personnalisation peut retourner un classement rapidement, la plus grande partie de la durée de l’appel étant dédiée à la communication via l’API REST. Azure mettra à l’échelle la capacité à répondre rapidement aux demandes.

##  <a name="low-latency-scenarios"></a>Scénarios de faible latence

Certaines applications nécessitent des latences faibles lors du retour d’un classement. Des latences faibles sont nécessaires :

* Pour empêcher l’utilisateur d’avoir à attendre avant d’afficher un contenu classé.
* Pour aider un serveur qui subit un trafic extrême à éviter de mobiliser le temps de calcul et les connexions réseau.


## <a name="scalability-and-training-throughput"></a>Extensibilité et rythme d’apprentissage

Personalizer fonctionne en mettant à jour un modèle qui est reformé en fonction des messages envoyés de façon asynchrone par Personalizer après les API de classement et de récompense. Ces messages sont envoyés à l’aide d’un EventHub Azure pour l’application.

 La plupart des applications n’atteindront probablement pas les niveaux de jointure et de rythme d’apprentissage maximaux de Personalizer. Bien qu’atteindre ce maximum ne ralentirait pas l’application, cela signifierait que les files d’attente d’Event Hub se rempliraient en interne plus rapidement qu’elles peuvent être nettoyées.

## <a name="how-to-estimate-your-throughput-requirements"></a>Comment estimer vos besoins en débit

* Estimez le nombre moyen d’octets par événement de classement en ajoutant les longueurs des documents JSON de contexte et d’action.
* Divisez 20 Mo par seconde par ce nombre moyen d’octets estimé.

Par exemple, si votre charge utile moyenne comprend 500 fonctionnalités et que chacune est estimée à 20 caractères, chaque événement est d’environ 10 ko. Avec ces estimations, 20 000 000/10 000 = 2 000 événements par seconde, ce qui équivaut à environ 173 millions d’événements par jour. 

Si vous atteignez ces limites, contactez notre équipe de support pour des conseils d’architecture.

## <a name="next-steps"></a>Étapes suivantes

[Créer et configurer Personalizer](how-to-settings.md).
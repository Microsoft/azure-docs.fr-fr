---
title: Test de simulation Azure DDoS Protection
description: Découvrez comment tester à l’aide de simulations
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/08/2020
ms.author: yitoh
ms.openlocfilehash: eff738e24b3abce52e80291c55a3ae64c3c8c853
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92904887"
---
# <a name="test-through-simulations"></a>Tester à l’aide de simulations

Nous vous conseillons de tester vos hypothèses sur la façon dont vos services répondront à une attaque au moyen de simulations périodiques. Ces tests permettent de vérifier que vos services ou applications continuent à fonctionner comme prévu et que l’expérience des utilisateurs n’est pas interrompue. Identifiez les lacunes du point de vue de la technologie et du processus, et incorporez-les dans la stratégie de réponse DDoS. Nous vous recommandons d’effectuer ces tests dans des environnements intermédiaires ou durant les heures creuses afin de minimiser l’impact sur l’environnement de production.

Nous travaillons en collaboration avec [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud), un générateur de trafic en libre service, pour créer une interface à l’aide de laquelle les clients Azure peuvent générer du trafic sur les points de terminaison publics dotés de DDoS Protection à des fins de simulation. Vous pouvez utiliser la simulation pour :

- Valider la façon dont Azure DDoS Protection contribue à protéger vos ressources Azure contre les attaques DDoS.
- Optimiser votre processus de réponse aux incidents en cas d’attaque DDoS.
- Documenter la conformité DDoS.
- Former des équipes de sécurité réseau.

## <a name="prerequisites"></a>Prérequis

- Avant de pouvoir exécuter la procédure de ce tutoriel, vous devez créer un [plan de protection Azure DDoS Standard](manage-ddos-protection.md) avec des adresses IP publiques protégées.
- Vous devez d’abord créer un compte avec [BreakingPoint Cloud](http://breakingpoint.cloud/). 

## <a name="configure-a-ddos-test-attack"></a>Configurer une attaque de test DDoS

1. Entrez ou sélectionnez les valeurs suivantes, puis sélectionnez **Démarrer le test**  :

    |Paramètre        |Valeur                                              |
    |---------      |---------                                          |
    |Adresse IP cible           | Entrez l’une de vos adresses IP publiques que vous voulez tester.                     |
    |Numéro de port   | Entrez _443_.                       |
    |Profil DDoS | Sélectionnez **TCP SYN Flood**.|
    |Taille du test       | Sélectionnez **200 000 pps, 100 Mbits/s et 8 adresses IP sources.**                                  |
    |Durée du test | Sélectionnez **10 minutes**.|

Cela devrait maintenant ressembler à ceci :

![Exemple de simulation d’attaque DDoS : BreakingPoint Cloud](./media/ddos-attack-simulation/ddos-attack-simulation-example-1.png)

## <a name="monitor-and-validate"></a>Surveiller et valider

1. Connectez-vous à https://portal.azure.com et accédez à votre abonnement.
1. Sélectionnez l’adresse IP publique sur laquelle vous avez testé l’attaque.
1. Sous **Supervision** , sélectionnez **Métriques**.
1. Pour **Métrique** , sélectionnez _Sous attaque DDoS ou non_.

Une fois la ressource attaquée, vous devez voir que la valeur passe de **0** à **1** , comme dans l’image suivante :

![Exemple de simulation d’attaque DDoS : Portail](./media/ddos-attack-simulation/ddos-attack-simulation-example-2.png)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [consulter et configurer la télémétrie de la protection DDoS](telemetry-monitoring-alerting.md).
- Découvrez comment [configurer les rapports de prévention des attaques DDoS et les journaux de flux](reports-and-flow-logs.md).
- Découvrez comment [faire appel à une réponse rapide DDoS](ddos-rapid-response.md).
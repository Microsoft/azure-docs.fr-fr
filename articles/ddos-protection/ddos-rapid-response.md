---
title: Réponse rapide Azure DDoS
description: Découvrez comment faire appel à des experts DDoS pendant une attaque active en vue d’un support spécialisé.
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/28/2020
ms.author: yitoh
ms.openlocfilehash: 3655b117b641a6179bd6539aaf3f75af07f21a1a
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94992367"
---
# <a name="azure-ddos-rapid-response"></a>Réponse rapide Azure DDoS

Lors d’un accès actif, les clients Azure DDoS Protection Standard ont accès à l’équipe de réponse rapide DDoS (DRR), qui peut aider à faire des recherches sur une attaque au cours d’une attaque, ainsi qu’une analyse après l’attaque.

## <a name="prerequisites"></a>Prérequis

- Avant de pouvoir exécuter la procédure de ce tutoriel, vous devez créer un [plan de protection Azure DDoS Standard](manage-ddos-protection.md).

## <a name="when-to-engage-drr"></a>Quand faire appel à DRR

Vous ne devez faire appel à DRR que dans les cas suivants : 

- Au cours d’une attaque DDoS, vous constatez que les performances de la ressource protégée sont considérablement altérées ou que la ressource n’est pas disponible. Passez en revue l’étape 2 ci-dessus sur la configuration des analyses pour détecter les problèmes de disponibilité et de performances des ressources.
- Vous pensez que votre ressource fait l’objet d’une attaque DDoS, mais le service de protection DDoS n’atténue pas efficacement l’attaque.
- Vous planifiez un événement viral qui peut accroître considérablement votre trafic réseau.
- Pour les attaques qui ont un impact critique sur l’activité.

## <a name="engage-drr-during-an-active-attack"></a>Faire appel à DRR pendant une attaque active

1. Sur le portail Azure lors de la création d’une nouvelle demande de support, choisissez le **Type de problème** Technique.
2. Choisissez la **Protection DDOS** **Service**.
3. Choisissez une ressource dans le menu déroulant des ressources. _Vous devez sélectionner un plan DDoS lié au réseau virtuel protégé par la protection DDoS standard pour faire appel à DRR._

    ![Choisissez Ressource](./media/ddos-rapid-response/choose-resource.png)

4. Sur la page suivante **Problème**, sélectionnez la **gravité** A - Impact critique et le **Type de problème** « Attaque en cours ».

    ![Gravité et type de problème](./media/ddos-rapid-response/severity-and-problem-type.png)

5. Complétez les détails supplémentaires et envoyez la demande de support.

DRR suit le modèle de support Azure Rapid Response. Pour plus d’informations sur Rapid Response, consultez [Étendue du support et réactivité](https://azure.microsoft.com/en-us/support/plans/response/).

Pour en savoir plus, lisez la [Documentation Azure DDoS Protection Standard](./ddos-protection-overview.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [tester à l’aide de simulations](test-through-simulations.md).
- Découvrez comment [consulter et configurer la télémétrie de la protection DDoS](telemetry-monitoring-alerting.md).
- Découvrez comment [configurer les rapports de prévention des attaques DDoS et les journaux de flux](reports-and-flow-logs.md).
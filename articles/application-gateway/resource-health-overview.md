---
title: Vue d’ensemble du contrôle d’intégrité des ressources pour Azure Application Gateway
description: Cet article est une vue d’ensemble de la fonctionnalité de contrôle d’intégrité des ressources pour Azure Application Gateway
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 7/9/2019
ms.author: victorh
ms.openlocfilehash: 7e30a93f8270cfaf8910130cc1e2633bb80c2b8e
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397170"
---
# <a name="azure-application-gateway-resource-health-overview"></a>Vue d’ensemble du contrôle d’intégrité des ressources pour Azure Application Gateway

[Azure Resource Health](../service-health/resource-health-overview.md) vous aide à diagnostiquer les problèmes et à obtenir du support quand un problème de service Azure a une incidence sur vos ressources. Il vous informe de l’intégrité (actuelle et passée) de vos ressources. Il fournit un support technique pour vous aider à résoudre les problèmes.

Pour Application Gateway, Resource Health s’appuie sur les signaux émis par la passerelle pour évaluer si elle est intègre ou non. Si la passerelle n’est pas intègre, Resource Health analyse des informations supplémentaires pour déterminer la source du problème. Il identifie également les actions adoptées par Microsoft ou ce que vous pouvez faire pour résoudre le problème.

Pour plus d’informations sur la procédure d’évaluation de l’intégrité, passez en revue la liste complète de types de ressource et de vérifications d’intégrité dans [Azure Resource Health](../service-health/resource-health-checks-resource-types.md#microsoftnetworkapplicationgateways).


L’état d’intégrité pour Application Gateway s’affiche comme l’un des états suivants :

## <a name="available"></a>Disponible

L’état **Disponible** signifie que le service n’a pas détecté d’événement qui influe sur l’intégrité de la ressource. Quand la passerelle a été récupérée après un arrêt non planifié au cours des dernières 24 heures, vous voyez la notification **Résolution récente**.

![État d’intégrité Disponible](media/resource-health-overview/available-full.png)

## <a name="unavailable"></a>Indisponible

L’état **Non disponible** signifie que le service a détecté un événement de plateforme ou hors plateforme en cours ayant un impact sur l’intégrité de la passerelle.

### <a name="platform-events"></a>Événements de plateforme

Les événements de plateforme sont déclenchés par plusieurs composants de l’infrastructure Azure. Ils incluent à la fois les actions planifiées (par exemple, une maintenance planifiée) et les incidents inattendus (par exemple, un redémarrage de l’hôte non planifié).

Resource Health fournit des détails supplémentaires sur l’événement et le processus de récupération. Vous pouvez aussi contacter le support même si vous n’avez pas de contrat de support Microsoft actif.

![État Non disponible](media/resource-health-overview/unavailable.png)

## <a name="unknown"></a>Inconnu

L’état d’intégrité **Inconnu** indique que Resource Health n’a reçu aucune information sur cette passerelle depuis plus de 10 minutes. Cet état n’est pas une indication définitive de l’état de la passerelle. Mais c’est un point de données important dans le processus de dépannage.

Si la passerelle s’exécute comme prévu, l’état passe à **Disponible** après quelques minutes.

Si vous rencontrez des problèmes avec la passerelle, l’état d’intégrité **Inconnu** peut suggérer qu’un événement de la plateforme influe sur la passerelle.

![État Inconnu](media/resource-health-overview/unknown.png)

## <a name="degraded"></a>Détérioré

L’état d’intégrité **Détérioré** indique que votre passerelle a détecté une perte de performances, bien qu’elle soit toujours disponible à l’utilisation.

![État Détérioré](media/resource-health-overview/degraded.png)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la résolution des problèmes du pare-feu d'applications web (WAF) d’Application Gateway, consultez [Résolution des problèmes de pare-feu d'applications web (WAF) pour Azure Application Gateway](../web-application-firewall/ag/web-application-firewall-troubleshoot.md).
---
title: Modèle d’intégration d’Azure Peering Service
description: Apprenez-en davantage sur l’intégration d’Azure Peering Service
services: peering-service
documentationcenter: na
author: derekolo
ms.service: peering-service
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Infrastructure-services
ms.date: 05/18/2020
ms.author: derekol
ms.openlocfilehash: 7a257b4c2bea3bd3384a55c0a6b85d7fdd2ca583
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84871399"
---
# <a name="onboarding-peering-service-model"></a>Modèle d’intégration de Peering Service

Le processus d’intégration de Peering Service est constitué de deux modèles, comme indiqué ci-dessous :

 - Intégration de connexion Peering Service

 - Télémétrie des connexions Azure Peering Service

Les plans d’action pour les modèles répertoriés ci-dessus sont décrits ci-dessous :

| **Étape** | **Action**| **Ce que vous obtenez**|
|-----------|---------|---------|---------|
|1|Client pour approvisionner la connectivité d’un partenaire de connectivité (sans interaction avec Microsoft). |Un fournisseur d’accès Internet bien connecté à Microsoft et répondant aux exigences techniques pour une connectivité performante et fiable à Microsoft.  |
|2 (Facultatif)|Le client enregistre les emplacements dans le portail Azure. Un emplacement est défini par le nom de l’ISP/IXP, l’emplacement physique du site du client (niveau d’état), le préfixe IP attribué à l’emplacement par le fournisseur de services ou l’entreprise.  |Télémétrie : surveillance des itinéraires Internet, hiérarchisation du trafic de Microsoft vers l’emplacement POP Edge le plus proche de l’utilisateur. |



## <a name="onboarding-peering-service-connection"></a>Intégration de connexion Peering Service

Pour intégrer une connexion Peering Service, procédez comme suit :

Faites appel à un fournisseur de services Internet ou à un partenaire d’échange Internet (IX) afin d’obtenir Peering Service pour connecter votre réseau au réseau Microsoft.

Assurez-vous que les [fournisseurs de connectivité](location-partners.md) sont des partenaires de Microsoft pour Peering Service. 

## <a name="onboarding-peering-service-connection-telemetry"></a>Télémétrie des connexions Azure Peering Service

Les clients peuvent choisir une télémétrie telle que l’analyses d’itinéraire BGP pour surveiller la latence et les performances de mise en réseau lors de l’accès au réseau Microsoft. Pour ce faire, vous devez inscrire la connexion dans le portail Azure.

Pour intégrer la télémétrie de connexion Peering Service, le client doit inscrire la connexion Peering Service dans le portail Azure. Pour savoir comment procéder, consultez [Inscrire Peering Service – Portail Azure](azure-portal.md).

Après cela, vous pouvez mesurer la télémétrie [ici](measure-connection-telemetry.md).

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir le processus pas à pas d’inscription de connexion Peering Service, consultez [Inscrire Peering Service – Portail Azure](azure-portal.md).

Pour en savoir plus sur la télémétrie de connexion, consultez [Mesurer la télémétrie de la connexion](measure-connection-telemetry.md).

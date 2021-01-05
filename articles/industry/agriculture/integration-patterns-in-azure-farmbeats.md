---
title: Architecture Azure FarmBeats
description: Décrit l’architecture d’Azure FarmBeats
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: d64c2175072d9979cfda2ea5f75beb34d3ad0d6b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75475719"
---
# <a name="integration-patterns"></a>Modèles d’intégration

Azure FarmBeats est une offre interentreprise, disponible dans la Place de marché Azure. FarmBeats permet l’agrégation de jeux de données agricoles entre les fournisseurs et la génération d’informations exploitables en créant des modèles d’intelligence artificielle (IA) ou de Machine Learning (ML) en fusionnant les jeux de données.

![Projet FarmBeats](./media/architecture-for-farmbeats/farmbeats-architecture-1.png)

Les sections suivantes décrivent le modèle d’intégration pour Azure FarmBeats.

## <a name="why-integrate-with-azure-farmbeats"></a>Pourquoi intégrer avec Azure FarmBeats ?

Cette section est axée sur les partenaires qui souhaitent intégrer leurs systèmes de données (comme les capteurs, les drones, les stations météo) dans Azure FarmBeats.

Azure FarmBeats est une offre extensible qui permet aux entreprises agricoles d’ajouter leurs différents jeux de données agricoles historiques et en temps réel à une seule plateforme. Azure FarmBeats aide une entreprise agricole à normaliser, contextualiser et agréger ses données dans le contexte d’une exploitation agricole.

En devenant un partenaire de données avec Azure FarmBeats, vous pouvez ouvrir vos systèmes à une adoption plus vaste et atteindre plus de clients avec vos offres de données. Azure FarmBeats fournit une couche d’API extensible appelée Datahub, qui vous permet d’ingérer systématiquement les données de vos appareils dans un schéma standardisé.

Une fois que les données sont disponibles dans l’instance Azure FarmBeats de vos clients, vos clients peuvent créer des analytiques et des outils plus riches en plus de vos données.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’intégration des données de capteurs, consultez [Intégration des données de capteurs](sensor-partner-integration-in-azure-farmbeats.md), et pour l’intégration des partenaires d’imagerie, consultez [Intégration des partenaires d’imagerie](imagery-partner-integration-in-azure-farmbeats.md).

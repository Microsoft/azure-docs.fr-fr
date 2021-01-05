---
title: Vue d’ensemble de l’ingestion – Azure Time Series Insights Gen2 | Microsoft Docs
description: Apprenez-en davantage sur l’ingestion des données dans Azure Time Series Insights Gen2.
author: lyrana
ms.author: lyhughes
manager: deepakpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 12/02/2020
ms.custom: seodec18
ms.openlocfilehash: 2b76490ab23cb14755f6f90a27ec8b176bb30a11
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780023"
---
# <a name="azure-time-series-insights-gen2-data-ingestion-overview"></a>Vue d’ensemble de l’ingestion de données dans Azure Time Series Insights Gen2

Votre environnement Azure Time Series Insights Gen2 contient un *moteur d’ingestion* pour collecter, traiter et stocker des données de série chronologique de diffusion en continu. À mesure que les données arrivent dans vos sources d'événements, Azure Time Series Insights Gen2 consomme et stocke vos données en quasi-temps réel.

[![Vue d’ensemble de l’ingestion](media/concepts-ingress-overview/ingress-overview.png)](media/concepts-ingress-overview/ingress-overview.png#lightbox)

## <a name="ingestion-topics"></a>Rubriques relatives à l’ingestion

Les articles suivants décrivent en détail le traitement des données, notamment les bonnes pratiques à suivre :

* En savoir plus sur les [sources d’événements](./concepts-streaming-ingestion-event-sources.md) et des conseils sur la sélection d’un horodateur de source d’événement.

* Vérifier les [types de données](./concepts-supported-data-types.md) pris en charge

* Découvrez comment le moteur d’ingestion appliquera un ensemble de [règles](./concepts-json-flattening-escaping-rules.md) à vos propriétés JSON pour créer vos colonnes de compte de stockage.

* Passez en revue les [limitations du débit](./concepts-streaming-ingress-throughput-limits.md) de votre environnement pour planifier vos besoins en matière de mise à l’échelle.

## <a name="next-steps"></a>Étapes suivantes

* Continuez pour en savoir plus sur les [sources d’événements](./concepts-streaming-ingestion-event-sources.md) pour votre environnement Azure Time Series Insights Gen2.

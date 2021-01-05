---
title: Qu’est-ce qu’Azure Service Health ?
description: Obtenez des informations personnalisées concernant l’incidence des problèmes et de la maintenance actuels et futurs d’Azure sur vos applications Azure.
ms.topic: overview
ms.date: 05/10/2019
ms.openlocfilehash: cb1eb8eb75a18a73e8f024f02217df37f45981fe
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87499770"
---
# <a name="what-is-azure-service-health"></a>Qu’est-ce qu’Azure Service Health ?

Azure propose une suite d’expériences destinées à vous tenir informé de l’état de vos ressources cloud. Ces informations incluent les problèmes actuels et à venir tels que les événements ayant un impact sur le service, la maintenance planifiée et d’autres modifications susceptibles d’affecter votre disponibilité.

Azure Service Health est une combinaison de trois services distincts plus petits.

L’article [État Azure](azure-status-overview.md) vous informe des pannes de service d’Azure indiquées dans la **[page État d’Azure](https://status.azure.com)** . Cette page vous présente une vue globale de l’intégrité des services Azure dans l’ensemble des régions Azure. Cette page constitue une bonne référence pour les incidents ayant un large impact. Toutefois, il est fortement recommandé aux utilisateurs Azure d’utiliser Azure Service Health pour rester informés des incidents et de la maintenance concernant les services Azure.

[Service Health](service-health-overview.md) fournit une vue personnalisée de l’intégrité des services et régions Azure que vous utilisez. C’est le meilleur endroit où rechercher des informations relatives aux pannes, aux activités de maintenance planifiée et aux autres avis concernant l’intégrité, car l’expérience Service Health authentifiée a connaissance des services et des ressources que vous utilisez actuellement. Le meilleur moyen d’utiliser Service Health est de configurer des alertes pour recevoir, par le biais de vos canaux de communication préférés, des notifications vous informant des problèmes affectant les services, des maintenances planifiées et autres changements pouvant impacter les services et régions Azure que vous utilisez.

[Resource Health](resource-health-overview.md) fournit des informations sur l’intégrité de vos ressources cloud individuelles, comme une instance de machine virtuelle spécifique. Avec Azure Monitor, vous pouvez également configurer des alertes pour être averti des changements de disponibilité affectant vos ressources cloud. Resource Health et les notifications Azure Monitor vous permettent d’être mieux informé de la disponibilité de vos ressources à chaque minute. En outre, ils vous permettent de déterminer rapidement si un problème est de votre côté ou s’il est lié à un événement de la plateforme Azure.

Ensemble, ces expériences vous donnent une vue complète de l’intégrité d’Azure, avec la granularité la plus appropriée pour vous.

**Regarder une présentation de la page État d’Azure, d’Azure Service Health et d’Azure Resource Health**

>[!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2OgX6]

[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]
---
author: IEvangelist
ms.author: dapine
ms.date: 10/02/2019
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: a6f399e19cadf3d6ce9edfaecb3d904e62c498aa
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006869"
---
Le conteneur a les paramètres de configuration suivants :

|Obligatoire|Paramètre|Objectif|
|--|--|--|
|Oui|[ApiKey](#apikey-configuration-setting)|Assure le suivi des informations de facturation.|
|Non|[ApplicationInsights](#applicationinsights-setting)|Permet d’ajouter la prise en charge de la télémétrie [Azure Application Insights](/azure/application-insights) à votre conteneur.|
|Oui|[Billing](#billing-configuration-setting)|Spécifie l’URI de point de terminaison de la ressource de service sur Azure.|
|Oui|[Eula](#eula-setting)| Indique que vous avez accepté la licence pour le conteneur.|
|Non|[Fluentd](#fluentd-settings)|Écrit les données des journaux et, éventuellement, des métriques, sur un serveur Fluentd.|
|Non|Proxy HTTP|Configure un proxy HTTP pour effectuer des requêtes sortantes.|
|Non|[Logging](#logging-settings)|Fournit la prise en charge de la journalisation ASP.NET Core pour votre conteneur. |
|Non|[Mounts](#mount-settings)|Lit et écrit des données de l’ordinateur hôte sur le conteneur, et du conteneur sur l’ordinateur hôte.|
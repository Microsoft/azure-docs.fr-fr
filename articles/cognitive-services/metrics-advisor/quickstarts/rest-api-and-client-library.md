---
title: API REST des bibliothèques de client Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Utilisez ce guide de démarrage rapide pour connecter vos applications à l’API Metrics Advisor d’Azure Cognitive Services.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: quickstart
ms.date: 10/14/2020
ms.author: mbullwin
zone_pivot_groups: programming-languages-metrics-monitor
ms.openlocfilehash: 5b3df5f4b41b2beeec68b667863f6ca7715df47b
ms.sourcegitcommit: 7a7b6c7ac0aa9dac678c3dfd4b5bcbc45dc030ca
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2020
ms.locfileid: "93186936"
---
# <a name="quickstart-use-the-client-libraries-or-rest-apis-to-customize-your-solution"></a>Démarrage rapide : Utiliser les bibliothèques de client ou les API REST pour personnaliser votre solution

Démarrez avec les bibliothèques de client ou les API REST Metrics Advisor. Suivez les étapes suivantes pour installer le package et essayer l’exemple de code pour les tâches de base.

Utilisez Metrics Advisor pour :

* Ajouter un flux de données à partir d’une source de données
* Vérifier l’état d’ingestion
* Configurer la détection et les alertes 
* Interroger les résultats de la détection des anomalies
* Diagnostiquer les anomalies


::: zone pivot="programming-language-csharp"

[!INCLUDE [REST API quickstart](../includes/quickstarts/csharp.md)]

::: zone-end

::: zone pivot="programming-language-java"

[!INCLUDE [REST API quickstart](../includes/quickstarts/java.md)]

::: zone-end

::: zone pivot="programming-language-javascript"

[!INCLUDE [REST API quickstart](../includes/quickstarts/javascript.md)]

::: zone-end

::: zone pivot="programming-language-python"

[!INCLUDE [REST API quickstart](../includes/quickstarts/python.md)]

::: zone-end

::: zone pivot="programming-language-rest-api"

[!INCLUDE [REST API quickstart](../includes/quickstarts/rest-api.md)]

::: zone-end

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser le portail web](web-portal.md)
- [Intégrer vos flux de données](../how-tos/onboard-your-data.md)
    - [Gérer les flux de données](../how-tos/manage-data-feeds.md)
    - [Configurations pour différentes sources de données](../data-feeds-from-different-sources.md)
- [Configurer des métriques et affiner la configuration de la détection](../how-tos/configure-metrics.md)
- [Ajuster la détection des anomalies à l’aide de commentaires](../how-tos/anomaly-feedback.md)
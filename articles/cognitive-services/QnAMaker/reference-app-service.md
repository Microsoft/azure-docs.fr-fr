---
title: Configuration du service – QnA Maker
description: Comprendre comment et où configurer les ressources.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 11/9/2020
ms.openlocfilehash: e7dfc6db50432b19d515671fabe26edf9cfb45ef
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96345666"
---
# <a name="service-configuration"></a>Configuration de service

Chaque version de QnA Maker utilise un ensemble différent de ressources (services) Azure. Cet article décrit les personnalisations prises en charge pour ces services. 

## <a name="app-service"></a>App Service

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

QnA Maker utilise App Service pour fournir le runtime de requête utilisé par l’[API generateAnswer](/rest/api/cognitiveservices/qnamaker4.0/runtime/generateanswer).

Ces paramètres sont disponibles dans le Portail Azure, pour App Service. Les paramètres sont disponibles en sélectionnant **Paramètres**, puis **Configuration**.

Vous pouvez définir un paramètre individuel via la liste des paramètres de l’application ou modifier plusieurs paramètres en sélectionnant **Modification avancée**.

|Ressource|Paramètre|
|--|--|
|AzureSearchAdminKey|Recherche cognitive : utilisée pour le stockage des paires de questions et réponses et le classement n°1|
|AzureSearchName|Recherche cognitive : utilisée pour le stockage des paires de questions et réponses et le classement n°1|
|DefaultAnswer|Texte de réponse quand aucune correspondance n’est trouvée|
|UserAppInsightsAppId|Journal et télémétrie de conversation|
|UserAppInsightsKey|Journal et télémétrie de conversation|
|UserAppInsightsName|Journal et télémétrie de conversation|

Vous devez **redémarrer** le service à partir de la page **Vue d’ensemble** du Portail Azure, une fois que vous avez apporté des modifications.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Les personnalisations App Service ne s’appliquent pas à QnA Maker managé (préversion).

---

## <a name="qna-maker-service"></a>Service d’établissement QnA

Le service QnA Maker fournit une configuration permettant aux utilisateurs suivants de collaborer sur un seul service QnA Maker, ainsi que toutes ses bases de connaissances.

Découvrez [comment ajouter des collaborateurs](./reference-role-based-access-control.md) à votre service.

## <a name="change-azure-cognitive-search"></a>Changer le service Recherche cognitive Azure

Découvrez [comment changer le service Recherche cognitive](./how-to/set-up-qnamaker-service-azure.md#configure-qna-maker-to-use-different-cognitive-search-resource) lié à votre service QnA Maker.

## <a name="change-default-answer"></a>Changer la réponse par défaut

Découvrez [comment changer le texte de vos réponses par défaut](How-To/change-default-answer.md). 

## <a name="telemetry"></a>Télémétrie

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Application Insights est utilisé pour superviser la télémétrie avec QnA Maker GA. Il n’existe aucun paramètre de configuration propre à QnA Maker.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Découvrez [comment ajouter des données de télémétrie à votre service QnA Maker managé (préversion).](How-To/get-analytics-knowledge-base.md) 

---

## <a name="app-service-plan"></a>Plan App Service

# <a name="qnamaker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Plan App Service n’a aucun paramètre de configuration spécifique à QnA Maker.

# <a name="qnamaker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Plan App Service n’est pas utilisé avec QnA Maker managé (préversion).

---

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [formats](reference-document-format-guidelines.md) pour les documents et les URL que vous souhaitez importer dans une base de connaissances.
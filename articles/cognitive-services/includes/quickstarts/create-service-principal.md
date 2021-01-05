---
title: Créer un principal du service Azure
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/01/2020
ms.author: pafarley
ms.openlocfilehash: 99487fe962228f775582aa2f0a6fb60fe52862ac
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95096617"
---
## <a name="create-an-azure-service-principal"></a>Créer un principal de service Azure

Pour que votre application interagisse avec votre compte Azure, un principal de service Azure doit gérer les autorisations. Suivez les instructions de l'article [Créer un principal de service Azure](/powershell/azure/create-azure-service-principal-azureps?view=azps-4.4.0&viewFallbackFrom=azps-3.3.0).

Lorsque vous créez un principal de service, vous constatez qu'il dispose d'une valeur de secret, d'un ID et d'un ID d'application. Enregistrez l'ID d'application et le secret à un emplacement temporaire. Vous en aurez besoin ultérieurement.
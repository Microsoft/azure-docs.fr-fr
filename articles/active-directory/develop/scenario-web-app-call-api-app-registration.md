---
title: Inscrire une application web appelant des API web - Plateforme d’identités Microsoft | Azure
description: Découvrir comment inscrire une application web qui appelle des API web
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 863d8d87d5de771293199dd60618128b0ac11bba
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94442597"
---
# <a name="a-web-app-that-calls-web-apis-app-registration"></a>Application web appelant des API web : Inscription d'application

Une application web appelant des API web a la même inscription qu’une application web connectant des utilisateurs. Par conséquent, suivez les instructions de [une application Web qui se connecte aux utilisateurs : Inscription d’application](scenario-web-app-sign-user-app-registration.md).

Cependant, comme l’application web appelle maintenant des API web, elle devient une application cliente confidentielle. C’est pourquoi une inscription supplémentaire est requise. L’application doit partager les informations d’identification de client, ou *secrets*, avec la plateforme d’identités Microsoft.

[!INCLUDE [Registration of client secrets](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="api-permissions"></a>Autorisations des API

Les applications web appellent des API pour le compte de l’utilisateur connecté. Pour ce faire, elles doivent demander des *autorisations déléguées*. Pour plus d’informations, consultez [Ajouter des autorisations pour accéder à votre API web](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api).

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Configuration du code](scenario-web-app-call-api-app-configuration.md).

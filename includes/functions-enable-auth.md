---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 04/21/2020
ms.author: glenga
ms.openlocfilehash: 380878fedaa2f7ea6160c3c4801c8dece6e1a9ff
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83648970"
---
#### <a name="enable-app-service-authenticationauthorization"></a>Activer l’authentification/autorisation App Service

La plateforme App Service vous permet d’utiliser AAD (Azure Active Directory) et plusieurs fournisseurs d’identité tiers pour authentifier les clients. Vous pouvez utiliser cette stratégie afin d'implémenter des règles d'autorisation personnalisées pour vos fonctions, et vous pouvez utiliser les informations utilisateur dans le code de votre fonction. Pour plus d’informations, consultez [Authentification et autorisation dans Azure App Service](../articles/app-service/overview-authentication-authorization.md) et [Utilisation des identités de clients](../articles/azure-functions/functions-bindings-http-webhook-trigger.md#working-with-client-identities).

#### <a name="use-azure-api-management-apim-to-authenticate-requests"></a>Utiliser la Gestion des API Azure (APIM) pour authentifier les requêtes

Gestion des API Azure offre une variété d’options de sécurité des API pour les requêtes entrantes. Pour plus d’informations, consultez [Stratégies d’authentification dans Gestion des API](../articles/api-management/api-management-authentication-policies.md). Avec Gestion des API Azure en place, vous pouvez configurer votre application de fonction pour qu’elle accepte seulement les requêtes provenant de l’adresse IP de votre instance Gestion des API Azure. Pour plus d’informations, consultez [Restriction des adresses IP](../articles/azure-functions/ip-addresses.md#ip-address-restrictions).

---
title: Obtenir un jeton d’accès à l’aide d’Azure CLI – API Azure pour FHIR
description: Cet article explique comment obtenir un jeton d’accès pour l’API Azure pour FHIR à l’aide d’Azure CLI.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/26/2019
ms.author: matjazl
ms.openlocfilehash: 4d1c4cfcb15d97a2c54a04344f0bd098f65c1392
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660370"
---
# <a name="get-access-token-for-azure-api-for-fhir-using-azure-cli"></a>Obtenir un jeton d’accès pour l’API Azure pour FHIR à l’aide d’Azure CLI

Cet article explique comment obtenir un jeton d’accès pour l’API Azure pour FHIR à l’aide d’Azure CLI. Lorsque vous [approvisionnez l’API Azure pour FHIR](fhir-paas-portal-quickstart.md), vous pouvez configurer un ensemble d’utilisateurs ou de principaux de service qui ont accès au service. Si votre ID d’objet utilisateur figure dans la liste des ID d’objet autorisés, vous pouvez accéder le service à l’aide d’un jeton obtenu à l’aide d’Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="obtain-a-token"></a>Obtenir un jeton

L’API Azure pour FHIR utilise `resource` ou `Audience` avec l’URI du serveur FHIR `https://<FHIR ACCOUNT NAME>.azurehealthcareapis.com`. Vous pouvez obtenir un jeton et le stocker dans une variable (nommée `$token`) avec la commande suivante :

```azurecli-interactive
token=$(az account get-access-token --resource=https://<FHIR ACCOUNT NAME>.azurehealthcareapis.com --query accessToken --output tsv)
```

## <a name="use-with-azure-api-for-fhir"></a>Utilisation avec l’API Azure pour FHIR

```azurecli-interactive
curl -X GET --header "Authorization: Bearer $token" https://<FHIR ACCOUNT NAME>.azurehealthcareapis.com/Patient
```

## <a name="next-steps"></a>Étapes suivantes

Cet article explique comment obtenir un jeton d’accès pour l’API Azure pour FHIR à l’aide d’Azure CLI. Pour savoir comment accéder à l’API FHIR à l’aide de Postman, effectuez le didacticiel Postman.

>[!div class="nextstepaction"]
>[Accédez à l’API FHIR à l’aide de Postman](access-fhir-postman-tutorial.md)

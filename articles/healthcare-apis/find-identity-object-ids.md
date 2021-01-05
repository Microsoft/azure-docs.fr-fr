---
title: 'Rechercher des ID d’objet d’identité pour l’authentification : API Azure pour FHIR'
description: Cet article explique comment localiser les ID d’objet d’identité nécessaires pour configurer l’authentification pour l’API Azure pour FHIR
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 706d7e081743f2bab1f593e00dc792f218a000ea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90033623"
---
# <a name="find-identity-object-ids-for-authentication-configuration"></a>Rechercher des ID d’objet d’identité pour la configuration de l’authentification

Cet article explique comment rechercher des ID d’objet d’identité requis lors de la configuration de l’API Azure pour FHIR afin d’[utiliser un locataire Active Directory externe ou secondaire](configure-local-rbac.md) pour un plan de données.

## <a name="find-user-object-id"></a>Rechercher un ID d’objet d’utilisateur

Si vous avez un utilisateur avec le nom d’utilisateur `myuser@contoso.com`, vous pouvez localiser les utilisateurs `ObjectId` à l’aide de la commande PowerShell suivante :

```azurepowershell-interactive
$(Get-AzureADUser -Filter "UserPrincipalName eq 'myuser@contoso.com'").ObjectId
```

ou vous pouvez utiliser l’interface de ligne de commande Azure :

```azurecli-interactive
az ad user show --id myuser@contoso.com --query objectId --out tsv
```

## <a name="find-service-principal-object-id"></a>Rechercher un ID d’objet de principal de service

Supposons que vous avez enregistré une [application cliente de service](register-service-azure-ad-client-app.md) et que vous souhaitez autoriser ce client de service pour accéder à l’API Azure pour FHIR, vous pouvez trouver l’ID d’objet du principal de service client avec la commande PowerShell suivante :

```azurepowershell-interactive
$(Get-AzureADServicePrincipal -Filter "AppId eq 'XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX'").ObjectId
```

où `XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX` est l’ID d’application client du service. Sinon, vous pouvez utiliser le `DisplayName` du client de service :

```azurepowershell-interactive
$(Get-AzureADServicePrincipal -Filter "DisplayName eq 'testapp'").ObjectId
```

Si vous utilisez l’interface Azure CLI, vous pouvez utiliser :

```azurecli-interactive
az ad sp show --id XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX --query objectId --out tsv
```

## <a name="find-a-security-group-object-id"></a>Rechercher un ID d’objet de groupe de sécurité

Si vous souhaitez trouver l’ID d’objet d’un groupe de sécurité, vous pouvez utiliser la commande PowerShell suivante :

```azurepowershell-interactive
$(Get-AzureADGroup -Filter "DisplayName eq 'mygroup'").ObjectId
```
Où `mygroup` est le nom du groupe qui vous intéresse.

Si vous utilisez l’interface Azure CLI, vous pouvez utiliser :

```azurecli-interactive
az ad group show --group "mygroup" --query objectId --out tsv
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à rechercher des ID d’objet d’identité nécessaires pour configurer l’API Azure pour FHIR afin d’utiliser un locataire Azure Active Directory externe ou secondaire. Découvrez ensuite comment utiliser les ID d’objet pour configurer des paramètres RBAC locaux :
 
>[!div class="nextstepaction"]
>[Configurer des paramètres RBAC locaux](configure-local-rbac.md)

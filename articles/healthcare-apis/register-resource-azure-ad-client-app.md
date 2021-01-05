---
title: Inscrire une ressource d’application dans Azure AD – API Azure pour FHIR
description: Inscrivez une ressource d’application (ou API) dans Azure Active Directory, afin que les applications clientes puissent demander l’accès à la ressource lors de l’authentification.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 8d70a7b44893ba9c9a0cc2d1d01c65e8e1584e0f
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024480"
---
# <a name="register-a-resource-application-in-azure-active-directory"></a>Inscrire une ressource d'application dans Azure Active Directory

Dans cet article, vous allez découvrir comment inscrire une ressource (ou API) d'application dans Azure Active Directory. Une ressource d’application est une représentation d’Azure Active Directory de l’API du serveur FHIR et les applications clientes peuvent demander l’accès aux ressources lors de l’authentification. La ressource d'application est aussi également appelée *audience* dans OAuth.

## <a name="azure-api-for-fhir"></a>API Azure pour FHIR

Si vous utilisez l’API Azure pour FHIR, une ressource d’application est automatiquement créée lorsque vous déployez le service. Tant que vous utilisez l’API Azure pour FHIR dans le même locataire Azure Active Directory que celui qui déploie votre application, vous pouvez ignorer ce guide pratique et déployer votre API Azure pour FHIR à la place pour la prendre en main.

Si vous utilisez un autre locataire Azure Active Directory (non associé à votre abonnement), vous pouvez importer la ressource d’application API Azure pour FHIR dans votre locataire à l’aide de PowerShell :

```azurepowershell-interactive
New-AzADServicePrincipal -ApplicationId 4f6778d8-5aef-43dc-a1ff-b073724b9495
```

Ou en utilisant Azure CLI :

```azurecli-interactive
az ad sp create --id 4f6778d8-5aef-43dc-a1ff-b073724b9495
```

## <a name="fhir-server-for-azure"></a>Serveur FHIR pour Azure

Si vous utilisez le serveur FHIR open source pour Azure, suivez les étapes dans le [référentiel GitHub](https://github.com/microsoft/fhir-server/blob/master/docs/Register-Resource-Application.md) pour inscrire une ressource d’application. 

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à inscrire une ressource d'application dans Azure Active Directory. Maintenant, inscrivez votre application cliente confidentielle.
 
>[!div class="nextstepaction"]
>[Inscrire une application cliente confidentielle](register-confidential-azure-ad-client-app.md)
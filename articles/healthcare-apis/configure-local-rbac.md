---
title: Configurer un contrôle d’accès en fonction du rôle local (RBAC local) pour l’API Azure pour FHIR
description: Cet article explique comment configurer l’API Azure pour FHIR afin d’utiliser un locataire Azure AD externe pour le plan de données
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 03/15/2020
ms.author: matjazl
ms.openlocfilehash: 096e4e3ecbcedaec674e074a2baccbb336e03c94
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95524196"
---
# <a name="configure-local-rbac-for-fhir"></a>Configurer un RBAC local pour FHIR 

Cet article explique comment configurer l’API Azure pour FHIR afin d’utiliser un locataire Azure Active Directory externe secondaire pour gérer l’accès au plan de données. Utilisez ce mode uniquement s’il n’est pas possible d’utiliser le locataire Azure Active Directory associé à votre abonnement.

> [!NOTE]
> Si votre plan de données de service FHIR est configuré pour utiliser votre locataire Azure Active Directory principal associé à votre abonnement, [utilisez un RBAC Azure pour affecter des rôles de plan de données](configure-azure-rbac.md).

## <a name="add-service-principal"></a>Ajouter un principal du service

Le RBAC local vous permet d’utiliser un locataire Azure Active Directory externe avec votre serveur FHIR. Afin d’autoriser le système RBAC local à vérifier les appartenances de groupe dans ce locataire, l’API Azure pour FHIR doit avoir un principal de service dans le locataire. Ce principal de service sera créé automatiquement dans les locataires liés aux abonnements qui ont déployé l’API Azure pour FHIR. Toutefois, si aucun abonnement n’est lié à votre locataire, un administrateur de locataire doit créer ce principal de service avec l’une des commandes suivantes :

En utilisant le module PowerShell `Az` :

```azurepowershell-interactive
New-AzADServicePrincipal -ApplicationId 3274406e-4e0a-4852-ba4f-d7226630abb7
```

Ou en utilisant le module PowerShell `AzureAd` :

```azurepowershell-interactive
New-AzureADServicePrincipal -AppId 3274406e-4e0a-4852-ba4f-d7226630abb7
```

Ou en utilisant Azure CLI :

```azurecli-interactive
az ad sp create --id 3274406e-4e0a-4852-ba4f-d7226630abb7
```

## <a name="configure-local-rbac"></a>Configurer un RBAC local

Vous pouvez configurer l’API Azure pour FHIR afin qu’elle utilise un locataire Azure Active Directory externe ou secondaire dans le panneau **Authentification** :

![Affectations de RBAC local](media/rbac/local-rbac-guids.png).

Dans la zone Autorité, entrez un locataire Azure Active Directory valide. Une fois le locataire validé, la zone **ID d’objet autorisés** doit être activée et vous pouvez entrer une liste d’ID d’objet d’identité. Ces ID peuvent être les ID d’objet d’identité de :

* Un utilisateur Azure Active Directory.
* Un principal de service Azure Active Directory.
* Un groupe de sécurité Azure Active Directory.

Pour plus d’informations, consultez l’article sur la façon de [Rechercher des ID d’objet d’identité](find-identity-object-ids.md).

Après avoir entré les ID d’objet requis, cliquez sur **Enregistrer** et attendez que les modifications soient enregistrées avant d’essayer d’accéder au plan de données à l’aide des utilisateurs, des principaux de service ou des groupes affectés.

## <a name="caching-behavior"></a>Comportement de mise en cache

L’API Azure pour FHIR met en cache les décisions pendant jusqu’à 5 minutes. Si vous accordez à un utilisateur l’accès au serveur FHIR en l’ajoutant à la liste des ID d’objet autorisés, ou le supprimez de la liste, jusqu’à cinq minutes peuvent être nécessaires pour que les modifications apportées aux autorisations soient propagées.

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a expliqué comment attribuer un accès au plan de données FHIR à l’aide d’un locataire Azure Active Directory externe (secondaire). Découvrez ensuite des paramètres supplémentaires pour l’API Azure pour FHIR :
 
>[!div class="nextstepaction"]
>[Paramètres supplémentaires de l’API Azure pour FHIR](azure-api-for-fhir-additional-settings.md)

---
title: Exemples de code de connecteur d’API pour les flux utilisateur - Azure AD
description: Exemples de code pour les connecteurs d’API dans les flux d’inscription en libre-service pour les identités externes Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: sample
ms.date: 06/16/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: a3995395cfd6256689bedc7a4a3c83effc65c0b4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87906860"
---
# <a name="samples-for-external-identities-self-service-sign-up"></a>Exemples pour l’inscription en libre-service des identités externes

Les tableaux suivants fournissent des liens vers des exemples de code pour tirer parti des API web dans vos flux utilisateur d’inscription en libre-service avec des [connecteurs d’API](api-connectors-overview.md).

## <a name="api-connector-azure-function-quickstarts"></a>Guides de démarrage rapide sur les fonctions Azure et les connecteurs d’API

| Exemple                                                                                                                          | Description                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [.NET Core](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-api-connector-azure-function-validate) | Cet exemple de fonction Azure .NET Core montre comment limiter les connexions à des domaines de locataire spécifiques et valider les informations fournies par l’utilisateur. |
| [Node.JS](https://github.com/Azure-Samples/active-directory-nodejs-external-identities-api-connector-azure-function-validate)   | Cet exemple de fonction Azure Node.js montre comment limiter les connexions à des domaines de locataire spécifiques et valider les informations fournies par l’utilisateur.  |
| [Python](https://github.com/Azure-Samples/active-directory-python-external-identities-api-connector-azure-function-validate)    | Cet exemple de fonction Azure Python montre comment limiter les connexions à des domaines de locataire spécifiques et valider les informations fournies par l’utilisateur.    |

<!-- \| [Java](../../azure-docs-pr/articles/active-directory/b2b/invite-internal-users.md#use-the-invitation-api-to-send-a-b2b-invitation) |  The sample below illustrates how to call the invitation API to invite an internal user as a B2B user. | -->

## <a name="custom-approval-workflows"></a>Workflows d’approbation personnalisés

| Exemple | Description |
|--------| ----------- |
| [Workflow d’approbation manuel](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-api-connectors-approvals) | Cet exemple illustre un workflow d’approbation de bout en bout pour gérer la création de comptes d’utilisateur invités dans le cadre de l’inscription en libre-service. |

## <a name="identity-verification"></a>Vérification d’identité

| Exemple                                                                                                            | Description                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| [IDology](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-idology-identity-verification) | Cet exemple montre comment vérifier une identité d’utilisateur dans le cadre de votre inscription en libre-service à l’aide d’un connecteur d’API pour l’intégration à IDology. |
| [Experian](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-experian-identity-verification) | Cet exemple montre comment vérifier une identité d’utilisateur dans le cadre de votre inscription en libre-service à l’aide d’un connecteur d’API pour l’intégration avec Experian. |

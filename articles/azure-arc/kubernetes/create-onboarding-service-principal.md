---
title: Créer un principal de service d’intégration compatible avec Azure Arc (préversion)
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: 'Créer un principal de service d’intégration compatible avec Azure Arc '
keywords: Kubernetes, Arc, Azure, conteneurs
ms.openlocfilehash: 02689dba32c8cc91e4a4a4de4dee98bc990b4dd6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87050079"
---
# <a name="create-an-azure-arc-enabled-onboarding-service-principal-preview"></a>Créer un principal de service d’intégration compatible avec Azure Arc (préversion)

## <a name="overview"></a>Vue d’ensemble

Il est possible d’utiliser des principaux de service associés à une attribution de rôle avec des privilèges limités pour l’intégration de clusters Kubernetes à Azure Arc. Cela est utile dans les pipelines d’intégration et de déploiement continus (CI/CD) comme Azure Pipelines et GitHub Actions.

Les étapes suivantes fournissent une procédure pas à pas permettant d’utiliser des principaux de service pour l’intégration de clusters Kubernetes à Azure Arc.

## <a name="create-a-new-service-principal"></a>Créer un principal de service

Créez un principal de service avec un nom informatif. N’oubliez pas que ce nom doit être unique au sein de votre locataire Azure Active Directory :

```console
az ad sp create-for-RBAC --skip-assignment --name "https://azure-arc-for-k8s-onboarding"
```

**Output:**

```console
{
  "appId": "22cc2695-54b9-49c1-9a73-2269592103d8",
  "displayName": "azure-arc-for-k8s-onboarding",
  "name": "https://azure-arc-for-k8s-onboarding",
  "password": "09d3a928-b223-4dfe-80e8-fed13baa3b3d",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"
}
```

## <a name="assign-permissions"></a>Affecter des autorisations

Après avoir créé le principal du service, affectez le rôle « Kubernetes Cluster - Azure Arc Onboarding » au principal nouvellement créé. Il s’agit d’un rôle Azure intégré doté d’autorisations limitées, qui permet uniquement au principal d’enregistrer des clusters dans Azure. Le principal ne peut pas mettre à jour, supprimer ou modifier les autres clusters ou ressources au sein de l’abonnement.

Étant donné les capacités limitées, les clients peuvent facilement réutiliser ce principal pour intégrer plusieurs clusters.

Vous pouvez limiter encore les autorisations en passant l’argument `--scope` approprié lors de l’affectation du rôle. Cela permet aux clients de limiter l’inscription des clusters. Les scénarios suivants sont pris en charge par différents paramètres `--scope` :

| Ressource  | Argument `scope`| Résultat |
| ------------- | ------------- | ------------- |
| Abonnement | `--scope /subscriptions/0b1f6471-1bf0-4dda-aec3-111122223333` | Le principal du service peut inscrire n’importe quel cluster dans un groupe de ressources existant au sein de l’abonnement donné |
| Groupe de ressources | `--scope /subscriptions/0b1f6471-1bf0-4dda-aec3-111122223333/resourceGroups/myGroup`  | Le principal du service peut __uniquement__ inscrire les clusters dans le groupe de ressources `myGroup` |

```console
az role assignment create \
    --role 34e09817-6cbe-4d01-b1a2-e0eac5743d41 \      # this is the id for the built-in role
    --assignee 22cc2695-54b9-49c1-9a73-2269592103d8 \  # use the appId from the new SP
    --scope /subscriptions/<<SUBSCRIPTION_ID>>         # apply the appropriate scope
```

**Output:**

```console
{
  "canDelegate": null,
  "id": "/subscriptions/<<SUBSCRIPTION_ID>>/providers/Microsoft.Authorization/roleAssignments/fbd819a9-01e8-486b-9eb9-f177ba400ba6",
  "name": "fbd819a9-01e8-486b-9eb9-f177ba400ba6",
  "principalId": "ddb0ddb4-ba84-4cde-b936-affc272a4b90",
  "principalType": "ServicePrincipal",
  "roleDefinitionId": "/subscriptions/<<SUBSCRIPTION_ID>>/providers/Microsoft.Authorization/roleDefinitions/34e09817-6cbe-4d01-b1a2-e0eac5743d41",
  "scope": "/subscriptions/<<SUBSCRIPTION_ID>>",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="use-service-principal-with-the-azure-cli"></a>Utiliser le principal de service avec Azure CLI

Faites référence au principal de service nouvellement créé :

```console
az login --service-principal -u mySpnClientId -p mySpnClientSecret --tenant myTenantID
az connectedk8s connect -n myConnectedClusterName -g myResoureGroupName
```

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser Azure Policy pour gérer la configuration du cluster](./use-azure-policy.md)

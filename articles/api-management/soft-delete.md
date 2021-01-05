---
title: Suppression réversible de Gestion des API Azure (version préliminaire) | Microsoft Docs
description: La suppression réversible vous permet de récupérer des instances de Gestion des API supprimées.
ms.service: api-management
ms.topic: conceptual
author: vladvino
ms.author: apimpm
ms.date: 11/27/2020
ms.openlocfilehash: 72e91715398b4920c62afae5f36aa09954a577f9
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97092140"
---
# <a name="api-management-soft-delete-preview"></a>Suppression réversible de Gestion des API (version préliminaire)

Avec la fonctionnalité de suppression réversible (préversion) de Gestion des API, vous pouvez récupérer et restaurer des instances de Gestion des API (APIM) supprimées récemment.

> [!IMPORTANT]
> Seules les instances de Gestion des API supprimées à l’aide de `2020-01-01-preview` et versions ultérieures d’API seront supprimées de façon réversible et récupérables à l’aide des étapes décrites dans cet article. Les instances APIM supprimées à l’aide des versions précédentes de l’API continueront d’être supprimées définitivement. Azure PowerShell et Azure CLI n’utilisent pas actuellement la version `2020-06-01-preview` et entraînent également un comportement de suppression irréversible.

## <a name="supporting-interfaces"></a>Prise en charge des interfaces

La fonctionnalité de suppression réversible est disponible via [API REST](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/restore).

> [!TIP]
> Pour obtenir des conseils et des outils sur l’appel des API REST Azure, consultez [Référence API REST Azure](/rest/api/azure/).

| Opération | Description | Espace de noms de Gestion des API | Version d’API minimale |
|--|--|--|--|
| [Créer ou mettre à jour](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/createorupdate) | Crée ou met à jour un service Gestion des API.  | Service Gestion des API | Quelconque |
| [Créer ou mettre à jour](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/createorupdate) avec la propriété `restore` définie sur **true** | Supprime le service Gestion des API s’il a été précédemment supprimé de manière réversible. Si `restore` est spécifié et défini sur `true`, toutes les autres propriétés seront ignorées.  | Service Gestion des API |  2020-06-01-preview |
| [Supprimer](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/delete) | Supprime un service Gestion des API existant. | Service Gestion des API | 2020-01-01-preview|
| [Récupérer par nom](/rest/api/apimanagement/2020-06-01-preview/deletedservices/getbyname) | Procurez-vous le service Gestion des API supprimé de manière réversible par nom. | Services supprimés | 2020-06-01-preview |
| [Lister par abonnement](/rest/api/apimanagement/2020-06-01-preview/deletedservices/listbysubscription) | Répertorie tous les services supprimés de manière réversible et disponibles pour l’annulation de la suppression pour l’abonnement donné. | Services supprimés | 2020-06-01-preview
| [Purge](/rest/api/apimanagement/2020-06-01-preview/deletedservices/purge) | Vide le service Gestion des API (le supprime sans option d’annulation de suppression). | Services supprimés | 2020-06-01-preview

## <a name="soft-delete-behavior"></a>Comportement de la suppression réversible

Vous pouvez utiliser n’importe quelle version d’API pour créer votre instance Gestion des API. Toutefois, vous devez utiliser `2020-01-01-preview` ou versions ultérieures pour supprimer de manière réversible votre instance APIM (et avoir la possibilité de la récupérer).

Lors de la suppression d’une instance Gestion des API, le service existe dans un état supprimé, ce qui le rend inaccessible à toutes les opérations APIM. Dans cet état, l’instance APIM peut uniquement être listée, récupérée ou vidée (supprimée définitivement).

Au même moment, Azure planifie la suppression des données sous-jacentes correspondant à l’instance APIM pour une exécution après l’intervalle de rétention prédéterminé (de 48 heures). L’enregistrement DNS correspondant à l’instance est également conservé pendant la durée de l’intervalle de rétention. Vous ne pouvez pas réutiliser le nom d’une instance Gestion des API ayant fait l’objet d’une suppression réversible tant que la période de rétention n’est pas écoulée.

Si votre instance APIM n’est pas récupérée dans les 48 heures, elle sera supprimée définitivement (irrécupérable). Vous pouvez également choisir de [vider](#purge-a-soft-deleted-apim-instance) (supprimer définitivement) votre instance APIM, en annulant la période de rétention de suppression réversible.

## <a name="list-deleted-apim-instances"></a>Répertorier les instances APIM supprimées

Vous pouvez vérifier qu’une instance APIM supprimée de manière réversible est disponible pour la restauration (annulation de suppression) à l’aide des opérations des services supprimés [Récupérer par nom](/rest/api/apimanagement/2020-06-01-preview/deletedservices/getbyname) ou [Lister par abonnement](/rest/api/apimanagement/2020-06-01-preview/deletedservices/listbysubscription).

### <a name="get-a-soft-deleted-instance-by-name"></a>Récupérer une instance supprimée de manière réversible par nom

Utilisez l’opération [Récupérer par nom](/rest/api/apimanagement/2020-06-01-preview/deletedservices/getbyname) de Gestion des API, en remplaçant `{subscriptionId}`, `{location}` et `{serviceName}` par votre abonnement Azure, l’emplacement de la ressource et le nom de l’instance Gestion des API :

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.ApiManagement/locations/{location}/deletedservices/{serviceName}?api-version=2020-06-01-preview
```

S’il est disponible pour l’annulation de la suppression, Azure renvoie un enregistrement de l’instance APIM qui affiche son `deletionDate` et `scheduledPurgeDate`, par exemple :

```json
{
    "id": "subscriptions/########-####-####-####-############/providers/Microsoft.ApiManagement/locations/southcentralus/deletedservices/apimtest",
    "name": "apimtest",
    "type": "Microsoft.ApiManagement/deletedservices",
    "location": "South Central US",
    "properties": {
        "serviceId": "/subscriptions/########-####-####-####-############/resourceGroups/apimtestgroup/providers/Microsoft.ApiManagement/service/apimtest",
        "scheduledPurgeDate": "2020-11-26T19:40:26.3596893Z",
        "deletionDate": "2020-11-24T19:40:50.1013572Z"
    }
}
```

### <a name="list-all-soft-deleted-instances-for-a-given-subscription"></a>Répertorier toutes les instances supprimées de manière réversible pour un abonnement donné

Utilisez l’opération [Lister par abonnement](/rest/api/apimanagement/2020-06-01-preview/deletedservices/listbysubscription) de Gestion des API, en remplaçant `{subscriptionId}` par votre ID d’abonnement :

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.ApiManagement/deletedservices?api-version=2020-06-01-preview
```

Cette opération renvoie la liste de tous les services supprimés de manière réversible disponibles pour l’annulation de la suppression dans le cadre de l’abonnement donné, en affichant le `deletionDate` et le `scheduledPurgeDate` pour chacun.

## <a name="recover-a-deleted-apim-instance"></a>Récupérer une instance APIM supprimée

Utilisez l’opération [Créer ou mettre à jour](/rest/api/apimanagement/2020-06-01-preview/apimanagementservice/createorupdate) de Gestion des API, en remplaçant `{subscriptionId}`, `{resourceGroup}` et `{apimServiceName}` par votre abonnement Azure, le nom du groupe de ressources et le nom de Gestion des API :

```rest
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ApiManagement/service/{apimServiceName}?api-version=2020-06-01-preview
```

. . . et affectez à la propriété `restore` la valeur `true` dans le corps de la demande. (Lorsque cet indicateur est spécifié et défini sur *true*, toutes les autres propriétés sont ignorées.) Exemple :

```json
{
  "properties": {
    "publisherEmail": "help@contoso.com",
    "publisherName": "Contoso",
    "restore": true
  },
  "sku": {
    "name": "Developer",
    "capacity": 1
  },
  "location": "South Central US"
}
```

## <a name="purge-a-soft-deleted-apim-instance"></a>Vider une instance APIM supprimée de manière réversible

Utilisez l’opération [Vider](/rest/api/apimanagement/2020-06-01-preview/deletedservices/purge) de Gestion des API, en remplaçant `{subscriptionId}`, `{location}` et `{serviceName}` par votre abonnement Azure, l’emplacement de la ressource et le nom de Gestion des API :

```rest
DELETE https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.ApiManagement/locations/{location}/deletedservices/{serviceName}?api-version=2020-06-01-preview
```

Cela va entraîner la suppression définitive de votre instance Gestion des API d’Azure.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les options de sauvegarde et de récupération de Gestion des API à long terme :

- [Comment implémenter une récupération d’urgence à l’aide d’une sauvegarde de service et la récupérer dans Gestion des API Azure](api-management-howto-disaster-recovery-backup-restore.md)
---
title: FAQ et problèmes connus en lien avec des identités managées – Azure AD
description: Problèmes connus en lien avec des identités managées pour les ressources Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.assetid: 2097381a-a7ec-4e3b-b4ff-5d2fb17403b6
ms.service: active-directory
ms.subservice: msi
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 12/01/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 2be66904898ecdf2006952f5e80c17dc78b81c06
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825808"
---
# <a name="faqs-and-known-issues-with-managed-identities-for-azure-resources"></a>FAQ et problèmes connus en lien avec des identités managées pour les ressources Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

## <a name="frequently-asked-questions-faqs"></a>Forum Aux Questions (FAQ)

> [!NOTE]
> Identités managées pour les ressources Azure est le nouveau nom du service anciennement nommé Managed Service Identity (MSI).

### <a name="how-can-you-find-resources-that-have-a-managed-identity"></a>Comment trouver les ressources qui ont une identité managée ?

Vous pouvez obtenir la liste des ressources qui ont une identité managée affectée par le système en exécutant la commande Azure CLI suivante : 

```azurecli-interactive
az resource list --query "[?identity.type=='SystemAssigned'].{Name:name,  principalId:identity.principalId}" --output table
```

### <a name="do-managed-identities-have-a-backing-app-object"></a>Les identités managées ont-elles un objet de sauvegarde d’application ?

Non. Les identités managées et les inscriptions d’applications Azure AD sont deux choses différentes dans l’annuaire. 

Les inscriptions d’applications ont deux composants : un objet application et un objet principal de service. Les identités managées des ressources Azure ont un seul de ces composants : un objet principal de service. 

Les identités managées n’ont pas d’objet application dans l’annuaire, qui est l’objet couramment utilisé pour accorder des autorisations d’application pour MS Graph. Au lieu de cela, les autorisations MS Graph pour les identités managées doivent être accordées directement au principal de service.  

### <a name="does-managed-identities-for-azure-resources-work-with-azure-cloud-services"></a>Les identités managées pour les ressources Azure fonctionnent-elles avec Azure Cloud Services ?

Non, il n’existe aucun plan de prise en charge des identités managées pour les ressources Azure dans Azure Cloud Services.

### <a name="what-is-the-credential-associated-with-a-managed-identity-how-long-is-it-valid-and-how-often-is-it-rotated"></a>Quelles sont les informations d’identification associées à une identité managée ? Quelle est la durée de validité et quelle est la fréquence de rotation ?

> [!NOTE]
> Le mode d’authentification des identités managées est un détail d’implémentation interne susceptible d’être modifié sans préavis.

Les identités managées utilisent l’authentification par certificat. Les informations d’identification de chaque identité managée ont une date d’expiration de 90 jours et elles sont renouvelées après 45 jours.

### <a name="what-is-the-security-boundary-of-managed-identities-for-azure-resources"></a>Quelle est la limite de sécurité des identités managées pour les ressources Azure ?

La limite de sécurité de l’identité est la ressource à laquelle elle est jointe. Par exemple, la limite de sécurité activée pour une machine virtuelle avec des identités managées pour des ressources Azure est la machine virtuelle. Tout code s’exécutant sur cette machine virtuelle est en mesure d’appeler les identités managées pour des jetons de point de terminaison et de requête de ressources Azure. L’expérience est similaire à celle d’autres ressources qui prennent en charge les identités managées pour des ressources Azure.

### <a name="what-identity-will-imds-default-to-if-dont-specify-the-identity-in-the-request"></a>Quelle est l’identité utilisée par défaut par IMDS si vous ne spécifiez pas l’identité dans la requête ?

- Si l’identité managée affectée par le système est activée et qu’aucune identité n’est spécifiée dans la requête, IMDS utilise par défaut l’identité managée affectée par le système.
- Si l’identité managée affectée par le système n’est pas activée et qu’il n’existe qu’une seule identité managée affectée par l’utilisateur, IMDS utilise par défaut l’identité managée affectée par ce seul utilisateur. 
- Si l’identité managée affectée par le système n’est pas activée et qu’il existe plusieurs identités managées affectées par l’utilisateur, la spécification d’une identité managée dans la requête est obligatoire.

### <a name="will-managed-identities-be-recreated-automatically-if-i-move-a-subscription-to-another-directory"></a>Les identités managées sont-elles recréées automatiquement si je déplace un abonnement vers un autre répertoire ?

Non. Si vous déplacez un abonnement vers un autre répertoire, vous devez les recréer manuellement et accorder à nouveau les attributions de rôle Azure.
- Pour les identités managées affectées par le système : désactivez et réactivez-les. 
- Pour les identités managées affectées par l’utilisateur : supprimez, recréez et joignez-les à nouveau aux ressources nécessaires (par exemple, des machines virtuelles)

### <a name="can-i-use-a-managed-identity-to-access-a-resource-in-a-different-directorytenant"></a>Puis-je utiliser une identité managée pour accéder à une ressource dans un autre répertoire/abonné ?

Non. Les identités managées ne prennent actuellement pas en charge les scénarios entre répertoires. 

### <a name="what-azure-rbac-permissions-are-required-to-managed-identity-on-a-resource"></a>Quelles sont les autorisations runbook automation Azure nécessaires pour une identité managée sur une ressource ? 

- Identité managée affectée par le système : Vous avez besoin d’autorisations en écriture sur la ressource. Ainsi, pour les machines virtuelles vous avez besoin de Microsoft.Compute/virtualMachines/write. Cette action est incluse dans les rôles intégrés spécifiques de la ressource, tel que le [Contributeur de machines virtuelles](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).
- Identité managée affectée par l’utilisateur : Vous avez besoin d’autorisations en écriture sur la ressource. Ainsi, pour les machines virtuelles vous avez besoin de Microsoft.Compute/virtualMachines/write. En plus de l’attribution de rôle [Opérateur d’identités managées](../../role-based-access-control/built-in-roles.md#managed-identity-operator) sur l’identité managée.

### <a name="how-do-i-prevent-the-creation-of-user-assigned-managed-identities"></a>Comment empêcher la création d’identités managées affectées par l’utilisateur ?

Vous pouvez empêcher vos utilisateurs de créer des identités managées affectées par l’utilisateur avec [Azure Policy](../../governance/policy/overview.md)

- Accédez au [portail Azure](https://portal.azure.com), puis à **Stratégie**.
- Choisissez **Définitions**
- Sélectionnez **+ Définition de stratégie** et entrez les informations nécessaires.
- Dans la section Règle de stratégie, collez

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "field": "type",
      "equals": "Microsoft.ManagedIdentity/userAssignedIdentities"
    },
    "then": {
      "effect": "deny"
    }
  },
  "parameters": {}
}

```

Une fois la stratégie créée, affectez-la au groupe de ressources que vous souhaitez utiliser.

- Accédez aux groupes de ressources.
- Recherchez le groupe de ressources que vous utilisez pour le test.
- Choisissez **Stratégies** dans le menu de gauche.
- Sélectionnez **Attribuer la stratégie**
- Dans la section **De base**, indiquez :
    - **Étendue** : groupe de ressources utilisé pour le test
    - **Définition de stratégie** : stratégie que nous avons créée précédemment.
- Conservez les valeurs par défaut pour tous les autres paramètres, puis choisissez **Vérifier + créer**

À ce stade, toute tentative de création d’une identité managée affectée par l’utilisateur dans le groupe de ressources échoue.

  ![Violation de stratégie](./media/known-issues/policy-violation.png)

## <a name="known-issues"></a>Problèmes connus

### <a name="automation-script-fails-when-attempting-schema-export-for-managed-identities-for-azure-resources-extension"></a>Le « Script d’automatisation » échoue lors de la tentative d’exportation de schéma pour l’extension des entités managées pour ressources Azure

Lorsque des identités managées pour les ressources Azure sont activées sur une machine virtuelle, l’erreur suivante s’affiche en cas de tentative d’utilisation de la fonctionnalité « Script d’automatisation » pour la machine virtuelle ou son groupe de ressources :

![Erreur d’exportation de script d’automatisation d’identités managées pour ressources Azure](./media/msi-known-issues/automation-script-export-error.png)

L’extension de machines virtuelles des identités managées pour les ressources Azure (dont la dépréciation est prévue en janvier 2019) ne prend pas en charge actuellement la possibilité d’exporter son schéma vers un modèle de groupe de ressources. Par conséquent, le modèle généré n’affiche pas les paramètres de configuration permettant d’activer des identités managées pour ressources Azure sur la ressource. Ces sections peuvent être ajoutées manuellement en suivant les exemples fournis dans [Configurer des identités managées pour les ressources Azure sur une machine virtuelle Azure en utilisant un modèle](qs-configure-template-windows-vm.md).

Lorsque la fonctionnalité d’exportation de schéma sera disponible pour l’extension de machine virtuelle des identités managées pour les ressources Azure (dont la dépréciation est prévue en janvier 2019), elle sera répertoriée dans [Exportation de groupes de ressources contenant des extensions de machines virtuelles](../../virtual-machines/extensions/export-templates.md#supported-virtual-machine-extensions).

### <a name="vm-fails-to-start-after-being-moved-from-resource-group-or-subscription"></a>La machine virtuelle ne démarre pas après avoir été déplacée d’un groupe de ressources ou d’un abonnement

Si vous déplacez une machine virtuelle en cours d’exécution, elle continue de s’exécuter pendant le déplacement. Toutefois, si la machine virtuelle est arrêtée et redémarrée après le déplacement, elle ne démarre plus. Ce problème se produit parce que la machine virtuelle ne met pas à jour la référence à l’identité des identités managées pour les ressources Azure, mais continue de pointer dessus dans l’ancien groupe de ressources.

**Solution de contournement** 
 
Déclenchez une mise à jour sur la machine virtuelle afin qu’elle obtienne les valeurs correctes pour les identités managées pour les ressources Azure. Vous pouvez modifier les propriétés d’une machine virtuelle pour mettre à jour la référence à l’identité des identités managées pour les ressources Azure. Par exemple, vous pouvez définir une nouvelle valeur de balise sur la machine virtuelle avec la commande suivante :

```azurecli-interactive
az vm update -n <VM Name> -g <Resource Group> --set tags.fixVM=1
```
 
Cette commande définit une nouvelle balise « fixVM » avec une valeur de 1 sur la machine virtuelle. 
 
En définissant cette propriété, la machine virtuelle se met à jour avec l’URI de ressource appropriée des identités managées pour les ressources Azure. Vous devriez ensuite être en mesure de démarrer la machine virtuelle. 
 
Une fois la machine virtuelle démarrée, la balise peut être supprimée en utilisant la commande suivante :

```azurecli-interactive
az vm update -n <VM Name> -g <Resource Group> --remove tags.fixVM
```

### <a name="transferring-a-subscription-between-azure-ad-directories"></a>Transfert d’un abonnement entre des répertoires Azure AD

Les identités managées ne sont pas mises à jour lorsqu’un abonnement est déplacé/transféré vers un autre répertoire. Par conséquent, toutes les identités managées existantes affectées par le système ou par l’utilisateur seront rompues. 

Solution de contournement pour les identités managées dans un abonnement déplacé vers un autre répertoire :

 - Pour les identités managées affectées par le système : désactivez et réactivez-les. 
 - Pour les identités managées affectées par l’utilisateur : supprimez, recréez et joignez-les à nouveau aux ressources nécessaires (par exemple, des machines virtuelles)

Pour plus d’informations, consultez [Transférer un abonnement Azure vers une autre instance Azure AD Directory](../../role-based-access-control/transfer-subscription.md).

### <a name="moving-a-user-assigned-managed-identity-to-a-different-resource-groupsubscription"></a>Le déplacement d’une identité managée affectée par l’utilisateur à un autre groupe de ressources/abonnement

Le déplacement d’une identité managée affectée par l’utilisateur à un autre groupe de ressources.

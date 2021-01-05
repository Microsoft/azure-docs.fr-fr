---
title: Intégrer un client à Azure Lighthouse
description: Apprenez à intégrer un client à Azure Lighthouse pour permettre l'accès à ses ressources et la gestion de celles-ci via votre propre locataire à l'aide de la gestion des ressources déléguées Azure.
ms.date: 12/15/2020
ms.topic: how-to
ms.openlocfilehash: 023b44a77cb38a14df8aa6a885ff137c02942061
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516123"
---
# <a name="onboard-a-customer-to-azure-lighthouse"></a>Intégrer un client à Azure Lighthouse

Cet article explique comment, en tant que fournisseur de services, vous pouvez intégrer un client à Azure Lighthouse. Les ressources déléguées (abonnements et/ou groupes de ressources) dans le locataire Azure Active Directory (Azure AD) du client peuvent être gérées via votre propre locataire en utilisant la [gestion des ressources déléguées Azure](../concepts/azure-delegated-resource-management.md).

> [!TIP]
> Même si nous faisons ici référence aux fournisseurs de services et aux clients, les [entreprises gérant plusieurs locataires](../concepts/enterprise.md) peuvent suivre le même processus pour configurer Azure Lighthouse et consolider leur expérience de gestion.

Vous pouvez répéter le processus d’intégration pour plusieurs clients. Lorsqu'un utilisateur disposant des autorisations appropriées se connecte à votre locataire gérant, il peut être autorisé à accéder aux étendues de location du client pour effectuer des opérations de gestion sans avoir à se connecter individuellement à chaque locataire du client.

Pour suivre votre impact sur les engagements client et recevoir une reconnaissance, associez votre ID Microsoft Partner Network (MPN) à au moins un compte d’utilisateur qui a accès à tous vos abonnements intégrés. Vous devez effectuer cette association dans le locataire de votre fournisseur de services. Nous vous recommandons de créer un compte de principal de service dans votre locataire associé à votre ID MPN, puis d’inclure ce principal de service chaque fois que vous intégrez un client. Pour plus d’informations, consultez [Lier votre ID de partenaire pour activer le crédit Partenaires sur des ressources déléguées](partner-earned-credit.md).

> [!NOTE]
> Les clients peuvent également être intégrés à Azure Lighthouse lorsqu’ils acquièrent une offre de service managé (publique ou privée) que vous [publiez dans la Place de marché Azure](publish-managed-services-offers.md). Vous pouvez également utiliser le processus d’intégration décrit ici en même temps qu’une offre publiée dans la Place de marché Azure.

Le processus d’intégration requiert d’exécuter des actions tant dans le locataire du fournisseur de services que dans le locataire du client. Toutes ces étapes sont décrites dans cet article.

## <a name="gather-tenant-and-subscription-details"></a>Collecter les détails du locataire et de l’abonnement

Pour qu’il soit possible d’intégrer le locataire d’un client, celui-ci doit disposer d’un abonnement Azure actif. Vous devez disposer des informations suivantes :

- L’ID de locataire du locataire du fournisseur de services (où vous allez gérer les ressources du client)
- L’ID de locataire du locataire du client (dont les ressources seront gérées par le fournisseur de services)
- Les ID d’abonnement de chaque abonnement dans le locataire du client qui sera géré par le fournisseur de services (ou qui contient les groupes de ressources qui seront gérés par le fournisseur de services).

Si vous ne disposez pas de ces valeurs d’ID, vous pouvez les récupérer de l’une des manières suivantes. Veillez à utiliser ces valeurs exactes dans votre déploiement.

### <a name="azure-portal"></a>Portail Azure

Vous pouvez afficher votre ID de locataire en pointant sur le nom de votre compte dans l’angle supérieur droit du portail Azure, ou en sélectionnant **Changer de répertoire**. Pour sélectionner et copier votre ID de locataire, recherchez « Azure Active Directory » dans le portail, puis sélectionnez **Propriétés** et copiez la valeur affichée dans le champ **ID de répertoire**. Pour trouver l’ID d’un abonnement dans un locataire du client, recherchez « Abonnements », puis sélectionnez l’ID d’abonnement approprié.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

Select-AzSubscription <subscriptionId>
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az account set --subscription <subscriptionId/name>
az account show
```

> [!NOTE]
> Lors de l’intégration d’un abonnement (ou d’un ou de plusieurs groupes de ressources au sein d’un abonnement) à l’aide du processus décrit ici, le fournisseur de ressources **Microsoft.ManagedServices** sera inscrit pour cet abonnement.

## <a name="define-roles-and-permissions"></a>Définir des rôles et des autorisations

En tant que fournisseur de services, vous souhaitez peut-être effectuer plusieurs tâches pour un seul client, ce qui requiert un accès différent pour les différentes étendues. Vous pouvez définir autant d'autorisations que nécessaire pour attribuer les [rôles intégrés Azure](../../role-based-access-control/built-in-roles.md) appropriés aux utilisateurs de votre locataire.

Pour faciliter la gestion, nous vous recommandons d’utiliser des groupes d’utilisateurs Azure AD pour chaque rôle. Cela vous donne la possibilité d’ajouter ou de supprimer des utilisateurs individuels dans le groupe qui a accès, afin de ne pas avoir à répéter le processus d’intégration pour apporter des modifications à l’utilisateur. Vous pouvez assigner des rôles à un principal de service, ce qui peut être utile pour les scénarios d’automatisation.

> [!IMPORTANT]
> Pour que vous puissiez ajouter des autorisations pour un groupe Azure AD, le **Type de groupe** doit être défini sur **Sécurité**. Cette option est sélectionnée lors de la création du groupe. Pour plus d’informations, consultez [Créer un groupe de base et ajouter des membres avec Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Lorsque vous définissez vos autorisations, veillez à suivre le principe du privilège minimum afin que les utilisateurs disposent uniquement des autorisations nécessaires pour accomplir leur travail. Pour des consignes et des informations sur les rôles pris en charge, consultez [Locataires, utilisateurs et rôles dans les scénarios Azure Lighthouse](../concepts/tenants-users-roles.md).

Pour définir des autorisations, vous devez connaître les valeurs d’ID pour chaque utilisateur, groupe d’utilisateurs ou principal de service dans le locataire du fournisseur de services auquel vous souhaitez accorder l’accès. Vous aurez également besoin de l’ID de définition de rôle pour chaque rôle intégré que vous souhaitez attribuer. Si vous ne les avez pas déjà, vous pouvez les récupérer en exécutant les commandes ci-dessous à partir du locataire du fournisseur de services.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

# To retrieve the objectId for an Azure AD group
(Get-AzADGroup -DisplayName '<yourGroupName>').id

# To retrieve the objectId for an Azure AD user
(Get-AzADUser -UserPrincipalName '<yourUPN>').id

# To retrieve the objectId for an SPN
(Get-AzADApplication -DisplayName '<appDisplayName>' | Get-AzADServicePrincipal).Id

# To retrieve role definition IDs
(Get-AzRoleDefinition -Name '<roleName>').id
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

# To retrieve the objectId for an Azure AD group
az ad group list --query "[?displayName == '<yourGroupName>'].objectId" --output tsv

# To retrieve the objectId for an Azure AD user
az ad user show --id "<yourUPN>" --query "objectId" --output tsv

# To retrieve the objectId for an SPN
az ad sp list --query "[?displayName == '<spDisplayName>'].objectId" --output tsv

# To retrieve role definition IDs
az role definition list --name "<roleName>" | grep name
```

> [!TIP]
> Nous vous recommandons d’attribuer le [rôle de suppression de l’affectation d’inscription des services managés](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) lors de l’intégration d’un client, afin que les utilisateurs de votre locataire puissent [supprimer l’accès à la délégation](remove-delegation.md) ultérieurement si nécessaire. Si ce rôle n’est pas attribué, les ressources déléguées ne peuvent être supprimées que par un utilisateur dans le locataire du client.

## <a name="create-an-azure-resource-manager-template"></a>Créer un modèle Azure Resource Manager

Pour intégrer votre client, vous devez créer un modèle [Azure Resource Manager](../../azure-resource-manager/index.yml) pour votre offre et y inclure les éléments suivants : Le client pourra voir les valeurs **mspOfferName** et **mspOfferDescription** dans la [page Fournisseurs de services](view-manage-service-providers.md) du Portail Azure.

|Champ  |Définition  |
|---------|---------|
|**mspOfferName**     |Nom décrivant cette définition. Cette valeur est présentée au client comme titre de l'offre. Il doit s'agir d'une valeur unique.        |
|**mspOfferDescription**     |Brève description de votre offre (par exemple, « Offre de gestion des machines virtuelles Contoso »).      |
|**managedByTenantId**     |Votre ID de client.          |
|**authorizations**     |Valeurs **principalId** pour les utilisateurs/groupes/noms de principal du service de votre locataire, chacune avec un **principalIdDisplayName** pour aider votre client à comprendre l’objectif de l’autorisation, et mappées à une valeur **roleDefinitionId** intégrée pour spécifier le niveau d’accès.      |

Le processus d’intégration nécessite un modèle Azure Resource Manager (fourni dans nos [exemples de référentiel](https://github.com/Azure/Azure-Lighthouse-samples/)) et un fichier de paramètres correspondant que vous modifiez pour correspondre à votre configuration et pour définir vos autorisations.

> [!IMPORTANT]
> Le processus décrit ici nécessite un déploiement distinct pour chaque abonnement en cours d'intégration, même si vous intégrez des abonnements dans le même locataire du client. Des déploiements distincts sont également requis si vous intégrez plusieurs groupes de ressources dans différents abonnements dans le même locataire du client. Toutefois, l’intégration de plusieurs groupes de ressources au sein d’un même abonnement peut être effectuée dans un seul déploiement.
>
> De plus, vous devez déployer séparément chacune des offres qui sont appliquées à un même abonnement (ou aux groupes de ressources d’un abonnement). Chaque offre appliquée doit utiliser un **mspOfferName** différent.

Les modèles que vous choisissez dépendront de votre choix d’intégrer un abonnement entier, un groupe de ressources ou plusieurs groupes de ressources au sein d’un abonnement. Nous fournissons également un modèle qui peut être utilisé pour des clients qui ont acheté une offre de service managé que vous avez publiée sur la Place de marché Azure, si vous préférez intégrer leurs abonnements de cette façon.

|Pour intégrer ceci  |Utiliser ce modèle Azure Resource Manager  |Et modifier ce fichier de paramètres |
|---------|---------|---------|
|Abonnement   |[delegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management/delegatedResourceManagement.json)  |[delegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management/delegatedResourceManagement.parameters.json)    |
|Resource group   |[rgDelegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/rgDelegatedResourceManagement.json)  |[rgDelegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/rgDelegatedResourceManagement.parameters.json)    |
|Plusieurs groupes de ressources au sein d’un abonnement   |[multipleRgDelegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/multipleRgDelegatedResourceManagement.json)  |[multipleRgDelegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/multipleRgDelegatedResourceManagement.parameters.json)    |
|Abonnement (lors de l’utilisation d’une offre publiée sur la Place de marché Azure)   |[marketplaceDelegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/marketplace-delegated-resource-management/marketplaceDelegatedResourceManagement.json)  |[marketplaceDelegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/marketplace-delegated-resource-management/marketplaceDelegatedResourceManagement.parameters.json)    |

> [!TIP]
> Bien que vous ne puissiez pas intégrer un groupe d’administration entier dans un déploiement, vous pouvez [déployer une stratégie au niveau du groupe d’administration](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-delegate-management-groups). La stratégie vérifie si chaque abonnement au sein du groupe d’administration a été délégué au locataire de gestion spécifié et, si ce n’est pas le cas, crée l’attribution en fonction des valeurs que vous fournissez.

L’exemple suivant montre un fichier **delegatedResourceManagement.parameters.json** modifié qui peut être utilisé pour l’intégration d’un abonnement. Les fichiers de paramètres de groupe de ressources (situés dans le dossier [rg-delegated-resource-management](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/rg-delegated-resource-management)) sont similaires, mais incluent en outre un paramètre **rgName** pour identifier les groupes de ressources à intégrer.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mspOfferName": {
            "value": "Fabrikam Managed Services - Interstellar"
        },
        "mspOfferDescription": {
            "value": "Fabrikam Managed Services - Interstellar"
        },
        "managedByTenantId": {
            "value": "df4602a3-920c-435f-98c4-49ff031b9ef6"
        },
        "authorizations": {
            "value": [
                {
                    "principalId": "0019bcfb-6d35-48c1-a491-a701cf73b419",
                    "principalIdDisplayName": "Tier 1 Support",
                    "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"
                },
                {
                    "principalId": "0019bcfb-6d35-48c1-a491-a701cf73b419",
                    "principalIdDisplayName": "Tier 1 Support",
                    "roleDefinitionId": "36243c78-bf99-498c-9df9-86d9f8d28608"
                },
                {
                    "principalId": "0afd8497-7bff-4873-a7ff-b19a6b7b332c",
                    "principalIdDisplayName": "Tier 2 Support",
                    "roleDefinitionId": "acdd72a7-3385-48ef-bd42-f606fba81ae7"
                },
                {
                    "principalId": "9fe47fff-5655-4779-b726-2cf02b07c7c7",
                    "principalIdDisplayName": "Service Automation Account",
                    "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"
                },
                {
                    "principalId": "3kl47fff-5655-4779-b726-2cf02b05c7c4",
                    "principalIdDisplayName": "Policy Automation Account",
                    "roleDefinitionId": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
                    "delegatedRoleDefinitionIds": [
                        "b24988ac-6180-42a0-ab88-20f7382dd24c",
                        "92aaf0da-9dab-42b6-94a3-d43ce8d16293"
                    ]
                }
            ]
        }
    }
}
```

La dernière autorisation dans l’exemple ci-dessus ajoute un **principalId** avec le rôle Administrateur de l’accès utilisateur (18d7d88d-d35e-4fb5-A5C3-7773c20a72d9). Lorsque vous attribuez ce rôle, vous devez inclure la propriété **delegatedRoleDefinitionIds** et un ou plusieurs rôles intégrés. L’utilisateur créé dans cette autorisation pourra attribuer ces rôles intégrés aux [identités managées](../../active-directory/managed-identities-azure-resources/overview.md) dans le même locataire du client, ce qui est nécessaire au [déploiement des stratégies pouvant être corrigées](deploy-policy-remediation.md).  L’utilisateur peut également créer des incidents de support.  Aucune autre autorisation normalement associée au rôle Administrateur de l’accès utilisateur ne s’appliquera à cet utilisateur.

## <a name="deploy-the-azure-resource-manager-templates"></a>Déployer les modèles Azure Resource Manager

Une fois que vous avez mis à jour votre fichier de paramètres, un utilisateur du locataire du client doit déployer le modèle Azure Resource Manager au sein de son locataire. Un déploiement distinct est nécessaire pour chaque abonnement que vous souhaitez intégrer (ou pour chaque abonnement contenant des groupes de ressources que vous souhaitez intégrer).

> [!IMPORTANT]
> Ce déploiement doit être effectué par un compte non invité du locataire du client qui dispose du [rôle intégré Propriétaire](../../role-based-access-control/built-in-roles.md#owner) pour l'abonnement en cours d'intégration (ou qui contient les groupes de ressources en cours d'intégration). Pour voir tous les utilisateurs qui peuvent déléguer l’abonnement, un utilisateur du locataire du client peut sélectionner l’abonnement dans le portail Azure, ouvrir **Contrôle d’accès (IAM)** et [afficher tous les utilisateurs ayant le rôle Propriétaire](../../role-based-access-control/role-assignments-list-portal.md#list-owners-of-a-subscription). 
>
> Si l’abonnement a été créé par le biais du [programme Fournisseur de solutions Cloud (CSP)](../concepts/cloud-solution-provider.md), tout utilisateur disposant du rôle [Agent d’administration](/partner-center/permissions-overview#manage-commercial-transactions-in-partner-center-azure-ad-and-csp-roles) dans le locataire de votre fournisseur de services peut effectuer le déploiement.

Le déploiement peut être effectué sur le portail Azure, à l'aide de PowerShell ou d'Azure CLI, comme indiqué ci-dessous.

### <a name="azure-portal"></a>Portail Azure

1. Dans notre [référentiel GitHub](https://github.com/Azure/Azure-Lighthouse-samples/), sélectionnez le bouton **Déployer sur Azure** disponible en regard du modèle que vous souhaitez utiliser. Le modèle s’ouvre dans le portail Azure.
1. Entrez vos valeurs dans les champs **Nom de l'offre MSP**, **Description de l'offre MSP**, **ID géré par le locataire** et **Autorisations**. Si vous préférez, vous pouvez sélectionner **Modifier les paramètres** pour entrer les valeurs de `mspOfferName`, `mspOfferDescription`, `managedbyTenantId` et `authorizations` directement dans le fichier de paramètres. Veillez à mettre à jour ces valeurs plutôt que d'utiliser les valeurs par défaut du modèle.
1. Sélectionnez **Vérifier et créer**, puis **Créer**.

Au bout de quelques minutes, une notification indique que le déploiement est terminé.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

# Deploy Azure Resource Manager template using template and parameter file locally
New-AzSubscriptionDeployment -Name <deploymentName> `
                 -Location <AzureRegion> `
                 -TemplateFile <pathToTemplateFile> `
                 -TemplateParameterFile <pathToParameterFile> `
                 -Verbose

# Deploy Azure Resource Manager template that is located externally
New-AzSubscriptionDeployment -Name <deploymentName> `
                 -Location <AzureRegion> `
                 -TemplateUri <templateUri> `
                 -TemplateParameterUri <parameterUri> `
                 -Verbose
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

# Deploy Azure Resource Manager template using template and parameter file locally
az deployment sub create --name <deploymentName> \
                         --location <AzureRegion> \
                         --template-file <pathToTemplateFile> \
                         --parameters <parameters/parameterFile> \
                         --verbose

# Deploy external Azure Resource Manager template, with local parameter file
az deployment sub create --name <deploymentName> \
                         --location <AzureRegion> \
                         --template-uri <templateUri> \
                         --parameters <parameterFile> \
                         --verbose
```

## <a name="confirm-successful-onboarding"></a>Vérifier la réussite de l’intégration

Lorsqu'un abonnement de client a été correctement intégré à Azure Lighthouse, les utilisateurs du locataire du fournisseur de services peuvent voir l'abonnement et ses ressources (si l'accès à ceux-ci leur a été octroyé via le processus ci-dessus, soit individuellement, soit en tant que membres d'un groupe Azure AD disposant des autorisations appropriées). Pour vérifier cela, assurez-vous que l’abonnement s’affiche de l’une des manières suivantes.  

### <a name="azure-portal"></a>Portail Azure

Dans le locataire du fournisseur de services :

1. Accédez à la [page Mes clients](view-manage-customers.md).
2. Sélectionnez **Clients**.
3. Vérifiez que vous pouvez voir les abonnements portant le nom de l’offre que vous avez fourni dans le modèle Resource Manager.

> [!IMPORTANT]
> Pour afficher l'abonnement délégué dans [Mes clients](view-manage-customers.md), les utilisateurs du locataire du fournisseur de services doivent avoir reçu le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader) (ou un autre rôle intégré qui comprend un accès Lecteur) lorsque l'abonnement a été intégré.

Dans le locataire du client :

1. Accédez à la [page Fournisseurs de services](view-manage-service-providers.md).
2. Sélectionnez **Offres de fournisseur de services**.
3. Vérifiez que vous pouvez voir les abonnements portant le nom de l’offre que vous avez fourni dans le modèle Resource Manager.

> [!NOTE]
> Quelques minutes peuvent s’écouler après votre déploiement, avant que les mises à jour apparaissent dans le portail Azure.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

Get-AzContext

# Confirm successful onboarding for Azure Lighthouse

Get-AzManagedServicesDefinition
Get-AzManagedServicesAssignment
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az account list
```

Si vous devez apporter des modifications après l’intégration du client, vous pouvez [mettre à jour la délégation](update-delegation.md). Vous pouvez également [supprimer complètement l’accès à la délégation](remove-delegation.md).

## <a name="troubleshooting"></a>Résolution des problèmes

Si vous ne parvenez pas à intégrer votre client ou si vos utilisateurs ont des difficultés à accéder aux ressources déléguées, vérifiez les conseils et les conditions suivantes, puis réessayez.

- La valeur `managedbyTenantId` ne doit pas être identique à l’ID de locataire pour l’abonnement en cours d’intégration.
- Vous ne pouvez pas avoir plusieurs affectations dans la même étendue avec le même `mspOfferName`. 
- Le fournisseur de ressources **Microsoft.ManagedServices** doit être inscrit pour l’abonnement délégué. Cela doit se produire automatiquement pendant le déploiement, mais si ce n’est pas le cas, vous pouvez [l’inscrire manuellement](../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider).
- Les autorisations ne doivent pas inclure d’utilisateurs dotés du rôle intégré [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) ni d’aucun rôle intégré avec [DataActions](../../role-based-access-control/role-definitions.md#dataactions).
- Les groupes doivent être créés avec un [**type de groupe**](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md#group-types) défini sur **Sécurité** et non **Microsoft 365**.
- Les utilisateurs qui ont besoin d’afficher les ressources dans le portail Azure doivent avoir le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader) (ou un autre rôle intégré qui inclut l’accès Lecteur).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les [Expériences de gestion inter-locataire](../concepts/cross-tenant-management-experience.md).
- [Affichez et gérez les clients](view-manage-customers.md) en accédant à **Mes clients** sur le portail Azure.
- Apprenez à [mettre à jour](update-delegation.md) ou à [supprimer](remove-delegation.md) une délégation.

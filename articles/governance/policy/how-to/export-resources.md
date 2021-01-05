---
title: Exporter les ressources Azure Policy
description: Apprenez à exporter des ressources Azure Policy vers GitHub, comme les définitions de stratégie et les attributions de stratégie.
ms.date: 10/29/2020
ms.topic: how-to
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 923b063244f6f47def1c3e6a63d6e4d6b3b88083
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94919563"
---
# <a name="export-azure-policy-resources"></a>Exporter les ressources Azure Policy

Cet article fournit des informations sur l’exportation de vos ressources Azure Policy existantes. L’exportation de vos ressources est utile et recommandée pour la sauvegarde, mais il s’agit également d’une étape importante de la gouvernance cloud et du traitement de votre [stratégie en tant que code](../concepts/policy-as-code.md). Les ressources Azure Policy peuvent être exportées via [le Portail Azure](#export-with-azure-portal), [Azure CLI](#export-with-azure-cli), [Azure PowerShell](#export-with-azure-powershell) et chacun des kits de développement logiciel (SDK) pris en charge.

## <a name="export-with-azure-portal"></a>Exporter avec le Portail Azure

Pour exporter une définition de stratégie à partir du Portail Azure, procédez comme suit :

1. Lancez le service Azure Policy dans le portail Azure en cliquant sur **Tous les services**, puis en recherchant et en cliquant sur **Stratégie**.

1. Sélectionnez **Définitions** sur le côté gauche de la page Azure Policy.

1. Utilisez le bouton **Exporter les définitions** ou sélectionnez les points de suspension sur la ligne d’une définition de stratégie, puis sélectionnez **Exporter la définition**.

1. Sélectionnez le bouton **Sign in with GitHub** (Se connecter avec GitHub). Si vous n’avez pas encore été authentifié avec GitHub pour autoriser Azure Policy à exporter la ressource, passez en revue les besoins [GitHub Actions](https://github.com/features/actions) dans la nouvelle fenêtre qui s’ouvre, puis sélectionnez **Autoriser AzureGitHubActions** pour poursuivre le processus d’exportation. Une fois l’opération terminée, la nouvelle fenêtre se ferme automatiquement.

1. Sous l’onglet **Général**, définissez les options suivantes, puis sélectionnez l’onglet **Stratégies** ou **Suivant : Stratégies** en bas de la page.

   - **Filtre de référentiel** : Affectez la valeur _Mes référentiels_ pour afficher uniquement les référentiels que vous possédez, ou _Tous les référentiels_ pour voir tous les référentiels auxquels vous avez accordé l’accès à GitHub Action.
   - **Référentiel** : Définissez sur le référentiel dans lequel vous souhaitez exporter les ressources Azure Policy.
   - **Branche** : Définissez la branche dans le référentiel. L’utilisation d’une autre branche que la branche par défaut est un bon moyen de valider vos mises à jour avant de les fusionner dans votre code source.
   - **Répertoire** : Le _dossier racine_ vers lequel exporter les ressources Azure Policy. Les sous-dossiers de ce répertoire sont créés en fonction des ressources exportées.

1. Sous l’onglet **Stratégies**, définissez l’étendue à rechercher en sélectionnant les points de suspension et en choisissant une combinaison de groupes d’administration, d’abonnements ou de groupes de ressources.
   
1. Utilisez le bouton **Ajouter une ou plusieurs définitions de stratégie** pour rechercher l’étendue des objets à exporter. Dans la fenêtre latérale qui s’ouvre, sélectionnez chaque objet à exporter. Filtrez la sélection avec la zone de recherche ou en sélectionnant un type. Une fois que vous avez sélectionné tous les objets à exporter, utilisez le bouton **Ajouter** au bas de la page.

1. Pour chaque objet sélectionné, sélectionnez les options d’exportation souhaitées, par exemple _Définition seulement_ ou _Définition et affectation(s)_ pour une définition de stratégie. Sélectionnez ensuite l’onglet **Vérifier + exporter** ou le bouton **Suivant : Vérifier + exporter** en bas de la page.

   > [!NOTE]
   > Si vous avez choisi l’option _Définition et affectation(s)_ , seules les affectations de stratégie au sein de l’étendue définie par le filtre lors de l’ajout de la définition de stratégie sont exportées.

1. Sous l’onglet **Vérifier + exporter**, vérifiez que les détails correspondent, puis utilisez le bouton **Exporter** au bas de la page.

1. Vérifiez votre référentiel, votre branche et votre _dossier de niveau racine_ GitHub pour vérifier que les ressources sélectionnées sont désormais exportées vers votre contrôle de code source.

Les ressources Azure Policy sont exportées dans la structure suivante dans le référentiel et le _dossier de niveau racine_ GitHub sélectionnés :

```text
|
|- <root level folder>/  ________________ # Root level folder set by Directory property
|  |- policies/  ________________________ # Subfolder for policy objects
|     |- <displayName>_<name>____________ # Subfolder based on policy displayName and name properties
|        |- policy.json _________________ # Policy definition
|        |- assign.<displayName>_<name>__ # Each assignment (if selected) based on displayName and name properties
|
```

## <a name="export-with-azure-cli"></a>Exporter avec Azure CLI

Les définitions, initiatives et affectations d’Azure Policy peuvent être exportées au format JSON avec [Azure CLI](/cli/azure/install-azure-cli). Chacune de ces commandes utilise un paramètre **name** pour spécifier l’objet dont vous souhaitez obtenir le JSON. La propriété **name** est souvent un _GUID_ et n’est pas le **displayName** de l’objet.

- Définition : [az policy definition show](/cli/azure/policy/definition#az_policy_definition_show)
- Initiative : [az policy set-definition show](/cli/azure/policy/set-definition#az_policy_set_definition_show)
- Affectation : [az policy assignment show](/cli/azure/policy/assignment#az_policy_assignment_show)

Voici un exemple d’obtention de JSON pour une définition de stratégie avec la propriété **name** de _VirtualMachineStorage_ :

```azurecli-interactive
az policy definition show --name 'VirtualMachineStorage'
```

## <a name="export-with-azure-powershell"></a>Exporter avec Azure PowerShell

Les définitions, initiatives et affectations d’Azure Policy peuvent être exportées au format JSON avec [Azure PowerShell](/powershell/azure/). Chacune de ces applets de commande utilise un paramètre **Name** pour spécifier l’objet dont vous souhaitez obtenir le JSON. La propriété **Name** est souvent un _GUID_ et n’est pas le **displayName** de l’objet.

- Définition : [Get-AzPolicyDefinition](/powershell/module/az.resources/get-azpolicydefinition)
- Initiative : [Get-AzPolicySetDefinition](/powershell/module/az.resources/get-azpolicysetdefinition)
- Affectation : [Get-AzPolicyAssignment](/powershell/module/az.resources/get-azpolicyassignment)

Voici un exemple d’obtention de JSON pour une définition de stratégie avec la propriété **Name** de _VirtualMachineStorage_ :

```azurepowershell-interactive
Get-AzPolicyDefinition -Name 'VirtualMachineStorage' | ConvertTo-Json -Depth 10
```

## <a name="next-steps"></a>Étapes suivantes

- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).
- Découvrez comment [créer des stratégies par programmation](programmatically-create.md).
- Découvrez comment [corriger des ressources non conformes](remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).

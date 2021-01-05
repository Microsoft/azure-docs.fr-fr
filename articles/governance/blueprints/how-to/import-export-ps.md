---
title: Importer et exporter des blueprints avec PowerShell
description: Découvrez comment utiliser vos définitions de blueprint en tant que code. Partagez-les, contrôlez leur code source et gérez-les à l’aide des commandes d’exportation et d’importation.
ms.date: 08/27/2020
ms.topic: how-to
ms.openlocfilehash: 87e8f00013fec3abb8132f92c1c717f152d1e264
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89051438"
---
# <a name="import-and-export-blueprint-definitions-with-powershell"></a>Importer et exporter des définitions de blueprint avec PowerShell

Vous pouvez gérer Azure Blueprints entièrement via le portail Azure. À mesure que les organisations progressent dans leur utilisation d'Azure Blueprints, elles doivent commencer à considérer les définitions de blueprint comme du code géré. Ce concept est souvent appelé infrastructure en tant que code (IaC). Le traitement de vos définitions de blueprint en tant que code offre des avantages supplémentaires au-delà de ce que propose le portail Azure. Les avantages sont les suivants :

- Partage de définitions de blueprint
- Sauvegarde de vos définitions de blueprint
- Réutilisation de définitions de blueprint dans différents locataires ou abonnements
- Placement des définitions de blueprint dans un contrôle de code source
  - Tests automatisés de définitions de blueprint dans des environnements de test
  - Prise en charge des pipelines d’intégration continue et de déploiement continu (CI/CD)

Quelles que soient vos raisons, la gestion de vos définitions de blueprint en tant que code présente des avantages. Cet article explique comment utiliser les commandes `Import-AzBlueprintWithArtifact` et `Export-AzBlueprintWithArtifact` dans le module [AZ.blueprint](https://powershellgallery.com/packages/Az.Blueprint/).

## <a name="prerequisites"></a>Prérequis

Cet article part tu principe que vous disposez d’une connaissance pratique modérée d’Azure Blueprints. Si vous ne l’avez pas encore fait, consultez les articles suivants :

- [Créer un blueprint dans le portail](../create-blueprint-portal.md)
- Découvrir les [phases de déploiement](../concepts/deployment-stages.md) et le [cycle de vie de blueprint](../concepts/lifecycle.md)
- [Création](../create-blueprint-powershell.md) et [gestion](./manage-assignments-ps.md) de définitions et d’affectations de blueprint avec PowerShell

Si le module n’est pas encore installé, suivez les instructions fournies dans [Ajouter le module Az.Blueprint](./manage-assignments-ps.md#add-the-azblueprint-module) pour installer et valider le module **AZ.Blueprint** à partir de PowerShell Gallery.

## <a name="folder-structure-of-a-blueprint-definition"></a>Structure de dossiers d’une définition de blueprint

Avant d’examiner l’exportation et l’importation de blueprints, voyons comment les fichiers qui composent la définition de blueprint sont structurés. Une définition de blueprint doit être stockée dans son propre dossier.

> [!IMPORTANT]
> Si aucune valeur n’est transmise au paramètre **Name** de la cmdlet `Import-AzBlueprintWithArtifact`, le nom du dossier dans lequel la définition de blueprint est stockée est utilisé.

La définition de blueprint qui doit être nommée `blueprint.json` est accompagnée des artefacts composant la définition de blueprint. Chaque artefact doit se trouver dans le sous-dossier nommé `artifacts`.
La structure de votre définition de blueprint en tant que fichiers JSON dans des dossiers doit se présenter comme suit :

```text
.
|
|- MyBlueprint/  _______________ # Root folder name becomes default name of blueprint definition
|  |- blueprint.json  __________ # The blueprint definition. Fixed name.
|
|  |- artifacts/  ______________ # Subfolder for all blueprint artifacts. Fixed name.
|     |- artifact.json  ________ # Blueprint artifact as JSON file. Artifact named from file.
|     |- ...
|     |- more-artifacts.json

```

## <a name="export-your-blueprint-definition"></a>Exporter votre définition de blueprint

Les étapes d’exportation de votre définition de blueprint sont simples. L’exportation de la définition de blueprint peut être utile pour le partage, la sauvegarde ou le placement dans le contrôle de code source.

- **Blueprint** [obligatoire]
  - Décrit la définition de blueprint.
  - Utilisez `Get-AzBlueprint` pour obtenir l’objet de référence
- **OutputPath** [obligatoire]
  - Spécifie le chemin d’accès pour l’enregistrement des fichiers JSON
  - Les fichiers de sortie se trouvent dans un sous-dossier portant le nom de la définition de blueprint.
- **Version** (facultatif)
  - Spécifie la version à produire si l’ objet de référence **Blueprint** contient des références à plusieurs versions.

1. Obtenez une référence à la définition de blueprint à exporter de l’abonnement représentée en tant que `{subId}` :

   ```azurepowershell-interactive
   # Login first with Connect-AzAccount if not using Cloud Shell

   # Get version '1.1' of the blueprint definition in the specified subscription
   $bpDefinition = Get-AzBlueprint -SubscriptionId '{subId}' -Name 'MyBlueprint' -Version '1.1'
   ```

1. Utilisez la cmdlet `Export-AzBlueprintWithArtifact` pour exporter la définition de blueprint spécifiée :

   ```azurepowershell-interactive
   Export-AzBlueprintWithArtifact -Blueprint $bpDefinition -OutputPath 'C:\Blueprints'
   ```

## <a name="import-your-blueprint-definition"></a>Importer votre définition de blueprint

Une fois que vous disposez d’une [définition de blueprint exportée](#export-your-blueprint-definition) ou que vous avez créé manuellement une définition de plan dans la [structure de dossiers requise](#folder-structure-of-a-blueprint-definition), vous pouvez importer cette définition de blueprint dans une autre groupe d’administration ou abonnement.

Pour obtenir des exemples de définitions de blueprint intégrées, voir le [dépôt GitHub Azure Blueprints](https://github.com/Azure/azure-blueprints/tree/master/samples/001-builtins).

- **Name** [obligatoire]
  - Spécifie le nom de la nouvelle définition de blueprint
- **InputPath** [obligatoire]
  - Spécifie le chemin d’accès à partir duquel créer la définition de blueprint
  - Doit correspondre à [la structure des dossiers requise](#folder-structure-of-a-blueprint-definition)
- **ManagementGroupId** (facultatif)
  - ID du groupe d’administration dans lequel enregistrer la définition de blueprint si ce n’est pas le contexte actuel par défaut
  - **ManagementGroupId** ou **SubscriptionId** doivent être spécifiés
- **SubscriptionId** (facultatif)
  - ID de l’abonnement dans lequel enregistrer la définition de blueprint si ce n’est pas le contexte actuel par défaut
  - **ManagementGroupId** ou **SubscriptionId** doivent être spécifiés

1. Utilisez la cmdlet `Import-AzBlueprintWithArtifact` pour importer la définition de blueprint spécifiée :

   ```azurepowershell-interactive
   # Login first with Connect-AzAccount if not using Cloud Shell

   Import-AzBlueprintWithArtifact -Name 'MyBlueprint' -ManagementGroupId 'DevMG' -InputPath 'C:\Blueprints\MyBlueprint'
   ```

Une fois la définition de blueprint importée, [attribuez-la avec PowerShell](./manage-assignments-ps.md#create-blueprint-assignments).

Pour plus d’informations sur la création de définitions de blueprint avancées, voir les articles suivants :

- Utilisez des [paramètres statiques et dynamiques](../concepts/parameters.md).
- Personnalisez l’[ordre de séquencement des blueprints](../concepts/sequencing-order.md).
- Protégez les déploiements avec un [verrouillage des ressources blueprint](../concepts/resource-locking.md).
- [Gérez les blueprints en tant que code](https://github.com/Azure/azure-blueprints/blob/master/README.md).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [cycle de vie des blueprints](../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../concepts/resource-locking.md).
- Résoudre les problèmes durant l’affectation d’un blueprint en suivant les étapes de [dépannage général](../troubleshoot/general.md).
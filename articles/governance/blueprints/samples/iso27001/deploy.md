---
title: Déployer l’exemple de blueprint ISO 27001
description: Étapes de déploiement pour l’exemple de blueprint ISO 27001 autonome et informations détaillées sur les paramètres d’artefact de blueprint.
ms.date: 11/02/2020
ms.topic: sample
ms.openlocfilehash: 75403710ecbfb0cc7046804b45c2da042531e98b
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420279"
---
# <a name="deploy-the-iso-27001-blueprint-sample"></a>Déployer l’exemple de blueprint ISO 27001

Pour déployer l’exemple de blueprint ISO 27001 pour Azure Blueprints, vous devez effectuer les étapes suivantes :

> [!div class="checklist"]
> - Créer un blueprint à partir de l’exemple
> - Marquer la copie de l’exemple en tant que **Publié**
> - Affecter votre copie du blueprint à un abonnement existant

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

## <a name="create-blueprint-from-sample"></a>Créer un blueprint à partir de l’exemple

Commencez par implémenter l’exemple de blueprint. Pour cela, créez un blueprint dans votre environnement en vous servant de l’exemple comme point de départ.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Dans la page **Démarrage** à gauche, sélectionnez le bouton **Créer** sous _Créer un blueprint_.

1. Recherchez l’exemple de blueprint **ISO 27001** sous _Autres exemples_ et sélectionnez **Utiliser cet exemple**.

1. Entrez les _Fonctions de base_ de l’exemple de blueprint :

   - **Nom du blueprint**  : entrez le nom de votre copie de l’exemple de blueprint ISO 27001.
   - **Emplacement de la définition**  : utilisez les points de suspension et sélectionnez le groupe d’administration dans lequel vous souhaitez enregistrer votre copie de l’exemple.

1. Sélectionnez l’onglet _Artefacts_ dans le haut de la page ou **Suivant : Artefacts** dans le bas de la page.

1. Passez en revue la liste des artefacts qui composent l’exemple de blueprint. De nombreux artefacts ont des paramètres que nous définirons plus tard. Sélectionnez **Enregistrer comme brouillon** lorsque vous avez terminé de passer en revue l’exemple de blueprint.

## <a name="publish-the-sample-copy"></a>Publier la copie de l’exemple

Votre copie de l’exemple de blueprint est à présent créée dans votre environnement. Elle est créée en mode **Brouillon** et doit être **publiée** avant de pouvoir être attribuée et déployée. Vous pouvez personnaliser la copie de l’exemple de blueprint en fonction de votre environnement et de vos besoins, mais de telles modifications peuvent l’éloigner de la norme ISO 27001.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Publier le blueprint** dans le haut de la page. Dans la nouvelle page à droite, indiquez la **Version** de votre copie de l’exemple de blueprint. Cette propriété est utile si vous prévoyez d’effectuer une modification ultérieurement. Dans **Notes de changement** , indiquez par exemple « Première version publiée à partir de l’exemple de blueprint ISO 27001 ». Sélectionnez ensuite **Publier** en bas de la page.

## <a name="assign-the-sample-copy"></a>Affecter la copie de l’exemple

Une fois que la copie de l’exemple de blueprint a été **publiée** , elle peut être affectée à un abonnement dans le groupe d’administration où elle a été enregistrée. C’est à cette étape que les paramètres sont fournis pour que chaque déploiement de la copie de l’exemple de blueprint soit unique.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Affecter le blueprint** dans le haut de la page.

1. Indiquez les valeurs des paramètres pour l’affectation du blueprint :

   - Concepts de base

     - **Abonnements**  : Sélectionnez un ou plusieurs des abonnements qui font partie du groupe d’administration où vous avez enregistré votre copie de l’exemple de blueprint. Si vous sélectionnez plusieurs abonnements, une affectation est créée pour chacun d’eux à l’aide des paramètres saisis.
     - **Nom de l’affectation**  : Le nom est prérempli en fonction du nom du blueprint.
       Changez-le si nécessaire ou laissez-le tel quel.
     - **Emplacement** : Sélectionnez une région dans laquelle créer l’identité managée. Azure Blueprint utilise cette identité managée pour déployer tous les artefacts dans le blueprint affecté. Pour en savoir plus, consultez [Identités managées pour les ressources Azure](../../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Version de définition du blueprint**  : Choisissez une version **publiée** de votre copie de l’exemple de blueprint.

   - Verrouiller l'affectation

     Sélectionnez le paramètre de verrouillage de blueprint pour votre environnement. Pour plus d’informations, consultez [Verrouillage des ressources des blueprints](../../concepts/resource-locking.md).

   - Identité managée

     Conservez l’option par défaut, _Affecté(e) par le système_ , pour l’identité managée.

   - Paramètres de blueprint

     Les paramètres définis dans cette section sont utilisés par de nombreux artefacts dans la définition du blueprint à des fins de cohérence.

     - **Emplacement autorisé pour les ressources et les groupes de ressources**  : valeur qui indique les emplacements autorisés pour les groupes de ressources et les ressources.

   - Paramètres d'artefact

     Les paramètres définis dans cette section s’appliquent à l’artefact sous lequel elle est définie. Ces paramètres sont des [paramètres dynamiques](../../concepts/parameters.md#dynamic-parameters) puisqu’ils sont définis lors de l’affectation du blueprint. Pour obtenir la liste complète des paramètres d’artefact et leur description, consultez le [tableau des paramètres d’artefact](#artifact-parameters-table).

1. Une fois tous les paramètres entrés, sélectionnez **Affecter** au bas de la page. L’affectation du blueprint est créée et le déploiement de l’artefact démarre. Le déploiement prend environ une heure. Pour vérifier l’état du déploiement, ouvrez l’affectation du blueprint.

> [!WARNING]
> Le service Azure Blueprints et les exemples de blueprint intégrés sont **gratuits**. Les ressources Azure sont [facturées par produit](https://azure.microsoft.com/pricing/). Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour estimer le coût d’exécution des ressources déployées par cet exemple de blueprint.

## <a name="artifact-parameters-table"></a>Tableau des paramètres d’artefact

Le tableau suivant fournit la liste des paramètres d’artefact de blueprint :

|Nom de l’artefact|Type d’artefact|Nom du paramètre|Description|
|-|-|-|-|
|\[Préversion\] : Déployer Log Analytics Agent pour Linux VM Scale Sets (VMSS)|Affectation de rôle|Espace de travail Log Analytics pour Linux VM Scale Sets (VMSS)|Si cet espace de travail se trouve en dehors de l’étendue de l’affectation, vous devez accorder manuellement des autorisations de « Contributeur Log Analytics » (ou similaires) à l’ID du principal de l’affectation de stratégie.|
|\[Préversion\] : Déployer Log Analytics Agent pour Linux VM Scale Sets (VMSS)|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Linux pris en charge à ajouter à l’étendue|Un tableau vide peut être utilisé pour indiquer l’absence de paramètres facultatifs : \[\]|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Linux|Affectation de rôle|Espace de travail Log Analytics pour les machines virtuelles Linux|Si cet espace de travail se trouve en dehors de l’étendue de l’affectation, vous devez accorder manuellement des autorisations de « Contributeur Log Analytics » (ou similaires) à l’ID du principal de l’affectation de stratégie.|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Linux|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Linux pris en charge à ajouter à l’étendue|Un tableau vide peut être utilisé pour indiquer l’absence de paramètres facultatifs : \[\]|
|\[Préversion\] : Déployer Log Analytics Agent pour Windows VM Scale Sets (VMSS)|Affectation de rôle|Espace de travail Log Analytics pour Windows VM Scale Sets (VMSS)|Si cet espace de travail se trouve en dehors de l’étendue de l’affectation, vous devez accorder manuellement des autorisations de « Contributeur Log Analytics » (ou similaires) à l’ID du principal de l’affectation de stratégie.|
|\[Préversion\] : Déployer Log Analytics Agent pour Windows VM Scale Sets (VMSS)|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Windows pris en charge à ajouter à l’étendue|Un tableau vide peut être utilisé pour indiquer l’absence de paramètres facultatifs : \[\]|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Windows|Affectation de rôle|Espace de travail Log Analytics pour les machines virtuelles Windows|Si cet espace de travail se trouve en dehors de l’étendue de l’affectation, vous devez accorder manuellement des autorisations de « Contributeur Log Analytics » (ou similaires) à l’ID du principal de l’affectation de stratégie.|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Windows|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Windows pris en charge à ajouter à l’étendue|Un tableau vide peut être utilisé pour indiquer l’absence de paramètres facultatifs : \[\]|
|Références SKU de compte de stockage autorisées|Affectation de rôle|Liste de références SKU de stockage autorisées|Liste des références qui peuvent être spécifiées pour les comptes de stockage.|
|Références SKU de machine virtuelle autorisées|Affectation de rôle|Liste de références SKU de machine virtuelle autorisées|Liste des références qui peuvent être spécifiées pour les machines virtuelles.|
|Initiative de blueprint pour ISO 27001|Affectation de rôle|Liste des types de ressources devant avoir les journaux de diagnostic activés|Liste des types de ressources utilisés pour déterminer si le paramètre de journal de diagnostic n’est pas activé. Les valeurs acceptables sont listées dans les [schémas de journaux de diagnostic Azure Monitor](../../../../azure-monitor/platform/resource-logs-schema.md#service-specific-schemas).|

## <a name="next-steps"></a>Étapes suivantes

Vous venez de passer en revue les étapes à suivre pour déployer l’exemple de blueprint ISO 27001. Consultez à présent les articles suivants pour découvrir l’architecture et le mappage des contrôles :

> [!div class="nextstepaction"]
> [Blueprint ISO 27001 - Vue d’ensemble](./index.md)
> [Blueprint ISO 27001 - Mappage des contrôles](./control-mapping.md)

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).

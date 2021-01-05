---
title: Vue d’ensemble de l’exemple de blueprint Azure Security Benchmark
description: Vue d’ensemble de l’exemple de blueprint Azure Security Benchmark. Cet exemple de blueprint aide les clients à évaluer des contrôles spécifiques.
ms.date: 10/14/2020
ms.topic: sample
ms.openlocfilehash: 6b147ffd4efbf814b6119f5d7e90dc7af2ea2687
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057584"
---
# <a name="azure-security-benchmark-blueprint-sample"></a>Exemple de blueprint Azure Security Benchmark

L’exemple de blueprint Azure Security Benchmark fournit des garde-fous de gouvernance avec [Azure Policy](../../policy/overview.md), qui vous permettent d’évaluer des contrôles [Azure Security Benchmark](../../../security/benchmarks/overview.md) spécifiques. Avec ce blueprint, les clients peuvent déployer un ensemble de stratégies pour toute architecture déployée par Azure où ils ont l’intention d’implémenter des contrôles Azure Security Benchmark.

## <a name="control-mapping"></a>Correspondance des contrôles

Le [mappage de contrôles Azure Policy](../../policy/samples/azure-security-benchmark.md) fournit des détails sur les définitions de stratégie incluses dans ce blueprint et sur la façon dont ces définitions de stratégie sont mappées aux **domaines de conformité** et aux **contrôles** dans Azure Security Benchmark. Quand des ressources sont attribuées à une architecture, elles sont évaluées par Azure Policy pour savoir si elles sont en conformité avec les stratégies affectées. Pour plus d’informations, consultez [Présentation d’Azure Policy](../../policy/overview.md).

## <a name="deploy"></a>Déployer

Pour déployer l’exemple de blueprint Azure Security Benchmark pour Azure Blueprints, vous devez effectuer les étapes suivantes :

> [!div class="checklist"]
> - Créer un blueprint à partir de l’exemple
> - Marquer la copie de l’exemple en tant que **Publié**
> - Affecter votre copie du blueprint à un abonnement existant

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

### <a name="create-blueprint-from-sample"></a>Créer un blueprint à partir de l’exemple

Commencez par implémenter l’exemple de blueprint. Pour cela, créez un blueprint dans votre environnement en vous servant de l’exemple comme point de départ.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Dans la page **Démarrage**à gauche, sélectionnez le bouton **Créer** sous _Créer un blueprint_.

1. Recherchez l’exemple de blueprint **Azure Security Benchmark** sous _Autres exemples_, puis sélectionnez le nom pour choisir cet exemple.

1. Entrez les _Fonctions de base_ de l’exemple de blueprint :

   - **Nom du blueprint** : Spécifiez un nom pour votre copie de l’exemple de blueprint Azure Security Benchmark.
   - **Emplacement de la définition** : utilisez les points de suspension et sélectionnez le groupe d’administration dans lequel vous souhaitez enregistrer votre copie de l’exemple.

1. Sélectionnez l’onglet _Artefacts_ dans le haut de la page ou **Suivant : Artefacts** dans le bas de la page.

1. Passez en revue la liste des artefacts qui sont inclus dans l’exemple de blueprint. De nombreux artefacts ont des paramètres que nous définirons plus tard. Sélectionnez **Enregistrer comme brouillon** lorsque vous avez terminé de passer en revue l’exemple de blueprint.

### <a name="publish-the-sample-copy"></a>Publier la copie de l’exemple

Votre copie de l’exemple de blueprint est à présent créée dans votre environnement. Elle est créée en mode **Brouillon** et doit être **publiée** avant de pouvoir être attribuée et déployée. Vous pouvez personnaliser la copie de l’exemple de blueprint en fonction de votre environnement et de vos besoins, mais ces modifications peuvent l’éloigner de son alignement avec les recommandations d’Azure Security Benchmark.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Publier le blueprint** dans le haut de la page. Dans la nouvelle page à droite, indiquez la **Version** de votre copie de l’exemple de blueprint. Cette propriété est utile si vous prévoyez d’effectuer une modification ultérieurement. Dans **Notes de changement**, indiquez par exemple « Première version publiée à partir de l’exemple de blueprint Azure Security Benchmark ». Sélectionnez ensuite **Publier** en bas de la page.

### <a name="assign-the-sample-copy"></a>Affecter la copie de l’exemple

Une fois que la copie de l’exemple de blueprint a été **publiée**, elle peut être affectée à un abonnement dans le groupe d’administration où elle a été enregistrée. C’est à cette étape que les paramètres sont fournis pour que chaque déploiement de la copie de l’exemple de blueprint soit unique.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Affecter le blueprint** dans le haut de la page.

1. Indiquez les valeurs des paramètres pour l’affectation du blueprint :

   - Concepts de base

     - **Abonnements** : Sélectionnez un ou plusieurs des abonnements qui font partie du groupe d’administration où vous avez enregistré votre copie de l’exemple de blueprint. Si vous sélectionnez plusieurs abonnements, une affectation est créée pour chacun d’eux à l’aide des paramètres saisis.
     - **Nom de l’affectation** : Le nom est prérempli en fonction du nom du blueprint.
       Changez-le si nécessaire ou laissez-le tel quel.
     - **Emplacement** : Sélectionnez une région dans laquelle créer l’identité managée. Azure Blueprint utilise cette identité managée pour déployer tous les artefacts dans le blueprint affecté. Pour en savoir plus, consultez [Identités managées pour les ressources Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Version de définition du blueprint** : Choisissez une version **publiée** de votre copie de l’exemple de blueprint.

   - Verrouiller l'affectation

     Sélectionnez le paramètre de verrouillage de blueprint pour votre environnement. Pour plus d’informations, consultez [Verrouillage des ressources des blueprints](../concepts/resource-locking.md).

   - Identité managée

     Conservez l’option par défaut, _Affecté(e) par le système_, pour l’identité managée.

   - Paramètres d'artefact

     Les paramètres définis dans cette section s’appliquent à l’artefact sous lequel elle est définie. Ces paramètres sont des [paramètres dynamiques](../concepts/parameters.md#dynamic-parameters) puisqu’ils sont définis lors de l’affectation du blueprint. Pour obtenir la liste complète des paramètres d’artefact et leur description, consultez le [tableau des paramètres d’artefact](#artifact-parameters-table).

1. Une fois tous les paramètres entrés, sélectionnez **Affecter** au bas de la page. L’affectation du blueprint est créée et le déploiement de l’artefact démarre. Le déploiement prend environ une heure. Pour vérifier l’état du déploiement, ouvrez l’affectation du blueprint.

> [!WARNING]
> Le service Azure Blueprints et les exemples de blueprint intégrés sont **gratuits**. Les ressources Azure sont [facturées par produit](https://azure.microsoft.com/pricing/). Utilisez la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour estimer le coût d’exécution des ressources déployées par cet exemple de blueprint.

### <a name="artifact-parameters-table"></a>Tableau des paramètres d’artefact

Le tableau suivant fournit la liste des paramètres d’artefact de blueprint :

|Nom de l’artefact|Type d’artefact|Nom du paramètre|Description|
|-|-|-|-|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Liste des utilisateurs exclus du groupe Administrateurs de machine virtuelle Windows|Liste séparée par des points-virgules des membres qui doivent être exclus du groupe Administrateurs local. Exemple : Administrator; myUser1; myUser2|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Liste des utilisateurs à inclure dans le groupe Administrateurs de machine virtuelle Windows|Liste séparée par des points-virgules des membres qui doivent être inclus dans le groupe Administrateurs local. Exemple : Administrator; myUser1; myUser2|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Liste des utilisateurs que le groupe Administrateurs de machine virtuelle Windows doit *seulement* inclure|Liste séparée par des points-virgules de tous les membres attendus dans le groupe Administrateurs local. Exemple : Administrator; myUser1; myUser2|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Liste des régions où Network Watcher doit être activé|Pour voir la liste complète des régions, utilisez Get-AzLocation|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Réseau virtuel où les machines virtuelles doivent être connectées|Exemple : /subscriptions/YourSubscriptionId/resourceGroups/YourResourceGroupName/providers/Microsoft.Network/virtualNetworks/Name|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Passerelle réseau que les réseaux virtuels doivent utiliser|Exemple : /subscriptions/YourSubscriptionId/resourceGroups/YourResourceGroup/providers/Microsoft.Network/virtualNetworkGateways/Name|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Liste des ID d’espace de travail où les agents Log Analytics doivent se connecter|Liste délimitée par des points-virgules des ID d’espaces de travail auxquels l’agent Log Analytics doit être connecté|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Liste des types de ressources devant avoir les journaux de diagnostic activés|Auditez le paramètre de diagnostic pour les types de ressources sélectionnés|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Dernière version de PHP|Dernière version de PHP prise en charge pour App Services|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Dernière version de Java|Dernière version de Java prise en charge pour App Services|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Dernière version de Python sur Windows|Dernière version de Python prise en charge pour App Services|
|Auditer les recommandations d’Azure Security Benchmark et déployer des extensions de machine virtuelle de prise en charge spécifiques|Affectation de rôle|Dernière version de Python sur Linux|Dernière version de Python prise en charge pour App Services|

## <a name="next-steps"></a>Étapes suivantes

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../how-to/update-existing-assignments.md).
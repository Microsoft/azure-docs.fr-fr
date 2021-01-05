---
title: Vue d’ensemble de l’exemple de blueprint NIST SP 800-53 R4
description: Vue d’ensemble de l’exemple de blueprint NIST SP 800-53 R4. Cet exemple de blueprint aide les clients à évaluer des contrôles NIST SP 800-53 R4 spécifiques.
ms.date: 08/27/2020
ms.topic: sample
ms.openlocfilehash: f40451e1e82833aba4f626052e708f2c1136f832
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89048769"
---
# <a name="nist-sp-800-53-r4-blueprint-sample"></a>Exemple de blueprint NIST SP 800-53 R4

L’exemple de blueprint NIST SP 800-53 R4 fournit des garde-fous de gouvernance avec [Azure Policy](../../policy/overview.md), qui vous permettent d’évaluer des contrôles NIST SP 800-53 R4 spécifiques. Avec ce blueprint, les clients peuvent déployer un ensemble de stratégies pour toute architecture déployée par Azure devant implémenter des contrôles NIST SP 800-53 R4.

## <a name="control-mapping"></a>Correspondance des contrôles

Le [mappage de contrôles Azure Policy](../../policy/samples/nist-sp-800-53-r4.md) fournit des détails sur les définitions de stratégie incluses dans ce blueprint et sur la façon dont ces définitions de stratégie sont mappées aux **domaines de conformité** et aux **contrôles** dans NIST SP 800-53 R4. Quand des ressources sont attribuées à une architecture, elles sont évaluées par Azure Policy pour savoir si elles sont en conformité avec les définitions de stratégies affectées. Pour plus d’informations, consultez [Présentation d’Azure Policy](../../policy/overview.md).

## <a name="deploy"></a>Déployer

Pour déployer l’exemple de blueprint NIST SP 800-53 R4 pour Azure Blueprints, vous devez effectuer les étapes suivantes :

> [!div class="checklist"]
> - Créer un blueprint à partir de l’exemple
> - Marquer la copie de l’exemple en tant que **Publié**
> - Affecter votre copie du blueprint à un abonnement existant

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

### <a name="create-blueprint-from-sample"></a>Créer un blueprint à partir de l’exemple

Commencez par implémenter l’exemple de blueprint. Pour cela, créez un blueprint dans votre environnement en vous servant de l’exemple comme point de départ.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Dans la page **Démarrage**à gauche, sélectionnez le bouton **Créer** sous _Créer un blueprint_.

1. Recherchez l’exemple de blueprint **NIST SP 800-53 R4** sous _Autres exemples_ et sélectionnez **Utiliser cet exemple**.

1. Entrez les _Fonctions de base_ de l’exemple de blueprint :

   - **Nom du blueprint** : Entrez le nom de votre copie de l’exemple de blueprint NIST SP 800-53 R4.
   - **Emplacement de la définition** : utilisez les points de suspension et sélectionnez le groupe d’administration dans lequel vous souhaitez enregistrer votre copie de l’exemple.

1. Sélectionnez l’onglet _Artefacts_ dans le haut de la page ou **Suivant : Artefacts** dans le bas de la page.

1. Passez en revue la liste des artefacts qui composent l’exemple de blueprint. De nombreux artefacts ont des paramètres que nous définirons plus tard. Sélectionnez **Enregistrer comme brouillon** lorsque vous avez terminé de passer en revue l’exemple de blueprint.

### <a name="publish-the-sample-copy"></a>Publier la copie de l’exemple

Votre copie de l’exemple de blueprint est à présent créée dans votre environnement. Elle est créée en mode **Brouillon** et doit être **publiée** avant de pouvoir être attribuée et déployée. Vous pouvez personnaliser la copie de l’exemple de blueprint en fonction de votre environnement et de vos besoins, mais ces modifications peuvent l’éloigner de son alignement avec les contrôles NIST SP 800-53.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour rechercher votre copie de l’exemple de blueprint, puis sélectionnez-la.

1. Sélectionnez **Publier le blueprint** dans le haut de la page. Dans la nouvelle page à droite, indiquez la **Version** de votre copie de l’exemple de blueprint. Cette propriété est utile si vous prévoyez d’effectuer une modification ultérieurement. Dans **Notes de changement**, indiquez par exemple « Première version publiée à partir de l’exemple de blueprint NIST SP 800-53 R4 ». Sélectionnez ensuite **Publier** en bas de la page.

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
|\[Préversion\] : Auditer les contrôles NIST SP 800-53 R4 et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit|Affectation de rôle|ID de l’espace de travail Log Analytics pour lequel les machines virtuelles doivent être configurées|Il s’agit de l’ID (GUID) de l’espace de travail Log Analytics pour lequel les machines virtuelles doivent être configurées.|
|\[Préversion\] : Auditer les contrôles NIST SP 800-53 R4 et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit|Affectation de rôle|Liste des types de ressources devant avoir les journaux de diagnostic activés|Liste des types de ressources utilisés pour déterminer si le paramètre de journal de diagnostic n’est pas activé. Les valeurs acceptables sont listées dans les [schémas de journaux de diagnostic Azure Monitor](../../../azure-monitor/platform/resource-logs-schema.md#service-specific-schemas).|
|\[Préversion\] : Auditer les contrôles NIST SP 800-53 R4 et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit|Affectation de rôle|Liste des utilisateurs qui doivent être exclus du groupe Administrateurs de machines virtuelles Windows|Liste séparée par des points-virgules des membres qui doivent être exclus du groupe Administrateurs local. Exemple : Administrator; myUser1; myUser2|
|\[Préversion\] : Auditer les contrôles NIST SP 800-53 R4 et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit|Affectation de rôle|Liste des utilisateurs qui doivent être inclus dans le groupe Administrateurs de machines virtuelles Windows|Liste séparée par des points-virgules des membres qui doivent être inclus dans le groupe Administrateurs local. Exemple : Administrator; myUser1; myUser2|
|\[Préversion\] : Déployer Log Analytics Agent pour Linux VM Scale Sets (VMSS)|Affectation de rôle|Espace de travail Log Analytics pour Linux VM Scale Sets (VMSS)|Si cet espace de travail se trouve en dehors de l’étendue de l’affectation, vous devez accorder manuellement des autorisations de « Contributeur Log Analytics » (ou similaires) à l’ID du principal de l’affectation de stratégie.|
|\[Préversion\] : Déployer Log Analytics Agent pour Linux VM Scale Sets (VMSS)|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Linux pris en charge à ajouter à l’étendue|Un tableau vide peut être utilisé pour indiquer l’absence de paramètres facultatifs : \[\]|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Linux|Affectation de rôle|Espace de travail Log Analytics pour les machines virtuelles Linux|Si cet espace de travail se trouve en dehors de l’étendue de l’affectation, vous devez accorder manuellement des autorisations de « Contributeur Log Analytics » (ou similaires) à l’ID du principal de l’affectation de stratégie.|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Linux|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Linux pris en charge à ajouter à l’étendue|Un tableau vide peut être utilisé pour indiquer l’absence de paramètres facultatifs : \[\]|
|\[Préversion\] : Déployer Log Analytics Agent pour Windows VM Scale Sets (VMSS)|Affectation de rôle|Espace de travail Log Analytics pour Windows VM Scale Sets (VMSS)|Si cet espace de travail se trouve en dehors de l’étendue de l’affectation, vous devez accorder manuellement des autorisations de « Contributeur Log Analytics » (ou similaires) à l’ID du principal de l’affectation de stratégie.|
|\[Préversion\] : Déployer Log Analytics Agent pour Windows VM Scale Sets (VMSS)|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Windows pris en charge à ajouter à l’étendue|Un tableau vide peut être utilisé pour indiquer l’absence de paramètres facultatifs : \[\]|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Windows|Affectation de rôle|Espace de travail Log Analytics pour les machines virtuelles Windows|Si cet espace de travail se trouve en dehors de l’étendue de l’affectation, vous devez accorder manuellement des autorisations de « Contributeur Log Analytics » (ou similaires) à l’ID du principal de l’affectation de stratégie.|
|\[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Windows|Affectation de rôle|Facultatif : Liste des images de machine virtuelle ayant un système d’exploitation Windows pris en charge à ajouter à l’étendue|Un tableau vide peut être utilisé pour indiquer l’absence de paramètres facultatifs : \[\]|
|Déployer Advanced Threat Protection sur les comptes de stockage|Affectation de rôle|Résultat|Vous trouverez des informations sur les effets de la stratégie dans [Comprendre les effets d’Azure Policy](../../policy/concepts/effects.md).|
|Déployer l’audit sur des serveurs SQL|Affectation de rôle|Valeur en jours de la période de conservation (0 indique une conservation illimitée)|Durée de conservation en jours (facultatif ; 180 jours si non spécifié)|
|Déployer l’audit sur des serveurs SQL|Affectation de rôle|Nom du groupe de ressources pour le compte de stockage pour l’audit SQL Server|L’audit écrit des événements de base de données dans un journal d’audit dans votre compte Stockage Azure (un compte de stockage sera créé dans chaque région où est créé un serveur SQL Server qui sera partagé par tous les serveurs de cette région). Important : Pour un bon fonctionnement de l’audit, vous ne devez ni supprimer ni renommer le groupe de ressources ou les comptes de stockage.|
|Déployer les paramètres de diagnostic pour les groupes de sécurité réseau|Affectation de rôle|Préfixe du compte de stockage pour les diagnostics du groupe de sécurité réseau|Ce préfixe peut être combiné à l’emplacement du groupe de sécurité réseau pour former le nom de compte de stockage créé.|
|Déployer les paramètres de diagnostic pour les groupes de sécurité réseau|Affectation de rôle|Nom du groupe de ressources pour le compte de stockage pour les diagnostics du groupe de sécurité réseau (doit exister)|Groupe de ressources dans lequel le compte de stockage sera créé. Ce groupe de ressources doit déjà exister.|

## <a name="next-steps"></a>Étapes suivantes

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../how-to/update-existing-assignments.md).

---
title: Activer Azure Monitor pour machines virtuelles à l’aide d’Azure Policy
description: Décrit l’activation d’Azure Monitor pour machines virtuelles sur plusieurs groupes de machines virtuelles identiques ou plusieurs machines virtuelles Azure à l’aide d’Azure Policy.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: 240c96016304c009c36485869ac15f5f38076fb7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90088287"
---
# <a name="enable-azure-monitor-for-vms-by-using-azure-policy"></a>Activer Azure Monitor pour machines virtuelles à l’aide d’Azure Policy
Cet article explique comment activer Azure Monitor pour machines virtuelles sur des machines virtuelles Azure ou une machine virtuelle hybride connectée à Azure Arc (préversion) à l’aide d’Azure Policy. Azure Policy vous permet d’attribuer des définitions de stratégie qui installent les agents nécessaires à Azure Monitor pour machines virtuelles dans votre environnement Azure et activent automatiquement la supervision des machines virtuelles à mesure qu’elles sont créées. Azure Monitor pour machines virtuelles offre une fonctionnalité permettant de découvrir et corriger les machines virtuelles non conformes dans votre environnement. Utilisez cette fonctionnalité au lieu d’utiliser directement Azure Policy.

Si vous n’êtes pas familiarisé avec Azure Policy, consultez la brève introduction de l’article [Déployer les fonctionnalités Azure Monitor à la bonne échelle à l’aide d’Azure Policy](../deploy-scale.md).

> [!NOTE]
> Pour utiliser Azure Policy avec des groupes de machines virtuelles identiques Azure ou pour utiliser directement Azure Policy pour activer des machines virtuelles Azure, consultez [Déployer les fonctionnalités Azure Monitor à la bonne échelle à l’aide d’Azure Policy](../deploy-scale.md#azure-monitor-for-vms).

## <a name="prerequisites"></a>Prérequis
- [Créer et configurer un espace de travail Log Analytics](vminsights-configure-workspace.md)
- Consulter [Systèmes d’exploitation pris en charge](vminsights-enable-overview.md#supported-operating-systems) pour vous assurer que le système d’exploitation de la machine virtuelle ou du groupe de machines virtuelles identiques que vous activez est pris en charge 


## <a name="azure-monitor-for-vms-initiative"></a>Initiative Azure Monitor pour machines virtuelles
Azure Monitor pour machines virtuelles fournit des définitions de stratégie intégrées pour l’installation de l’agent Log Analytics et de l’agent de dépendances sur les machines virtuelles Azure. L’initiative **Activer Azure Monitor pour machines virtuelles** inclut chacune de ces définitions de stratégie. Attribuez cette initiative à un groupe d’administration, un abonnement ou un groupe de ressources pour installer automatiquement les agents sur toutes les machines virtuelles Windows ou Linux de cette étendue.

## <a name="open-policy-coverage-feature"></a>Accéder à la fonctionnalité de couverture de stratégie
Pour accéder à la **couverture de la stratégie Azure Monitor pour machines virtuelles**, sélectionnez **Machines virtuelles** dans le menu **Azure Monitor** du portail Azure. Sélectionnez **Autres options d’intégration**, puis **Activer** sous **Activer l’utilisation de la stratégie**.

[![Onglet Prise en main d’Azure Monitor pour machines virtuelles](./media/vminsights-enable-at-scale-policy/get-started-page.png)](./media/vminsights-enable-at-scale-policy/get-started-page.png#lightbox)

## <a name="create-new-assignment"></a>Créer une attribution
Si vous ne disposez pas encore d’une attribution, créez-en une en cliquant sur **Assigner une stratégie**.

[![Créer une attribution](media/vminsights-enable-at-scale-policy/create-assignment.png)](media/vminsights-enable-at-scale-policy/create-assignment.png#lightbox)

Cette page est identique à celle permettant d’attribuer une initiative dans Azure Policy, à ceci près qu’elle est codée en dur avec l’étendue que vous avez sélectionnée et la définition d’initiative **Activer Azure Monitor pour machines virtuelles**. Si vous le souhaitez, vous pouvez modifier le champ **Nom de l’attribution** et remplir la zone **Description**. Sélectionnez **Exclusions** si vous souhaitez spécifier une exclusion pour l’étendue. Par exemple, si votre étendue est un groupe d’administration, vous pouvez spécifier un abonnement de ce groupe d’administration à exclure de l’attribution.

[![Attribuer l’initiative](media/vminsights-enable-at-scale-policy/assign-initiative.png)](media/vminsights-enable-at-scale-policy/assign-initiative.png#lightbox)

Dans la page **Paramètres**, sous **Espace de travail Log Analytics**, sélectionnez un espace de travail Log Analytics qui sera utilisé par toutes les machines virtuelles de l’attribution. Si vous souhaitez spécifier des espaces de travail différents pour différentes machines virtuelles, vous devez créer plusieurs attributions, chacune avec sa propre étendue. 

   > [!NOTE]
   > Si l’espace de travail se trouve en dehors de l’étendue de l’affectation, accordez des autorisations de *Contributeur Log Analytics* à l’ID du principal de l’affectation de stratégie. Si vous ne le faites pas, vous risquez d’obtenir un message d’échec de déploiement comme celui-ci : `The client '343de0fe-e724-46b8-b1fb-97090f7054ed' with object id '343de0fe-e724-46b8-b1fb-97090f7054ed' does not have authorization to perform action 'microsoft.operationalinsights/workspaces/read' over scope ...`

[![Espace de travail](media/vminsights-enable-at-scale-policy/assignment-workspace.png)](media/vminsights-enable-at-scale-policy/assignment-workspace.png#lightbox)

Cliquez sur **Vérifier + créer** pour vérifier les informations sur l’attribution, puis cliquez sur **Créer** pour la créer. Ne créez pas de tâche de correction à ce stade, car vous aurez probablement besoin de plusieurs tâches de correction pour activer les machines virtuelles existantes. Consultez [Corriger les résultats de conformité](#remediate-compliance-results) plus bas.

## <a name="review-compliance"></a>Vérifier la conformité
Quand l’attribution est créée, vous pouvez vérifier et gérer la couverture de l’initiative **Activer Azure Monitor pour machines virtuelles** dans vos groupes d’administration et abonnements. Vous pouvez voir combien de machines virtuelles existent dans chacun des groupes d’administration et des abonnements ainsi que leur état de conformité.

[![Page de gestion de stratégie d’Azure Monitor pour machines virtuelles](media/vminsights-enable-at-scale-policy/manage-policy-page-01.png)](media/vminsights-enable-at-scale-policy/manage-policy-page-01.png#lightbox)


Le tableau suivant décrit les informations fournies dans cette vue.

| Fonction | Description | 
|----------|-------------| 
| **Portée** | Groupe d’administration et abonnements auxquels vous avez accès ou vous avez hérité l’accès avec la possibilité de descendre dans la hiérarchie du groupe d’administration.|
| **Rôle** | Votre rôle dans l’étendue, qui peut être lecteur, propriétaire ou contributeur. Cette valeur est vide si vous avez accès à l’abonnement, mais pas au groupe d’administration auquel il appartient. Ce rôle détermine les données que vous pouvez voir et les actions que vous pouvez effectuer quant à l’attribution des stratégies ou des initiatives (propriétaire), leur modification ou l’affichage de la conformité. |
| **Nombre total de machines virtuelles** | Nombre total de machines virtuelles dans cette étendue quel que soit leur état. Pour un groupe d’administration, il s’agit de la somme des machines virtuelles imbriquées sous les abonnements ou les groupes d’administration enfants. |
| **Couverture d’attribution** | Pourcentage de machines virtuelles couvertes par l’initiative. |
| **État de l’attribution** | **Réussite** - L’agent Log Analytics et l’agent de dépendances sont déployés sur toutes les machines virtuelles dans l’étendue.<br>**Avertissement** - L’abonnement n’est pas sous un groupe d’administration.<br>**Non démarrée** - Une nouvelle attribution a été ajoutée.<br>**Verrouillage** - Vous ne disposez pas de privilèges suffisants sur le groupe d’administration.<br>**Vide** - Aucune machine virtuelle n’existe ou aucune stratégie n’est attribuée. |
| **Machines virtuelles conformes** | Nombre de machines virtuelles conformes, soit le nombre de machines virtuelles sur lesquelles l’agent Log Analytics et l’agent de dépendances sont installés. Ce champ est vide s’il n’y a aucune attribution, aucune machine virtuelle dans l’étendue ou aucune autorisation appropriée. |
| **Conformité** | La conformité globale est la somme des ressources distinctes conformes divisée par la somme de toutes les ressources distinctes. |
| **État de conformité** | **Conforme** - L’agent Log Analytics et l’agent de dépendances sont déployés sur toutes les machines virtuelles de l’étendue, ou de nouvelles machines virtuelles dans l’étendue, faisant l’objet de l’attribution, n’ont pas encore été évaluées.<br>**Non conforme** - Certaines machines virtuelles ont été évaluées, mais ne sont pas activées et nécessitent une correction.<br>**Non démarrée** - Une nouvelle attribution a été ajoutée.<br>**Verrouillage** - Vous ne disposez pas de privilèges suffisants sur le groupe d’administration.<br>**Vide** - Aucune stratégie n’est attribuée.  |


Quand vous attribuez l’initiative, l’étendue sélectionnée dans l’attribution peut être l’étendue indiquée ou un sous-ensemble de celle-ci. Par exemple, vous avez peut-être créé une attribution pour un abonnement (étendue de la stratégie) et non pas un groupe d’administration (étendue de la couverture). Dans ce cas, la valeur de **Couverture d’attribution** indique les machines virtuelles dans l’étendue d’initiative divisée par les machines virtuelles dans l’étendue de couverture. Dans un autre cas, vous pouvez avoir exclu des machines virtuelles, des groupes de ressources ou un abonnement de l’étendue de stratégie. Si la valeur est vide, elle indique que la stratégie ou l’initiative n’existe pas ou que vous n’avez pas l’autorisation. Des informations sont fournies sous **État de l’attribution**.


## <a name="remediate-compliance-results"></a>Corriger les résultats de conformité
L’initiative sera appliquée aux machines virtuelles à mesure qu’elles sont créées ou modifiées, mais ne sera pas appliquée aux machines virtuelles existantes. Si votre attribution n’est pas conforme à 100 %, créez des tâches de correction pour évaluer et activer les machines virtuelles existantes et sélectionnez **Afficher la conformité** en sélectionnant les points de suspension (...).

[![Afficher la conformité](media/vminsights-enable-at-scale-policy/view-compliance.png)](media/vminsights-enable-at-scale-policy/view-compliance.png#lightbox)

La page **Conformité** liste les attributions correspondant au filtre spécifié et indique si elles sont conformes. Cliquez sur une attribution pour voir les informations correspondantes.

[![Conformité de stratégie pour les machines virtuelles Azure](./media/vminsights-enable-at-scale-policy/policy-view-compliance.png)](./media/vminsights-enable-at-scale-policy/policy-view-compliance.png#lightbox)

La page **Conformité de l’initiative** liste les définitions de stratégie dans l’initiative et indique la conformité de chacune d’elles.

[![Détails de conformité](media/vminsights-enable-at-scale-policy/compliance-details.png)](media/vminsights-enable-at-scale-policy/compliance-details.png#lightbox)

Cliquez sur une définition de stratégie pour voir les informations correspondantes. Voici certains scénarios dans lesquels des définitions de stratégie sont indiquées comme non conformes :

* L’agent Log Analytics ou l’agent de dépendances n’est pas déployé. Créez une tâche de correction pour appliquer une atténuation.
* L’image de machine virtuelle (système d’exploitation) n’est pas identifiée dans la définition de stratégie. Les critères de la stratégie de déploiement incluent uniquement les machines virtuelles qui sont déployées à partir d’images de machine virtuelle Azure connues. Consultez la documentation pour savoir si le système d’exploitation de la machine virtuelle est pris en charge.
* Les machines virtuelles ne sont pas connectées à l’espace de travail Log Analytics spécifié. Certaines machines virtuelles de l’étendue de l’initiative sont connectées à un espace de travail Log Analytics différent de celui spécifié dans l’attribution de stratégie.

[![Détails de conformité de la stratégie](media/vminsights-enable-at-scale-policy/policy-compliance-details.png)](media/vminsights-enable-at-scale-policy/policy-compliance-details.png#lightbox)

Pour créer une tâche de correction afin d’atténuer les problèmes de conformité, cliquez sur **Créer une tâche de correction**. 

[![Nouvelle tâche de correction](media/vminsights-enable-at-scale-policy/new-remediation-task.png)](media/vminsights-enable-at-scale-policy/new-remediation-task.png#lightbox)

Cliquez sur **Corriger** pour créer la tâche de correction, puis cliquez sur **Corriger** pour la démarrer. Vous devrez probablement créer plusieurs tâches de correction (une pour chaque définition de stratégie). Vous ne pouvez pas créer de tâche de correction pour une initiative.

[![Capture d'écran du volet de correction des stratégies de Monitor | Machines virtuelles.](media/vminsights-enable-at-scale-policy/remediation.png)](media/vminsights-enable-at-scale-policy/remediation.png#lightbox)


Après l’exécution des tâches de correction, vos machines virtuelles doivent être conformes aux agents installés et activées pour Azure Monitor pour machines virtuelles. 

## <a name="next-steps"></a>Étapes suivantes

Une fois la supervision activée pour vos machines virtuelles, ces informations peuvent être analysées par Azure Monitor pour machines virtuelles. 

- Pour afficher les dépendances des applications détectées, consultez [Utilisation de la fonctionnalité Map d’Azure Monitor pour machines virtuelles dans le but de comprendre les composants d’application](vminsights-maps.md). 
- Pour identifier les goulots d’étranglement et l’utilisation globale avec les performances de votre machine virtuelle, consultez [Consulter les performances des machines virtuelles Azure](vminsights-performance.md). 

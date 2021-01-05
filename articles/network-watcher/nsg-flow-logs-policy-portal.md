---
title: 'Démarrage rapide : déployer et gérer les journaux de flux NSG à l’aide d’Azure Policy'
titleSuffix: Azure Network Watcher
description: Cet article explique comment utiliser les stratégies intégrées pour gérer le déploiement des journaux de flux NSG
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/01/2020
ms.author: damendo
ms.openlocfilehash: 54b87da73d4427234e65e406d183525d55c6c00d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94948542"
---
# <a name="quickstart-deploy-and-manage-nsg-flow-logs-using-azure-policy"></a>Démarrage rapide : Déployer et gérer les journaux de flux NSG à l’aide d’Azure Policy 

## <a name="overview"></a>Vue d’ensemble
Azure Policy aide à appliquer les normes organisationnelles et à évaluer la conformité à l’échelle. Les cas d’usage courants pour Azure Policy incluent la mise en œuvre de la gouvernance pour la cohérence des ressources, la conformité réglementaire, la sécurité, le coût et la gestion. Dans cet article, nous allons utiliser deux stratégies intégrées disponibles pour les journaux de flux NSG afin de gérer l’installation de vos journaux de flux. La première stratégie marque tous les NSG sans les journaux de flux activés. La deuxième stratégie déploie automatiquement les journaux de flux pour les NSG sans les journaux de flux activés. 

Si vous créez une stratégie Azure pour la première fois, vous pouvez lire : 
- [Vue d’ensemble d’Azure Policy](../governance/policy/overview.md) 
- [Tutoriel sur la création d’une stratégie](../governance/policy/assign-policy-portal.md#create-a-policy-assignment).


## <a name="locate-the-policies"></a>Rechercher les stratégies
1. Accédez au Portail Azure – [portal.azure.com](https://portal.azure.com) 

Accédez à la page d’Azure Policy en recherchant Stratégie dans la barre de recherche supérieure ![Page d’accueil de Stratégie](./media/network-watcher-builtin-policy/1_policy-search.png)

2. Accédez à l’onglet **Attributions** à partir du volet gauche

![onglet Attributions](./media/network-watcher-builtin-policy/2_assignments-tab.png)

3. Cliquez sur le bouton **Attribuer une stratégie** 

![Bouton Attribuer une stratégie](./media/network-watcher-builtin-policy/3_assign-policy-button.png)

4. Cliquez sur le menu à trois points sous « Définitions de stratégie » pour afficher les stratégies disponibles

5. Utilisez le filtre Type et choisissez « Intégré ». Puis recherchez « Journal de flux »

Vous devez voir les deux stratégies intégrées pour les journaux de flux ![Liste de stratégies](./media/network-watcher-builtin-policy/4_filter-for-flow-log-policies.png)

6. Choisissez la stratégie que vous souhaitez attribuer

- *« Le journal de flux doit être configuré pour chaque groupe de sécurité réseau »* est la stratégie d’audit qui signale les NSG non conformes, c’est-à-dire les NSG sans la journalisation des flux activée
- *« Déployer une ressource de journal de flux avec un groupe de sécurité réseau cible »* est la stratégie avec une action de déploiement, elle active les journaux de flux sur tous les NSG sans journaux de flux

Les instructions distinctes ci-dessous s’appliquent pour chaque stratégie.  

## <a name="audit-policy"></a>Stratégie d’audit 

### <a name="how-the-policy-works"></a>Comment fonctionne la stratégie

La stratégie vérifie tous les objets ARM existants de type « Microsoft.Network/networkSecurityGroups », c’est-à-dire qu’elle examine tous les NSG dans une étendue donnée et vérifie l’existence de journaux de flux liés à l’aide de la propriété Journaux de flux du NSG. Si la propriété n’existe pas, le NSG est marqué.

Si vous souhaitez voir la définition complète de la stratégie, accédez à l’onglet [Définitions](https://ms.portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyMenuBlade/Definitions) et recherchez « Journaux de flux » pour trouver la stratégie

### <a name="assignment"></a>Affectation

1. Renseignez les détails de votre stratégie

- Étendue : Un abonnement est le choix le plus courant. Vous pouvez également choisir un groupe d’administration ou un groupe de ressources qui vous convient.  
- Définition de stratégie : Doit être choisie comme indiqué dans la section « Rechercher les stratégies ».
- AssignmentName : Choisissez un nom descriptif 

2. Cliquez sur « Vérifier + créer » pour passer en revue votre attribution

La stratégie ne requiert aucun paramètre. Lorsque vous affectez une stratégie d’audit, vous n’avez pas besoin de renseigner les détails dans l’onglet « Correction ».  

![Révision de la stratégie d’audit](./media/network-watcher-builtin-policy/5_1_audit-policy-review.png)

### <a name="results"></a>Résultats

Pour vérifier les résultats, ouvrez l’onglet Conformité et recherchez le nom de votre attribution.
Une fois votre stratégie exécutée, vous devriez voir quelque chose de similaire à la capture d’écran suivante. Si votre stratégie n’a pas été exécutée, patientez quelques instants. 

![Résultats de la stratégie d’audit](./media/network-watcher-builtin-policy/7_1_audit-policy-results.png)

## <a name="deploy-if-not-exists-policy"></a>Stratégie de déploiement si inexistant 

### <a name="policy-structure"></a>Structure de la stratégie

La stratégie vérifie tous les objets ARM existants de type « Microsoft.Network/networkSecurityGroups », c’est-à-dire qu’elle examine tous les NSG dans une étendue donnée et vérifie l’existence de journaux de flux liés à l’aide de la propriété Journaux de flux du NSG. Si la propriété n’existe pas, la stratégie déploie un journal de flux. 

Si vous souhaitez voir la définition complète de la stratégie, accédez à l’onglet [Définitions](https://ms.portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyMenuBlade/Definitions) et recherchez « Journaux de flux » pour trouver la stratégie. 

### <a name="assignment"></a>Affectation

1. Renseignez les détails de votre stratégie

- Étendue : Un abonnement est le choix le plus courant. Vous pouvez également choisir un groupe d’administration ou un groupe de ressources qui vous convient.  
- Définition de stratégie : Doit être choisie comme indiqué dans la section « Rechercher les stratégies ».
- AssignmentName : Choisissez un nom descriptif 

2. Ajoutez des paramètres de stratégie 

Le service Azure Network Watcher est un service régional. Ces paramètres permettent d’exécuter l’action de stratégie de déploiement des journaux de flux. 
- Région NSG : Régions Azure sur lesquelles la stratégie est ciblée
- ID de stockage : ID de ressource complet du compte de stockage. Remarque : Ce compte de stockage doit se trouver dans la même région que le NSG. 
- Network Watchers RG : Nom du groupe de ressources qui contient votre ressource Network Watcher. Si vous ne l’avez pas renommé, vous pouvez entrer « NetworkWatcherRG », qui est la valeur par défaut.
- Nom de Network Watcher : Nom du service d’observateur de réseau régional. Format: NetworkWatcher_NomRégion. Exemple : NetworkWatcher_centralus. Consultez la liste complète.

![Paramètres de stratégie DINE](./media/network-watcher-builtin-policy/5_2_1_dine-policy-details-alt.png)

3. Ajoutez des détails de la correction

- Activez la case à cocher « Créer une tâche de correction » si vous souhaitez que la stratégie affecte les ressources existantes 
- L’option « Créer une identité managée » doit être déjà cochée
- Le même emplacement que le précédent pour votre identité managée doit être sélectionné 
- Vous aurez besoin de l’autorisation Contributeur ou Propriétaire pour utiliser cette stratégie. Si vous disposez de ces autorisations, vous ne devriez pas voir d’erreurs.

![Correction de stratégie DINE](./media/network-watcher-builtin-policy/5_2_2_dine-remediation.png) 

4. Cliquez sur « Vérifier + créer » pour passer en revue votre attribution. Vous devriez voir un résultat similaire à la capture d’écran suivante.

![Révision de la stratégie DINE](./media/network-watcher-builtin-policy/5_2_3_dine-review.png) 


### <a name="results"></a>Résultats

Pour vérifier les résultats, ouvrez l’onglet Conformité et recherchez le nom de votre attribution.
Une fois votre stratégie exécutée, vous devez voir quelque chose de semblable à la capture d’écran suivante. Si votre stratégie n’a pas été exécutée, patientez quelques instants.

![Résultats de la stratégie DINE](./media/network-watcher-builtin-policy/7_2_dine-policy-results.png)  


## <a name="next-steps"></a>Étapes suivantes 

-   Utilisez ce [tutoriel](./quickstart-configure-network-security-group-flow-logs-from-arm-template.md) aller plus loin en utilisant des modèles ARM pour déployer des journaux de flux et Traffic Analytics.
-   En savoir plus sur [Network Watcher](./index.yml)
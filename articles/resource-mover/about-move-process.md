---
title: À propos du processus de déplacement dans Azure Resource Mover
description: Découvrir le processus de déplacement des ressources d’une région à l’autre avec Azure Resource Mover
author: rayne-wiselman
manager: evansma
ms.service: resource-move
ms.topic: overview
ms.date: 09/09/2020
ms.author: raynew
ms.openlocfilehash: 5261904dd1ee7f280209015d8f756a055dfab57e
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95522940"
---
# <a name="about-the-move-process"></a>À propos du processus de déplacement

[Azure Resource Mover](overview.md) vous permet de déplacer des ressources Azure d’une région Azure à l’autre. Cet article résume les composants utilisés par Resource Mover et décrit le processus de déplacement. 


## <a name="components"></a>Components

Ces composants sont utilisés pendant le changement de région.

**Composant** | **Détails**
--- | ---
**Resource Mover** |  Resource Mover se coordonne avec des [fournisseurs de ressources Azure](../azure-resource-manager/management/resource-providers-and-types.md) pour orchestrer le déplacement des ressources entre les régions. Resource Mover analyse les dépendances des ressources, puis gère leur état pendant le processus de déplacement. 
**Collection de déplacement** |  Une collection de déplacement est un objet [Azure Resource Manager](../azure-resource-manager/management/overview.md).<br/><br/> Elle est créée pendant le processus de changement de région, pour chaque combinaison de régions source et cible dans un abonnement. La collection contient des métadonnées et des informations de configuration sur les ressources à déplacer.<br/><br/>Les ressources ajoutées à une collection de déplacement doivent figurer dans le même abonnement, mais peuvent se trouver dans des groupes de ressources différents. 
**Ressource de déplacement** | Lorsque vous ajoutez une ressource à une collection de déplacement, elle est suivie par Resource Mover en tant que ressource de déplacement.<br/><br/> Resource Mover conserve des informations sur toutes les ressources de déplacement dans la collection de déplacement, puis gère une relation un-à-un entre la ressource source et la ressource cible. 
**Dépendances** | Resource Mover valide les ressources que vous ajoutez à une collection, puis vérifie si ces ressources ont des dépendances qui ne sont pas dans la collection de déplacement.<br/><br/> Après avoir identifié les dépendances d’une ressource, vous pouvez soit les ajouter à la collection de déplacement pour les déplacer également, soit sélectionner d’autres ressources existantes dans la région cible. Toutes les dépendances doivent être résolues avant de commencer le déplacement. 



## <a name="move-region-process"></a>Processus de changement de région 

![Diagramme montrant les étapes du déplacement](./media/about-move-process/move-steps.png)

Chaque ressource de déplacement passe par les étapes récapitulées ici.

**Étape** | **Détails** | **État/Problèmes**
--- | --- | --- 
**Étape 1 : Sélectionner des ressources** | Sélectionnez une ressource. La ressource est ajoutée à la collection de déplacement. | L’état de la ressource passe à *Préparation en attente*.<br/><br/> Si la ressource a des dépendances, *Valider la dépendance* indique que vous avez besoin d’ajouter des ressources dépendantes à la collection de déplacement.
**Étape 2 : Valider les dépendances** | Lancez le processus de validation.<br/><br/> Si la validation indique que des ressources dépendantes sont en attente, ajoutez-les à la collection de déplacement. <br/><br/> Ajoutez toutes les ressources dépendantes, même si vous ne voulez pas les déplacer. Vous pouvez spécifier ultérieurement que les ressources que vous déplacez doivent plutôt utiliser des ressources différentes dans la région cible.<br/><br/> Vous revalidez jusqu’à ne plus avoir de dépendances en suspens. | Une fois que toutes les dépendances sont ajoutées et que la validation aboutit, l’état des ressources passe à *Préparation en attente* et aucun problème ne subsiste.
**Étape 3: Préparer** | Lancez le processus de préparation. Les étapes de préparation dépendent des ressources que vous déplacez :<br/><br/> - **Ressources sans état** : Les ressources sans état contiennent uniquement des informations de configuration. Elles n’ont alors pas besoin de réplication continue des données pour être déplacées. Ce sont, par exemple, des réseaux virtuels Azure, des cartes réseau, des équilibreurs de charge et des groupes de sécurité réseau. Pour ce type de ressource, le processus de préparation génère un modèle Azure Resource Manager.<br/><br/> - **Ressources avec état** : Les ressources avec état contiennent à la fois des informations de configuration et des données qui ont besoin d’être déplacées. Il peut s’agir, par exemple, de machines virtuelles Azure et de bases de données Azure SQL. Le processus de préparation diffère pour chaque ressource. Il peut inclure la réplication de la ressource source dans la région cible. | Le lancement de cette étape fait passer l’état des ressources à *Préparation en cours*.<br/><br/> Une fois la préparation terminée, l’état des ressources passe à *Lancement du déplacement en attente* et aucun problème ne subsiste.<br/><br/> Si le processus échoue, l’état passe à *Échec de la préparation*.
**Étape 4 : Lancer le déplacement** | Lancez le processus de déplacement. La méthode de déplacement dépend du type de ressource :<br/><br/> - **Sans état** : En général, pour les ressources sans état, le processus de déplacement déploie un modèle importé sur la région cible. Ce modèle se base sur les paramètres de la ressource source, ainsi que sur toutes les éventuelles modifications manuelles que vous apportez aux paramètres cibles.<br/><br/> - **Avec état** : Pour les ressources avec état, le processus de déplacement peut impliquer de créer la ressource, ou d’en activer une copie, dans la région cible.<br/><br/>  Pour les ressources avec état uniquement, le lancement d’un déplacement peut entraîner un temps d’arrêt des ressources sources, comme des machines virtuelles et SQL. | Le lancement du déplacement fait passer l’état à *Lancement du déplacement en cours*.<br/><br/> Quand le lancement du déplacement aboutit, l’état des ressources passe à *Validation du déplacement en attente* et aucun problème ne subsiste. <br/><br/> Si le processus de déplacement échoue, l’état passe à *Impossible de lancer le déplacement*.
**Étape 5, option 1 : Abandonner le déplacement** | Après le déplacement initial, vous pouvez décider de poursuivre avec un déplacement complet. Si vous ne voulez pas continuer, vous pouvez abandonner le déplacement. Resource Mover supprime alors les ressources créées dans la cible. Le processus de réplication des ressources avec état se poursuit après le processus d’abandon. Cette option s’avère utile à des fins de test. | L’abandon des ressources fait passer l’état à *Abandon en cours*.<br/><br/> Si l’abandon aboutit, l’état passe à *Lancement du déplacement en attente* et aucun problème ne subsiste.<br/><br/> Si l’abandon échoue, l’état passe à *Impossible d’abandonner le déplacement*. 
**Étape 5, option 2 : Valider le déplacement** | Après le déplacement initial, si vous voulez poursuivre avec un déplacement complet, vous vérifiez les ressources dans la région cible et, quand vous êtes prêt, vous validez le déplacement.<br/><br/> Pour les ressources avec état uniquement, la validation peut entraîner l’inaccessibilité de ressources sources comme des machines virtuelles ou SQL. | Si vous validez le déplacement, l’état des ressources passe à *Validation du déplacement en cours**.<br/><br/> Si la validation aboutit, l’état des ressources indique *Validation du déplacement terminée* et aucun problème ne subsiste.<br/><br/> Si la validation échoue, l’état passe à *Impossible de valider le déplacement*.
**Étape 6 : Supprimer la source** | Après avoir validé le déplacement et vérifié les ressources dans la région cible, vous pouvez supprimer la ressource source. | Après la validation du déplacement, l’état des ressources passe à *Suppression de la source en attente*.


## <a name="move-region-states"></a>États du changement de région

Le processus de déplacement comporte plusieurs états, ainsi que des problèmes pouvant survenir pendant chacun d’eux. Ces états sont récapitulés dans l’organigramme.

![Organigramme des états et problèmes possibles](./media/about-move-process/process.png)


### <a name="dependency-analysis"></a>Analyse des dépendances

Au fur et à mesure que vous avancez dans le processus de déplacement, vous pouvez être invité à valider des dépendances dans les cas suivants :
- Une ressource utilise des ressources dépendantes qui ne sont pas dans la collection de déplacement.
- Une ressource dépendante dans la collection de déplacement a ses propres dépendances qui ne sont pas dans la collection de déplacement.
- Vous avez modifié des paramètres cibles de la ressource et vous devez revalider les dépendances.


### <a name="remove-resources"></a>Supprimer des ressources

Si vous ne voulez pas déplacer une ressource, vous pouvez la supprimer de la collection de déplacement. En général, la ressource est alors supprimée de la collection, ainsi que les actions ou objets associés, comme la réplication ou les modèles stockés. Ce qui se produit exactement quand vous supprimez une ressource dépend de son type et de son état au moment de la suppression. [En savoir plus](remove-move-resources.md)

## <a name="move-impact"></a>Impact du déplacement

Le tableau récapitule les effets d’un changement de région.

**Comportement** | **Entre les régions**
--- | --- | --- 
**Données** | Les données et métadonnées de ressource sont déplacées.<br/><br/> Les métadonnées sont stockées temporairement pour suivre l’état des dépendances et opérations des ressources.
**Ressource** | La ressource source reste intacte pour veiller à ce que les applications continuent de fonctionner. Vous pouvez éventuellement la supprimer après le déplacement.<br/><br/> Une ressource est créée dans la région cible.
Processus de déplacement | Processus en plusieurs étapes nécessitant une intervention manuelle et une supervision.
Test | Il est important de tester le déplacement car les applications doivent continuer à fonctionner comme prévu dans la région cible, après le déplacement.
**Temps d’arrêt** : |  Aucune perte de données n’est attendue, mais un temps d’arrêt est nécessaire pour déplacer les ressources.



## <a name="next-steps"></a>Étapes suivantes

[Déplacez](tutorial-move-region-virtual-machines.md) des machines virtuelles Azure vers une autre région.
[Déplacez](tutorial-move-region-sql.md) des ressources Azure SQL vers une autre région.
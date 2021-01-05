---
title: Déplacer des ressources Azure vers une autre région
description: Fournit une vue d’ensemble du déplacement des ressources Azure entre les régions Azure.
author: rayne-wiselman
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 09/10/2020
ms.author: raynew
ms.openlocfilehash: 7a71502ec361004079e0962d8bc6433316a4ba81
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90007636"
---
# <a name="moving-azure-resources-across-regions"></a>Déplacement des ressources Azure entre les régions

Cet article fournit des informations sur le déplacement des ressources Azure entre les régions Azure.

Les zones géographiques, les régions et les zones de disponibilité Azure constituent la base de l’infrastructure mondiale d’Azure. Les [zones géographiques](https://azure.microsoft.com/global-infrastructure/geographies/) d’Azure contiennent généralement au moins deux [régions Azure](https://azure.microsoft.com/global-infrastructure/regions/). Une région est une zone géographique contenant des zones de disponibilité et plusieurs centres de données. 

Une fois les ressources déployées dans une région Azure spécifique, il existe plusieurs raisons pour lesquelles vous pouvez souhaiter déplacer des ressources vers une autre région.

- **Aligner sur un lancement d’une région** : Déplacer vos ressources vers une région Azure récemment introduite qui n’était pas disponible auparavant.
- **Aligner en raison des services/fonctionnalités** : Déplacer des ressources pour profiter des services ou fonctionnalités qui sont disponibles dans une région spécifique.
- **Répondre au développement de l’activité** : Déplacer des ressources vers une région en réponse à des changements de l’activité, comme des fusions ou des acquisitions.
- **Aligner pour cause de proximité** : Déplacer des ressources vers une région proche de votre entreprise.
- **Respecter les exigences relatives aux données** : Déplacer des ressources afin de respecter les exigences de résidence des données ou les besoins en matière de classification des données. [Plus d’informations](https://azure.microsoft.com/mediahandler/files/resourcefiles/achieving-compliant-data-residency-and-security-with-azure/Achieving_Compliant_Data_Residency_and_Security_with_Azure.pdf)
- **Respecter les exigences de déploiement** : Déplacer des ressources qui ont été déployées par erreur ou déplacez-les en réponse aux besoins de capacité. 
- **Respecter la désaffectation** : Déplacer des ressources en raison de la désaffectation des régions.

## <a name="move-resources-with-resource-mover"></a>Déplacer des ressources avec Resource Mover

Vous pouvez déplacer des ressources vers une autre région avec [Azure Resource Mover](../../resource-mover/overview.md). Resource Mover fournit :

- Un hub unique pour déplacer des ressources entre les régions.
- Une réduction de la complexité et du temps nécessaire aux déplacements. Tout ce dont vous avez besoin se trouve à un même emplacement.
- Une expérience simple et cohérente pour le déplacement des différents types de ressources Azure.
- Un moyen simple d’identifier les dépendances des ressources que vous voulez déplacer. Ceci vous permet de déplacer des ressources associées ensemble, pour que tout fonctionne comme prévu dans la région cible après le déplacement.
- Le nettoyage automatique des ressources dans la région source, si vous voulez les supprimer après le déplacement.
- Tests. Vous pouvez essayer un déplacement, puis l’annuler si vous ne voulez pas effectuer un déplacement complet.

Vous pouvez déplacer des ressources vers une autre région en utilisant deux méthodes différentes :

- **Commencer le déplacement des ressources à partir d’un groupe de ressources** : Avec cette méthode, vous commencez le changement de région à partir d’un groupe de ressources. Une fois que vous avez sélectionné les ressources que vous voulez déplacer, le processus continue dans le hub Resource Mover, où a lieu la vérification des dépendances des ressources et l’orchestration du processus de déplacement. [Plus d’informations](../../resource-mover/move-region-within-resource-group.md)
- **Commencer le déplacement des ressources directement à partir du hub Resource Mover** : Avec cette méthode, vous commencez le changement de région directement dans le hub. [Plus d’informations](../../resource-mover/tutorial-move-region-virtual-machines.md)


## <a name="support-for-region-move"></a>Prise en charge du changement de région

Actuellement, vous pouvez utiliser Resource Mover pour déplacer ces ressources vers une autre région :

- Machines virtuelles Azure et disques associés
- Cartes réseau
- Groupes à haute disponibilité
- Réseaux virtuels Azure
- Adresses IP publiques
- Groupes de sécurité réseau (NSG)
- Équilibreurs de charge internes et publics
- Bases de données et pools élastiques Azure SQL

## <a name="region-move-process"></a>Processus de changement de région

Le processus réel de déplacement des ressources entre les régions dépend des ressources que vous déplacez. Toutefois, il existe quelques étapes clés communes :

1. **Vérifier les prérequis** : Les prérequis incluent de s’assurer que les ressources dont vous avez besoin dont disponibles dans la région source, vérifier que votre quota est suffisant et vérifier que votre abonnement peut accéder à la région cible.
2. **Analyser les dépendances** : Vos ressources peuvent avoir des dépendances avec d’autres ressources. Avant le déplacement, déterminez les dépendances afin que les ressources déplacées continuent à fonctionner comme prévu après le déplacement.
3. **Préparer le déplacement** : Les étapes à suivre dans votre région principale avant le déplacement. Par exemple, vous devrez peut-être exporter un modèle Azure Resource Manager ou démarrer la réplication des ressources de la source vers la cible.
4. **Déplacer les ressources** : Le mode de déplacement des ressources dépend de ce qu’elles sont. Vous devrez peut-être déployer un modèle dans la région cible ou basculer les ressources sur la cible.
5. **Abandonner les ressources cibles** : Après avoir déplacé des ressources, vous souhaiterez peut-être examiner les ressources qui sont maintenant dans la région cible et décider si vous en avez besoin ou pas.
6. **Valider le déplacement** : Après avoir vérifié les ressources dans la région cible, certaines ressources peuvent nécessiter une action de validation finale. Par exemple, dans une région cible qui est désormais la région principale, vous devrez peut-être configurer la récupération d’urgence vers une nouvelle région secondaire. 
7. **Nettoyer la source** : Enfin, une fois que tout est en cours d’exécution dans la nouvelle région, vous pouvez nettoyer et désactiver les ressources que vous avez créées pour le déplacement et les ressources dans votre région principale.



## <a name="next-steps"></a>Étapes suivantes

[Découvrez plus d’informations](../../resource-mover/about-move-process.md) sur le processus de déplacement dans Resource Mover.

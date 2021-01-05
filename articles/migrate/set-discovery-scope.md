---
title: Définir l’étendue de la découverte de machines virtuelles VMware avec Azure Migrate
description: Décrit comment définir l’étendue de la découverte pour l’évaluation et la migration de machines virtuelles VMware avec Azure Migrate.
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: how-to
ms.date: 06/09/2020
ms.openlocfilehash: dc5bd178c837deea7a22fb3be5ba438085c0e748
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753550"
---
# <a name="set-discovery-scope-for-vmware-vms"></a>Définir l’étendue de la découverte des machines virtuelles VMware

Cet article explique comment limiter l’étendue des machines virtuelles VMware que vous découvrez quand vous :

- Découvrez des machines virtuelles à l’aide de l’[appliance Azure Migrate](migrate-appliance-architecture.md) quand vous utilisez l’outil Azure Migrate : Server Assessment.
- Découvrez des machines virtuelles à l’aide de l’[appliance Azure Migrate](migrate-appliance-architecture.md) quand vous utilisez l’outil Azure Migrate : Server Migration, pour la migration sans agent des machines virtuelles VMware vers Azure.

Quand vous configurez l’appliance, elle se connecte à vCenter Server et démarre la découverte. Avant de connecter l’appliance à vCenter Server, vous pouvez définir l’étendue de la découverte pour limiter la détection à des centres de donnés vCenter Server, des clusters, un dossier de clusters, des hôtes, un dossier d’hôtes ou des machines virtuelles individuelles. Pour définir l’étendue, vous affectez des autorisations sur le compte utilisé par l’appliance pour accéder à vCenter Server.

## <a name="before-you-start"></a>Avant de commencer

Si vous n’avez pas configuré de compte d’utilisateur vCenter utilisé par Azure Migrate pour la découverte, faites-le maintenant pour l’[évaluation](./tutorial-discover-vmware.md#prepare-vmware) ou la [migration sans agent](./migrate-support-matrix-vmware-migration.md#agentless-migration).


## <a name="assign-permissions-and-roles"></a>Affecter des autorisations et des rôles

Vous pouvez affecter des autorisations sur les objets d’inventaire VMware en appliquant l’une des deux méthodes suivantes :

- Sur le compte utilisé par l’appliance, attribuez un rôle avec les autorisations requises sur les objets à inclure dans l’étendue.
- Vous pouvez également attribuer un rôle au compte au niveau du centre de données et le propager aux objets enfants. Attribuez ensuite au compte un rôle **Aucun accès** pour chaque objet que vous ne voulez pas inclure dans l’étendue. Nous ne recommandons pas cette approche, car elle est fastidieuse et peut exposer des contrôles d’accès, chaque nouvel objet enfant créé héritant automatiquement de l’accès de son parent.

Vous ne pouvez pas limiter la découverte d’inventaire au niveau du dossier de machine virtuelle vCenter. Si vous devez limiter la découverte aux machines virtuelles contenues dans un dossier de machine virtuelle, créez un utilisateur et accordez l’accès individuellement à chaque machine virtuelle requise. Les dossiers d’hôtes et de clusters sont pris en charge.


### <a name="assign-a-role-for-assessment"></a>Attribuer un rôle pour l’évaluation

1. Sur le compte vCenter d’appliance que vous utilisez pour la découverte, appliquez le rôle **Lecture seule** pour tous les objets parents qui hébergent des machines virtuelles que vous souhaitez découvrir et évaluer (hôte, cluster, dossier d’hôtes, dossier de clusters, jusqu’au centre de données).
2. Propagez ces autorisations aux objets enfants dans la hiérarchie.

    ![Affecter des autorisations](./media/tutorial-assess-vmware/assign-perms.png)

### <a name="assign-a-role-for-agentless-migration"></a>Attribuer un rôle pour la migration sans agent

1. Sur le compte vCenter d’appliance que vous utilisez pour la migration, appliquez un rôle défini par l’utilisateur qui dispose des [autorisations nécessaires](migrate-support-matrix-vmware-migration.md#vmware-requirements-agentless) sur tous les objets parents qui hébergent les machines virtuelles que vous souhaitez découvrir et migrer.
2. Vous pouvez donner au rôle un nom plus facile à identifier, par exemple <em>Azure_Migrate</em>.

## <a name="work-around-vm-folder-restriction"></a>Contourner la restriction relative aux dossiers de machines virtuelles

Actuellement, l’outil Server Assessment ne peut pas découvrir de machine virtuelle si l’accès est accordé au niveau du dossier de machines virtuelles vCenter. Si vous souhaitez limiter votre découverte et votre évaluation à des dossiers de machines virtuelles spécifiques, utilisez cette solution de contournement.

1. Affectez des autorisations de lecture seule sur toutes les machines virtuelles situées dans les dossiers de machines virtuelles auxquelles vous voulez limiter la découverte et l’évaluation.
2. Accordez un accès en lecture seule à tous les objets parents qui hébergent l’hôte de machines virtuelles, le cluster, le dossier d’hôtes, le dossier de clusters, jusqu’au centre de donnes. Vous n’avez pas besoin de propager les autorisations à tous les objets enfants.
3. Pour utiliser les informations d’identification pour la découverte, sélectionnez le centre de données en tant qu’**Étendue de la collecte**.


La configuration du contrôle d’accès en fonction du rôle garantit que le compte d’utilisateur vCenter correspondant a accès seulement aux machines virtuelles spécifiques d’un locataire.


## <a name="next-steps"></a>Étapes suivantes

[Configurer l’appliance](how-to-set-up-appliance-vmware.md)
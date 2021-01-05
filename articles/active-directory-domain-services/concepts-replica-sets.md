---
title: Concepts des jeux de réplicas pour Azure AD Domain Services | Microsoft Docs
description: Découvrez les jeux de réplicas qui se trouvent dans Azure Active Directory Domain Services et la façon dont ils assurent la redondance des applications qui requièrent des services d’identité.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/16/2020
ms.author: justinha
ms.openlocfilehash: 5359a955ea97b559b7e3d244bfb6c4fb09e8681b
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620033"
---
# <a name="replica-sets-concepts-and-features-for-azure-active-directory-domain-services-preview"></a>Concepts et fonctionnalités des jeux de réplicas pour Azure Active Directory Domain Services (préversion)

Lorsque vous créez un domaine managé Azure Active Directory Domain Services (Azure AD DS), vous définissez un espace de noms unique. Cet espace de noms est le nom de domaine, par exemple *aaddscontoso.com*. Deux contrôleurs de domaine sont ensuite déployés dans la région Azure sélectionnée. Ce déploiement de contrôleurs de domaine est appelé jeu de réplicas.

Vous pouvez étendre un domaine managé pour avoir plusieurs jeux de réplicas par locataire Azure AD. Les jeux de réplicas peuvent être ajoutés à n’importe quel réseau virtuel appairé dans toute région Azure prenant en charge Azure AD DS. D’autres jeux de réplicas dans des régions Azure différentes assurent la récupération d’urgence géographique pour les applications héritées si une région Azure est mise hors connexion.

Les jeux de réplicas sont actuellement en préversion.

> [!NOTE]
> Les jeux de réplicas ne vous permettent pas de déployer plusieurs domaines managés uniques dans un seul locataire Azure. Chaque jeu de réplicas contient les mêmes données.

## <a name="how-replica-sets-work"></a>Fonctionnement des jeux de réplicas

Quand vous créez un domaine managé, tel que *aaddscontoso.com*, un jeu de réplicas initial est créé. Les jeux de réplicas supplémentaires partagent le même espace de noms et la même configuration. Les modifications apportées à Azure AD DS, notamment en ce qui concerne la configuration, l’identité de l’utilisateur et les informations d’identification, les groupes, les objets de stratégie de groupe, les objets ordinateur et autres modifications, sont appliquées à tous les jeux de réplicas du domaine managé via la réplication AD DS.

Vous créez chaque jeu de réplicas dans un réseau virtuel. Chaque réseau virtuel doit être appairé à tous les autres réseaux virtuels qui hébergent le jeu de réplicas d’un domaine managé. Cette configuration crée une topologie de réseau maillé qui prend en charge la réplication de répertoire. Un réseau virtuel peut prendre en charge plusieurs jeux de réplicas, à condition que chaque jeu de réplicas soit dans un sous-réseau virtuel différent.

Tous les jeux de réplicas sont placés dans le même site Active Directory. Par conséquent, toutes les modifications sont propagées à l’aide de la réplication intrasite pour une convergence rapide.

> [!NOTE]
> Il n’est pas possible de définir des sites distincts ni de définir des paramètres de réplication entre des jeux de réplicas.

Le diagramme suivant montre un domaine managé avec deux jeux de réplicas. Le premier jeu de réplicas est créé avec l’espace de noms du domaine. Un deuxième jeu de réplicas est créé après cela :

![Diagramme d’un exemple de domaine managé avec deux jeux de réplicas](./media/concepts-replica-sets/two-replica-set-example.png)

> [!NOTE]
> Les jeux de réplicas garantissent la disponibilité des services d’authentification dans les régions où un jeu de réplicas est configuré. Pour qu’une application ait une redondance géographique en cas de panne régionale, la plateforme d’application qui s’appuie sur le domaine managé doit également résider dans l’autre région.
>
> La résilience des autres services requis pour que l’application fonctionne, comme Machines virtuelles Azure ou Azure App Service, n’est pas fournie par les jeux de réplicas. La conception de la disponibilité d’autres composants d’application doit tenir compte des caractéristiques de résilience des services qui composent l’application.

L’exemple suivant montre un domaine managé avec trois jeux de réplicas afin de renforcer la résilience et de garantir la disponibilité des services d’authentification. Dans les deux exemples, les charges de travail des applications se trouvent dans la même région que le jeu de réplicas du domaine managé :

![Diagramme d’un exemple de domaine managé avec trois jeux de réplicas](./media/concepts-replica-sets/three-replica-set-example.png)

## <a name="deployment-considerations"></a>Points à prendre en considération pour le déploiement

Le niveau tarifaire par défaut pour un domaine managé est le SKU *Entreprise*, qui prend en charge plusieurs jeux de réplicas. Pour créer des jeux de réplicas supplémentaires si vous êtes passé au niveau tarifaire *Standard*, [mettez à niveau le domaine managé](change-sku.md) vers *Entreprise* ou *Premium*.

Le nombre maximal de jeux de réplicas pris en charge pendant la préversion est de quatre, y compris le premier réplica créé lorsque vous avez créé le domaine managé.

La facturation de chaque jeu de réplicas est basée sur le niveau tarifaire de la configuration du domaine. Par exemple, si vous avez un domaine managé qui utilise le niveau tarifaire *Entreprise* et que vous avez trois jeux de réplicas, votre abonnement est facturé à l’heure pour chacun des trois jeux de réplicas.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="can-i-use-my-production-managed-domain-with-this-preview"></a>Puis-je utiliser mon domaine managé de production avec cette préversion ?

Les jeux de réplicas sont une fonctionnalité d’évaluation publique d’Azure AD Domain Services. Vous pouvez utiliser un domaine managé de production, mais ne perdez pas de vue que les fonctionnalités toujours en préversion présentent des différences sur le plan de la prise en charge. Pour plus d’informations sur les préversions, consultez [Contrat SLA des préversions Azure Active Directory](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="can-i-create-a-replica-set-in-subscription-different-from-my-managed-domain"></a>Puis-je créer un jeu de réplicas dans un abonnement différent de celui de mon domaine managé ?

Non. Les jeux de réplicas doivent se trouver dans le même abonnement que le domaine managé.

### <a name="how-many-replica-sets-can-i-create"></a>Combien de jeux de réplicas puis-je créer ?

La préversion est limitée à un maximum de quatre jeux de réplicas : le jeu de réplicas initial pour le domaine managé, plus trois jeux de réplicas supplémentaires.

### <a name="how-does-user-and-group-information-get-synchronized-to-my-replica-sets"></a>Comment les informations d’utilisateur et de groupe sont-elles synchronisées avec mes jeux de réplicas ?

Tous les jeux de réplicas sont connectés les uns aux autres à l’aide d’un appairage de réseaux virtuels maillés. Un jeu de réplicas reçoit les mises à jour des utilisateurs et des groupes d’Azure AD. Ces modifications sont ensuite répliquées sur les autres jeux de réplicas à l’aide de la réplication AD DS intrasite sur le réseau appairé.

À l’instar de l’instance AD DS locale, un état déconnecté prolongé peut entraîner une interruption de la réplication. Les réseaux virtuels appairés n’étant pas transitifs, les exigences de conception pour les jeux de réplicas nécessitent une topologie réseau entièrement maillée.

### <a name="how-do-i-make-changes-in-my-managed-domain-after-i-have-replica-sets"></a>Comment apporter des modifications à mon domaine managé après la création des jeux de réplicas ?

Les modifications au sein du domaine managé fonctionnent comme auparavant. Vous [créez et utilisez une machine virtuelle de gestion, avec ses outils RSAT, qui est jointe au domaine managé](tutorial-create-management-vm.md). Vous pouvez joindre autant de machines virtuelles de gestion au domaine managé que vous le souhaitez.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer à utiliser des jeux de réplicas, [créez et configurez un domaine managé Azure AD DS][tutorial-create-advanced]. Une fois le domaine déployé, [créez et utilisez des jeux de réplicas supplémentaires][create-replica-set].

<!-- LINKS - INTERNAL -->
[tutorial-create-advanced]: tutorial-create-instance-advanced.md
[create-replica-set]: tutorial-create-replica-set.md

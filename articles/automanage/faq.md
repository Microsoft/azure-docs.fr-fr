---
title: FAQ sur Azure Automanage pour machines virtuelles
description: Réponses aux questions fréquemment posées sur Azure Automanage pour machines virtuelles.
author: deanwe
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: troubleshooting
ms.date: 09/04/2020
ms.author: deanwe
ms.openlocfilehash: 89ad5c2b973a74bc9fa0f6ca2f59f92a26b666e7
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891458"
---
# <a name="frequently-asked-questions-for-azure-automanage-for-vms"></a>Forum aux questions sur Azure Automanage pour machines virtuelles

Cet article fournit des réponses à certaines des questions les plus courantes sur [Azure Automanage pour machines virtuelles](automanage-virtual-machines.md).

Si le problème lié à Azure n’est pas traité dans cet article, parcourez les forums Azure sur [MSDN et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez publier votre problème sur ces forums ou [@AzureSupport sur Twitter](https://twitter.com/AzureSupport). Vous pouvez également soumettre une demande de support Azure. Pour soumettre une demande de support, sur la [page de support Azure](https://azure.microsoft.com/support/options/), sélectionnez **Obtenir de l’aide** .


## <a name="azure-automanage-for-virtual-machines"></a>Azure Automanage pour machines virtuelles

**Quelles sont les conditions préalables requises pour activer Azure Automanage ?**

Les conditions préalables à l’activation d’Azure Automanage sont les suivantes :
- Les machines virtuelles concernées doivent être exclusivement Windows Server.
- Les machines virtuelles doivent être en cours d’exécution.
- Les machines virtuelles doivent se trouver dans une région prise en charge.
- L’utilisateur doit disposer des autorisations appropriées.
- Les machines virtuelles ne peuvent pas faire partie d’un groupe identique.
- Automanage ne prend pas en charge les abonnements sandbox à ce stade

**Quelle autorisation Azure RBAC est nécessaire pour activer Automanage ?**

Si vous activez Automanage sur une machine virtuelle avec un compte Automanage existant, vous devez disposer du rôle Contributeur pour le groupe de ressources où réside la machine virtuelle.

Si vous utilisez un nouveau compte Automanage lors de l’activation, vous devez avoir le rôle Propriétaire ou le rôle Contributeur + Administrateur de l’accès utilisateur sur l’abonnement.


**Quelles sont les régions prises en charge ?**

Les machines virtuelles prises en charge sont situées dans les régions suivantes : Europe Ouest, USA Est, USA Ouest 2, Canada Centre, USA Centre-Ouest.


**Quelles sont les fonctionnalités qu’Azure Automanage automatise ?**

Automanage inscrit, configure et surveille tout au long du cycle de vie de la machine virtuelle les services répertoriés [ici](virtual-machines-best-practices.md).

**Azure Automanage fonctionne-t-il avec les machines virtuelles avec Azure Arc ?**

Actuellement, Automanage ne prend pas en charge les machines virtuelles avec Azure Arc.

**Puis-je personnaliser les configurations sur Azure Automanage ?**

Les clients peuvent personnaliser les paramètres de services spécifiques, tels que la rétention de Sauvegarde Azure, par le biais de préférences de configuration. Pour obtenir la liste complète des paramètres qui peuvent être modifiés, consultez notre documentation [ici](virtual-machines-best-practices.md).


**Azure Automanage fonctionne-t-il avec les machines virtuelles tant Linux que Windows ?**

Actuellement, Automanage prend en charge les machines virtuelles Azure Windows Server.


**Puis-je appliquer Automanage de manière sélective uniquement à un ensemble de machines virtuelles ?**

Vous pouvez activer Automanage en un clic, en pointant simplement sur les machines virtuelles nouvelles et existantes sélectionnées. Vous pouvez également désactiver Automanage à tout moment.


**Azure Automanage prend-il en charge les machines virtuelles d’un groupe de machines virtuelles identiques ?**

Non, Azure Automanage ne prend pas actuellement en charge les machines virtuelles d’un groupe de machines virtuelles identiques.


**Combien coûte Azure Automanage ?**

Azure Automanage est disponible sans coût supplémentaire en préversion publique. En revanche, les ressources Azure associées, telles que Sauvegarde Azure occasionnent des coûts.


**Puis-je appliquer Automanage par le biais d’Azure Policy ?**

Oui, nous disposons d’une stratégie intégrée qui applique automatiquement Automanage à toutes les machines virtuelles au sein de votre étendue définie. Vous devez également spécifier le profil de configuration (DevTest ou Production) en même temps que votre compte Automanage. Apprenez-en davantage sur l’activation d’Automanage via Azure Policy [ici](virtual-machines-policy-enable.md).


**Qu’est-ce qu’un compte Automanage ?**

Le compte Automanage est une MSI (Managed Service Identity) qui fournit le contexte de sécurité ou l’identité sous laquelle les opérations automatisées sont effectuées.


**Lors de l’activation d’Automanage, des machines virtuelles supplémentaires sont-elles affectées, en plus de celles que j’ai sélectionnées ?**

Si votre machine virtuelle est liée à un espace de travail Log Analytics existant, nous réutiliserons cet espace de travail pour appliquer ces solutions : Change Tracking, Inventaire et Update Management. Ces solutions seront activées sur toutes les machines virtuelles connectées à cet espace de travail.


**Puis-je modifier le profil de configuration de ma machine virtuelle ?**

Actuellement, vous devez désactiver Automanage pour cette machine virtuelle, puis le réactiver avec le profil de configuration et les préférences souhaités.


**Si ma machine virtuelle est déjà configurée pour un service, tel Update Management, Automanage va-t-il la reconfigurer ?**
Non, Automanage ne va pas la reconfigurer. Nous allons commencer à surveiller la dérive des ressources associées à ce service.


**Pourquoi l’état de ma machine virtuelle est-il Échec dans le portail Automanage ?**

Si vous voyez l’état *Échec* , vous pouvez résoudre des problèmes de déploiement via le groupe de ressources dans lequel se trouve votre machine virtuelle. Accédez à **Groupes de ressources** , sélectionnez votre groupe de ressources, cliquez sur **Déploiements** , puis consultez l’état *Échec* contenant tous les détails de l’erreur.

**Comment puis-je obtenir une assistance pour la résolution des problèmes en lien avec Automanage ?**

Vous pouvez créer un [ticket de cas de support technique](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). Pour l’option **Service** , recherchez et sélectionnez *Automanage* dans la section *Surveillance et gestion* .


## <a name="next-steps"></a>Étapes suivantes

Essayez d’activer le service Automanage pour machines virtuelles dans le portail Azure.

> [!div class="nextstepaction"]
> [Activer le service Automanage pour machines virtuelles dans le portail Azure](quick-create-virtual-machines-portal.md)
---
title: Contrôle d’Azure HANA - Grandes instances à l’aide du portail Azure | Microsoft Docs
description: Décrit la façon dont vous pouvez identifier Azure HANA - Grandes instances et interagir avec par le biais du portail
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/15/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 550e22ac861b92994f2695594d09fc2935d273d1
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967752"
---
# <a name="azure-hana-large-instances-control-through-azure-portal"></a>Contrôle des grandes instances Azure HANA à l’aide du portail Azure
Ce document décrit la façon dont [HANA - Grandes instances](./hana-overview-architecture.md) est présenté dans le [portail Azure](https://portal.azure.com) et les activités qui peuvent être effectuées par le biais du portail Azure avec des unités de Grande instance HANA qui sont automatiquement déployées. La visibilité de la fonctionnalité HANA - Grandes instances dans le portail Azure est fournie par le biais d’un fournisseur de ressources Azure pour HANA - Grandes instances, qui est actuellement en préversion publique

## <a name="register-hana-large-instance-resource-provider"></a>Inscrire le fournisseur de ressources Grande instance HANA
Généralement, l’abonnement Azure que vous utilisez pour les déploiements HANA - Grandes instances est inscrit auprès le fournisseur de ressources HANA - Grandes instances. Toutefois, si vous ne voyez pas les unités de Grande instance HANA que vous avez déployées, vous devez inscrire le fournisseur de ressources dans votre abonnement Azure. Vous pouvez inscrire le fournisseur de ressources Grande instance HANA de deux façons.

### <a name="register-through-cli-interface"></a>Effectuer l’inscription par le biais de l’interface CLI
Vous devez être connecté à votre abonnement Azure utilisé pour le déploiement de Grande instance HANA par le biais de l’interface Azure CLI. Vous pouvez (ré)inscrire le fournisseur de Grande instance HANA avec cette commande :
    
```azurecli
az provider register --namespace Microsoft.HanaOnAzure
```

Pour plus d’informations, consultez l’article [Fournisseurs et types de ressources Azure](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-cli).


### <a name="register-through-azure-portal"></a>Effectuer l’inscription par le biais du portail Azure
Vous pouvez (ré)inscrire le fournisseur de ressources HANA - Grandes instances par le biais du portail Azure. Vous devez afficher, dans le portail Azure, votre abonnement qui a été utilisé pour déployer votre ou vos unités de Grande instance HANA, puis double-cliquer dessus. Une fois dans la page de vue d’ensemble de votre abonnement, sélectionnez « Fournisseurs de ressources », comme indiqué ci-dessous, puis tapez « HANA » dans la fenêtre de recherche. 

![Inscrire le fournisseur de ressources HLI (HANA - Grandes instances) par le biais du portail Azure](./media/hana-li-portal/portal-register-hli-rp.png)

Dans la capture d’écran affichée, le fournisseur de ressources a déjà été inscrit. Dans le cas où le fournisseur de ressources n’est pas encore inscrit, appuyez sur « Réinscrire » ou sur « Inscrire ».

Pour plus d’informations, consultez l’article [Fournisseurs et types de ressources Azure](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-powershell).


## <a name="display-of-hana-large-instance-units-in-the-azure-portal"></a>Affichage d’unités de Grande instance HANA dans le portail Azure
Quand vous soumettez une demande de déploiement de Grande instance HANA, vous êtes invité à spécifier l’abonnement Azure auquel vous vous connectez également pour HANA - Grandes instances. Il est recommandé d’utiliser le même abonnement que celui que vous utilisez pour déployer la couche Application SAP qui fonctionne sur les unités de Grande instance HANA.
Lors du déploiement de votre première unités de Grande instance HANA, un [groupe de ressources Azure](../../../azure-resource-manager/management/manage-resources-portal.md) est créé dans l’abonnement Azure que vous avez soumis dans la demande de déploiement pour votre ou vos Grandes instances HANA.  Le nouveau groupe de ressources liste toutes vos unités de Grande instance HANA que vous avez déployées dans l’abonnement spécifique.

Afin de trouver le groupe de ressources Azure, vous affichez la liste des groupes de ressources de votre abonnement en parcourant le volet de navigation de gauche du portail Azure

![Capture d’écran de l’option Groupes de ressources](./media/hana-li-portal/portal-resource-group.png)

Vous êtes répertorié dans la liste des groupes de ressources. Vous devrez peut-être filtrer sur l’abonnement que vous avez utilisé pour déployer HANA - Grandes instances.

![Filtrer des groupes de ressources dans le portail Azure](./media/hana-li-portal/portal-filtering-subscription.png)

Après avoir filtré pour afficher l’abonnement approprié, il est possible que vous ayez malgré tout une longue liste de groupes de ressources. Recherchez celui ayant un suffixe **-Txxx** où « xxx » représentent trois chiffres ; par exemple, **-T050**. 

Une fois que vous avez trouvé le groupe de ressources, affichez la liste des détails le concernant. La liste que vous recevez peut ressembler à ceci :

![Liste d’unités HLI (HANA - Grandes instances) dans le portail Azure](./media/hana-li-portal/portal-hli-units-list.png)

Toutes les unités listées représentent une unité de Grande instance HANA unique qui a été déployée dans votre abonnement. Dans le cas présent, vous examinez huit unités de Grande instance HANA différentes qui ont été déployées dans votre abonnement.

Si vous avez déployé plusieurs locataires de Grande instance HANA dans le même abonnement Azure, plusieurs groupes de ressources Azure sont affichés. 


## <a name="look-at-attributes-of-single-hli-unit"></a>Examen des attributs d’une unité HLI unique
Dans la liste des unités de Grande instance HANA, vous pouvez cliquer sur une unité précise et obtenir les détails la concernant. 

Dans l’écran de vue d’ensemble, après avoir cliqué sur « Afficher plus »vous obtenez une présentation de l’unité, qui ressemble à ceci :

![Afficher la vue d’ensemble d’une unité HLI](./media/hana-li-portal/portal-show-overview.png)

Si vous examinez les différents attributs affichés, vous remarquez qu’ils ne semblent pas vraiment différents des attributs de machine virtuelle Azure. Sur le côté gauche de l’en-tête sont affichés le groupe de ressources, la région Azure, le nom de l’abonnement et son ID, ainsi que certaines étiquettes que vous avez ajoutées. Par défaut, aucune étiquette n’est attribuée aux unités de Grande instance HANA. Sur le côté droit de l’en-tête est affiché le nom de l’unité tel qu’affecté au moment du déploiement. Le système d’exploitation est indiqué, ainsi que l’adresse IP. Comme avec les machines virtuelles, le type de l’unité de Grande instance HANA avec le nombre de threads de processeur et la mémoire est également affiché. Des détails supplémentaires sur les différentes unités de Grande instance HANA sont indiqués ici :

- [Références (SKU) disponibles pour HLI](./hana-available-skus.md)
- [Architecture de stockage de SAP HANA (Grandes instances)](./hana-storage-architecture.md) 

Les données supplémentaires sur le côté inférieur droit sont la révision du tampon de grande instance HANA. Les valeurs possibles sont les suivantes :

- Révision 3
- Révision 4

La révision 4 est la dernière architecture des grandes instances HANA avec des améliorations majeures de la latence du réseau entre les machines virtuelles Azure et les unités de grande instance HANA déployées dans les tampons ou les lignes de révision 4.
Vous trouverez d’autres informations très importantes dans le coin inférieur droit de la vue d’ensemble avec le nom du groupe de placement de proximité Azure créé automatiquement pour chaque unité de grande instance HANA déployée. Ce groupe de placement de proximité doit être référencé lors du déploiement des machines virtuelles Azure qui hébergent la couche application SAP. En utilisant le [groupe de placement de proximité Azure](../../linux/co-location.md) associé à l’unité de grande instance HANA, vous vous assurez que les machines virtuelles Azure sont déployées à proximité de l’unité de grande instance HANA. La méthode d’utilisation des groupes de placement de proximité pour localiser la couche d’application SAP dans le même centre de données Azure que les unités de grande instance HANA hébergées en version 4 est décrite dans la section [Groupes de placement de proximité Azure pour une latence réseau optimale avec les applications SAP](sap-proximity-placement-scenarios.md).

Un champ supplémentaire dans la colonne de droite de l’en-tête indique l’état d’alimentation de l’unité de Grande instance HANA.

> [!NOTE]
> L’état d’alimentation indique si l’unité matérielle est sous tension ou hors tension. Il ne donne pas d’informations indiquant si le système d’exploitation est opérationnel. Quand vous redémarrez une unité de Grande instance HANA, un peu de temps s’écoule pendant lequel l’état de l’unité devient **Démarrage en cours** pour passer à l’état **Démarré**. L’état **Démarré** signifie que le système d’exploitation est en train de démarrer ou que son démarrage est complètement terminé. Par conséquent, après un redémarrage de l’unité, vous ne pouvez pas espérer vous connecter à l’unité dès que l’état bascule vers **Démarré**.
> 

Si vous sélectionnez « En savoir plus », des informations supplémentaires s’affichent. Ces informations supplémentaires contiennent la révision du tampon de grande instance HANA, dans laquelle l’unité a été déployée. Pour découvrir les différentes révisions des tampons de grande instance HANA, veuillez consulter l’article [Qu’est-ce que SAP HANA sur Azure (grandes instances) ?](./hana-overview-architecture.md)

## <a name="check-activities-of-a-single-hana-large-instance-unit"></a>Vérifier les activités d’une seule unité de Grande instance HANA 
Outre le fait que vous disposez d’une vue d’ensemble des unités de Grande instance HANA, vous pouvez vérifier les activités de l’unité en particulier. Un journal d’activité peut ressembler à ceci :

![Volet de navigation du portail Azure](./media/hana-li-portal/portal-activity-list.png)

L’une des principales activités enregistrées est le redémarrage d’une unité. Les données listées incluent l’état de l’activité, la date et l’heure auxquelles elle a été déclenchée, l’ID d’abonnement à partir duquel elle a été déclenchée et l’utilisateur Azure qui l’a déclenché. 

Les changements apportés à l’unité dans les métadonnées Azure constituent une autre activité qui est enregistrée. En plus du redémarrage lancé, vous pouvez voir l’activité **Write HANAInstances**. Ce type d’activité n’apporte aucun changement à l’unité de Grande instance HANA proprement dite, mais il documente les changements apportés aux métadonnées de l’unité dans Azure. Dans le cas indiqué, nous avons ajouté et supprimé une étiquette (voir la section suivante).

## <a name="add-and-delete-an-azure-tag-to-a-hana-large-instance-unit"></a>Ajouter et supprimer une étiquette Azure à une unité de Grande instance HANA
Une autre possibilité s’offre à vous : ajouter une [étiquette](../../../azure-resource-manager/management/tag-resources.md) à une unité de Grande instance HANA. La façon dont les étiquettes sont attribuées ne diffère pas de l’attribution d’étiquettes à des machines virtuelles. Comme avec les machines virtuelles, les étiquettes existent dans les métadonnées Azure et, pour HANA - Grandes instances, elles ont les mêmes restrictions que les étiquettes définies pour les machines virtuelles.

La suppression d’étiquettes fonctionne de la même façon qu’avec les machines virtuelles. Les deux activités, à savoir l’application et la suppression d’une étiquette, seront listées dans le journal d’activité de l’unité de Grande instance HANA spécifique.

## <a name="check-properties-of-a-hana-large-instance-unit"></a>Vérifier les propriétés d’une unité de Grande instance HANA
La section **Propriétés** inclut des informations importantes que vous obtenez quand les instances vous sont remises. Vous trouverez dans cette section toutes les informations que vous pourriez exiger dans des cas de support ou dont vous avez besoin lors de la définition de la configuration de la capture instantanée de stockage. Par conséquent, cette section est une collection de données concernant votre instance, la connectivité de l’instance à Azure et le back-end de stockage. Le haut de la section ressemble à ceci :


![partie supérieure des propriétés HLI dans le portail Azure](./media/hana-li-portal/portal-properties-top.png)

Vous avez déjà vu les premiers éléments de données dans l’écran de vue d’ensemble. Mais une partie importante de données est constituée de l’ID de Circuit ExpressRoute, que vous avez obtenu lors de la remise des premières unités déployées. Vous pouvez être invité à fournir ces données dans certains cas de support. Une entrée de données importante est affichée en bas de la capture d’écran. Les données affichées sont l’adresse IP de l’en-tête de stockage NFS qui isole votre stockage de votre **locataire** dans la pile Grande instance HANA. Cette adresse IP est également nécessaire quand vous modifiez le [fichier de configuration pour les sauvegardes de captures instantanées de stockage](./hana-backup-restore.md#set-up-storage-snapshots). 

Quand vous faites défiler le volet de propriétés vers le bas, des données supplémentaires s’affichent, comme un ID de ressource unique pour votre unité de Grande instance HANA ou l’ID d’abonnement qui a été affecté au déploiement.

## <a name="restart-a-hana-large-instance-unit-through-azure-portal"></a>Redémarrer une unité de Grande instance HANA à l’aide du portail Azure
Lors du lancement d’un redémarrage du système d’exploitation Linux, il y avait diverses situations dans lesquelles le système d’exploitation ne parvenait pas à redémarrer correctement. Pour forcer un redémarrage, vous deviez ouvrir une demande de service pour que des opérations Microsoft effectuent un redémarrage de l’alimentation de l’unité de Grande instance HANA. Les fonctionnalités d’un redémarrage de l’alimentation d’une unité de Grande instance HANA sont maintenant intégrées au portail Azure. Quand vous vous trouvez dans la partie de vue d’ensemble de l’unité de Grande instance HANA, le bouton de redémarrage est affiché en haut de la section de données.

![Redémarrer l’étape n° 1 dans le portail Azure](./media/hana-li-portal/portal-restart-first-step.png)

Quand vous appuyez sur le bouton de redémarrage, un message vous demande si vous voulez vraiment redémarrer l’unité. Quand vous confirmez en appuyant sur le bouton « Oui », l’unité redémarre.

> [!NOTE]
> Dans le processus de redémarrage, un peu de temps s’écoule pendant lequel l’état de l’unité devient **Démarrage en cours** pour passer à l’état **Démarré**. L’état **Démarré** signifie que le système d’exploitation est en train de démarrer ou que son démarrage est complètement terminé. Par conséquent, après un redémarrage de l’unité, vous ne pouvez pas espérer vous connecter à l’unité dès que l’état bascule vers **Démarré**.

> [!IMPORTANT]
> En fonction de la quantité de mémoire présente dans votre unité de Grande instance HANA, un redémarrage du matériel et du système d’exploitation peut prendre jusqu’à une heure.


## <a name="open-a-support-request-for-hana-large-instances"></a>Ouvrir une demande de support pour les Grandes instances HANA
En dehors de l’affichage du portail Azure d’unités de Grande instance HANA, vous pouvez également créer des demandes de support spécifiquement pour une unité de Grande instance HANA. Pour cela, suivez le lien **Nouvelle demande de support**. 

![lancer l’étape n° 1 de la demande de service dans le portail Azure](./media/hana-li-portal/portal-initiate-support-request.png)

Afin que le service de SAP HANA - Grandes instances soit listé dans l’écran suivant, vous devrez peut-être sélectionner « Tous les services », comme indiqué ci-dessous.

![Sélectionner tous les services dans le portail Azure](./media/hana-li-portal/portal-create-service-request.png)

Dans la liste des services, vous trouverez le service **Grande instance SAP HANA**. Quand vous choisissez ce service, vous pouvez sélectionner des types de problèmes spécifiques comme indiqué :


![Sélectionner la classe de problème dans le portail Azure](./media/hana-li-portal/portal-select-problem-class.png)

Sous chacun des différents types de problèmes, une sélection des sous-types de problèmes que vous devez sélectionner pour mieux caractériser votre problème est disponible. Après avoir sélectionné le sous-type, vous pouvez nommer l’objet. Une fois que vous avez terminé le processus de sélection, vous pouvez passer à l’étape suivante de la création. Dans la section **Solutions**, vous êtes dirigé vers une documentation concernant HANA - Grandes instances, qui peut donner des indications pour résoudre votre problème. Si vous ne trouvez aucune solution à votre problème dans la documentation suggérée, vous passez à l’étape suivante. À l’étape suivante, vous devrez indiquer si le problème concerne les machines virtuelles ou les unités de Grande instance HANA. Ces informations permettent d’adresser la demande de support aux spécialistes appropriés. 

![Détails d’un cas de support dans le portail Azure](./media/hana-li-portal/portal-support-request-details.png)

Une fois que vous avez répondu aux questions et fourni des détails supplémentaires, vous pouvez passer à l’étape suivante afin d’examiner la demande de support et de l’envoyer.

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour superviser SAP HANA (grandes instances) sur Azure](./troubleshooting-monitoring.md)
- [Surveillance et dépannage à partir de HANA](./hana-monitor-troubleshoot.md)

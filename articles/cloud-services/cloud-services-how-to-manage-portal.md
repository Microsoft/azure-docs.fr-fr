---
title: Tâches courantes de gestion de service cloud | Microsoft Docs
description: Découvrez comment gérer Azure Cloud Services dans le portail Azure. Ces exemples utilisent le portail Azure.
services: cloud-services
documentationcenter: ''
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 07/05/2017
ms.author: tagore
ms.openlocfilehash: 84e450a55f9482c3a713943adfcc4d7940ab873f
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92076706"
---
# <a name="manage-cloud-services-in-the-azure-portal"></a>Gérer Azure Cloud Services dans le portail Azure
Dans la zone **Services cloud** du portail Azure, vous pouvez :

* Mettre à jour d’un rôle de service ou un déploiement.
* Promouvoir un déploiement intermédiaire en production.
* Lier des ressources à votre service cloud afin de pouvoir visualiser ensemble les dépendances de ressources et l’échelle des ressources.
* Supprimez ou service cloud ou un déploiement.

Pour plus d’informations sur la mise à l’échelle votre service cloud, voir [Configurer la mise à l’échelle automatique pour un service cloud dans le portail](cloud-services-how-to-scale-portal.md).

## <a name="update-a-cloud-service-role-or-deployment"></a>Mettre à jour rôle de service cloud ou un déploiement
Si vous devez mettre à jour le code de l'application pour votre service cloud, utilisez **Update** dans le panneau de service cloud. Vous pouvez mettre à jour un ou plusieurs rôles. Pour effectuer la mise à jour, vous pouvez charger un nouveau package de service ou un nouveau fichier de configuration de service.

1. Dans le [portail Azure][Azure portal], sélectionnez le service cloud que vous souhaitez mettre à jour. Cette étape ouvre le panneau d'instance de service cloud.

2. Sur le panneau, sélectionnez **Mettre à jour**.

    ![Bouton Mettre à jour](./media/cloud-services-how-to-manage-portal/update-button.png)

3. Mettez à jour le déploiement avec un nouveau fichier de package de service (.cspkg) et un fichier de configuration de service (.cscfg).

    ![UpdateDeployment](./media/cloud-services-how-to-manage-portal/update-blade.png)

4. Si vous le souhaitez, mettez à jour le compte de stockage et de l’étiquette du déploiement.

5. Si l’un des rôles ne comporte qu’une seule instance, activez la case à cocher **Déployer même si un ou plusieurs rôles contiennent une seule instance** pour permettre la mise à niveau.

    Azure ne peut garantir 99,95 % de disponibilité du service pendant la mise à jour d’un service cloud que si chaque rôle dispose d’au moins deux instances (machines virtuelles). Avec deux instances de rôle, une machine virtuelle traite les demandes du client pendant que l'autre est mise à jour.

6. Activez la case à cocher **Démarrer le déploiement** pour appliquer la mise à jour une fois le chargement du package terminé.

7. Sélectionnez **OK** pour démarrer la mise à jour du service.

## <a name="swap-deployments-to-promote-a-staged-deployment-to-production"></a>Permuter des déploiements pour promouvoir un déploiement intermédiaire en production
Lorsque vous décidez de déployer une nouvelle version d'un service cloud, créez un déploiement intermédiaire et testez la nouvelle version dans l'environnement intermédiaire de votre service cloud. Utilisez **Swap** pour inverser les URL qui permettent d'accéder aux deux déploiements et de faire passer une nouvelle version en production.

Vous pouvez inverser les déploiements à partir de la page **Cloud Services** ou du tableau de bord.

1. Dans le [portail Azure][Azure portal], sélectionnez le service cloud que vous souhaitez mettre à jour. Cette étape ouvre le panneau d'instance de service cloud.

2. Sur le panneau, sélectionnez **Permuter**.

    ![Bouton Permuter les services cloud](./media/cloud-services-how-to-manage-portal/swap-button.png)

3. L’invite de confirmation suivante s’ouvre :

    ![Inverser les services cloud](./media/cloud-services-how-to-manage-portal/swap-prompt.png)

4. Après avoir vérifié les informations de déploiement, sélectionnez **OK** pour permuter les déploiements.

    L'inversion des déploiements se fait rapidement, car la seule chose qui change est l'adresse IP virtuelle (VIP) des déploiements.

    Afin de réduire les coûts liés au calcul, vous pouvez supprimer le déploiement intermédiaire une fois que vous êtes certain que votre déploiement de production se comporte comme vous le souhaitez.

### <a name="common-questions-about-swapping-deployments"></a>Questions courantes sur l’échange de déploiements

**Quelles sont les conditions préalables à l’échange des déploiements ?**

Il existe deux conditions préalables principales pour qu’un échange de déploiements réussisse :

- Si vous souhaitez utiliser une adresse IP statique pour votre emplacement de production, vous devez en réserver une pour votre emplacement intermédiaire également. Sinon, l’échange échoue.

- Toutes les instances de vos rôles doivent être en cours d’exécution pour que vous puissiez effectuer l’échange. Vous pouvez vérifier l’état de vos instances sur le panneau **Aperçu** du portail Azure. Vous pouvez également utiliser la commande [Get-AzureRole](/powershell/module/servicemanagement/azure.service/get-azurerole?view=azuresmps-3.7.0) dans Windows PowerShell.

Notez que les mises à jour du système d’exploitation invité et les opérations de réparation de service peuvent également entraîner l’échec du déploiement. Pour plus d’informations, consultez [Résoudre les problèmes de déploiement de service cloud](cloud-services-troubleshoot-deployment-problems.md).

**Un échange implique-t-il un temps d’arrêt pour mon application ? Comment dois-je le gérer ?**

Comme décrit dans la section précédente, une permutation de déploiements est généralement rapide, car il s’agit simplement d’une modification de configuration dans Azure Load Balancer. Dans certains cas, elle peut prendre au moins 10 secondes et entraîner des échecs de connexion temporaires. Pour limiter l’impact sur vos clients, envisagez d’implémenter la [logique de nouvelle tentative client](/azure/architecture/best-practices/transient-faults).

## <a name="delete-deployments-and-a-cloud-service"></a>Supprimer des déploiements et un service cloud
Avant de pouvoir supprimer un service cloud, vous devez supprimer tous les déploiements existants.

Afin de réduire les coûts liés au calcul, vous pouvez supprimer le déploiement intermédiaire une fois que vous êtes certain que votre déploiement de production se comporte comme vous le souhaitez. Vous êtes facturé pour les coûts de calcul des instances de rôle déployées qui ont été arrêtées.

Utiliser la procédure suivante pour supprimer un déploiement ou un service cloud.

1. Dans le [portail Azure][Azure portal], sélectionnez le service cloud que vous souhaitez supprimer. Cette étape ouvre le panneau d'instance de service cloud.

2. Sur le panneau, sélectionnez **Supprimer**.

    ![Bouton Supprimer des services cloud](./media/cloud-services-how-to-manage-portal/delete-button.png)

3. Pour supprimer tout le service cloud, activez la service cloud **Service cloud et ses déploiements**. Vous pouvez également activer la case à cocher **Déploiement de production** ou **Déploiement intermédiaire** case à cocher.

    ![Suppression de services cloud](./media/cloud-services-how-to-manage-portal/delete-blade.png)

4. Sélectionnez **Supprimer** en bas.

5. Pour supprimer le service cloud, sélectionnez **Supprimer le service cloud**. Ensuite, à l’invite de confirmation, sélectionnez **Oui**.

> [!NOTE]
> Quand un service cloud est supprimé et que la surveillance détaillée est configurée, vous devez supprimer manuellement les données de votre compte de stockage. Pour savoir où trouver les tables de métriques, voir [Introduction à la surveillance des services cloud](cloud-services-how-to-monitor.md).


## <a name="find-more-information-about-failed-deployments"></a>Trouver des informations supplémentaires sur les déploiements ayant échoué
Le panneau **d’aperçu** possède une barre d’état en haut. Lorsque vous sélectionnez la barre, un nouveau panneau s’ouvre et affiche toutes les informations d’erreur. Si le déploiement ne contient pas d’erreur, le panneau d’informations est vide.

![Vue d’ensemble des services cloud](./media/cloud-services-how-to-manage-portal/status-info.png)



[Azure portal]: https://portal.azure.com

## <a name="next-steps"></a>Étapes suivantes
* [Configuration générale de votre service cloud](cloud-services-how-to-configure-portal.md).
* Découvrez comment [déployer un service cloud](cloud-services-how-to-create-deploy-portal.md).
* Configurez un [nom de domaine personnalisé](cloud-services-custom-domain-name-portal.md).
* Configurez des [certificats TLS/SSL](cloud-services-configure-ssl-certificate-portal.md).
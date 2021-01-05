---
title: Préparer Azure à la reprise d’activité après sinistre pour Hyper-V à l’aide d’Azure Site Recovery
description: Découvrez comment préparer Azure à la reprise d’activité des machines virtuelles Hyper-V locales à l’aide d’Azure Site Recovery.
author: rayne-wiselman
ms.service: site-recovery
services: site-recovery
ms.topic: tutorial
ms.date: 11/14/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: f4f040288574a9ddfc5f7c56878c7f1e795b09a3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88653849"
---
# <a name="prepare-azure-resources-for-hyper-v-disaster-recovery"></a>Préparer les ressources Azure à la reprise d’activité après sinistre pour Hyper-V

 [Azure Site Recovery](site-recovery-overview.md) fournit une aide à la continuité d’activité et reprise d’activité en garantissant le bon fonctionnement et la disponibilité des applications métier pendant les interruptions planifiées et non planifiées. Site Recovery gère et orchestre la récupération d’urgence des machines locales et des machines virtuelles Azure, notamment la réplication, le basculement et la récupération.

Ce tutoriel est le premier d’une série qui montre comment configurer la reprise d’activité après sinistre pour des machines virtuelles Hyper-V locales.

> [!NOTE]
> Nous concevons les tutoriels pour montrer le chemin de déploiement le plus simple pour un scénario. Ces tutoriels utilisent les options par défaut dans la mesure du possible, et n’affichent pas tous les paramètres et chemins possibles. Pour plus d’informations, consultez la section « Guide pratique » du scénario correspondant.

Ce tutoriel vous comment préparer les composants Azure quand vous souhaitez répliquer des machines virtuelles locales (Hyper-V) sur Azure. Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Vérifiez que votre compte Azure dispose des autorisations de réplication.
> * Créer un compte Stockage Azure qui stocke les images des machines répliquées.
> * Créer un coffre Recovery Services qui stocke les métadonnées et les informations de configuration pour les machines virtuelles et autres composants de réplication.
> * Configurez un réseau Azure Une fois les machines virtuelles Azure créées après le basculement, elles sont jointes à ce réseau.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.

## <a name="sign-in"></a>Se connecter

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="verify-account-permissions"></a>Vérifier les autorisations du compte

Si vous venez de créer un compte Azure gratuit, vous êtes l’administrateur de cet abonnement. Si vous n’êtes pas l’administrateur, collaborez avec celui-ci pour affecter les autorisations dont vous avez besoin. Pour activer la réplication pour une nouvelle machine virtuelle, vous devez avoir l’autorisation d’effectuer les actions ci-après :

- Créer une machine virtuelle dans le groupe de ressources sélectionné
- Créer une machine virtuelle dans le réseau virtuel sélectionné
- Écrire dans le compte de stockage sélectionné

Pour effectuer ces tâches, le rôle prédéfini de contributeur de machines virtuelles doit être assigné à votre compte. Pour gérer les opérations Site Recovery dans un coffre, le rôle prédéfini de contributeur Site Recovery doit être assigné à votre compte.

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

Les images des machines répliquées sont conservées dans le stockage Azure. Les machines virtuelles Azure sont créées à partir du stockage quand vous basculez du site local vers Azure. Le compte de stockage doit se trouver dans la même région que le coffre Recovery Services.

1. Dans le menu du [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** > **Stockage** > **Compte de stockage - blob, fichier, table, file d’attente**.
2. Dans **Créer un compte de stockage** , entrez un nom correspondant au compte.  Le nom que vous choisissez doit être unique dans Azure, comprendre entre 3 et 24 caractères, et contenir uniquement des lettres minuscules et des chiffres. Pour ce tutoriel, utilisez **contosovmsacct1910171607**.
3. Dans **Modèle de déploiement**, sélectionnez **Resource Manager**.
4. Dans **Type de compte**, sélectionnez **Stockage (v1 universel)** . Ne sélectionnez pas Stockage Blob.
5. Dans **Réplication**, sélectionnez la valeur par défaut **Stockage géo-redondant avec accès en lecture** pour la redondance de stockage. Laissez le paramètre Transfert sécurisé requis désactivé.
6. Dans **Performances**, sélectionnez **Standard**. Ensuite, dans **Niveau d’accès**, sélectionnez l’option par défaut **Chaud**.
7. Dans **Abonnement**, choisissez l’abonnement dans lequel vous souhaitez créer le compte de stockage.
8. Dans **Groupe de ressources**, entrez un nouveau groupe de ressources. Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Pour ce tutoriel, utilisez **ContosoRG**.
9. Dans **Emplacement**, choisissez un emplacement géographique pour votre compte de stockage. Pour ce tutoriel, utilisez **Europe Ouest**.
10. Sélectionnez **Créer** pour créer le compte de stockage.

   ![Capture d’écran des options Créer un compte de stockage](media/tutorial-prepare-azure/create-storageacct.png)

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, puis recherchez Recovery Services sur la Place de marché Azure.
2. Sélectionnez **Backup and Site Recovery (OMS)** . Ensuite, dans la page **Backup and Site Recovery**, sélectionnez **Créer**.
1. Dans **Coffre Recovery Services > Nom**, entrez un nom convivial permettant d’identifier le coffre. Pour ce didacticiel, utilisez **ContosoVMVault**.
2. Dans **Groupe de ressources**, sélectionnez un groupe de ressources existant ou créez-en un. Pour ce tutoriel, utilisez **contosoRG**.
3. Dans **Emplacement**, sélectionnez la région où le coffre doit se trouver. Pour ce tutoriel, utilisez **Europe Ouest**.
4. Pour accéder rapidement au coffre à partir du tableau de bord, sélectionnez **Épingler au tableau de bord** > **Créer**.

![Capture d’écran de la page Créer un coffre Recovery Services](./media/tutorial-prepare-azure/new-vault-settings.png)

Le nouveau coffre apparaît dans **Tableau de bord** > **Toutes les ressources** et dans la page principale **Coffres Recovery Services**.

## <a name="set-up-an-azure-network"></a>Configurer un réseau Azure

Quand les machines virtuelles Azure sont créées à partir du stockage après le basculement, elles sont jointes à ce réseau.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** > **Mise en réseau** > **Réseau virtuel**. Laissez Resource Manager sélectionné comme modèle de déploiement.
2. Dans **Nom**, entrez un nom de réseau. Le nom doit être unique au sein du groupe de ressources Azure. Pour ce tutoriel, utilisez **ContosoASRnet**.
3. Spécifiez le groupe de ressources dans lequel créer le réseau. Pour ce tutoriel, utilisez le groupe de ressources existant **contosoRG**.
4. Dans **Plage d’adresses**, entrez **10.0.0.0/24** comme plage du réseau. Il n’existe aucun sous-réseau pour ce réseau.
5. Dans **Abonnement**, sélectionnez l’abonnement dans lequel créer le réseau.
6. Dans **Emplacement**, choisissez **Europe Ouest**. Ce réseau doit se trouver dans la même région que le coffre Recovery Services.
7. Conservez les options par défaut de la protection DDoS de base, sans point de terminaison de service sur le réseau.
8. Sélectionnez **Create** (Créer).

![Capture d’écran des options Créer un réseau virtuel](media/tutorial-prepare-azure/create-network.png)

La création du réseau virtuel prend quelques secondes. Une fois qu’il est créé, vous le voyez dans le tableau de bord du portail Azure.

## <a name="useful-links"></a>Liens utiles

Vous en saurez plus sur :
- [Réseaux Azure](../virtual-network/virtual-networks-overview.md)
- [Disques managés](../virtual-machines/managed-disks-overview.md)



## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Préparer l’infrastructure Hyper-V locale à la reprise d’activité sur Azure](hyper-v-prepare-on-premises-tutorial.md)

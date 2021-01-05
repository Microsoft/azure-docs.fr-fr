---
title: Configurer la reprise d’activité après sinistre pour Hyper-V (avec VMM) en utilisant Azure Site Recovery
description: Découvrez comment configurer la reprise d’activité des machines virtuelles Hyper-V locales hébergées dans des clouds System Center VMM sur Azure avec Site Recovery.
ms.topic: tutorial
ms.date: 03/19/2020
ms.custom: MVC
ms.openlocfilehash: c806f968bc6530879f64ddbf6fd4c7d45aa7a8d3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89442818"
---
# <a name="set-up-disaster-recovery-of-on-premises-hyper-v-vms-in-vmm-clouds-to-azure"></a>Configurer la récupération d’urgence dans Azure de machines virtuelles Hyper-V locales hébergées dans des clouds VMM

Ce tutoriel explique comment activer la réplication de machines virtuelles Hyper-V locales gérées par System Center Virtual Machine Manager (VMM) sur Azure à l’aide d’[Azure Site Recovery](site-recovery-overview.md). Si vous n’utilisez pas VMM, [suivez ce tutoriel](hyper-v-azure-tutorial.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Sélectionner la source et la cible de réplication.
> * Configurer l’environnement de réplication source, y compris les composants Site Recovery locaux, et l’environnement de réplication cible.
> * Configurer le mappage réseau entre les réseaux de machines virtuelles VMM et les réseaux virtuels Azure.
> * Créer une stratégie de réplication.
> * Activer la réplication pour une machine virtuelle.

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario. Ils utilisent les options par défaut lorsque cela est possible et n’affichent pas tous les paramètres et chemins d’accès possibles. Pour obtenir des instructions détaillées, consultez les articles de la section **Guides pratiques** de la [documentation sur Site Recovery](./index.yml).

## <a name="prerequisites"></a>Prérequis

Ce tutoriel suppose que vous avez déjà effectué les tutoriels suivants :

1. [Préparer Azure](tutorial-prepare-azure.md)
1. [Préparer des serveurs Hyper-V locaux](hyper-v-prepare-on-premises-tutorial.md)

## <a name="select-a-replication-goal"></a>Sélectionner un objectif de réplication

1. Dans le portail Azure, accédez à **Coffres Recovery Services**, puis sélectionnez le coffre **ContosoVMVault** qui a été créé dans le tutoriel [Préparer Azure](tutorial-prepare-azure.md#create-a-recovery-services-vault).
1. Dans **Bien démarrer**, sélectionnez **Site Recovery** > **Préparer l’infrastructure** et configurez les paramètres suivants :
    1. **Objectif de protection** > **Où se trouvent vos machines ?** , sélectionnez **Local**.
    1. **Où voulez-vous répliquer vos machines ?** , sélectionnez **Dans Azure**.
    1. **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec Hyper-V**.
    1. **Utilisez-vous System Center VMM pour gérer vos hôtes Hyper-V ?** , sélectionnez **Oui**.
1. Sélectionnez **OK**.

   ![Objectif de réplication](./media/hyper-v-vmm-azure-tutorial/replication-goal.png)

## <a name="confirm-deployment-planning"></a>Confirmer la planification d’un déploiement

1. Dans **Planification du déploiement**, si vous planifiez un déploiement de grande envergure, téléchargez l’outil Planificateur de déploiement pour Hyper-V à partir du lien figurant dans la page. [En savoir plus](hyper-v-deployment-planner-overview.md) sur la planification du déploiement Hyper-V.
1. Pour ce tutoriel, nous n’avons pas besoin de l’outil Planificateur de déploiement. Dans **Avez-vous effectué la planification du déploiement ?** , sélectionnez **Je le ferai plus tard**, puis sélectionnez **OK**.

## <a name="set-up-the-source-environment"></a>Configurer l’environnement source

Quand vous configurez l’environnement source, vous installez le fournisseur Azure Site Recovery sur le serveur VMM et inscrivez ce dernier dans le coffre. Vous installez l’agent Azure Recovery Services sur chaque hôte Hyper-V.

1. **Préparer l’infrastructure**. Sélectionnez **Source**.
1. **Préparer la source**. Sélectionnez **+ VMM** pour ajouter un serveur VMM. Dans **Ajouter un serveur**, vérifiez que **Serveur System Center VMM** s’affiche dans **Type de serveur**.
1. Téléchargez le programme d’installation du fournisseur Microsoft Azure Site Recovery.
1. Téléchargez la clé d’inscription du coffre. Vous avez besoin de cette clé quand vous exécuterez le programme d’installation du fournisseur. Une fois générée, la clé reste valide pendant 5 jours.
1. Téléchargez le programme d’installation de l’agent Microsoft Azure Recovery Services.

   ![Télécharger le fournisseur, la clé d’inscription et l’agent](./media/hyper-v-vmm-azure-tutorial/download-vmm.png)

### <a name="install-the-provider-on-the-vmm-server"></a>Installer le fournisseur sur le serveur VMM

1. Dans l’Assistant Installation du fournisseur Azure Site Recovery **Microsoft Update**. Choisissez d’utiliser Microsoft Update pour rechercher les mises à jour du fournisseur.
1. **Installation**. Acceptez l’emplacement d’installation par défaut pour le fournisseur et sélectionnez **Installer**.
1. Après l’installation, dans l’Assistant Inscription de Microsoft Azure Site Recovery, sélectionnez **Paramètres de coffre**, **Parcourir** et, dans **Fichier de clé**, sélectionnez le fichier de clé de coffre que vous avez téléchargé.
1. Spécifiez l’abonnement Azure Site Recovery et le nom du coffre (**ContosoVMVault**). Spécifiez un nom convivial pour le serveur VMM afin de l’identifier dans le coffre.
1. **Paramètres proxy**. Sélectionnez **Se connecter directement à Azure Site Recovery sans serveur proxy**.
1. Acceptez l’emplacement par défaut du certificat utilisé pour chiffrer les données. Les données chiffrées sont déchiffrées pendant le basculement.
1. **Synchroniser les métadonnées du cloud**. Sélectionnez **Synchroniser les métadonnées du cloud par rapport au portail Azure Site Recovery**. Cette action ne doit se produire qu’une seule fois sur chaque serveur. Sélectionnez ensuite **Inscription**.
1. Une fois que le serveur est inscrit dans le coffre, sélectionnez **Terminer**.

Une fois l’inscription terminée, les métadonnées du serveur sont récupérées par Azure Site Recovery et le serveur VMM s’affiche dans **Infrastructure Site Recovery**.

### <a name="install-the-recovery-services-agent-on-hyper-v-hosts"></a>Installer l’agent Recovery Services sur des hôtes Hyper-V

Installez l’agent sur chaque hôte Hyper-V contenant les machines virtuelles à répliquer.

Dans l’Assistant Installation de l’Agent Microsoft Azure Recovery Services, configurez les paramètres suivants :

1. **Vérification de la configuration requise**. Sélectionnez **Suivant**. Tous les prérequis manquants sont installés automatiquement.
1. **Paramètres d’installation**. Acceptez l’emplacement d’installation. Sélectionnez **Installer**.

    >[!NOTE]
    >Pour Azure Site Recovery, l’**emplacement du cache** n’est pas obligatoire.

1. **Installation**. Quand l’installation est terminée, sélectionnez **Fermer** pour quitter l’Assistant.

   ![Installer l'agent](./media/hyper-v-vmm-azure-tutorial/mars-install.png)

## <a name="set-up-the-target-environment"></a>Configurer l’environnement cible

1. Sélectionnez **Préparer l’infrastructure** > **Cible**.
1. Sélectionnez l’abonnement et le groupe de ressources (**ContosoRG**) où créer les machines virtuelles Azure après le basculement.
1. Sélectionnez le modèle de déploiement **Resource Manager**.

Site Recovery vérifie qu’il y a un ou plusieurs réseaux et comptes Azure Storage compatibles.

## <a name="configure-network-mapping"></a>Configurer le mappage réseau

1. **Infrastructure Site Recovery** > **Mappages réseau** > **Mappage réseau**. Sélectionnez l’icône **+Mappage réseau**.
1. **Ajouter un mappage réseau**. Sélectionnez le serveur **Source System Center VMM**. Pour la **Cible**, sélectionnez Azure.
1. Vérifiez l’abonnement et le modèle de déploiement après le basculement.
1. **Réseau source**. Sélectionnez le réseau de machines virtuelles locales sources.
1. **Réseau cible**. Choisissez le réseau Azure dans lequel les machines virtuelles Azure de réplica se trouveront, une fois créées après le basculement. Ensuite, sélectionnez **OK**.

   ![Mappage réseau](./media/hyper-v-vmm-azure-tutorial/network-mapping-vmm.png)

## <a name="set-up-a-replication-policy"></a>Configurer une stratégie de réplication

1. Sélectionnez **Préparer l’infrastructure** > **Paramètres de réplication** >  **+Créer et associer**.
1. Dans **Créer et associer une stratégie**, indiquez le nom de la stratégie. Nous utilisons **ContosoReplicationPolicy**.
1. Acceptez les paramètres par défaut et sélectionnez **OK** :
   - **Fréquence de copie** indique que les données delta (après la réplication initiale) sont répliquées toutes les cinq minutes.
   - **Rétention des points de récupération** indique que les points de récupération vont être conservés pendant deux heures.
   - **Fréquence des instantanés de cohérence des applications** indique que les points de récupération contenant des instantanés de cohérence des applications sont créés toutes les heures.
   - **Heure de début de la réplication initiale** indique que la réplication initiale démarre immédiatement.
   - **Chiffrer les données stockées sur Azure** est défini sur la valeur par défaut (**Désactivé**) et indique que les données au repos dans Azure ne sont pas chiffrées.
1. Après avoir créé la stratégie, sélectionnez **OK**. Lorsque vous créez une stratégie, elle est automatiquement associée au cloud VMM.

## <a name="enable-replication"></a>Activer la réplication

1. **Répliquer l’application**. Sélectionnez **Source**.
1. **Source**. Sélectionnez le cloud VMM. Ensuite, sélectionnez **OK**.
1. **Cible**. Vérifiez la cible (Azure) et l’abonnement du coffre, puis sélectionnez le modèle **Resource Manager**.
1. Sélectionnez le compte de stockage **contosovmsacct1910171607** et le réseau Azure **ContosoASRnet**.
1. **Machines virtuelles** > **Sélectionner**. Sélectionnez la machine virtuelle à répliquer. Ensuite, sélectionnez **OK**.

   Vous pouvez suivre la progression de l’action **Activer la protection** dans **Travaux** > **Travaux Site Recovery**. Quand le travail de **finalisation de la protection** est terminé, la réplication initiale est également terminée et la machine virtuelle est prête à être basculée.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Effectuer un test de récupération d’urgence](tutorial-dr-drill-azure.md)

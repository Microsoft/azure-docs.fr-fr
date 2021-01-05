---
title: Configurer la reprise d’activité après sinistre pour Hyper-V à l’aide d’Azure Site Recovery
description: Découvrez comment configurer la reprise d’activité de machines virtuelles Hyper-V locales (sans VMM) dans Azure à l’aide de Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 11/12/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 5ce98c785700301bba92926d7d5a243b614eca7f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87504226"
---
# <a name="set-up-disaster-recovery-of-on-premises-hyper-v-vms-to-azure"></a>Configurer la récupération d’urgence de machines virtuelles Hyper-V locales vers Azure

Le service [Azure Site Recovery](site-recovery-overview.md) contribue à votre stratégie de reprise d’activité après sinistre en gérant et en coordonnant la réplication, le basculement et la restauration automatique des machines locales et des machines virtuelles Azure.

Il s’agit du troisième didacticiel d’une série. Il vous montre comment configurer la reprise d’activité de machines virtuelles Hyper-V locales sur Azure. Ce tutoriel s’applique aux machines virtuelles Hyper-V qui ne sont pas gérées par Microsoft System Center Virtual Machine Manager (VMM).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Sélectionner la source et la cible de réplication.
> * Configurer l’environnement de réplication source, y compris les composants Site Recovery locaux, et l’environnement de réplication cible.
> * Créer une stratégie de réplication.
> * Activer la réplication pour une machine virtuelle.

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario. Ils utilisent les options par défaut lorsque cela est possible et n’affichent pas tous les paramètres et chemins d’accès possibles. Pour obtenir des instructions détaillées, consultez les articles de la section **Guides pratiques** de la [documentation sur Site Recovery](./index.yml).



## <a name="before-you-begin"></a>Avant de commencer

Il s’agit du troisième didacticiel d’une série. Il suppose que vous avez déjà effectué les tâches des tutoriels précédents :

1. [Préparer Azure](tutorial-prepare-azure.md)
2. [Préparer un serveur Hyper-V local](./hyper-v-prepare-on-premises-tutorial.md)

## <a name="select-a-replication-goal"></a>Sélectionner un objectif de réplication

1. Dans le portail Azure, accédez aux **coffres Recovery Services**, puis sélectionnez le coffre. Nous avons préparé le coffre-fort **ContosoVMVault** au cours du tutoriel précédent.
2. Dans **Démarrage**, sélectionnez **Site Recovery**, puis sélectionnez **Préparer l’infrastructure**.
3. Dans **Objectif de protection** > **Où se trouvent vos machines ?** , sélectionnez **Local**.
4. Dans **Où voulez-vous répliquer vos machines ?** , sélectionnez **Dans Azure**.
5. Dans **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec Hyper-V**.
6. Dans **Utilisez-vous System Center VMM pour gérer vos hôtes Hyper-V ?** , sélectionnez **Non**.
7. Sélectionnez **OK**.

    ![Capture d’écran des options Objectif de protection sous Préparer l’infrastructure.](./media/hyper-v-azure-tutorial/replication-goal.png)

## <a name="confirm-deployment-planning"></a>Confirmer la planification d’un déploiement

1. Dans **Planification du déploiement**, si vous planifiez un déploiement de grande envergure, téléchargez l’outil Planificateur de déploiement pour Hyper-V à partir du lien figurant dans la page. [En savoir plus](hyper-v-deployment-planner-overview.md) sur la planification du déploiement Hyper-V.
2. Pour ce tutoriel, nous n’avons pas besoin de l’outil Planificateur de déploiement. Dans **Avez-vous effectué la planification du déploiement ?** , sélectionnez **Je le ferai plus tard**, puis sélectionnez **OK**.

    ![Capture d’écran des options Planification du déploiement sous Préparer l’infrastructure.](./media/hyper-v-azure-tutorial/deployment-planning.png)

## <a name="set-up-the-source-environment"></a>Configurer l’environnement source

Pour configurer l’environnement source, créez un site Hyper-V et ajoutez à ce site les hôtes Hyper-V contenant les machines virtuelles à répliquer. Ensuite, vous téléchargez et installez le fournisseur Azure Site Recovery et l’agent Azure Recovery Services sur chaque hôte et inscrivez le site Hyper-V dans le coffre.

1. Sous **Préparer l’infrastructure**, sélectionnez **Source**.
2. Dans **Préparer la source**, sélectionnez **+ Site Hyper-V**.
3. Dans **Créer un site Hyper-V**, spécifiez le nom du site. Nous utilisons **ContosoHyperVSite**.

    ![Capture d’écran montrant la sélection du site Hyper-V sous Préparer l’infrastructure](./media/hyper-v-azure-tutorial/hyperv-site.png)

4. Une fois le site créé, dans **Préparer la source** > **Étape 1 : Sélectionner le site Hyper-V**, sélectionnez le site que vous avez créé.
5. Sélectionnez **+ Serveur Hyper-V**.

    ![Capture d’écran montrant la sélection du serveur Hyper-V sous Préparer l’infrastructure](./media/hyper-v-azure-tutorial/hyperv-server.png)

6. Téléchargez le programme d’installation du fournisseur Microsoft Azure Site Recovery.
7. Téléchargez la clé d’inscription du coffre. Vous devez avoir cette clé pour installer le fournisseur. Une fois générée, la clé reste valide pendant 5 jours.

    ![Capture d’écran des options permettant de télécharger le fournisseur et la clé d’inscription.](./media/hyper-v-azure-tutorial/download.png)
    

### <a name="install-the-provider"></a>Installer le fournisseur

Installez le fichier de configuration téléchargé (AzureSiteRecoveryProvider.exe) sur chaque hôte Hyper-V à ajouter au site Hyper-V. Le programme d’installation installe le fournisseur Azure Site Recovery et l’agent Recovery Services sur chaque hôte Hyper-V.

1. Exécutez le fichier de configuration.
2. Dans l’Assistant Installation du fournisseur Azure Site Recovery > **Microsoft Update**, choisissez d’utiliser Microsoft Update pour rechercher les mises à jour du fournisseur.
3. Dans **Installation**, acceptez l’emplacement d’installation par défaut pour le fournisseur et l’agent, et sélectionnez **Installer**.
4. Après l’installation, dans l’Assistant Inscription de Microsoft Azure Site Recovery > **Paramètres de coffre**, sélectionnez **Parcourir** et, dans **Fichier de clé**, sélectionnez le fichier de clé de coffre que vous avez téléchargé.
5. Spécifiez l’abonnement Azure Site Recovery, le nom du coffre (**ContosoVMVault**) et le site Hyper-V (**ContosoHyperVSite**) auquel appartient le serveur Hyper-V.
6. Dans **Paramètres de proxy**, sélectionnez **Se connecter directement à Azure Site Recovery sans proxy**.
7. Dans **Inscription**, une fois que le serveur est inscrit dans le coffre, sélectionnez **Terminer**.

Les métadonnées du serveur Hyper-V sont récupérées par Azure Site Recovery et le serveur s’affiche dans **Infrastructure Site Recovery** > **Hôtes Hyper-V**. Ce processus peut prendre jusqu’à 30 minutes.

#### <a name="install-the-provider-on-a-hyper-v-core-server"></a>Installer le fournisseur sur un serveur Hyper-V (installation minimale)

Si vous exécutez un serveur Hyper-V (installation minimale), téléchargez le fichier d’installation et effectuez les étapes suivantes :

1. Extrayez les fichiers d’AzureSiteRecoveryProvider.exe dans un répertoire local en exécutant cette commande :

    `AzureSiteRecoveryProvider.exe /x:. /q`
 
2. Exécutez `.\setupdr.exe /i`. Les résultats sont journalisés dans %Programdata%\ASRLogs\DRASetupWizard.log.

3. Inscrivez le serveur en exécutant cette commande :

    ```
    cd  "C:\Program Files\Microsoft Azure Site Recovery Provider\DRConfigurator.exe" /r /Friendlyname "FriendlyName of the Server" /Credentials "path to where the credential file is saved"
    ```

## <a name="set-up-the-target-environment"></a>Configurer l’environnement cible

Sélectionnez et vérifiez les ressources cibles :

1. Sélectionnez **Préparer l’infrastructure** > **Cible**.
2. Sélectionnez l’abonnement et le groupe de ressources **ContosoRG** où créer les machines virtuelles Azure après le basculement.
3. Sélectionnez le modèle de déploiement **Resource Manager**.

Site Recovery vérifie que vous disposez d’un ou de plusieurs réseaux et comptes Azure Storage compatibles.

## <a name="set-up-a-replication-policy"></a>Configurer une stratégie de réplication

1. Sélectionnez **Préparer l’infrastructure** > **Paramètres de réplication** >  **+Créer et associer**.
2. Dans **Créer et associer une stratégie**, indiquez le nom de la stratégie. Nous utilisons **ContosoReplicationPolicy**.
3. Pour ce tutoriel, nous allons laisser les paramètres par défaut :
    - **Fréquence de copie** indique la fréquence de réplication des données delta (après la réplication initiale). Il effectue une synchronisation toutes les cinq minutes par défaut.
    - **Rétention des points de récupération** indique que les points de récupération vont être conservés pendant deux heures. La valeur maximale autorisée de conservation pour la protection des machines virtuelles hébergées sur des hôtes Hyper-V est de 24 heures.
    - **Fréquence des instantanés de cohérence des applications** indique que les points de récupération contenant des instantanés de cohérence des applications sont créés toutes les heures.
    - **Heure de début de la réplication initiale** indique que la réplication initiale démarre immédiatement.
4. Après avoir créé la stratégie, sélectionnez **OK**. Quand vous créez une stratégie, celle-ci est automatiquement associée au site Hyper-V spécifié. Dans notre tutoriel, il s’agit de **ContosoHyperVSite**.

    ![Stratégie de réplication](./media/hyper-v-azure-tutorial/replication-policy.png)

## <a name="enable-replication"></a>Activer la réplication

1. Dans **Répliquer l’application**, sélectionnez **Source**.
2. Dans **Source**, sélectionnez le site **ContosoHyperVSite**. Ensuite, sélectionnez **OK**.
3. Dans **Cible**, vérifiez la cible (Azure), l’abonnement du coffre et le modèle de déploiement **Resource Manager**.
4. Si vous utilisez les paramètres du tutoriel, sélectionnez le compte de stockage **contosovmsacct1910171607** créé dans le tutoriel précédent pour les données répliquées. Sélectionnez également le réseau **ContosoASRnet**, dans lequel les machines virtuelles Azure sont placées après un basculement.
5. Dans **Machines virtuelles** > **Sélectionner**, sélectionnez les machines virtuelles à répliquer. Ensuite, sélectionnez **OK**.

   Vous pouvez suivre la progression de l’action **Activer la protection** dans **Travaux** > **Travaux Site Recovery**. Quand le travail de **finalisation de la protection** est terminé, la réplication initiale est également terminée et la machine virtuelle est prête à être basculée.

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Effectuer un test de récupération d’urgence](tutorial-dr-drill-azure.md)

---
title: Migrer des serveurs Windows Server 2008 vers Azure à l’aide d’Azure Migrate/Site Recovery
description: Cet article explique comment migrer des machines Windows Server 2008 locales vers Azure, et recommande Azure Migrate.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 07/27/2020
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 6cc0855d3a4540de780a566a4613b4dbc647cfc5
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369486"
---
# <a name="migrate-servers-running-windows-server-2008-to-azure"></a>Migrer des serveurs exécutant Windows Server2008 vers Azure

Ce tutoriel montre comment migrer des serveurs locaux exécutant Windows Server 2008 ou 2008 R2 vers Azure à l’aide d’Azure Site Recovery. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Préparer votre environnement local pour la migration.
> * Configurer l’environnement cible.
> * Configurer une stratégie de réplication.
> * Activer la réplication.
> * Exécuter une migration de test pour vérifier que tout fonctionne comme prévu.
> * Basculer vers Azure et effectuer la migration.

## <a name="migrate-with-azure-migrate"></a>Migrer avec Azure Migrate

Nous vous recommandons de migrer des machines vers Azure à l’aide du service [Azure Migrate](../migrate/migrate-services-overview.md). 

- Azure Migrate est spécialement conçu pour la migration de serveur.
- Azure Migrate fournit un hub centralisé pour la découverte, l’évaluation et la migration de machines locales vers Azure. Azure Site Recovery ne doit être utilisé que pour la récupération d’urgence, pas pour la migration.
- Azure Migrate prend en charge la migration de serveurs exécutant Windows Server 2008.


## <a name="migrate-with-site-recovery"></a>Migrer avec Site Recovery

### <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge


|Système d’exploitation  | Environnement  |
|---------|---------|
|Windows Server 2008 SP2 - 32 bits et 64 bits (IA-32 et cx86-64)</br>- Standard</br>- Entreprise</br>- Centre de données   |     Machines virtuelles VMware, machines virtuelles Hyper-V et serveurs physiques    |
|Windows Server 2008 R2 SP1 - 64 bits</br>- Standard</br>- Entreprise</br>- Centre de données     |     Machines virtuelles VMware, machines virtuelles Hyper-V et serveurs physiques|

> [!WARNING]
> - La migration de serveurs exécutant Server Core n’est pas prise en charge.
> - Avant la migration, assurez-vous que le dernier Service Pack et les dernières mises à jour de Windows sont installés.


### <a name="prerequisites"></a>Prérequis

Avant de commencer, il est utile d’examiner l’architecture d’Azure Site Recovery pour la [migration de VMware et de serveur physique](vmware-azure-architecture.md) ou la [migration de machine virtuelle Hyper-V](hyper-v-azure-architecture.md). 

Pour migrer des machines virtuelles Hyper-V exécutant Windows Server 2008 ou 2008 R2, suivez les étapes décrites dans le didacticiel [Migrer des machines locales vers Azure](migrate-tutorial-on-premises-azure.md).

Le reste de ce didacticiel montre comment migrer des machines virtuelles VMware locales et des serveurs physiques exécutant Windows Server 2008 ou 2008 R2.
> [!TIP]
> Vous recherchez un moyen de migrer des machines virtuelles VMware vers Azure sans agent ? [Cliquez ici](../migrate/tutorial-migrate-vmware.md)


### <a name="limitations-and-known-issues"></a>Limitations et problèmes connus

- Le serveur de configuration, les serveurs de processus supplémentaires et le service Mobilité utilisé pour migrer les serveurs Windows Server 2008 SP2 doivent exécuter la version 9.19.0.0 ou une version ultérieure du logiciel Azure Site Recovery.

- Les points de récupération cohérents d’application et la fonctionnalité de cohérence multimachine virtuelle ne sont pas pris en charge pour la réplication des serveurs exécutant Windows Server 2008 SP2. Les serveurs Windows Server 2008 SP2 doivent être migrés vers un point de récupération cohérent d’incident. Par défaut, des points de récupération cohérents d’incident sont générés toutes les 5 minutes. L’utilisation d’une stratégie de réplication avec une fréquence de capture instantanée de cohérence des applications configurée a pour effet que l’intégrité de la réplication devient critique à défaut de point de récupération cohérent d’application. Pour éviter les faux positifs, définissez la fréquence de capture instantanée de cohérence des applications dans la stratégie de réplication sur « Désactivé ».

- Pour que le service Mobilité fonctionne, .NET Framework 3.5 Service Pack 1 doit être installé sur les serveurs migrés.

- Si votre serveur dispose de disques dynamiques, vous remarquerez peut-être que, dans certaines configurations, ces disques sur le serveur basculé sont marqués hors connexion ou affichés en tant que disques étrangers. Vous pourriez également remarquer que l’état de mise en miroir défini pour les volumes en miroir des disques dynamiques est marqué « Échec de la redondance ». Vous pouvez résoudre ce problème à partir de diskmgmt.msc en important ces disques et en les réactivant manuellement.

- Les serveurs migrés doivent avoir le pilote vmstorfl.sys. Le basculement peut échouer si le pilote n’est pas présent sur le serveur migré. 
  > [!TIP]
  >Vérifiez si le pilote est présent dans « C:\Windows\system32\drivers\vmstorfl.sys ». Si le pilote est introuvable, vous pouvez contourner ce problème en créant un fichier factice. 
  >
  > Ouvrez l’invite de commandes (run > cmd), puis exécutez la commande suivante : « copy nul c:\Windows\system32\drivers\vmstorfl.sys »

- Il se peut que vous ne puissiez pas utiliser le protocole RDP sur des serveurs Windows Server 2008 SP2 exécutant le système d’exploitation 32 bits immédiatement après leur basculement ou leur test de basculement vers Azure. Redémarrez la machine virtuelle basculée à partir du portail Azure, puis réessayez de vous connecter. Si vous ne parvenez pas à vous connecter, vérifiez si le serveur est configuré pour autoriser les connexions Bureau à distance, et assurez-vous qu’il n’y pas de règle de pare-feu ou de groupe de sécurité réseau bloquant la connexion. 
  > [!TIP]
  > Un test de basculement est fortement recommandé avant de migrer les serveurs. Assurez-vous d’avoir effectué au moins un test de basculement réussi sur chaque serveur que vous migrez. Dans le cadre du test de basculement, connectez-vous à la machine de test et vérifiez que tout fonctionne comme prévu.
  >
  >L’opération de test de basculement est non perturbateur et vous aide à tester les migrations en créant des machines virtuelles dans un réseau isolé de votre choix. Contrairement à l’opération de basculement, pendant l’opération de test de basculement, la réplication des données continue à progresser. Vous pouvez effectuer autant de tests de basculement que vous le souhaitez pour préparer la migration. 
  >
  


### <a name="get-started"></a>Bien démarrer

Pour préparer l’abonnement Azure et l’environnement physique/VMware local, effectuez les tâches suivantes :

1. [Préparer Azure](tutorial-prepare-azure.md)
2. Préparer les machines virtuelles [VMware](vmware-azure-tutorial-prepare-on-premises.md) locales


### <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

1. Connectez-vous au [portail Azure](https://portal.azure.com) > **Recovery Services**.
2. Cliquez sur **Créer une ressource** > **Outils de gestion** > **Backup and Site Recovery**.
3. Dans **Nom**, indiquez le nom convivial **W2K8-migration**. Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié.
4. Créez un groupe de ressources **w2k8migrate**.
5. Spécifiez une région Azure. Pour découvrir les régions prises en charge, référez-vous à la disponibilité géographique de la page [Tarification de Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).
6. Pour accéder rapidement au coffre à partir du tableau de bord, cliquez sur **Épingler au tableau de bord**, puis sur **Créer**.

   ![Capture d’écran montrant les options de création de coffre](media/migrate-tutorial-windows-server-2008/migrate-windows-server-2008-vault.png)

Le nouveau coffre est ajouté à la zone **Tableau de bord** dans **Toutes les ressources** et dans la page principale **Coffres Recovery Services**.


### <a name="prepare-your-on-premises-environment-for-migration"></a>Préparer votre environnement local pour la migration

- Pour migrer des machines virtuelles Windows Server 2008 s’exécutant sur VMware, [configurez le serveur de configuration local sur VMware](vmware-azure-tutorial.md#set-up-the-source-environment).
- Si le serveur de configuration ne peut pas être configuré en tant que machine virtuelle VMware, [configurez le serveur de configuration localement sur un serveur physique ou une machine virtuelle](physical-azure-disaster-recovery.md#set-up-the-source-environment).

### <a name="set-up-the-target-environment"></a>Configurer l’environnement cible

Sélectionnez et vérifiez les ressources cibles.

1. Cliquez sur **Préparer l’infrastructure** > **Cible** et sélectionnez l’abonnement Azure à utiliser.
2. Spécifiez le modèle de déploiement Resource Manager.
3. Site Recovery vérifie que vous disposez d’un ou de plusieurs réseaux et comptes Azure Storage compatibles.


### <a name="set-up-a-replication-policy"></a>Configurer une stratégie de réplication

1. Pour créer une nouvelle stratégie de réplication, cliquez sur **Infrastructure de Site Recovery** > **Stratégies de réplication** >  **+Stratégie de réplication**.
2. Dans **Créer une stratégie de réplication**, indiquez le nom de la stratégie.
3. Dans **Seuil d’objectif de point de récupération**, spécifiez la limite de l’objectif de point de récupération (RPO). Une alerte est générée si l’objectif de point de récupération dépasse cette limite.
4. Dans **Rétention des points de récupération**, spécifiez la durée de la fenêtre de rétention pour chaque point de récupération (en heures). Les serveurs répliqués peuvent être récupérés à n’importe quel moment dans cette fenêtre. Les machines virtuelles répliquées vers Premium Storage peuvent prendre en charge jusqu’à 24 heures de rétention et 72 heures en cas de stockage standard.
5. Dans **Fréquence de capture instantanée de cohérence des applications**, spécifiez **Désactivé**. Cliquez sur **OK** pour créer la stratégie.

La stratégie est automatiquement associée au serveur de configuration.

> [!WARNING]
> Veillez à spécifier **Désactivé** pour le paramètre de fréquence de capture instantanée de cohérence des applications de la stratégie de réplication. Seuls les points de récupération cohérents d’incident sont pris en charge lors de la réplication de serveurs exécutant Windows Server 2008. La spécification de toute autre valeur pour la fréquence des captures instantanées de cohérence des applications entraîne de fausses alertes en rendant critique l’intégrité de réplication du serveur en raison d’un manque points de récupération de cohérence d’application.

   ![Capture d’écran montrant les options de création de stratégie de réplication](media/migrate-tutorial-windows-server-2008/create-policy.png)

### <a name="enable-replication"></a>Activer la réplication

[Activez la réplication](physical-azure-disaster-recovery.md#enable-replication) pour le serveur Windows Server 2008 SP2 ou 2008 R2 SP1 à migrer.
   
   ![Capture d’écran montrant les options d’ajout de machines physiques](media/migrate-tutorial-windows-server-2008/Add-physical-server.png)

   ![Capture d’écran montrant les options d’activation de la réplication](media/migrate-tutorial-windows-server-2008/Enable-replication.png)

### <a name="run-a-test-migration"></a>Exécuter un test de migration

Vous pouvez effectuer un test de basculement des serveurs de réplication une fois que la réplication initiale est terminée et que le serveur est dans l’état **Protégé**.

Exécutez un [test de basculement](tutorial-dr-drill-azure.md) vers Azure afin de vérifier que tout fonctionne bien.

   ![Capture d’écran montrant la commande Tester le basculement](media/migrate-tutorial-windows-server-2008/testfailover.png)


### <a name="migrate-to-azure"></a>Migrer vers Azure

Exécutez un basculement pour les machines que vous souhaitez migrer.

1. Dans **Paramètres** > **Éléments répliqués**, cliquez sur la machine > **Basculement**.
2. Dans **Basculer**, sélectionnez un **point de récupération** vers lequel basculer. Sélectionnez le dernier point de récupération.
3. Sélectionnez **Arrêter la machine avant de commencer le basculement**. Site Recovery tente d’arrêter le serveur avant de déclencher le basculement. Le basculement est effectué même en cas d’échec de l’arrêt. Vous pouvez suivre la progression du basculement sur la page **Tâches**.
4. Vérifiez que la machine virtuelle Azure s’affiche dans Azure comme prévu.
5. Dans **Éléments répliqués**, cliquez avec le bouton droit sur le serveur > **Terminer la migration**. Cette opération effectue les actions suivantes :

    - Termine le processus de migration, arrête la réplication pour le serveur et arrête la facturation Site Recovery pour le serveur.
    - Cette étape nettoie les données de réplication. Elle ne supprime pas les machines virtuelles migrées.

   ![Capture d’écran montrant la commande Terminer la migration](media/migrate-tutorial-windows-server-2008/complete-migration.png)


> [!WARNING]
> **N’annulez pas un basculement en cours** : la réplication du serveur est arrêtée avant le début du basculement. Si vous annulez un basculement en cours, le basculement s’arrête, mais le serveur n’est plus répliqué.

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Passez en revue les questions courantes](../migrate/resources-faq.md) sur Azure Migrate.
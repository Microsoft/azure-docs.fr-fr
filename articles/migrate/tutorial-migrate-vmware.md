---
title: Migrer des machines virtuelles VMware à l’aide de la migration de serveurs Azure Migrate sans agent
description: Découvrez comment exécuter la migration sans agent des machines virtuelles VMware avec Azure Migrate.
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: tutorial
ms.date: 06/09/2020
ms.custom: mvc
ms.openlocfilehash: f9e7547b8b9a8728c2c1ce7d1d01d06ad1d55dc1
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387239"
---
# <a name="migrate-vmware-vms-to-azure-agentless"></a>Migrer des machines virtuelles VMware vers Azure (sans agent)

Cet article explique comment effectuer la migration de machines virtuelles VMware locales vers Azure avec l’outil [Azure Migrate : Migration de serveur](migrate-services-overview.md#azure-migrate-server-migration-tool), en utilisant la migration sans agent. Vous pouvez également migrer des machines virtuelles VMware à l’aide de la migration basée sur un agent. [Comparez](server-migrate-overview.md#compare-migration-methods) les méthodes.

Ce tutoriel est le troisième d’une série qui explique comment évaluer et migrer des machines virtuelles VMware vers Azure. 

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario donné afin que vous puissiez configurer rapidement une preuve de concept. Ils utilisent des options par défaut, le cas échéant, et ne montrent pas tous les paramètres et chemins possibles. 


Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Ajouter l’outil Azure Migrate : Migration de serveur
> * Découvrir les machines virtuelles que vous souhaitez migrer
> * Démarrer la réplication des machines virtuelles
> * Exécuter une migration de test pour vérifier que tout fonctionne comme prévu.
> * Exécuter une migration de machine virtuelle complète

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce didacticiel, vous devez :

1. [Suivre le premier tutoriel](./tutorial-discover-vmware.md) pour configurer Azure et VMware en vue de la migration.
2. Nous vous recommandons de suivre le deuxième tutoriel afin d’[évaluer les machines virtuelles VMware](./tutorial-assess-vmware-azure-vm.md) avant de les migrer vers Azure, mais cela n’est pas une obligation. 
3. Accédez au projet déjà créé ou [créez un nouveau projet](./create-manage-projects.md).
4. Vérifiez les autorisations de votre compte Azure – Votre compte Azure a besoin d’autorisations pour créer une machine virtuelle et écrire sur un disque managé Azure.

## <a name="set-up-the-azure-migrate-appliance"></a>Configurer l’appliance Azure Migrate

Azure Migrate : Migration de serveur exécute une appliance de machine virtuelle VMware légère qui est utilisée pour la détection, l’évaluation et la migration sans agent des machines virtuelles VMware. Si vous suivez le [tutoriel d’évaluation](./tutorial-assess-vmware-azure-vm.md), vous avez en principe déjà configuré l’appliance. Si ce n’est pas le cas, configurez-la maintenant, en choisissant l’une des méthodes suivantes :

- **Modèle OVA** : [Configurez-la](how-to-set-up-appliance-vmware.md) sur une machine virtuelle VMware au moyen d’un modèle OVA téléchargé.
- **Script** : [configuration](deploy-appliance-script.md) sur une machine virtuelle VMware ou une machine physique, au moyen d’un script d’installation PowerShell. C’est la méthode à utiliser si vous ne pouvez pas configurer une machine virtuelle à l’aide d’un modèle OVA ou si vous êtes dans un cloud Azure Government.

Après avoir créé l’appliance, vérifiez qu’elle peut se connecter à Azure Migrate Server Assessment, configurez-la pour la première fois, puis inscrivez-la auprès du projet Azure Migrate.

## <a name="replicate-vms"></a>Répliquer des machines virtuelles

Une fois que vous avez configuré l’appliance et terminé la détection, vous pouvez commencer la réplication des machines virtuelles VMware sur Azure. 

- Vous pouvez exécuter jusqu’à 300 réplications simultanément.
- Dans le portail, vous pouvez sélectionner jusqu’à 10 machines virtuelles à la fois pour la migration. Pour migrer davantage de machines, ajoutez-les à des groupes par lots de 10.

Activez la réplication comme suit :

1. Dans le projet Azure Migrate > **Serveurs**, **Azure Migrate : Server Migration**, cliquez sur **Répliquer**.

    ![Répliquer des machines virtuelles](./media/tutorial-migrate-vmware/select-replicate.png)

2. Dans **Répliquer** > **Paramètres de la source** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec VMware vSphere**.
3. Dans **Appliance locale**, sélectionnez le nom de l’appliance Azure Migrate que vous avez configurée > **OK**. 

    ![Paramètres de la source](./media/tutorial-migrate-vmware/source-settings.png)

4. Dans **Machines virtuelles**, sélectionnez les machines à répliquer. Pour appliquer les paramètres de taille de machine virtuelle et de type de disque issus d’une évaluation, le cas échéant, dans **Importer les paramètres de migration à partir d’une évaluation Azure Migrate ?** , sélectionnez **Oui**, puis sélectionnez le groupe de machines virtuelles et le nom de l’évaluation. Si vous n’utilisez pas les paramètres d’évaluation, sélectionnez **Non**.
   
    ![Sélectionner l’évaluation](./media/tutorial-migrate-vmware/select-assessment.png)

5. Dans **Machines virtuelles**, sélectionnez les machines virtuelles que vous souhaitez migrer. Cliquez ensuite sur **Suivant : Paramètres de la cible**.

    ![Sélectionner les machines virtuelles](./media/tutorial-migrate-vmware/select-vms.png)

6. Dans **Paramètres cibles**, sélectionnez l’abonnement et la région cible. Spécifiez le groupe de ressources dans lequel résideront les machines virtuelles Azure après la migration.
7. Dans **Réseau virtuel**, sélectionnez le réseau virtuel/sous-réseau Azure auquel joindre les machines virtuelles Azure après la migration.
8. Dans **Options de disponibilité**, sélectionnez :
    -  Zone de disponibilité pour épingler la machine migrée à une Zone de disponibilité spécifique dans la région. Utilisez cette option pour distribuer les serveurs qui forment une couche Application à plusieurs nœuds entre des Zones de disponibilité. Si vous sélectionnez cette option, vous devez spécifier la Zone de disponibilité à utiliser pour chaque machine sélectionnée dans l’onglet Calcul. Cette option est disponible uniquement si la région cible sélectionnée pour la migration prend en charge les Zones de disponibilité
    -  Groupe à haute disponibilité pour placer la machine migrée dans un groupe à haute disponibilité. Pour utiliser cette option, le groupe de ressources cible qui a été sélectionné doit avoir un ou plusieurs groupes à haute disponibilité.
    - Aucune option de redondance de l’infrastructure n’est requise si vous n’avez pas besoin de ces configurations de disponibilité pour les machines migrées.
9. Dans **Type de chiffrement de disque**, sélectionnez :
    - Chiffrement au repos avec une clé gérée par la plateforme
    - Chiffrement au repos avec une clé gérée par le client

   > [!NOTE]
   > Pour répliquer des machines virtuelles avec une clé gérée par le client, vous devez [créer un jeu de chiffrement de disque](https://go.microsoft.com/fwlink/?linkid=2151800) sous le groupe de ressources cible. Un objet de jeu de chiffrement de disque mappe les disques managés à un coffre de clés contenant les clés gérées par le client à utiliser pour le chiffrement côté serveur.
  
10. Dans **Azure Hybrid Benefit** :

    - Sélectionnez **Non** si vous ne souhaitez pas appliquer Azure Hybrid Benefit. Cliquez ensuite sur **Suivant**.
    - Sélectionnez **Oui** si vous avez des machines Windows Server couvertes par des abonnements Software Assurance ou Windows Server actifs et que vous souhaitez appliquer l’avantage aux machines que vous migrez. Cliquez ensuite sur **Suivant**.

    ![Paramètres de la cible](./media/tutorial-migrate-vmware/target-settings.png)

11. Dans **Calcul**, vérifiez le nom de la machine virtuelle, sa taille, le type de disque du système d’exploitation et la configuration de la disponibilité (si elle a été sélectionnée à l’étape précédente). Les machines virtuelles doivent satisfaire aux [exigences d’Azure](migrate-support-matrix-vmware-migration.md#azure-vm-requirements).

    - **Taille de la machine virtuelle** : si vous suivez les recommandations de l’évaluation, la liste déroulante Taille de la machine virtuelle affiche la taille recommandée. Sinon, Azure Migrate choisit une taille qui correspond à la taille la plus proche dans l’abonnement Azure. Vous pouvez également choisir une taille manuelle dans **Taille de la machine virtuelle Azure**. 
    - **Disque de système d’exploitation** : spécifiez le disque du système d’exploitation (démarrage) pour la machine virtuelle. Le disque du système d’exploitation est le disque qui contient le chargeur de démarrage et le programme d’installation du système d’exploitation. 
    - **Zone de disponibilité** : Spécifiez la zone de disponibilité à utiliser.
    - **Groupe à haute disponibilité** : Spécifiez le groupe à haute disponibilité à utiliser.

    > [!NOTE]
    > Si vous souhaitez sélectionner une autre option de disponibilité pour un ensemble de machines virtuelles, allez à l’étape 1 et répétez les étapes en sélectionnant différentes options de disponibilité après le démarrage de la réplication pour un ensemble de machines virtuelles.


 ![Paramètres de capacité de calcul de la machine virtuelle](./media/tutorial-migrate-vmware/compute-settings.png)

12. Dans **Disques**, indiquez si les disques de machine virtuelle doivent être répliqués sur Azure, puis sélectionnez le type de disque (SSD/HDD standard ou disques managés Premium) dans Azure. Cliquez ensuite sur **Suivant**.
   
    ![Capture d’écran montrant l’onglet Disques de la boîte de dialogue Répliquer.](./media/tutorial-migrate-vmware/disks.png)

13. Dans **Passer en revue et démarrer la réplication**, passez en revue les paramètres, puis cliquez sur **Répliquer** pour démarrer la réplication initiale pour les serveurs.

> [!NOTE]
> Vous pouvez mettre à jour les paramètres de réplication à tout moment avant le démarrage de la réplication (**Gérer** > **Réplication des machines**). En revanche, les paramètres ne sont plus modifiables après le démarrage de la réplication.

### <a name="provisioning-for-the-first-time"></a>Provisionnement initial

S’il s’agit de la première machine virtuelle que vous répliquez dans le projet, l’outil Migration de serveur provisionne automatiquement ces ressources, dans le même groupe de ressources que le projet.

- **Service Bus** : Migration de serveur utilise Service Bus pour envoyer des messages d’orchestration de réplication à l’appliance.
- **Compte de stockage de la passerelle** : Server Migration utilise le compte de stockage de la passerelle pour stocker des informations d’état relatives aux machines virtuelles en cours de réplication.
- **Compte de stockage de journal** : l’appliance Azure Migrate charge les journaux de réplication pour les machines virtuelles vers un compte de stockage de journal. Azure Migrate applique les informations de réplication aux disques managés de réplica.
- **Coffre de clés** : l’appliance Azure Migrate utilise le coffre de clés afin de gérer les chaînes de connexion pour Service Bus, et des clés d’accès pour les comptes de stockage utilisés dans la réplication.

## <a name="track-and-monitor"></a>Suivre et superviser

1. Suivez l’état du travail dans les notifications du portail.
2. Supervisez l’état de la réplication en cliquant sur **Réplication de serveurs** dans **Azure Migrate : Server Migration**.

     ![Superviser la réplication](./media/tutorial-migrate-vmware/replicating-servers.png)

La réplication se déroule comme suit :
- Une fois le travail Démarrer la réplication terminé, les machines commencent leur réplication initiale vers Azure.
- Lors de la réplication initiale, une capture instantanée de machine virtuelle est créée. Les données de disque de la capture instantanée sont répliquées sur les disques managés de réplica dans Azure.
- Au terme de la réplication initiale, la réplication différentielle commence. Les modifications incrémentielles apportées aux disques locaux sont répliquées régulièrement sur les disques de réplica dans Azure.

## <a name="run-a-test-migration"></a>Exécuter un test de migration


Quand la réplication différentielle commence, vous pouvez exécuter une migration de test avec les machines virtuelles avant d’exécuter une migration complète vers Azure. Avant la migration, nous vous recommandons vivement d’effectuer cette opération au moins une fois pour chaque machine.

- L’exécution d’une migration de test permet de vérifier que la migration fonctionne comme prévu, sans impact sur les machines locales qui restent opérationnelles et poursuivent la réplication. 
- Une migration de test simule la migration en créant une machine virtuelle Azure à l’aide de données répliquées (il s’agit en général d’une migration vers un réseau virtuel hors production dans votre abonnement Azure).
- Vous pouvez utiliser la machine virtuelle Azure de test répliquée pour valider la migration, effectuer des tests d’applications et résoudre les éventuels problèmes avant la migration complète.

Effectuez une migration de test de la façon suivante :


1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Migration**, cliquez sur **Tester les serveurs migrés**.

     ![Tester les serveurs migrés](./media/tutorial-migrate-vmware/test-migrated-servers.png)

2. Cliquez avec le bouton droit sur la machine virtuelle à tester, puis cliquez sur **Migration de test**.

    ![Migration de test](./media/tutorial-migrate-vmware/test-migrate.png)

3. Dans **Migration de test**, sélectionnez le réseau virtuel Azure dans lequel la machine virtuelle Azure se trouvera après la migration. Nous vous recommandons d’utiliser un réseau virtuel hors production.
4. Le travail **Migration de test** démarre. Supervisez le travail dans les notifications du portail.
5. Une fois la migration terminée, affichez la machine virtuelle Azure migrée dans **Machines virtuelles** dans le portail Azure. Le nom de la machine a le suffixe **-Test**.
6. Une fois le test terminé, cliquez avec le bouton droit sur la machine virtuelle Azure dans **Réplication des machines**, puis cliquez sur **Nettoyer la migration de test**.

    ![Nettoyer la migration](./media/tutorial-migrate-vmware/clean-up.png)


## <a name="migrate-vms"></a>Migrer des machines virtuelles

Après avoir vérifié que la migration de test fonctionne comme prévu, vous pouvez migrer des machines locales.

1. Dans le projet Azure Migrate > **Serveurs** > **Azure Migrate : Server Migration**, cliquez sur **Réplication de serveurs**.

    ![Réplication de serveurs](./media/tutorial-migrate-vmware/replicate-servers.png)

2. Dans **Réplication des machines**, cliquez avec le bouton droit sur la machine virtuelle > **Migrer**.
3. Dans **Migrer** > **Arrêter les machines virtuelles et effectuer une migration planifiée sans perte de données**, sélectionnez **Oui** > **OK**.
    - Par défaut, Azure Migrate arrête la machine virtuelle locale et exécute une réplication à la demande pour synchroniser tout changement apporté à la machine virtuelle depuis la dernière réplication. Cela permet d’éviter toute perte de données.
    - Si vous ne souhaitez pas arrêter la machine virtuelle, sélectionnez **Non**.
4. Un travail de migration démarre pour la machine virtuelle. Suivez le travail dans les notifications Azure.
5. Une fois le travail terminé, vous pouvez afficher et gérer la machine virtuelle à partir de la page **Machines virtuelles**.

## <a name="complete-the-migration"></a>Effectuer la migration

1. Une fois la migration terminée, cliquez avec le bouton droit sur la machine virtuelle > **Arrêter la réplication**. Cette action arrête la réplication pour la machine locale et nettoie les informations d’état de la réplication de la machine virtuelle.
2. Installez l’agent [Linux](../virtual-machines/extensions/agent-linux.md) de machine virtuelle Azure sur les machines migrées si la machine exécute le système d’exploitation Linux. Nous installons automatiquement l’agent de machine virtuelle pour les machines virtuelles Windows pendant la migration.
3. Effectuez les éventuels ajustements post-migration de l’application, comme la mise à jour des chaînes de connexion de base de données et les configurations du serveur web.
4. Effectuez les tests finaux de réception de l’application et de la migration sur l’application migrée qui s’exécute maintenant dans Azure.
5. Réduisez le trafic vers l’instance de machine virtuelle Azure migrée.
6. Supprimez les machines virtuelles locales de votre inventaire des machines virtuelles locales.
7. Supprimez les machines virtuelles locales des sauvegardes locales.
8. Mettez à jour la documentation interne en y mentionnant le nouvel emplacement et la nouvelle adresse IP des machines virtuelles Azure. 

## <a name="post-migration-best-practices"></a>Bonnes pratiques après la migration

- Pour une meilleure résilience :
    - Sécurisez les données en sauvegardant les machines virtuelles Azure avec le service Sauvegarde Azure. [Plus d’informations](../backup/quick-backup-vm-portal.md)
    - Conservez les charges de travail en cours d’exécution et disponibles en continu en répliquant des machines virtuelles Azure vers une région secondaire avec Site Recovery. [Plus d’informations](../site-recovery/azure-to-azure-tutorial-enable-replication.md)
- Pour renforcer la sécurité :
    - Verrouillez et limitez l’accès du trafic entrant avec l’[administration juste-à-temps d’Azure Security Center](../security-center/security-center-just-in-time.md).
    - Limitez le trafic réseau vers les points de terminaison de gestion avec des [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).
    - Déployez [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md) pour sécuriser les disques, et protégez les données contre le vol et les accès non autorisés.
    - Découvrez plus d’informations sur la [sécurisation des ressources IaaS](https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/) et visitez [Azure Security Center](https://azure.microsoft.com/services/security-center/).
- Pour la surveillance et la gestion :
-  Envisagez de déployer [Azure Cost Management](../cost-management-billing/cloudyn/overview.md) pour surveiller l’utilisation et les coûts des ressources.


## <a name="next-steps"></a>Étapes suivantes

Examinez le [parcours de migration cloud](/azure/architecture/cloud-adoption/getting-started/migrate) dans le framework d’adoption du cloud Azure.
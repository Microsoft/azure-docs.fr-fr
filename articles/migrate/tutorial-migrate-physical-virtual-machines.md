---
title: Migrez les machines en tant que serveurs physiques vers Azure avec Azure Migrate.
description: Cet article explique comment migrer des machines physiques vers Azure avec Azure Migrate.
author: rahulg1190
ms.author: rahugup
ms.manager: bsiva
ms.topic: tutorial
ms.date: 04/15/2020
ms.custom: MVC
ms.openlocfilehash: af1c321e5c537fbd3af770cb392c538e6056e075
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752870"
---
# <a name="migrate-machines-as-physical-servers-to-azure"></a>Migrer des machines en tant que serveurs physiques vers Azure

Cet article explique comment migrer des machines en tant que serveurs physiques vers Azure à l’aide de l’outil Azure Migrate : Server Migration. La migration de machines en les traitant comme des serveurs physiques est utile dans plusieurs scénarios :

- Serveurs physiques locaux
- Machines virtuelles virtualisées par des plateformes telles que Xen ou KVM
- Machines virtuelles Hyper-V ou VMware. Une telle migration s’avère utile si, pour une raison ou une autre, vous ne pouvez pas utiliser le processus de migration standard pour migrer les machines [Hyper-V](tutorial-migrate-hyper-v.md) ou [VMware ](server-migrate-overview.md).
- Machines virtuelles exécutées dans des clouds privés
- Machines virtuelles exécutées dans des clouds publics tels qu’Amazon Web Services (AWS) ou Google Cloud Platform (GCP).


Ce tutoriel est le troisième d’une série qui montre comment évaluer et migrer des serveurs physiques vers Azure. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Vous préparer à utiliser Azure avec Azure Migrate : Migration de serveur.
> * Vérifier les conditions requises pour les machines qui doivent faire l’objet de la migration, et préparer une machine pour l’appliance de réplication Azure Migrate utilisée pour découvrir et effectuer la migration des machines vers Azure
> * Ajouter l’outil Azure Migrate Server Migration dans le hub Azure Migrate
> * Configurer l’appliance de réplication
> * Installer Mobility Service sur chaque machine qui doit faire l’objet d’une migration
> * Activez la réplication.
> * Exécuter une migration de test pour vérifier que tout fonctionne comme prévu.
> * Exécuter une migration complète vers Azure.

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario donné afin que vous puissiez configurer rapidement une preuve de concept. Ils utilisent des options par défaut, le cas échéant, et ne montrent pas tous les paramètres et chemins possibles. Pour obtenir des instructions détaillées, consultez les articles de guide pratique pour Azure Migrate.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.


## <a name="prerequisites"></a>Prérequis

Avant de commencer ce didacticiel, vous devez :

[Passer en revue](./agent-based-migration-architecture.md) l’architecture de migration.

## <a name="prepare-azure"></a>Préparer Azure

Préparer Azure pour la migration avec Migration de serveur.

**Tâche** | **Détails**
--- | ---
**Créer un projet Azure Migrate** | Votre compte Azure doit disposer d’autorisations Contributeur ou Propriétaire pour [créer un projet](https://docs.microsoft.com/azure/migrate/create-manage-projects).
**Vérifier les autorisations pour votre compte Azure** | Votre compte Azure a besoin d’autorisations pour créer une machine virtuelle et écrire sur un disque managé Azure.


### <a name="assign-permissions-to-create-project"></a>Attribuer des autorisations pour créer un projet

1. Dans le portail Azure, ouvrez l’abonnement, puis sélectionnez **Contrôle d’accès (IAM)** .
2. Dans **Vérifier l’accès**, recherchez le compte approprié, puis cliquez dessus pour voir les autorisations correspondantes.
3. Vous devez disposer des autorisations de **Contributeur** ou de **Propriétaire**.
    - Si vous venez de créer un compte Azure gratuit, vous êtes le propriétaire de votre abonnement.
    - Si vous n’êtes pas le propriétaire de l’abonnement, demandez au propriétaire de vous attribuer le rôle.


### <a name="assign-azure-account-permissions"></a>Affecter des autorisations de compte Azure

Affectez le rôle Contributeur de machines virtuelles au compte Azure. Cela permet d’effectuer les opérations suivantes :

- Créer une machine virtuelle dans le groupe de ressources sélectionné
- Créer une machine virtuelle dans le réseau virtuel sélectionné
- Écrire sur un disque managé Azure. 

### <a name="create-an-azure-network"></a>Créer un réseau Azure

[Configurez](../virtual-network/manage-virtual-network.md#create-a-virtual-network) un réseau virtuel Azure. Quand vous effectuez une réplication sur Azure, des machines virtuelles Azure sont créées et jointes au réseau virtuel Azure que vous avez spécifié lors de la configuration de la migration.

## <a name="prepare-for-migration"></a>Préparation de la migration

Pour préparer la migration des serveurs physiques, vous devez vérifier les paramètres des serveurs physiques et préparer le déploiement d’une appliance de réplication.

### <a name="check-machine-requirements-for-migration"></a>Vérifier les conditions requises pour la migration des machines

Vérifiez que les machines sont conformes aux exigences de migration vers Azure. 

> [!NOTE]
> Lors de la migration de machines physiques, Azure Migrate : Migration de serveur utilise la même architecture de réplication que la fonctionnalité de reprise d’activité après sinistre basée sur un agent du service Azure Site Recovery, et certains des composants utilisés partagent la même base de code. Certains contenus peuvent être liés à la documentation Site Recovery.

1. [Vérifiez](migrate-support-matrix-physical-migration.md#physical-server-requirements) les conditions requises des serveurs physiques.
2. Vérifiez que les machines locales que vous répliquez sur Azure respectent les [exigences des machines virtuelles Azure](migrate-support-matrix-physical-migration.md#azure-vm-requirements).
3. Certaines modifications doivent être apportées aux machines virtuelles afin qu’elles puissent être migrées vers Azure.
    - Pour certains systèmes d’exploitation, Azure Migrate effectue ces modifications automatiquement. 
    - Il est important d’effectuer ces modifications avant de commencer la migration. Si vous migrez la machine virtuelle avant d’effectuer la modification, elle risque de ne pas démarrer dans Azure. Examinez les modifications à apporter aux machines [Windows](prepare-for-migration.md#windows-machines) et [Linux](prepare-for-migration.md#linux-machines).

### <a name="prepare-a-machine-for-the-replication-appliance"></a>Préparer une machine pour l’appliance de réplication

Azure Migrate : Migration de serveur utilise une appliance de réplication pour répliquer les machines sur Azure. L’appliance de réplication exécute les composants suivants :

- **Serveur de configuration** : Le serveur de configuration coordonne la communication entre les ordinateurs locaux et Azure.et gère la réplication des données.
- **Serveur de traitement**: Le serveur de processus fait office de passerelle de réplication. Il reçoit les données de réplication, les optimise grâce à une mise en cache, une compression et un chiffrement, puis les envoie à un compte de stockage de cache dans Azure. 

Préparez le déploiement de l’appliance comme suit :

- Préparez un ordinateur pour héberger l’appliance de réplication. [Passez en revue](migrate-replication-appliance.md#appliance-requirements) la configuration requise pour l’ordinateur.
- L’appliance de réplication utilise MySQL. Passez en revue les [options](migrate-replication-appliance.md#mysql-installation) d’installation de MySQL sur l’appliance.
- Passez en revue les URL Azure nécessaires à l’appliance de réplication pour accéder aux clouds [publics](migrate-replication-appliance.md#url-access) et du [secteur public](migrate-replication-appliance.md#azure-government-url-access).
- Passez en revue les exigences d’accès au [port] (migrate-replication-appliance.md#port-access) pour l’appliance de réplication.

> [!NOTE]
> L’appliance de réplication ne doit pas être installée sur une machine source que vous souhaitez répliquer, ou sur l’appliance de découverte et d’évaluation Azure Migrate que vous avez peut-être déjà installée.

## <a name="set-up-the-replication-appliance"></a>Configurer l’appliance de réplication

La première étape de la migration consiste à configurer l’appliance de réplication. Pour configurer l’appliance pour la migration des serveurs physiques, vous téléchargez le fichier d’installation de l’appliance, puis vous l’exécutez sur l’[ordinateur que vous avez préparé](#prepare-a-machine-for-the-replication-appliance). Une fois l’appliance installée, vous devez l’inscrire auprès d’Azure Migrate Server Migration.


### <a name="download-the-replication-appliance-installer"></a>Télécharger le programme d’installation de l’appliance de réplication

1. Dans le projet Azure Migrate > **Serveurs**, dans **Azure Migrate : Server Migration**, cliquez sur **Découvrir**.

    ![Détection des machines virtuelles](./media/tutorial-migrate-physical-virtual-machines/migrate-discover.png)

3. Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , cliquez sur **Non virtualisé/autre**.
4. Dans **Région cible**, sélectionnez la région Azure vers laquelle vous souhaitez migrer les machines.
5. Sélectionnez **Confirmer que la région cible pour la migration est « nom_région »** .
6. Cliquez sur **Créer des ressources**. Un coffre Azure Site Recovery est créé en arrière-plan.
    - Si vous avez déjà configuré la migration avec Azure Migrate Server Migration, l’option Cible ne peut pas être configurée, car les ressources ont déjà été configurées.    
    - Après avoir cliqué sur ce bouton, vous ne pouvez plus changer la région cible de ce projet.
    - Toutes les migrations suivantes sont effectuées dans cette région.

7. Dans **Voulez-vous installer une nouvelle appliance de réplication ?** , sélectionnez **Installer une appliance de réplication**.
9. Dans **Télécharger et installer le logiciel de l’appliance de réplication**, téléchargez le programme d’installation de l’appliance et la clé d’inscription. Vous avez besoin de la clé pour inscrire l’appliance. Une fois téléchargée, la clé reste valide pendant 5 jours.

    ![Télécharger le fournisseur](media/tutorial-migrate-physical-virtual-machines/download-provider.png)

10. Copiez le fichier de configuration de l’appliance et le fichier de clé sur la machine Windows Server 2016 que vous avez créée pour l’appliance.
11. Une fois l’installation terminée, l’Assistant Configuration de l’appliance démarre automatiquement (vous pouvez également le lancer manuellement à l’aide du raccourci cspsconfigtool créé sur le Bureau de l’appliance). Utilisez l’onglet Gérer les comptes de l’Assistant pour ajouter les détails du compte à utiliser pour l’installation push du service Mobilité. Dans ce tutoriel, nous allons installer manuellement le service Mobility sur les machines virtuelles sources à répliquer. Avant de continuer, vous devez donc créer un compte factice. Vous pouvez spécifier les informations suivantes pour créer le compte factice : « guest » comme nom convivial, « username » comme nom d’utilisateur et « password » comme mot de passe de compte. Vous utiliserez ce compte factice à l’étape Activer la réplication. 

12. Une fois l’appliance installée et redémarrée, dans **Découvrir des machines**, sélectionnez la nouvelle appliance dans **Sélectionner un serveur de configuration**, puis cliquez sur **Finaliser l’inscription**. L’option Finaliser l’inscription exécute quelques tâches finales pour préparer l’appliance de réplication.

    ![Finaliser l’inscription](./media/tutorial-migrate-physical-virtual-machines/finalize-registration.png)

Après la finalisation de l’inscription, il peut falloir du temps pour que les machines découvertes apparaissent dans Azure Migrate Server Migration. À mesure que des machines virtuelles sont découvertes, le nombre de **Serveurs découverts** augmente.

![Serveurs découverts](./media/tutorial-migrate-physical-virtual-machines/discovered-servers.png)


## <a name="install-the-mobility-service"></a>Installer le service Mobilité

Vous devez installer l’agent Mobility Service sur les machines qui feront l’objet de la migration. Les programmes d’installation de l’agent sont disponibles dans l’appliance de réplication. Recherchez le programme d’installation qui convient, puis installez l’agent sur chaque machine devant faire l’objet d’une migration. Procédez comme suit :

1. Connectez-vous à l’appliance de réplication.
2. Accédez au dossier **%ProgramData%\ASR\home\svsystems\pushinstallsvc\repository**.
3. Recherchez le programme d’installation correspondant à la version du système d’exploitation de la machine. Vérifiez quels sont les [systèmes d’exploitation pris en charge](../site-recovery/vmware-physical-azure-support-matrix.md#replicated-machines). 
4. Copiez le fichier du programme d’installation sur la machine qui doit faire l’objet d’une migration.
5. Vérifiez que vous disposez de la phrase secrète qui a été générée lorsque vous avez déployé l’appliance.
    - Stockez le fichier dans un fichier texte temporaire sur la machine.
    - Vous pouvez obtenir la phrase secrète dans l’appliance de réplication. Sur la ligne de commande, exécutez **C:\ProgramData\ASR\home\svsystems\bin\genpassphrase.exe -v** pour afficher la phrase secrète.
    - Ne regénérez pas la phrase secrète. Si vous la regénérez, la connectivité sera perdue et vous devrez réinscrire l’appliance de réplication.


### <a name="install-on-windows"></a>Installer sur Windows

1. Procédez de la façon suivante pour extraire le contenu du fichier du programme d’installation dans un dossier local (par exemple, C:\Temp) de la machine :

    ```
    ren Microsoft-ASR_UA*Windows*release.exe MobilityServiceInstaller.exe
    MobilityServiceInstaller.exe /q /x:C:\Temp\Extracted
    cd C:\Temp\Extracted
    ```
2. Exécutez le programme d’installation de Mobility Service :
    ```
   UnifiedAgent.exe /Role "MS" /Platform "VmWare" /Silent
    ```
3. Inscrivez l’agent auprès de l’appliance de réplication :
    ```
    cd C:\Program Files (x86)\Microsoft Azure Site Recovery\agent
    UnifiedAgentConfigurator.exe  /CSEndPoint <replication appliance IP address> /PassphraseFilePath <Passphrase File Path>
    ```

### <a name="install-on-linux"></a>Installer sur Linux

1. Procédez de la façon suivante pour extraire le contenu du tarball du programme d’installation dans un dossier local (par exemple, /tmp/MobSvcInstaller) de la machine :
    ```
    mkdir /tmp/MobSvcInstaller
    tar -C /tmp/MobSvcInstaller -xvf <Installer tarball>
    cd /tmp/MobSvcInstaller
    ```
2. Exécutez le script du programme d’installation :
    ```
    sudo ./install -r MS -v VmWare -q
    ```
3. Inscrivez l’agent auprès de l’appliance de réplication :
    ```
    /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i <replication appliance IP address> -P <Passphrase File Path>
    ```

## <a name="replicate-machines"></a>Répliquer les machines

À présent, sélectionnez les machines pour la migration. 

> [!NOTE]
> Vous pouvez répliquer ensemble jusqu’à 10 machines. Si vous avez besoin d’en répliquer davantage, répliquez-les simultanément par lots de 10.

1. Dans le projet Azure Migrate > **Serveurs**, **Azure Migrate : Server Migration**, cliquez sur **Répliquer**.

    ![Capture d’écran de l’écran Azure Migrate - Serveurs montrant le bouton Répliquer sélectionné dans Azure Migrate : Migration du serveur sous Outils de migration.](./media/tutorial-migrate-physical-virtual-machines/select-replicate.png)

2. Dans **Répliquer** > **Paramètres de la source** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Non virtualisé/autre**.
3. Dans **Appliance locale**, sélectionnez le nom de l’appliance Azure Migrate que vous avez configurée.
4. Dans **Process Server**, sélectionnez le nom de l’appliance de réplication.
6. Sous **Informations d’identification de l’invité**, sélectionnez le compte factice créé durant l’[installation de l’appliance de réplication](#download-the-replication-appliance-installer) pour installer manuellement le service Mobility (l’installation push n’est pas prise en charge). Cliquez ensuite sur **Suivant : Machines virtuelles**.   

    ![Capture d’écran de l’onglet Paramètres de la source dans l’écran Répliquer avec le champ Informations d’identification de l’invité en surbrillance.](./media/tutorial-migrate-physical-virtual-machines/source-settings.png)

7. Dans **Machines virtuelles**, dans **Importer les paramètres de migration à partir d’une évaluation ?** , conservez le paramètre par défaut **Non, je vais spécifier les paramètres de migration manuellement**.
8. Cochez chaque machine virtuelle devant faire l’objet d’une migration. Cliquez ensuite sur **Suivant : Paramètres de la cible**.

    ![Sélectionner les machines virtuelles](./media/tutorial-migrate-physical-virtual-machines/select-vms.png)


9. Dans **Paramètres de la cible**, sélectionnez l’abonnement et la région cible vers laquelle vous allez migrer, puis spécifiez le groupe de ressources dans lequel les machines virtuelles Azure résideront après la migration.
10. Dans **Réseau virtuel**, sélectionnez le réseau virtuel/sous-réseau Azure auquel les machines virtuelles Azure seront jointes après la migration.
11. Dans **Options de disponibilité**, sélectionnez :
    -  Zone de disponibilité pour épingler la machine migrée à une Zone de disponibilité spécifique dans la région. Utilisez cette option pour distribuer les serveurs qui forment une couche Application à plusieurs nœuds entre des Zones de disponibilité. Si vous sélectionnez cette option, vous devez spécifier la Zone de disponibilité à utiliser pour chaque machine sélectionnée dans l’onglet Calcul. Cette option est disponible uniquement si la région cible sélectionnée pour la migration prend en charge les Zones de disponibilité
    -  Groupe à haute disponibilité pour placer la machine migrée dans un groupe à haute disponibilité. Pour utiliser cette option, le groupe de ressources cible qui a été sélectionné doit avoir un ou plusieurs groupes à haute disponibilité.
    - Aucune option de redondance de l’infrastructure n’est requise si vous n’avez pas besoin de ces configurations de disponibilité pour les machines migrées.
12. Dans **Azure Hybrid Benefit** :

    - Sélectionnez **Non** si vous ne souhaitez pas appliquer Azure Hybrid Benefit. Cliquez ensuite sur **Suivant**.
    - Sélectionnez **Oui** si vous avez des machines Windows Server couvertes par des abonnements Software Assurance ou Windows Server actifs et que vous souhaitez appliquer l’avantage aux machines que vous migrez. Cliquez ensuite sur **Suivant**.

    ![Paramètres de la cible](./media/tutorial-migrate-physical-virtual-machines/target-settings.png)

13. Dans **Calcul**, vérifiez le nom de la machine virtuelle, sa taille, le type de disque du système d’exploitation et la configuration de la disponibilité (si elle a été sélectionnée à l’étape précédente). Les machines virtuelles doivent satisfaire aux [exigences d’Azure](migrate-support-matrix-physical-migration.md#azure-vm-requirements).

    - **Taille de la machine virtuelle** : si vous suivez les recommandations de l’évaluation, la liste déroulante Taille de la machine virtuelle affiche la taille recommandée. Sinon, Azure Migrate choisit une taille qui correspond à la taille la plus proche dans l’abonnement Azure. Vous pouvez également choisir une taille manuelle dans **Taille de la machine virtuelle Azure**.
    - **Disque de système d’exploitation** : spécifiez le disque du système d’exploitation (démarrage) pour la machine virtuelle. Le disque du système d’exploitation est le disque qui contient le chargeur de démarrage et le programme d’installation du système d’exploitation.
    - **Zone de disponibilité** : Spécifiez la zone de disponibilité à utiliser.
    - **Groupe à haute disponibilité** : Spécifiez le groupe à haute disponibilité à utiliser.

> [!NOTE]
> Si vous souhaitez sélectionner une autre option de disponibilité pour un ensemble de machines virtuelles, allez à l’étape 1 et répétez les étapes en sélectionnant différentes options de disponibilité après le démarrage de la réplication pour un ensemble de machines virtuelles.

   ![Paramètres de capacité de calcul](./media/tutorial-migrate-physical-virtual-machines/compute-settings.png)

13. Dans **Disques**, indiquez si les disques de machine virtuelle doivent être répliqués sur Azure, puis sélectionnez le type de disque (SSD/HDD standard ou disques managés Premium) dans Azure. Cliquez ensuite sur **Suivant**.
    - Vous pouvez exclure des disques de la réplication.
    - Si vous excluez des disques, ils ne seront pas présents sur la machine virtuelle Azure après la migration. 

    ![Paramètres des disques](./media/tutorial-migrate-physical-virtual-machines/disks.png)


14. Dans **Passer en revue et démarrer la réplication**, passez en revue les paramètres, puis cliquez sur **Répliquer** pour démarrer la réplication initiale pour les serveurs.

> [!NOTE]
> Vous pouvez mettre à jour les paramètres de réplication à tout moment avant le démarrage de la réplication (**Gérer** > **Réplication des machines**). Vous ne pouvez pas changer les paramètres après le démarrage de la réplication.



## <a name="track-and-monitor"></a>Suivre et superviser

- Quand vous cliquez sur **Répliquer**, un travail Démarrer la réplication commence. 
- Une fois le travail Démarrer la réplication terminé, les machines commencent leur réplication initiale vers Azure.
- Au terme de la réplication initiale, la réplication différentielle commence. Les modifications incrémentielles apportées aux disques locaux sont répliquées régulièrement sur les disques de réplica dans Azure.


Vous pouvez suivre l’état du travail dans les notifications du portail.

Vous pouvez superviser l’état de la réplication en cliquant sur **Réplication de serveurs** dans **Azure Migrate : Server Migration**.
![Superviser la réplication](./media/tutorial-migrate-physical-virtual-machines/replicating-servers.png)

## <a name="run-a-test-migration"></a>Exécuter un test de migration


Quand la réplication différentielle commence, vous pouvez exécuter une migration de test avec les machines virtuelles avant d’exécuter une migration complète vers Azure. Avant la migration, nous vous recommandons vivement d’effectuer cette opération au moins une fois pour chaque machine.

- L’exécution d’une migration de test permet de vérifier que la migration fonctionne comme prévu, sans impact sur les machines locales qui restent opérationnelles et poursuivent la réplication. 
- Une migration de test simule la migration en créant une machine virtuelle Azure à l’aide de données répliquées (il s’agit en général d’une migration vers un réseau virtuel hors production dans votre abonnement Azure).
- Vous pouvez utiliser la machine virtuelle Azure de test répliquée pour valider la migration, effectuer des tests d’applications et résoudre les éventuels problèmes avant la migration complète.

Effectuez une migration de test de la façon suivante :


1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Migration**, cliquez sur **Tester les serveurs migrés**.

     ![Tester les serveurs migrés](./media/tutorial-migrate-physical-virtual-machines/test-migrated-servers.png)

2. Cliquez avec le bouton droit sur la machine virtuelle à tester, puis cliquez sur **Migration de test**.

    ![Migration de test](./media/tutorial-migrate-physical-virtual-machines/test-migrate.png)

3. Dans **Migration de test**, sélectionnez le réseau virtuel Azure dans lequel la machine virtuelle Azure se trouvera après la migration. Nous vous recommandons d’utiliser un réseau virtuel hors production.
4. Le travail **Migration de test** démarre. Supervisez le travail dans les notifications du portail.
5. Une fois la migration terminée, affichez la machine virtuelle Azure migrée dans **Machines virtuelles** dans le portail Azure. Le nom de la machine a le suffixe **-Test**.
6. Une fois le test terminé, cliquez avec le bouton droit sur la machine virtuelle Azure dans **Réplication des machines**, puis cliquez sur **Nettoyer la migration de test**.

    ![Nettoyer la migration](./media/tutorial-migrate-physical-virtual-machines/clean-up.png)


## <a name="migrate-vms"></a>Migrer des machines virtuelles

Après avoir vérifié que la migration de test fonctionne comme prévu, vous pouvez migrer des machines locales.

1. Dans le projet Azure Migrate > **Serveurs** > **Azure Migrate : Server Migration**, cliquez sur **Réplication de serveurs**.

    ![Réplication de serveurs](./media/tutorial-migrate-physical-virtual-machines/replicate-servers.png)

2. Dans **Réplication des machines**, cliquez avec le bouton droit sur la machine virtuelle > **Migrer**.
3. Dans **Migrer** > **Arrêter les machines virtuelles et effectuer une migration planifiée sans perte de données**, sélectionnez **Oui** > **OK**.
    - Si vous ne souhaitez pas arrêter la machine virtuelle, sélectionnez **Non**.
    
    Remarque : Pour la migration de serveurs physiques, il est recommandé de fermer l’application dans le cadre de la fenêtre de migration (ne laissez pas les applications accepter de connexions), puis de lancer la migration (le serveur doit continuer à fonctionner pour que les modifications restantes puissent être synchronisées avant la fin de la migration).

4. Un travail de migration démarre pour la machine virtuelle. Suivez le travail dans les notifications Azure.
5. Une fois le travail terminé, vous pouvez afficher et gérer la machine virtuelle à partir de la page **Machines virtuelles**.

## <a name="complete-the-migration"></a>Effectuer la migration

1. Une fois la migration terminée, cliquez avec le bouton droit sur la machine virtuelle > **Arrêter la migration**. Cette opération effectue les actions suivantes :
    - Arrête la réplication pour l’ordinateur local.
    - Supprime l’ordinateur du nombre **Réplication de serveurs** dans Azure Migrate : Server Migration.
    - Nettoie les informations d’état de réplication pour la machine.
2. Installez l’agent [Windows](../virtual-machines/extensions/agent-windows.md) ou [Linux](../virtual-machines/extensions/agent-linux.md) de machine virtuelle Azure sur les machines migrées.
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
    - Envisagez de déployer [Azure Cost Management](../cost-management-billing/cloudyn/overview.md) pour surveiller l’utilisation et les coûts des ressources.


## <a name="next-steps"></a>Étapes suivantes

Examinez le [parcours de migration cloud](/azure/architecture/cloud-adoption/getting-started/migrate) dans le framework d’adoption du cloud Azure.
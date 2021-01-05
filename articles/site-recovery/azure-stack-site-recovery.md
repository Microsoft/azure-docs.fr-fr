---
title: Répliquer des machines virtuelles Azure Stack à Azure en utilisant Azure Site Recovery | Microsoft Docs
description: Découvrez comment configurer la reprise d’activité sur Azure de machines virtuelles Azure Stack avec le service Azure Site Recovery.
ms.topic: conceptual
ms.date: 08/05/2019
ms.openlocfilehash: 36e11bfe5354644f9ef6603ffe20cb2e86074323
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016900"
---
# <a name="replicate-azure-stack-vms-to-azure"></a>Répliquer des machines virtuelles Azure Stack dans Azure

Cet article explique comment configurer la reprise d’activité de machines virtuelles Azure Stack sur Azure en utilisant le [service Azure Site Recovery](site-recovery-overview.md).

Site Recovery contribue à votre stratégie de continuité d’activité et reprise d’activité (BCDR). Le service veille à ce que vos charges de travail de machine virtuelle restent disponibles en cas d’interruption attendue ou inattendue.

- Site Recovery orchestre et gère la réplication de machines virtuelles sur Stockage Azure.
- En cas d’interruption de votre site principal, Site Recovery vous permet de basculer vers Azure.
- Lors du basculement, des machines virtuelles Azure sont créées à partir des données des machines virtuelles stockées, et les utilisateurs peuvent continuer à accéder aux charges de travail exécutées sur ces machines virtuelles Azure.
- Lorsque tout est à nouveau opérationnel, vous pouvez restaurer les machines virtuelles Azure sur votre site principal et recommencer à répliquer sur Stockage Azure.


Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * **Étape 1 : Préparer des machines virtuelles Azure Stack pour la réplication**. Vérifier que les machines virtuelles sont conformes aux exigences de Site Recovery, et préparer l’installation du service Mobilité Azure Site Recovery. Ce service est installé sur chaque machine à répliquer.
> * **Étape 2 : Configurer un coffre Recovery Services**. Configurer un coffre pour Site Recovery, et spécifier ce que vous souhaitez répliquer. Les actions et composants de Site Recovery sont configurés et gérés dans le coffre.
> * **Étape 3 : Configurer l’environnement de réplication source**. Configurer un serveur de configuration Site Recovery. Le serveur de configuration est une machine virtuelle Azure Stack qui s’exécute tous les composants dont Site Recovery a besoin. Après avoir configuré le serveur de configuration, vous l’inscrivez dans le coffre.
> * **Étape 4 : Configurer l’environnement de réplication cible**. Sélectionnez votre compte Azure, ainsi que le compte de stockage Azure et le réseau que vous souhaitez utiliser. Lors de la réplication, les données des machines virtuelles sont copiées dans Stockage Azure. Après le basculement, les machines virtuelles Azure sont jointes au réseau spécifié.
> * **Étape 5 : Activer la réplication**. Configurer les paramètres de réplication et activer la réplication à partir des machines virtuelles. Le service Mobilité est installé sur une machine virtuelle lors de l’activation de la réplication. Site Recovery effectue une réplication initiale de la machine virtuelle, puis la réplication continue commence.
> * **Étape 6 : Exécuter une simulation de reprise d’activité** : Une fois la réplication opérationnelle, vous vérifiez que le basculement fonctionne comme prévu en exécutant une simulation. Pour lancer la simulation, exécutez un test de basculement dans Site Recovery. Le test de basculement n’affecte pas votre environnement de production.

Une fois ces étapes terminées, vous pouvez exécuter un basculement complet vers Azure comme et quand vous le souhaitez.

## <a name="architecture"></a>Architecture

![Le diagramme montre des coffres Recovery Services pour deux locataires dans des clouds associés à des abonnements de locataire sur une infrastructure Azure Stack commune.](./media/azure-stack-site-recovery/architecture.png)

**Lieu** | **Composant** |**Détails**
--- | --- | ---
**Serveur de configuration** | S’exécute sur une seule machine virtuelle Azure Stack. | Dans chaque abonnement, vous configurez une machine virtuelle serveur de configuration. Cette machine virtuelle exécute les composants Site Recovery suivants :<br/><br/> - Serveur de configuration : coordonne la communication entre les ordinateurs locaux et Azure, et gère la réplication des données. - Serveur de traitement : Fait office de passerelle de réplication. Il reçoit les données de réplication, optimise le processus avec la mise en cache, la compression et le chiffrement, puis envoie les données à Stockage Azure.<br/><br/> Si les machines virtuelles que vous souhaitez répliquer dépassent les limites indiquées ci-dessous, vous pouvez configurer un serveur de processus autonome distinct. [Plus d’informations](vmware-azure-set-up-process-server-scale.md)
**Service de mobilité** | Installé sur chaque machine virtuelle que vous souhaitez répliquer. | Les étapes décrites dans cet article permettent de préparer un compte de façon à ce que le service Mobilité soit installé automatiquement sur une machine virtuelle lorsque la réplication est activée. Si vous ne souhaitez pas installer le service automatiquement, vous pouvez utiliser d’autres méthodes. [Plus d’informations](vmware-azure-install-mobility-service.md)
**Microsoft Azure** | Dans Azure, vous avez besoin d’un coffre Recovery Services, d’un compte de stockage et d’un réseau virtuel. |  Les données répliquées sont stockées dans le compte de stockage. Des machines virtuelles Azure sont ajoutées au réseau Azure lors du basculement.


La réplication fonctionne comme suit :

1. Dans le coffre, vous spécifiez la source et la cible de réplication, configurez le serveur de configuration, créez une stratégie de réplication et activez la réplication.
2. Le service Mobilité est installé sur l’ordinateur (si vous avez utilisé une installation push), et les machines commencent la réplication conformément à la stratégie de réplication.
3. Une copie initiale des données du serveur est répliquée sur Stockage Azure.
4. La réplication des modifications delta dans Azure commence à l’issue de la réplication initiale. Le suivi des modifications d’une machine est conservé dans un fichier .hrl.
5. Le serveur de configuration orchestre la gestion de la réplication avec Azure (port de sortie HTTPS 443).
6. Le serveur de processus optimise et chiffre les données qu’il reçoit des machines sources, puis les envoie à Stockage Azure via le (port de sortie 443).
7. Les machines répliquées communiquent avec le serveur de configuration (port d’entrée HTTPS 443) pour gérer la réplication. Les machines envoient les données de réplication au serveur de processus (port d’entrée HTTPS 9443 ; modifiable).
8. Le trafic est répliqué sur des points de terminaison publics de stockage Azure via Internet. Une autre solution consiste à utiliser un peering public Azure ExpressRoute. La réplication du trafic à partir d’un site local vers Azure via un réseau VPN de site à site n’est pas prise en charge.

## <a name="prerequisites"></a>Prérequis

Voici les éléments requis pour configurer ce scénario.

**Prérequis** | **Détails**
--- | ---
**Compte d’abonnement Azure** | Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/).
**Autorisations du compte Azure** | Le compte Azure que vous utilisez doit être autorisé à effectuer les actions suivantes :<br/><br/> - Créer un coffre Recovery Services<br/><br/> - Créer une machine virtuelle dans le groupe de ressources et le réseau virtuel que vous utilisez pour le scénario<br/><br/> - Écrire dans le compte de stockage que vous spécifiez<br/><br/> Notez les points suivants :<br/><br/> \- Si vous créez un compte gratuit, vous êtes l’administrateur de votre abonnement et pouvez effectuer toutes les actions.<br/><br/> - Si vous utilisez un abonnement existant dont vous n’êtes pas l’administrateur, vous devez collaborer avec l’administrateur pour qu’il vous accorde les autorisations Propriétaire ou Contributeur.<br/><br/> - Si vous avez besoin d’autorisations plus précises, voir [cet article](site-recovery-role-based-linked-access-control.md).
**Machine virtuelle Azure Stack** | Vous avez besoin d’une machine virtuelle Azure Stack dans l’abonnement du locataire, qui sera déployée en tant que serveur de configuration Site Recovery.


### <a name="prerequisites-for-the-configuration-server"></a>Conditions préalables pour le serveur de configuration

[!INCLUDE [site-recovery-config-server-reqs-physical](../../includes/site-recovery-config-server-reqs-physical.md)]



## <a name="step-1-prepare-azure-stack-vms"></a>Étape 1 : Préparer les machines virtuelles Azure Stack

### <a name="verify-the-operating-system"></a>Vérifier le système d’exploitation

Assurez-vous que les machines virtuelles exécutent l’un des systèmes d’exploitation répertoriés dans le tableau.


**Système d’exploitation** | **Détails**
--- | ---
**Windows 64 bits** | Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 (à partir de SP1)
**CentOS** | 5.2 à 5.11, 6.1 à 6.9, 7.0 à 7.3
**Ubuntu** | Serveur LTS 14.04, Serveur LTS 16.04. Passer en revue les [noyaux pris en charge](vmware-physical-azure-support-matrix.md#ubuntu-kernel-versions)

### <a name="prepare-for-mobility-service-installation"></a>Préparer l’installation du service Mobilité

Le service Mobilité doit être installé sur toutes les machines virtuelles à répliquer. Pour que le serveur de processus installe automatiquement le service sur la machine virtuelle quand la réplication est activée, vérifiez les paramètres de la machine virtuelle.

#### <a name="windows-machines"></a>Machines Windows

- Vous avez besoin d’une connexion réseau entre la machine virtuelle sur laquelle vous souhaitez activer la réplication et l’ordinateur exécutant le serveur de processus (par défaut, la machine virtuelle du serveur configuration).
- Vous avez besoin d’un compte disposant de droits administratifs (de domaine ou locaux) sur la machine pour laquelle vous activez la réplication.
    - Vous spécifiez ce compte quand vous configurez Site Recovery. Le serveur de processus utilise ensuite ce compte pour installer le service Mobilité quand la réplication est activée.
    - Ce compte sera utilisé uniquement par Site Recovery pour l’installation push et la mise à jour du service Mobilité.
    - Si vous n’utilisez pas de compte de domaine, vous devez désactiver le contrôle d’accès des utilisateurs distants sur la machine virtuelle :
        - Dans le Registre, créez la valeur DWORD **LocalAccountTokenFilterPolicy** sous HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System.
        - Définissez la valeur sur 1.
        - Pour exécuter cette action à l’invite de commandes, tapez la commande suivante : **REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1**.
- Dans le Pare-feu Windows sur la machine virtuelle à répliquer, autorisez le partage de fichiers et d’imprimantes ainsi que WMI.
    - Pour ce faire, exécutez **wf.msc** pour ouvrir la console du Pare-feu Windows. Cliquez avec le bouton droit sur **Règles de trafic entrant** > **Nouvelle règle**. Sélectionnez **Prédéfinie**, puis choisissez **Partage de fichiers et d’imprimantes** dans la liste. Suivez la procédure de l’Assistant, choisissez d’autoriser la connexion > **Terminer**.
    - Pour les ordinateurs du domaine, vous pouvez utiliser un objet de stratégie de groupe (GPO) à cette fin.


#### <a name="linux-machines"></a>Machines Linux

- Vérifiez qu’il existe une connectivité réseau entre l’ordinateur Linux et le serveur de processus.
- Sur la machine pour laquelle vous activez la réplication, vous avez besoin d’un compte d’utilisateur racine sur le serveur Linux source :
    - Vous spécifiez ce compte quand vous configurez Site Recovery. Le serveur de processus utilise ensuite ce compte pour installer le service Mobilité quand la réplication est activée.
    - Ce compte sera utilisé uniquement par Site Recovery pour l’installation push et la mise à jour du service Mobilité.
- Vérifiez que le fichier /etc/hosts sur le serveur Linux source contient des entrées qui mappent le nom d’hôte local sur les adresses IP associées à toutes les cartes réseau.
- Installez les packages openssh, openssh-server et openssl les plus récents sur l’ordinateur à répliquer.
- Vérifiez que le protocole Secure Shell (SSH) est activé et s’exécute sur le port 22.
- Activez le sous-système SFTP et l'authentification par mot de passe dans le fichier sshd_config :
    1. Pour ce faire, connectez-vous en tant que racine.
    2. Recherchez la ligne commençant par **PasswordAuthentication**, dans le fichier /etc/ssh/sshd_config. Supprimez les commentaires de la ligne et remplacez la valeur par **oui**.
    3. Recherchez la ligne commençant par **Subsystem** et supprimez la mise en commentaire de la ligne.

        ![Service Mobilité Linux](./media/azure-stack-site-recovery/linux-mobility.png)

    4. Redémarrez le service sshd.


### <a name="note-the-vm-private-ip-address"></a>Noter l’adresse IP privée de la machine virtuelle

Pour chaque machine à répliquer, recherchez son adresse IP :

1. Dans le portail Azure Stack, cliquez sur la machine virtuelle.
2. Dans le menu **Ressource**, cliquez sur **Interfaces réseau**.
3. Notez l’adresse IP privée.

    ![Adresse IP privée](./media/azure-stack-site-recovery/private-ip.png)


## <a name="step-2-create-a-vault-and-select-a-replication-goal"></a>Étape 2 : Créer un coffre et sélectionner un objectif de réplication

1. Dans le portail Azure, sélectionnez **Créer une ressource** > **Outils de gestion** > **Backup and Site Recovery (Sauvegarde et Site Recovery)** .
2. Dans **Name**, entrez un nom convivial pour identifier le coffre.
3. Dans **Groupe de ressources**, créez ou sélectionnez un groupe de ressources. Nous utilisons **contosoRG**.
4. Dans **Emplacement**, entrez la région Azure. Nous utilisons **Europe Ouest**.
5. Pour accéder rapidement au coffre à partir du tableau de bord, sélectionnez **Épingler au tableau de bord** > **Créer**.

   ![Créer un coffre](./media/azure-stack-site-recovery/new-vault-settings.png)

   Le nouveau coffre apparaît dans **Tableau de bord** > **Toutes les ressources** et dans la page principale **Coffres Recovery Services**.

### <a name="select-a-replication-goal"></a>Sélectionner un objectif de réplication

1. Dans **Coffres Recovery Services**, spécifiez un nom de coffre. Nous utilisons **ContosoVMVault**.
2. Dans **Prise en main**, sélectionnez Site Recovery. Sélectionnez ensuite **Préparer l’infrastructure**.
3. Dans **Objectif de protection** > **Où se trouvent vos machines**, sélectionnez **Local**.
4. Dans **Où voulez-vous répliquer vos machines**, sélectionnez **Dans Azure**.
5. Dans **Vos machines sont-elles virtualisées**, sélectionnez **Non virtualisé / autre**. Sélectionnez ensuite **OK**.

    ![Objectif de protection](./media/azure-stack-site-recovery/protection-goal.png)

## <a name="step-3-set-up-the-source-environment"></a>Étape 3 : Configurer l’environnement source

Configurez la machine serveur de configuration, inscrivez-la dans le coffre, puis découvrez les machines que vous souhaitez répliquer.

1. Cliquez sur **Préparer l’infrastructure** > **Source**.
2. Dans **Préparer la source**, cliquez sur **+ Serveur de configuration**.

    ![Capture d’écran de la boîte de dialogue +Serveur de configuration avec le message « Cliquez sur +Serveur de configuration dans la barre de commandes ci-dessus pour configurer un... ».](./media/azure-stack-site-recovery/plus-config-srv.png)

3. Dans **Ajouter un serveur**, vérifiez que **Serveur de configuration** s’affiche dans **Type de serveur**.
5. Téléchargez le fichier d’installation unifiée de Site Recovery.
6. Téléchargez la clé d’inscription du coffre. Vous avez besoin de la clé d’inscription lorsque vous exécutez le programme d’installation unifiée. Une fois générée, la clé reste valide pendant 5 jours.

    ![Capture d’écran de la boîte de dialogue Ajouter un serveur, avec le type de serveur défini sur Serveur de configuration, et le bouton Télécharger la clé d’inscription du coffre mis en évidence.](./media/azure-stack-site-recovery/set-source2.png)


### <a name="run-azure-site-recovery-unified-setup"></a>Exécuter le programme d’installation unifiée Azure Site Recovery

Pour installer et inscrire le serveur de configuration, établissez une connexion RDP à la machine virtuelle que vous souhaitez utiliser pour le serveur de configuration, puis exécutez Installation unifiée.

Avant de commencer, assurez-vous que l’horloge est [synchronisée avec un serveur de temps](/windows-server/networking/windows-time-service/windows-time-service-top) sur la machine virtuelle. L’installation échoue si l’heure est décalée de plus de cinq minutes par rapport à l’heure locale.

Installez à présent le serveur de configuration :

[!INCLUDE [site-recovery-add-configuration-server](../../includes/site-recovery-add-configuration-server.md)]

> [!NOTE]
> Vous pouvez également installer le serveur de configuration à partir de la ligne de commande. [Plus d’informations](physical-manage-configuration-server.md#install-from-the-command-line)
>
> L’affichage du nom de compte dans le portail peut prendre plus de 15 minutes. Pour procéder à une mise à jour immédiate, sélectionnez **Serveurs de configuration** > **_nom du serveur_ *_ > _* Actualiser le serveur**.

## <a name="step-4-set-up-the-target-environment"></a>Étape 4 : Configurer l’environnement cible

Sélectionnez et vérifiez les ressources cibles.

1. Dans **Préparer l’infrastructure** > **Cible**, sélectionnez l’abonnement Azure à utiliser.
2. Spécifiez le modèle de déploiement cible.
3. Site Recovery vérifie que vous disposez d’un ou de plusieurs réseaux et comptes Azure Storage compatibles. S’il n’en trouve pas, vous devez créer au moins un compte de stockage et un réseau virtuel pour suivre la procédure de l’Assistant.


## <a name="step-5-enable-replication"></a>Étape 5 : Activer la réplication

### <a name="create-a-replication-policy"></a>Créer une stratégie de réplication

1. Cliquez sur **Préparer l’infrastructure** > **Paramètres de réplication**.
2. Dans **Créer une stratégie de réplication**, indiquez le nom de la stratégie.
3. Dans **Seuil d’objectif de point de récupération**, spécifiez la limite de l’objectif de point de récupération (RPO).
    - Des points de récupération pour les données répliquées sont créés en fonction de l’heure définie.
    - Ce paramètre n’affecte pas la réplication qui est continue. Il sert simplement à émettre une alerte si le seuil limite est atteint sans que cela déclenche la création d’un point de récupération.
4. Dans **Rétention des points de récupération**, spécifiez le temps de rétention de chaque point de récupération. Les machines virtuelles répliquées peuvent être restaurées à n’importe quel point de récupération dans la fenêtre de temps spécifiée.
5. Dans **Fréquence des captures instantanées de cohérence d’application**, spécifiez la fréquence de création d’instantanés de cohérence de l’application.

    - Un instantané de cohérence d’application est une capture instantanée à un point dans le temps des données d’application à l’intérieur de la machine virtuelle.
    - Le service VSS s’assure que les applications sur la machine virtuelle sont dans un état cohérent au moment de la capture instantanée.
6. Sélectionnez **OK** pour créer la stratégie.


### <a name="confirm-deployment-planning"></a>Confirmer la planification d’un déploiement

Vous pouvez ignorer cette étape pour l’instant. Dans la liste déroulante **Planification du déploiement**, cliquez sur **Oui, je l’ai fait**.



### <a name="enable-replication"></a>Activer la réplication

Vérifiez que vous avez accompli toutes les tâches décrites dans [Étape 1 : Préparer la machine](#step-1-prepare-azure-stack-vms). Activez ensuite la réplication comme suit :

1. Sélectionnez **Répliquer l’application** > **Source**.
2. Dans **Source**, sélectionnez le serveur de configuration.
3. Dans **Type de machine**, sélectionnez **Machines physiques**.
4. Sélectionnez le serveur de processus (serveur de configuration). Cliquez ensuite sur **OK**.
5. Dans **Cible**, sélectionnez l’abonnement et le groupe de ressources dans lesquels vous voulez créer les machines virtuelles après le basculement. Choisissez le modèle de déploiement à utiliser dans Azure pour les machines virtuelles basculées.
6. Sélectionnez le compte de stockage Azure dans lequel vous souhaitez stocker les données répliquées.
7. Sélectionnez le sous-réseau et le réseau Azure auxquels les machines virtuelles Azure se connectent lorsqu’elles sont créées après le basculement.
8. Sélectionnez **Effectuez maintenant la configuration pour les machines sélectionnées** pour appliquer le paramètre réseau à l’ensemble des machines que vous sélectionnez à des fins de protection. Sélectionnez **Configurer ultérieurement** si vous souhaitez sélectionner le réseau Azure séparément pour chaque machine.
9. Dans **Machines physiques**, cliquez sur **+Machines physiques**. Spécifiez le nom, l’adresse IP et le type de système d’exploitation de chaque machine à répliquer.

    - Utilisez l’adresse IP interne de la machine.
    - Si vous spécifiez l’adresse IP publique, il se peut que la réplication ne fonctionne pas comme prévu.

10. Dans **Propriétés** > **Configurer les propriétés**, sélectionnez le compte que le serveur de processus doit utiliser pour installer automatiquement le service Mobilité sur la machine.
11. Dans **Paramètres de réplication** > **Configurer les paramètres de réplication**, vérifiez que la stratégie de réplication sélectionnée est correcte.
12. Cliquez sur **Activer la réplication**.
13. Suivez la progression du travail **Activer la protection** dans **Paramètres** > **Travaux** > **Travaux Site Recovery**. Une fois le travail **Finaliser la protection** exécuté, la machine est prête pour le basculement.

> [!NOTE]
> Site Recovery installe le service Mobilité quand la réplication est activée pour une machine virtuelle.
>
> Il peut être nécessaire d’attendre 15 minutes ou plus avant que les modifications prennent effet et qu’elles apparaissent dans le portail.
>
> Pour surveiller les machines virtuelles que vous ajoutez, consultez l’heure de la dernière découverte des machines virtuelles dans **Serveurs de configuration** > **Dernier contact à**. Pour ajouter des machines virtuelles sans attendre la découverte planifiée, mettez en surbrillance le serveur de configuration (sans le sélectionner) et sélectionnez **Actualiser**.


## <a name="step-6-run-a-disaster-recovery-drill"></a>Étape 6 : Exécuter une simulation de récupération d'urgence

Vous exécutez un test de basculement vers Azure pour vérifier que tout fonctionne comme prévu. Ce basculement n’affecte pas votre environnement de production.

### <a name="verify-machine-properties"></a>Vérifier les propriétés de la machine virtuelle

Avant d’exécuter un test de basculement, vérifiez les propriétés de la machine virtuelle, et assurez-vous qu’elles sont conformes à la [configuration requise pour Azure](vmware-physical-azure-support-matrix.md#azure-vm-requirements). Vous pouvez afficher et modifier les propriétés comme suit :

1. Dans **Éléments protégés**, cliquez sur **Éléments répliqués** > Machine virtuelle.
2. Dans le volet **Élément répliqué**, vous voyez un récapitulatif des informations de la machine virtuelle, son état d’intégrité et ses derniers points de récupération disponibles. Cliquez sur **Propriétés** pour obtenir plus de détails.
3. Dans **Calcul et réseau**, modifiez les paramètres selon vos besoins.

    - Vous pouvez modifier les paramètres de nom de machine virtuelle Azure, de groupe de ressources, de taille cible, de [groupe à haute disponibilité](../virtual-machines/windows/tutorial-availability-sets.md) et de disque managé.
    - Vous pouvez également afficher et modifier les paramètres réseau. Ceux-ci incluent le réseau/sous-réseau auquel la machine virtuelle Azure sera jointe après le basculement, ainsi que l’adresse IP qui sera affectée à la machine virtuelle.
1. Dans **Disques**, vous pouvez voir les informations relatives au système d’exploitation aux disques de données de la machine virtuelle.


### <a name="run-a-test-failover"></a>Exécuter un test de basculement

Quand vous effectuez un test de basculement, voici ce qui se produit :

1. Une vérification des prérequis est effectuée pour garantir que toutes les conditions nécessaires pour le basculement sont en place.
2. Le basculement traite les données en utilisant le point de récupération spécifié :
    - **Dernier point traité** : la machine bascule vers le dernier point de récupération traité par Site Recovery. L’horodatage est affiché. Cette option, avec laquelle aucun temps n’est passé à traiter les données, offre un objectif de délai de récupération faible.
    - **Dernier point de cohérence des applications** : La machine virtuelle bascule vers le dernier point de récupération de cohérence des applications.
    - **Personnalisé** : Sélectionnez le point de récupération utilisé pour le basculement.

3. Une machine virtuelle Azure est créée à l’aide des données traitées.
4. Un test de basculement peut nettoyer automatiquement les machines virtuelles Azure créées durant la simulation.

Pour exécuter un test de basculement pour une machine virtuelle, procédez comme suit :

1. Dans **Paramètres** > **Éléments répliqués**, cliquez sur Machine virtuelle > **+Test de basculement**.
2. Pour cette procédure pas à pas, nous allons utiliser le point de récupération **Dernier point traité**.
3. Dans **Test de basculement**, sélectionnez le réseau Azure cible.
4. Cliquez sur **OK** pour commencer le basculement.
5. Vous pouvez suivre la progression en cliquant sur la machine virtuelle pour ouvrir ses propriétés. Vous pouvez également cliquer sur le travail **Test de basculement** dans *nom du coffre* > **Paramètres** > **Travaux** >**Travaux Site Recovery**.
6. Une fois le basculement terminé, la machine virtuelle Azure de réplication apparaît dans le portail Azure > **Machines virtuelles**. Vérifiez que la machine virtuelle est de la taille appropriée, qu’elle est connectée au bon réseau et qu’elle est en cours d’exécution.
7. Vous devriez à présent pouvoir vous connecter à la machine virtuelle répliquée dans Azure. [Plus d’informations](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover)
8. Pour supprimer les machines virtuelles créées lors du test de basculement, cliquez sur **Nettoyer le test de basculement** sur la machine virtuelle. Dans **Notes**, enregistrez d’éventuelles observations sur le test de basculement.

## <a name="fail-over-and-fail-back"></a>Basculement et restauration automatique

Après avoir configuré la réplication et exécuté une simulation de vous assurer que tout fonctionne, vous pouvez basculer des machines vers Azure en fonction des besoins.

Avant d’exécuter un basculement, si vous souhaitez vous connecter à la machine dans Azure après le basculement, vous devez [préparer la connexion](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover) avant de commencer.

Ensuite, exécutez un basculement comme suit :


1. Dans **Paramètres** > **Éléments répliqués**, cliquez sur la machine > **Basculement**.
2. Sélectionnez le point de récupération que vous souhaitez utiliser.
3. Dans **Test de basculement**, sélectionnez le réseau Azure cible.
4. Sélectionnez **Arrêter la machine avant de commencer le basculement**. Avec ce paramètre, Site Recovery tente d’arrêter la machine source avant de commencer le basculement. Le basculement se poursuit même en cas d’échec de l’arrêt.
5. Cliquez sur **OK** pour commencer le basculement. Vous pouvez suivre la progression du basculement sur la page **Tâches**.
6. Une fois le basculement terminé, la machine virtuelle Azure de réplication apparaît dans le portail Azure > **Machines virtuelles**. Si vous avez préparé la connexion après basculement, vérifiez que la machine virtuelle est de la taille appropriée, qu’elle est connectée au bon réseau et qu’elle est en cours d’exécution.
7. Après avoir vérifié la machine virtuelle, cliquez sur **Valider** pour achever le basculement. Cela a pour effet de supprimer tous les points de récupération disponibles.

> [!WARNING]
> N’annulez pas un basculement en cours : Avant le démarrage du basculement, la réplication de la machine virtuelle est arrêtée. Si vous annulez un basculement en cours, le basculement s’arrête mais la machine virtuelle ne sera pas à nouveau répliquée.


### <a name="fail-back-to-azure-stack"></a>Restaurer automatiquement sur Azure Stack

Lorsque votre site principal est à nouveau opérationnel, vous pouvez opérer une restauration automatique d’Azure vers Azure Stack. Pour ce faire, effectuez les étapes [listées ici](/azure-stack/operator/site-recovery-failback?view=azs-2005).

## <a name="conclusion"></a>Conclusion

Dans le cadre de cet article, nous avons répliqué des machines virtuelles Azure Stack sur Azure. Une fois la réplication configurée, nous avons effectué une simulation de reprise d’activité pour vérifier que le basculement vers Azure fonctionnait comme prévu. L’article nous a également permis de découvrir les étapes permettant d’effectuer un basculement complet vers Azure ainsi qu’une restauration automatique vers Azure Stack.

## <a name="next-steps"></a>Étapes suivantes

Après une restauration automatique, vous pouvez reprotéger la machine virtuelle et recommencer à la répliquer sur Azure. Pour ce faire, répétez les étapes décrites dans cet article.
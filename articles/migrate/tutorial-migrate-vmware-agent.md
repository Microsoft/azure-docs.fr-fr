---
title: Migrer des machines virtuelles VMware à l’aide de la migration de serveur Azure Migrate avec agent
description: Découvrez comment exécuter la migration avec agent des machines virtuelles VMware avec Azure Migrate.
author: rahulg1190
ms.author: rahugup
ms.manager: bsiva
ms.topic: tutorial
ms.date: 06/09/2020
ms.custom: MVC
ms.openlocfilehash: 0c3b86fc823e4b563dc747a5604fcb2e04995fed
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752649"
---
# <a name="migrate-vmware-vms-to-azure-agent-based"></a>Migrer des machines virtuelles VMware vers Azure (migration basée sur un agent)

Cet article explique comment effectuer la migration de machines virtuelles VMware locales vers Azure avec l’outil [Azure Migrate : Migration de serveur](migrate-services-overview.md#azure-migrate-server-migration-tool), en utilisant la migration basée sur un agent.  Vous pouvez également migrer des machines virtuelles VMware à l’aide de la migration basée sur un agent. [Comparez](server-migrate-overview.md#compare-migration-methods) les méthodes.


 Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Préparez Azure pour qu’il fonctionne avec Azure Migrate.
> * Préparer une migration basée sur un agent. Configurer un compte VMware afin de permettre à Azure Migrate de détecter les machines pour la migration. Configurer un compte afin que l’agent du service Mobility puisse être installé sur les machines que vous souhaitez migrer, et préparer une machine en tant qu’appliance de réplication.
> * Ajouter l’outil Azure Migrate : Migration de serveur
> * Configurer l’appliance de réplication
> * Répliquer des machines virtuelles.
> * Exécuter une migration de test pour vérifier que tout fonctionne comme prévu.
> * Exécuter une migration complète vers Azure.

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario donné afin que vous puissiez configurer rapidement une preuve de concept. Ils utilisent des options par défaut, le cas échéant, et ne montrent pas tous les paramètres et chemins possibles. 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.


## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, [examinez](./agent-based-migration-architecture.md) l’architecture de la migration VMware basée sur un agent.

## <a name="prepare-azure"></a>Préparer Azure

Effectuez les tâches du tableau pour préparer Azure en vue de la migration basée sur un agent.

**Tâche** | **Détails**
--- | ---
**Créer un projet Azure Migrate** | Votre compte Azure doit disposer d’autorisations Contributeur ou Propriétaire pour créer un projet.
**Vérifier les autorisations du compte Azure** | Votre compte Azure a besoin d’autorisations pour créer une machine virtuelle et écrire sur un disque managé Azure.
**Configurer un réseau Azure** | Configurez un réseau auquel joindre les machines virtuelles Azure après la migration.

### <a name="assign-permissions-to-create-project"></a>Attribuer des autorisations pour créer un projet
Si vous n’avez pas de projet Azure Migrate, vérifiez les autorisations pour en créer un.


1. Dans le portail Azure, ouvrez l’abonnement, puis sélectionnez **Contrôle d’accès (IAM)** .
2. Dans **Vérifier l’accès**, recherchez le compte approprié, puis cliquez dessus pour voir les autorisations correspondantes.
3. Vérifiez que vous disposez des autorisations de **Contributeur** ou de **Propriétaire**.

    - Si vous venez de créer un compte Azure gratuit, vous êtes le propriétaire de votre abonnement.
    - Si vous n’êtes pas le propriétaire de l’abonnement, demandez au propriétaire de vous attribuer le rôle.
    
### <a name="assign-azure-account-permissions"></a>Affecter des autorisations de compte Azure

Affectez le rôle Contributeur de machines virtuelles au compte, ce qui vous permettra d’effectuer les tâches suivantes :

- Créer une machine virtuelle dans le groupe de ressources sélectionné
- Créer une machine virtuelle dans le réseau virtuel sélectionné
- Écrire sur un disque managé Azure. 


### <a name="set-up-an-azure-network"></a>Configurer un réseau Azure

[Configurez un réseau Azure](../virtual-network/manage-virtual-network.md#create-a-virtual-network). Les machines locales sont répliquées sur des disques managés Azure. Quand vous basculez vers Azure pour la migration, les machines virtuelles Azure sont créées à partir de ces disques managés, et elles sont jointes au réseau Azure que vous avez configuré.

## <a name="prepare-for-migration"></a>Préparation de la migration

Vérifiez les conditions de prise en charge et les autorisations, et préparez le déploiement d’une appliance de réplication. 

### <a name="prepare-an-account-to-discover-vms"></a>Préparer un compte pour la détection des machines virtuelles

Azure Migrate : Migration de serveur doit accéder aux serveurs VMware pour détecter les machines virtuelles que vous comptez migrer. Créez le compte comme suit :

1. Pour utiliser un compte dédié, créez un rôle au niveau du vCenter. Donnez un nom au rôle, tel que **Azure_Migrate**.
2. Attribuez au rôle les autorisations résumées dans le tableau ci-dessous.
3. Créez un utilisateur sur le serveur vCenter ou sur l’hôte vSphere. Affectez le rôle à l’utilisateur.

#### <a name="vmware-account-permissions"></a>Autorisations du compte VMware

**Tâche** | **Rôle/Autorisations** | **Détails**
--- | --- | ---
**Détection des machines virtuelles** | Au moins un utilisateur en lecture seule<br/><br/> Objet de centre de données -> Propager vers l’objet enfant, rôle = lecture seule | L’utilisateur est affecté au niveau du centre de données et a accès à tous les objets dans le centre de données.<br/><br/> Pour restreindre l’accès, attribuez le rôle **Aucun accès** avec **Propager vers l’objet enfant** aux objets enfants (hôtes vSphere, magasins de données, machines virtuelles et réseaux).
**Réplication** |  Créez un rôle (Azure_Site_Recovery) avec les autorisations nécessaires, puis attribuez le rôle à un utilisateur ou à un groupe d’utilisateurs VMware<br/><br/> Objet de centre de données -> Propager vers l’objet enfant, rôle = Azure_Site_Recovery<br/><br/> Banque de données -> Allouer de l’espace, parcourir la banque de données, opérations de fichier de bas niveau, supprimer le fichier, mettre à jour les fichiers de machine virtuelle<br/><br/> Réseau -> Attribution de réseau<br/><br/> Ressource -> Affecter les machines virtuelles au pool de ressources, migrer des machines virtuelles hors tension, migrer des machines virtuelles sous tension<br/><br/> Tâches -> Créer une tâche, Mettre à jour une tâche<br/><br/> Machine virtuelle -> Configuration<br/><br/> Machine virtuelle -> Interagir -> répondre à la question, connexion d’appareil, configurer un support de CD, configurer une disquette, mettre hors tension, mettre sous tension, installation des outils VMware<br/><br/> Machine virtuelle -> Stock -> Créer, inscrire, désinscrire<br/><br/> Machine virtuelle -> Approvisionnement -> Autoriser le téléchargement de machines virtuelles, autoriser le chargement de fichiers de machine virtuelle<br/><br/> Machine virtuelle -> Instantanés -> Supprimer les instantanés | L’utilisateur est affecté au niveau du centre de données et a accès à tous les objets dans le centre de données.<br/><br/> Pour restreindre l’accès, attribuez le rôle **Aucun accès** avec **Propager vers l’objet enfant** aux objets enfants (hôtes vSphere, magasins de données, machines virtuelles et réseaux).

### <a name="prepare-an-account-for-mobility-service-installation"></a>Préparer un compte pour l’installation du service Mobilité

Le service Mobilité doit être installé sur les machines virtuelles que vous souhaitez répliquer.

- L’appliance de réplication Azure Migrate peut effectuer une installation de type push de ce service quand vous activez la réplication pour une machine, ou vous pouvez l’installer manuellement ou à l’aide des outils d’installation.
- Dans ce tutoriel, nous allons installer le service Mobilité avec l’installation de type push.
- Pour l’installation de type push, vous devez préparer un compte dont Azure Migrate Server Migration peut se servir pour accéder à la machine virtuelle. Ce compte est utilisé uniquement pour l’installation Push, si vous n’installez pas le service Mobility manuellement.

Préparez le compte comme suit :

1. Préparez un domaine ou un compte local avec les autorisations nécessaires pour l’installation sur la machine virtuelle.
2. Pour les machines virtuelles Windows, si vous n’utilisez pas de compte de domaine, désactivez le contrôle d’accès des utilisateurs distants sur la machine locale. Pour cela, ajoutez l’entrée DWORD **LocalAccountTokenFilterPolicy** dans le Registre, sous **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System**.
3. Pour des machines virtuelles Linux, préparez un compte racine sur le serveur Linux source.


### <a name="prepare-a-machine-for-the-replication-appliance"></a>Préparer une machine pour l’appliance de réplication

L’appliance est utilisée durant la réplication des machines vers Azure. L’appliance est une machine virtuelle VMware locale unique et hautement disponible qui héberge les composants suivants :

- **Serveur de configuration** : Le serveur de configuration coordonne la communication entre les ordinateurs locaux et Azure.et gère la réplication des données.
- **Serveur de traitement**: Le serveur de processus fait office de passerelle de réplication. Il reçoit les données de réplication, les optimise grâce à une mise en cache, une compression et un chiffrement, puis les envoie à un compte de stockage de cache dans Azure. De plus, le serveur de processus installe l’agent du service Mobilité sur les machines virtuelles que vous souhaitez répliquer, puis effectue la détection automatique des machines virtuelles VMware locales.

Préparez l’appliance de la façon suivante :

- [Vérifiez la configuration requise de l’appliance](migrate-replication-appliance.md#appliance-requirements). En règle générale, vous configurez l’appliance de réplication en tant que machine virtuelle VMware au moyen d’un fichier OVA préalablement téléchargé. Le modèle crée une appliance qui présente la configuration requise.
- MySQL doit être installé sur l’appliance. [Examinez](migrate-replication-appliance.md#mysql-installation) les méthodes d’installation.
- Examinez les [URL de cloud public](migrate-replication-appliance.md#url-access) et les [URL Azure Government](migrate-replication-appliance.md#azure-government-url-access) auxquelles la machine de l’appliance doit accéder.
- [Examinez les ports](migrate-replication-appliance.md#port-access) auxquels la machine de l’appliance de réplication doit accéder.



### <a name="check-vmware-requirements"></a>Vérifier les conditions requises VMware

Vérifiez que les machines virtuelles et serveurs VMware respectent les exigences de la migration vers Azure. 

1. [Vérifiez](migrate-support-matrix-vmware-migration.md#vmware-requirements-agent-based) les conditions requises pour les serveurs VMware.
2. [Vérifiez](migrate-support-matrix-vmware-migration.md#vm-requirements-agent-based) la configuration requise des machines virtuelles pour la migration.
3. Vérifiez les paramètres Azure. Les machines virtuelles locales que vous répliquez vers Azure doivent respecter les [exigences relatives aux machines virtuelles Azure](migrate-support-matrix-vmware-migration.md#azure-vm-requirements).
4. Certaines modifications doivent être apportées aux machines virtuelles afin qu’elles puissent être migrées vers Azure.
    - Il est important d’effectuer ces modifications avant de commencer la migration. Si vous migrez la machine virtuelle avant d’effectuer la modification, elle risque de ne pas démarrer dans Azure.
    - Examinez les modifications à apporter aux machines [Windows](prepare-for-migration.md#windows-machines) et [Linux](prepare-for-migration.md#linux-machines).

> [!NOTE]
> La migration basée sur un agent avec Azure Migrate Server Migration est basée sur les fonctionnalités du service Azure Site Recovery. Certaines exigences peuvent pointer vers la documentation de Site Recovery.

## <a name="set-up-the-replication-appliance"></a>Configurer l’appliance de réplication

Cette procédure décrit comment configurer l’appliance à l’aide d’un modèle OVA (Open Virtualization Application). Si vous ne pouvez pas utiliser cette méthode, configurez l’appliance [en utilisant un script](tutorial-migrate-physical-virtual-machines.md#set-up-the-replication-appliance). 

### <a name="download-the-replication-appliance-template"></a>Télécharger le modèle d’appliance de réplication

Téléchargez le modèle comme suit :

1. Dans le projet Azure Migrate, cliquez sur **Serveurs** sous **Objectifs de migration**.
2. Dans la page **Azure Migrate - Serveurs** > **Azure Migrate : Server Migration**, cliquez sur **Découvrir**.

    ![Détection des machines virtuelles](./media/tutorial-migrate-vmware-agent/migrate-discover.png)

3. Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , cliquez sur **Oui, avec l’hyperviseur vSphere VMware**.
4. Dans **Quel type de migration voulez-vous utiliser ?** , sélectionnez **Utilisation de la réplication basée sur un agent**.
5. Dans **Région cible**, sélectionnez la région Azure vers laquelle vous souhaitez migrer les machines.
6. Sélectionnez **Confirmer que la région cible pour la migration est « nom_région »** .
7. Cliquez sur **Créer des ressources**. Un coffre Azure Site Recovery est créé en arrière-plan. Vous ne pouvez pas changer la région cible du projet après avoir cliqué sur ce bouton : toutes les migrations suivantes seront effectuées dans cette région.

    ![Créer un coffre Recovery Services](./media/tutorial-migrate-vmware-agent/create-resources.png)

8. Dans **Voulez-vous installer une nouvelle appliance de réplication ?** , sélectionnez **Installer une appliance de réplication**.
9. Cliquez sur **Télécharger**. Un modèle OVF est téléchargé.
    ![Télécharger OVA](./media/tutorial-migrate-vmware-agent/download-ova.png)
10. Notez le nom du groupe de ressources et du coffre Recovery Services. Vous en aurez besoin lors du déploiement de l’appliance.


### <a name="import-the-template-in-vmware"></a>Importer le modèle dans VMware

Après avoir téléchargé le modèle OVF, importez-le dans VMware pour créer l’application de réplication sur une machine virtuelle VMware exécutant Windows Server 2016.

1. Connectez-vous au serveur VMware vCenter ou à l’hôte vSphere ESXi à l’aide du client VMware vSphere.
2. Dans le menu **Fichier**, sélectionnez **Déployer le modèle OVF** pour démarrer l’**Assistant Déploiement du modèle OVF**. 
3. Dans **Sélectionner une source**, saisissez l’emplacement du modèle OVF téléchargé.
4. Dans **Examiner les détails**, sélectionnez **Suivant**.
5. Dans **Sélectionner le nom et le dossier** et **Sélectionner la configuration**, acceptez les paramètres par défaut.
6. Dans **Sélectionner un stockage** > **Sélectionner le format de disque virtuel**, sélectionnez **Thick Provision Eager Zeroed** pour des performances optimales.
7. Dans le reste des pages de l’Assistant, acceptez les paramètres par défaut.
8. Dans **Prêt à terminer**, pour configurer la machine virtuelle avec les paramètres par défaut, sélectionnez **Mettre sous tension après le déploiement** > **Terminer**.

   > [!TIP]
   > Si vous souhaitez ajouter une carte réseau supplémentaire, décochez la case **Mettre sous tension après le déploiement** > **Terminer**. Par défaut, le modèle contient une seule carte réseau. mais vous pouvez en ajouter d’autres après le déploiement.

### <a name="start-appliance-setup"></a>Démarrer l’installation de l’appliance

1. Dans la console du client vSphere de VMware, activez la machine virtuelle. La machine virtuelle démarre sur une expérience d’installation de Windows Server 2016.
2. Acceptez le contrat de licence et entrez un mot de passe administrateur.
3. Une fois l’installation terminée, connectez-vous à la machine virtuelle en tant qu’administrateur en utilisant le mot de passe d’administrateur. L’outil de configuration d’appliance de réplication (outil de configuration Azure Site Recovery) démarre en quelques secondes la première fois que vous vous connectez.
5. Entrez le nom à utiliser pour inscrire l’appliance auprès de Migration de serveur. Cliquez ensuite sur **Suivant**.
6. L’outil vérifie que la machine virtuelle peut se connecter à Azure. Une fois la connexion établie, sélectionnez **Connecter** pour vous connecter à votre abonnement Azure.
7. Attendez que l’outil termine d’inscrire une application Azure AD pour identifier l’appliance. L’appliance redémarre.
1. Reconnectez-vous à la machine. En quelques secondes, l’Assistant Gestion de serveur de configuration démarre automatiquement.

### <a name="register-the-replication-appliance"></a>Inscrire l’appliance de réplication

Finissez l’installation et l’inscription de l’appliance de réplication.

1. Dans la configuration de l’appliance, sélectionnez **Configurer la connectivité**.
2. Sélectionnez la carte réseau (par défaut, il n’y en a qu’une) que l’appliance de réplication utilise pour la découverte des machines virtuelles et pour effectuer une installation de type push du service Mobilité sur les machines sources.
3. Sélectionnez la carte réseau que l’appliance de réplication utilise pour la connectivité avec Azure. Ensuite, sélectionnez **Enregistrer**. Vous ne pouvez pas modifier ce paramètre une fois qu’il a été configuré.
4. Si l’appliance se trouve derrière un serveur proxy, vous devez spécifier les paramètres du proxy.
    - Spécifiez le nom du proxy sous la forme **http://ip-address** ou **http://FQDN** . Les serveurs proxy HTTPS ne sont pas pris en charge.
5. Quand vous êtes invité à entrer l’abonnement, les groupes de ressources et les détails du coffre, ajoutez les détails que vous avez notés au moment du téléchargement du modèle d’appliance.
6. Dans **Installer le logiciel tiers**, acceptez le contrat de licence. Sélectionnez **Télécharger et installer** pour installer MySQL Server.
7. Sélectionnez **Installer VMware PowerCLI**. Assurez-vous que toutes les fenêtres du navigateur sont fermées avant de continuer. Sélectionnez **Continuer**.
8. Dans **Valider la configuration de l’appliance**, les conditions préalables sont vérifiées avant que vous ne poursuiviez.
9. Dans **Configurer le serveur vCenter Server/vSphere ESXi**, saisissez le nom de domaine complet ou l’adresse IP du serveur vCenter Server, ou l’hôte vSphere, sur lequel sont situées les machines virtuelles que vous souhaitez répliquer. Entrez le numéro de port sur lequel le serveur écoute. Entrez un nom convivial à utiliser pour le serveur VMware dans le coffre.
10. Entrez les informations d’identification du compte que vous avez [créé](#prepare-an-account-to-discover-vms) pour la découverte VMware. Sélectionnez **Ajouter** > **Continuer**.
11. Dans **Configurer les informations d’identification de la machine virtuelle**, entrez les informations d’identification que vous avez [créées](#prepare-an-account-for-mobility-service-installation) pour l’installation de type push du service Mobilité au moment de l’activation de la réplication pour les machines virtuelles.  
    - Pour les machines Windows, le compte doit disposer des privilèges d’administrateur local sur les machines que vous souhaitez répliquer.
    - Pour Linux, fournissez les détails du compte racine.
12. Sélectionnez **Finaliser la configuration** pour terminer l’inscription.


Une fois l’appliance de réplication inscrite, Azure Migrate Server Assessment se connecte aux serveurs VMware en utilisant les paramètres spécifiés et découvre les machines virtuelles. Vous pouvez afficher les machines virtuelles découvertes dans **Gérer** > **Éléments découverts** sous l’onglet **Autre**.



## <a name="replicate-vms"></a>Répliquer des machines virtuelles

Sélectionnez les machines virtuelles pour la migration.

> [!NOTE]
> Dans le portail, vous pouvez sélectionner jusqu’à 10 machines à la fois pour la réplication. Si vous avez besoin d’en répliquer davantage, regroupez-les par lots de 10.

1. Dans le projet Azure Migrate > **Serveurs**, **Azure Migrate : Server Migration**, cliquez sur **Répliquer**.

    ![Capture d’écran de l’écran Serveurs dans Azure Migrate. Le bouton Répliquer est sélectionné dans Azure Migrate : Migration du serveur sous Outils de migration.](./media/tutorial-migrate-vmware-agent/select-replicate.png)

2. Dans **Répliquer** > **Paramètres de la source** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec VMware vSphere**.
3. Dans **Appliance locale**, sélectionnez le nom de l’appliance Azure Migrate que vous avez configurée.
4. Dans **vCenter Server**, spécifiez le nom du serveur vCenter qui gère les machines virtuelles ou du serveur vSphere sur lequel les machines virtuelles sont hébergées.
5. Dans **Process Server**, sélectionnez le nom de l’appliance de réplication.
6. Dans **Informations d’identification de l’invité**, spécifiez le compte d’administrateur de machine virtuelle qui sera utilisé pour l’installation de type push du service Mobilité. Cliquez ensuite sur **Suivant : Machines virtuelles**.

    ![Capture d’écran de l’onglet Paramètres de la source dans l’écran Réplication. Le champ Informations d’identification de l’invité est mis en surbrillance, et la valeur est définie sur VM-admin-account.](./media/tutorial-migrate-vmware-agent/source-settings.png)

7. Dans **Machines virtuelles**, sélectionnez les machines à répliquer.

    - Si vous avez exécuté une évaluation des machines virtuelles, vous pouvez appliquer les recommandations relatives au dimensionnement et au type de disque (Premium/Standard) des machines virtuelles qui apparaissent dans les résultats de l’évaluation. Pour ce faire, dans **Importer les paramètres de migration à partir d’une évaluation Azure Migrate ?** , sélectionnez l’option **Oui**.
    - Si vous n’avez pas exécuté d’évaluation ou que vous ne souhaitez pas utiliser les paramètres de l’évaluation, sélectionnez l’option **Non**.
    - Si vous avez choisi d’utiliser l’évaluation, sélectionnez le groupe de machines virtuelles et le nom de l’évaluation.
8. Dans **Options de disponibilité**, sélectionnez :
    -  Zone de disponibilité pour épingler la machine migrée à une Zone de disponibilité spécifique dans la région. Utilisez cette option pour distribuer les serveurs qui forment une couche Application à plusieurs nœuds entre des Zones de disponibilité. Si vous sélectionnez cette option, vous devez spécifier la Zone de disponibilité à utiliser pour chaque machine sélectionnée dans l’onglet Calcul. Cette option est disponible uniquement si la région cible sélectionnée pour la migration prend en charge les Zones de disponibilité
    -  Groupe à haute disponibilité pour placer la machine migrée dans un groupe à haute disponibilité. Pour utiliser cette option, le groupe de ressources cible qui a été sélectionné doit avoir un ou plusieurs groupes à haute disponibilité.
    - Aucune option de redondance de l’infrastructure n’est requise si vous n’avez pas besoin de ces configurations de disponibilité pour les machines migrées.
9. Cochez chaque machine virtuelle devant faire l’objet d’une migration. Cliquez ensuite sur **Suivant : Paramètres de la cible**.
10. Dans **Paramètres de la cible**, sélectionnez l’abonnement et la région cible vers laquelle vous allez migrer, puis spécifiez le groupe de ressources dans lequel les machines virtuelles Azure résideront après la migration.
11. Dans **Réseau virtuel**, sélectionnez le réseau virtuel/sous-réseau Azure auquel les machines virtuelles Azure seront jointes après la migration.
12. Dans **Options de disponibilité**, sélectionnez :
    -  Zone de disponibilité pour épingler la machine migrée à une Zone de disponibilité spécifique dans la région. Utilisez cette option pour distribuer les serveurs qui forment une couche Application à plusieurs nœuds entre des Zones de disponibilité. Si vous sélectionnez cette option, vous devez spécifier la Zone de disponibilité à utiliser pour chaque machine sélectionnée dans l’onglet Calcul. Cette option est disponible uniquement si la région cible sélectionnée pour la migration prend en charge les Zones de disponibilité
    -  Groupe à haute disponibilité pour placer la machine migrée dans un groupe à haute disponibilité. Pour utiliser cette option, le groupe de ressources cible qui a été sélectionné doit avoir un ou plusieurs groupes à haute disponibilité.
    - Aucune option de redondance de l’infrastructure n’est requise si vous n’avez pas besoin de ces configurations de disponibilité pour les machines migrées.
    
13. Dans **Azure Hybrid Benefit** :

    - Sélectionnez **Non** si vous ne souhaitez pas appliquer Azure Hybrid Benefit. Cliquez ensuite sur **Suivant**.
    - Sélectionnez **Oui** si vous avez des machines Windows Server couvertes par des abonnements Software Assurance ou Windows Server actifs et que vous souhaitez appliquer l’avantage aux machines que vous migrez. Cliquez ensuite sur **Suivant**.

14. Dans **Calcul**, vérifiez le nom de la machine virtuelle, sa taille, le type de disque du système d’exploitation et la configuration de la disponibilité (si elle a été sélectionnée à l’étape précédente). Les machines virtuelles doivent satisfaire aux [exigences d’Azure](migrate-support-matrix-vmware-migration.md#azure-vm-requirements).

   - **Taille de la machine virtuelle** : si vous suivez les recommandations de l’évaluation, la liste déroulante Taille de la machine virtuelle affiche la taille recommandée. Sinon, Azure Migrate choisit une taille qui correspond à la taille la plus proche dans l’abonnement Azure. Vous pouvez également choisir une taille manuelle dans **Taille de la machine virtuelle Azure**. 
    - **Disque de système d’exploitation** : spécifiez le disque du système d’exploitation (démarrage) pour la machine virtuelle. Le disque du système d’exploitation est le disque qui contient le chargeur de démarrage et le programme d’installation du système d’exploitation. 
    - **Zone de disponibilité** : Spécifiez la zone de disponibilité à utiliser.
    - **Groupe à haute disponibilité** : Spécifiez le groupe à haute disponibilité à utiliser.

15. Dans **Disques**, indiquez si les disques de machine virtuelle doivent être répliqués sur Azure, puis sélectionnez le type de disque (SSD/HDD standard ou disques managés Premium) dans Azure. Cliquez ensuite sur **Suivant**.
    - Vous pouvez exclure des disques de la réplication.
    - Si vous excluez des disques, ils ne seront pas présents sur la machine virtuelle Azure après la migration. 

16. Dans **Passer en revue et démarrer la réplication**, passez en revue les paramètres, puis cliquez sur **Répliquer** pour démarrer la réplication initiale pour les serveurs.

> [!NOTE]
> Vous pouvez mettre à jour les paramètres de réplication à tout moment avant le démarrage de la réplication (**Gérer** > **Réplication des machines**). Vous ne pouvez pas changer les paramètres après le démarrage de la réplication.


## <a name="track-and-monitor"></a>Suivre et superviser

1. Suivez l’état du travail dans les notifications du portail. 

    ![Suivre le travail](./media/tutorial-migrate-vmware-agent/jobs.png)
    
2. Pour superviser l’état de la réplication, cliquez sur **Réplication de serveurs** dans **Azure Migrate : Server Migration**.

    ![Superviser la réplication](./media/tutorial-migrate-vmware-agent/replicate-servers.png)

La réplication se déroule comme suit :
- Une fois le travail Démarrer la réplication terminé, les machines commencent leur réplication initiale vers Azure.
- Au terme de la réplication initiale, la réplication différentielle commence. Les modifications incrémentielles apportées aux disques locaux sont répliquées régulièrement sur les disques de réplica dans Azure.


## <a name="run-a-test-migration"></a>Exécuter un test de migration


Quand la réplication différentielle commence, vous pouvez exécuter une migration de test avec les machines virtuelles avant d’exécuter une migration complète vers Azure. Avant la migration, nous vous recommandons vivement d’effectuer cette opération au moins une fois pour chaque machine.

- L’exécution d’une migration de test permet de vérifier que la migration fonctionne comme prévu, sans impact sur les machines locales qui restent opérationnelles et poursuivent la réplication. 
- Une migration de test simule la migration en créant une machine virtuelle Azure à l’aide de données répliquées (il s’agit en général d’une migration vers un réseau virtuel hors production dans votre abonnement Azure).
- Vous pouvez utiliser la machine virtuelle Azure de test répliquée pour valider la migration, effectuer des tests d’applications et résoudre les éventuels problèmes avant la migration complète.

Effectuez une migration de test de la façon suivante :


1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Migration**, cliquez sur **Tester les serveurs migrés**.

     ![Tester les serveurs migrés](./media/tutorial-migrate-vmware-agent/test-migrated-servers.png)

2. Cliquez avec le bouton droit sur la machine virtuelle à tester, puis cliquez sur **Migration de test**.

    ![Migration de test](./media/tutorial-migrate-vmware-agent/test-migrate.png)

3. Dans **Migration de test**, sélectionnez le réseau virtuel Azure dans lequel la machine virtuelle Azure se trouvera après la migration. Nous vous recommandons d’utiliser un réseau virtuel hors production.
4. Le travail **Migration de test** démarre. Supervisez le travail dans les notifications du portail.
5. Une fois la migration terminée, affichez la machine virtuelle Azure migrée dans **Machines virtuelles** dans le portail Azure. Le nom de la machine a le suffixe **-Test**.
6. Une fois le test terminé, cliquez avec le bouton droit sur la machine virtuelle Azure dans **Réplication des machines**, puis cliquez sur **Nettoyer la migration de test**.

    ![Nettoyer la migration](./media/tutorial-migrate-vmware-agent/clean-up.png)


## <a name="migrate-vms"></a>Migrer des machines virtuelles

Après avoir vérifié que la migration de test fonctionne comme prévu, vous pouvez migrer des machines locales.

1. Dans le projet Azure Migrate > **Serveurs** > **Azure Migrate : Server Migration**, cliquez sur **Réplication de serveurs**.

    ![Réplication de serveurs](./media/tutorial-migrate-vmware-agent/replicate-servers.png)

2. Dans **Réplication des machines**, cliquez avec le bouton droit sur la machine virtuelle > **Migrer**.
3. Dans **Migrer** > **Arrêter les machines virtuelles et effectuer une migration planifiée sans perte de données**, sélectionnez **Oui** > **OK**.
    - Par défaut, Azure Migrate arrête la machine virtuelle locale pour garantir une perte de données minimale. 
    - Si vous ne souhaitez pas arrêter la machine virtuelle, sélectionnez **Non**.
4. Un travail de migration démarre pour la machine virtuelle. Suivez le travail dans les notifications Azure.
5. Une fois le travail terminé, vous pouvez afficher et gérer la machine virtuelle à partir de la page **Machines virtuelles**.

## <a name="complete-the-migration"></a>Effectuer la migration

1. Une fois la migration terminée, cliquez avec le bouton droit sur la machine virtuelle > **Arrêter la migration**. Cette opération effectue les actions suivantes :
    - Arrête la réplication pour l’ordinateur local.
    - Supprime l’ordinateur du nombre **Réplication de serveurs** dans Azure Migrate : Server Migration.
    - Nettoie les informations d’état de réplication pour la machine virtuelle.
2. Installez l’agent [Windows](../virtual-machines/extensions/agent-windows.md) ou [Linux](../virtual-machines/extensions/agent-linux.md) de machine virtuelle Azure sur les machines migrées.
3. Effectuez les éventuels ajustements post-migration de l’application, comme la mise à jour des chaînes de connexion de base de données et les configurations du serveur web.
4. Effectuez les tests finaux de réception de l’application et de la migration sur l’application migrée qui s’exécute maintenant dans Azure.
5. Réduisez le trafic vers l’instance de machine virtuelle Azure migrée.
6. Supprimez les machines virtuelles locales de votre inventaire des machines virtuelles locales.
7. Supprimez les machines virtuelles locales des sauvegardes locales.
8. Mettez à jour la documentation interne en y mentionnant le nouvel emplacement et la nouvelle adresse IP des machines virtuelles Azure. 

## <a name="post-migration-best-practices"></a>Bonnes pratiques après la migration

- Local
    - Déplacez le trafic de l’application vers l’application en cours d’exécution sur l’instance de machine virtuelle Azure migrée.
    - Supprimez les machines virtuelles locales de votre inventaire des machines virtuelles locales.
    - Supprimez les machines virtuelles locales des sauvegardes locales.
    - Mettez à jour la documentation interne en y mentionnant le nouvel emplacement et la nouvelle adresse IP des machines virtuelles Azure.
- Ajuster les paramètres de machine virtuelle Azure après migration :
    - [L’agent de machine virtuelle Azure](../virtual-machines/extensions/agent-windows.md) gère les interactions entre une machine virtuelle Azure et le contrôleur Azure Fabric. Il est nécessaire pour certains services Azure, comme Sauvegarde Azure, Site Recovery et Azure Security. Lors de la migration de machines virtuelles VMare avec la migration basée sur l’agent, le programme d’installation du service Mobility installe l’agent de machine virtuelle Azure sur les ordinateurs Windows. Sur les machines virtuelles Linux, nous vous recommandons d’installer l’agent après la migration.
    - Désinstallez manuellement le service Mobility de la machine virtuelle Azure après la migration.
    - Désinstallez manuellement les outils VMware après la migration.
- Dans Azure :
    - Effectuez les éventuels ajustements post-migration de l’application, comme la mise à jour des chaînes de connexion de base de données et les configurations du serveur web.
    - Effectuez les tests finaux de réception de l’application et de la migration sur l’application migrée qui s’exécute maintenant dans Azure.
- Continuité d'activité/Récupération d'urgence
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
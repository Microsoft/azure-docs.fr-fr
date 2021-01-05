---
title: Découvrir les machines virtuelles VMware avec Azure Migrate Server Assessment
description: Apprenez à découvrir les machines virtuelles VMware locales avec l’outil Azure Migrate Server Assessment
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: tutorial
ms.date: 09/14/2020
ms.custom: mvc
ms.openlocfilehash: ff82b90189e134db7d39b259c2c135489d14595d
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752785"
---
# <a name="tutorial-discover-vmware-vms-with-server-assessment"></a>Tutoriel : Découvrir les machines virtuelles VMware avec Server Assessment

Dans le cadre de votre migration vers Azure, vous devez découvrir vos charges de travail locales et réaliser votre inventaire. 

Ce tutoriel vous montre comment découvrir des machines virtuelles VMware locales avec l’outil Azure Migrate Server Assessment, à l’aide d’une appliance Azure Migrate légère. Vous allez déployer l’appliance en tant que machine virtuelle VMware, afin de découvrir en continu les métadonnées de performances et de machines virtuelles, les applications exécutées sur les machines virtuelles et les dépendances des machines virtuelles.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer un compte Azure
> * Préparer l’environnement VMware pour la découverte
> * Créer un projet Azure Migrate.
> * Configurer l’appliance Azure Migrate
> * Démarrer la découverte continue

> [!NOTE]
> Les tutoriels indiquent le moyen le plus rapide de tester un scénario et utilisent les options par défaut lorsque c’est possible.  

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.


## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, vérifiez les prérequis.


**Prérequis** | **Détails**
--- | ---
**vCenter Server/Hôte ESXi** | Vous avez besoin d’une instance vCenter Server exécutant la version 5.5, 6.0, 6.5 ou 6.7.<br/><br/> Les machines virtuelles doivent être hébergées sur un hôte ESXi exécutant la version 5.5 ou ultérieure.<br/><br/> Dans vCenter Server, autorisez les connexions entrantes sur le port TCP 443 pour que l’appliance puisse collecter les données d’évaluation.<br/><br/> L’appliance se connecte à vCenter sur le port 443 par défaut. Si vCenter Server écoute un autre port, vous pouvez modifier le port lorsque vous vous connectez au serveur à partir de l’appliance afin de démarrer la découverte.<br/><br/> Sur le serveur EXSi qui héberge les machines virtuelles, vérifiez que l’accès entrant est autorisé sur le port TCP 443 pour la découverte des applications.
**Appliance** | vCenter Server doit allouer des ressources à une machine virtuelle pour l’appliance Azure Migrate :<br/><br/> - Windows Server 2016<br/><br/> - 32 Go de RAM, huit processeurs virtuels et environ 80 Go de stockage sur disque.<br/><br/> - Un commutateur virtuel externe et un accès à Internet pour la machine virtuelle, directement ou via un proxy.
**Machines virtuelles** | Pour utiliser ce tutoriel, les machines virtuelles Windows doivent exécuter Windows Server 2016, 2012 R2, 2012 ou 2008 R2.<br/><br/> Les machines virtuelles Linux doivent exécuter Red Hat Enterprise Linux 7/6/5, Ubuntu Linux 14.04/16.04, Debian 7/8, Oracle Linux 6/7 ou CentOS 5/6/7.<br/><br/> Les outils VMware (version ultérieure à 10.2.0) doivent être installés et exécutés sur les machines virtuelles.<br/><br/> Sur les machines virtuelles Windows, Windows PowerShell 2.0 ou ultérieur doit être installé.


## <a name="prepare-an-azure-user-account"></a>Préparer un compte de stockage Azure

Pour créer un projet Azure Migrate et inscrire l’appliance Azure Migrate, vous avez besoin d’un compte avec :
- Des autorisations de niveau Contributeur ou Propriétaire sur un abonnement Azure
- Des autorisations permettant d’inscrire des applications Azure Active Directory

Si vous venez de créer un compte Azure gratuit, vous êtes le propriétaire de votre abonnement. Si vous n’êtes pas le propriétaire de l’abonnement, demandez à celui-ci de vous attribuer les autorisations de la façon suivante :

1. Dans le portail Azure, recherchez « abonnements », puis sous **Services**, sélectionnez **Abonnements**.

    ![Zone de recherche pour rechercher l’abonnement Azure](./media/tutorial-discover-vmware/search-subscription.png)

2. Dans la page **Abonnements**, sélectionnez l’abonnement dans lequel vous souhaitez créer un projet Azure Migrate. 
3. Dans l’abonnement, sélectionnez **Contrôle d’accès (IAM)**  > **Vérifier l’accès**.
4. Dans **Vérifier l’accès**, recherchez le compte d’utilisateur correspondant.
5. Dans **Ajouter une attribution de rôle**, cliquez sur **Ajouter**.

    ![Rechercher un compte d’utilisateur pour vérifier l’accès et attribuer un rôle](./media/tutorial-discover-vmware/azure-account-access.png)

6. Dans **Ajouter une attribution de rôle**, sélectionnez le rôle Contributeur ou Propriétaire, puis sélectionnez le compte (azmigrateuser dans notre exemple). Ensuite, cliquez sur **Enregistrer**.

    ![Ouvre la page Ajouter une attribution de rôle pour attribuer un rôle au compte.](./media/tutorial-discover-vmware/assign-role.png)

7. Dans le portail, recherchez des utilisateurs, puis, sous **Services**, sélectionnez **Utilisateurs**.
8. Dans **Paramètres utilisateur**, vérifiez que les utilisateurs Azure AD peuvent inscrire des applications (défini sur **Oui** par défaut).

    ![Vérifier dans les paramètres utilisateur que les utilisateurs peuvent inscrire des applications Active Directory](./media/tutorial-discover-vmware/register-apps.png)

9. L’administrateur général ou le locataire peuvent également attribuer à un compte le rôle **Développeur d’applications** pour permettre l’inscription d’applications AAD. [Plus d’informations](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md)

## <a name="prepare-vmware"></a>Préparer VMware

Dans vCenter Server, créez un compte que l’appliance peut utiliser pour accéder à l’instance vCenter Server, et vérifiez que les ports nécessaires sont ouverts. Vous avez également besoin d’un compte que l’appliance peut utiliser pour accéder aux machines virtuelles. 

### <a name="create-an-account-to-access-vcenter"></a>Créer un compte pour accéder à vCenter

Dans le client web vSphere, configurez un compte de la façon suivante :

1. À l’aide d’un compte disposant de privilèges Administrateur, dans le client web vSphere, sélectionnez **Administration**.
2. Sous **Access** (Accès), sélectionnez **Single Sign-On - Users and Groups** (Authentification unique - Utilisateurs et groupes).
3. Dans **Users** (Utilisateurs), ajoutez un nouvel utilisateur.
4. Dans **New User** (Nouvel utilisateur), tapez les informations du compte. Cliquez ensuite sur **OK**.
5. Dans **Global Permissions** (Autorisations globales), sélectionnez le compte d’utilisateur, puis attribuez le rôle **Read-only** (Lecture seule) au compte. Cliquez ensuite sur **OK**.
6. Dans **Roles** (Rôles), sélectionnez le rôle **Read-only**, puis dans **Privileges** (Privilèges), sélectionnez **Guest Operations** (Opérations invité). Ces privilèges sont nécessaires pour découvrir les applications qui s’exécutent sur les machines virtuelles, ainsi que pour analyser les dépendances des machines virtuelles.
 
    ![Case à cocher permettant d’autoriser les opérations invité pour le rôle Lecture seule](./media/tutorial-discover-vmware/guest-operations.png)


### <a name="create-an-account-to-access-vms"></a>Créer un compte pour accéder aux machines virtuelles

L’appliance accède aux machines virtuelles pour découvrir des applications et analyser les dépendances. L’appliance n’installe pas d’agents sur les machines virtuelles.

1. Créez un compte d’administrateur local que l’appliance peut utiliser pour découvrir des applications et des dépendances sur les machines virtuelles Windows.
2. Pour les machines Linux, créez un compte d’utilisateur avec des privilèges racine ou un compte d’utilisateur disposant de ces autorisations sur les fichiers/bin/netstat et/bin/ls : CAP_DAC_READ_SEARCH et CAP_SYS_PTRACE.

> [!NOTE]
> Azure Migrate permet d’utiliser des informations d’identification pour la découverte d’applications sur l’ensemble des serveurs Windows, et d’autres informations d’identification pour la découverte des applications sur l’ensemble des machines Linux.


## <a name="set-up-a-project"></a>Configuration d’un projet

Configurez un nouveau projet Azure Migrate.

1. Dans le portail Azure, sélectionnez **Tous les services**, puis recherchez **Azure Migrate**.
2. Sous **Services**, sélectionnez **Azure Migrate**.
3. Dans **Vue d’ensemble**, sélectionnez **Créer un projet**.
5. Dans **Créer un projet**, sélectionnez votre abonnement Azure et votre groupe de ressources. Créez un groupe de ressources si vous n’en avez pas.
6. Dans **Détails du projet**, spécifiez le nom du projet ainsi que la zone géographique où vous souhaitez le créer. Passez en revue les zones géographiques prises en charge pour les clouds [publics](migrate-support-matrix.md#supported-geographies-public-cloud) et du [secteur public](migrate-support-matrix.md#supported-geographies-azure-government).

   ![Zones pour le nom et la région du projet](./media/tutorial-discover-vmware/new-project.png)

7. Sélectionnez **Create** (Créer).
8. Attendez quelques minutes, le temps nécessaire au déploiement du projet Azure Migrate.

L’outil **Azure Migrate Server Assessment** est ajouté par défaut au nouveau projet.

![Page montrant l’outil Server Assessment ajouté par défaut](./media/tutorial-discover-vmware/added-tool.png)


## <a name="set-up-the-appliance"></a>Configurer l’appliance

Pour configurer l’appliance avec un modèle OVA, vous :
- Fournissez un nom d’appliance et générez une clé de projet Azure Migrate sur le portail.
- Téléchargez un fichier de modèle OVA, puis importez-le dans vCenter Server.
- Créez l’appliance et vérifiez qu’elle peut se connecter à Azure Migrate Server Assessment.
- Configurez l’appliance pour la première fois, puis inscrivez-la auprès du projet Azure Migrate en utilisant la clé de projet Azure Migrate.

> [!NOTE]
> Si, pour une raison quelconque, vous ne pouvez pas configurer l’appliance à l’aide du modèle, vous pouvez la configurer à l’aide d’un script PowerShell. [Plus d’informations](deploy-appliance-script.md#set-up-the-appliance-for-vmware)


### <a name="deploy-with-ova"></a>Effectuer un déploiement avec OVA

Pour configurer l’appliance avec un modèle OVA, vous :
- Fournissez un nom d’appliance et générez une clé de projet Azure Migrate sur le portail.
- Téléchargez un fichier de modèle OVA, puis importez-le dans vCenter Server.
- Créez l’appliance et vérifiez qu’elle peut se connecter à Azure Migrate Server Assessment.
- Configurez l’appliance pour la première fois, puis inscrivez-la auprès du projet Azure Migrate en utilisant la clé de projet Azure Migrate.

### <a name="generate-the-azure-migrate-project-key"></a>Générer la clé de projet Azure Migrate

1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Assessment**, sélectionnez **Découvrir**.
2. Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec l’hyperviseur vSphere VMware**.
3. Dans **1 : Générer une clé de projet Azure Migrate**, attribuez un nom à l’appliance Azure Migrate que vous allez configurer pour la découverte de machines virtuelles VMware. Il doit s’agir d’un nom alphanumérique d’au maximum 14 caractères.
1. Cliquez sur **Générer une clé** pour lancer la création des ressources Azure nécessaires. Ne fermez pas la page Découvrir des machines pendant la création des ressources.
1. Une fois les ressources Azure créées, une **clé de projet Azure Migrate** est générée.
1. Copiez la clé, car vous en aurez besoin pour terminer l’inscription de l’appliance pendant sa configuration.

### <a name="download-the-ova-template"></a>Télécharger le modèle OVA

Dans **2 : Télécharger l’appliance Azure Migrate**, sélectionnez le fichier .OVA et cliquez sur **Télécharger**. 


### <a name="verify-security"></a>Vérifier la sécurité

Vérifiez que le fichier OVA est sécurisé avant de le déployer :

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.
2. Exécutez la commande suivante pour générer le code de hachage du fichier OVA :
  
   ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
   
   Exemple d’utilisation : ```C:\>CertUtil -HashFile C:\Users\Administrator\Desktop\MicrosoftAzureMigration.ova SHA256```

3. Vérifiez les toutes dernières versions de l’appliance et les valeurs de hachage :

    - Pour le cloud public Azure :
    
        **Algorithme** | **Télécharger** | **SHA256**
        --- | --- | ---
        VMware (11,9 Go) | [Version la plus récente](https://go.microsoft.com/fwlink/?linkid=2140333) | e9c9a1fe4f3ebae81008328e8f3a7933d78ff835ecd871d1b17f367621ce3c74

    - Pour Azure Government :
    
        **Algorithme** | **Télécharger** | **SHA256**
        --- | --- | ---
        VMware (85,8 Mo) | [Version la plus récente](https://go.microsoft.com/fwlink/?linkid=2140337) | 2daaa2a59302bf911e8ef195f8add7d7c8352de77a9af0b860e2a627979085ca




### <a name="create-the-appliance-vm"></a>Créer la machine virtuelle de l’appliance

Importez le fichier téléchargé, puis créez une machine virtuelle.

1. Dans la console du client vSphere, cliquez sur **File** (Fichier) > **Deploy OVF Template** (Déployer le modèle OVF).
2. Dans l’Assistant de déploiement du modèle OVF > **Source**, spécifiez l’emplacement du fichier .OVA.
3. Dans **Name** (Nom) et **Location** (Emplacement), spécifiez un nom convivial pour la machine virtuelle. Sélectionnez l’objet d’inventaire dans lequel la machine virtuelle doit être hébergée.
5. Dans **Host/Cluster** (Hôte/Cluster), spécifiez l’hôte ou le cluster sur lequel s’exécute la machine virtuelle.
6. Dans **Storage** (Stockage), spécifiez la destination du stockage de la machine virtuelle.
7. Dans **Disk Format** (Format de disque), spécifiez le type de disque et la taille.
8. Dans **Network Mapping** (Mappage réseau), spécifiez le réseau auquel se connectera la machine virtuelle. Le réseau a besoin d’une connexion à Internet pour envoyer des métadonnées à Azure Migrate Server Assessment.
9. Passez en revue les paramètres, confirmez-les, puis cliquez sur **Finish** (Terminer).


### <a name="verify-appliance-access-to-azure"></a>Vérifier l’accès de l’appliance à Azure

Vérifiez que la machine virtuelle de l’appliance peut se connecter aux URL Azure pour les clouds [publics](migrate-appliance.md#public-cloud-urls) et du [secteur public](migrate-appliance.md#government-cloud-urls).


### <a name="configure-the-appliance"></a>Configurer l’appliance

Configurez l’appliance pour la première fois.

> [!NOTE]
> Si vous configurez l’appliance à l’aide d’un [script PowerShell](deploy-appliance-script.md) au lieu du modèle OVA téléchargé, ne tenez pas compte des deux premières étapes de cette procédure.

1. Dans la console du client vSphere, cliquez avec le bouton droit sur la machine virtuelle, puis sélectionnez **Open Console** (Ouvrir la console).
2. Spécifiez la langue, le fuseau horaire et le mot de passe pour l’appliance.
3. Ouvrez un navigateur sur une machine qui peut se connecter à la machine virtuelle, puis ouvrez l’URL de l’application web de l’appliance : **https://*nom ou adresse IP de l’appliance* : 44368**.

   Vous pouvez aussi ouvrir l’application à partir du Bureau de l’appliance en sélectionnant le raccourci de l’application.
1. Acceptez les **termes du contrat de licence** et lisez les informations relatives aux tiers.
1. Dans l’application web > **Configurer les prérequis**, procédez comme suit :
   - **Connectivité** : L’application vérifie que la machine virtuelle a accès à Internet. Si la machine virtuelle utilise un proxy :
     - Cliquez sur **Configurer le proxy** et spécifiez l’adresse du proxy (sous la forme http://ProxyIPAddress ou http://ProxyFQDN) ) et le port d’écoute.
     - Spécifiez les informations d’identification si le proxy nécessite une authentification.
     - Seuls les proxys HTTP sont pris en charge.
     - Si vous avez ajouté les détails du proxy ou désactivé le proxy et/ou l’authentification, cliquez sur **Enregistrer** pour relancer la vérification de la connectivité.
   - **Synchronisation de l’heure** : L’heure de l’appliance doit être synchronisée avec l’heure Internet pour que la découverte fonctionne correctement.
   - **Installer les mises à jour** : L’appliance vérifie que les dernières mises à jour sont installées. Une fois la vérification terminée, vous pouvez cliquer sur **Afficher les services de l’appliance** pour voir l’état et les versions des composants s’exécutant sur l’appliance.
   - **Installer VDDK** : L’appliance vérifie que VDDK (VMware vSphere Virtual Disk Development Kit) est installé. S’il n’est pas installé, téléchargez VDDK 6.7 à partir de VMware, puis extrayez le contenu du fichier zip téléchargé sur l’appliance à l’emplacement indiqué dans les **instructions d’installation**.

     Azure Migrate Server Migration utilise VDDK pour répliquer les machines durant la migration vers Azure. 
1. Si vous le souhaitez, vous pouvez **réexécuter les prérequis** à tout moment pendant la configuration de l’appliance pour vérifier si celle-ci remplit les remplit tous.

### <a name="register-the-appliance-with-azure-migrate"></a>Inscrire l’appliance auprès d’Azure Migrate

1. Collez la **clé de projet Azure Migrate** copiée à partir du portail. Si vous n’avez pas la clé, accédez à **Évaluation de serveur > Découvrir > Gérer les appliances existantes**, sélectionnez le nom d’appliance que vous avez indiqué au moment de générer la clé, puis copiez la clé correspondante.
1. Cliquez sur **Connexion**. Une invite de connexion Azure s’ouvre dans un nouvel onglet du navigateur. S’il n’apparaît pas, vérifiez que vous avez désactivé le bloqueur de fenêtres publicitaires dans le navigateur.
1. Sous le nouvel onglet, connectez-vous avec votre nom d’utilisateur et votre mot de passe Azure.
   
   La connexion avec un code PIN n’est pas prise en charge.
3. Une fois la connexion établie, revenez à l’application web. 
4. Si le compte d’utilisateur Azure utilisé pour la connexion dispose des autorisations adéquates sur les ressources Azure créées au moment de la génération de la clé, l’inscription de l’appliance est lancée.
1. Une fois l’inscription de l’appliance terminée, vous pouvez consulter les détails de l’inscription en cliquant sur **Afficher les détails**.



## <a name="start-continuous-discovery"></a>Démarrer la découverte en continu

L’appliance doit se connecter à vCenter Server pour découvrir les données de configuration et de performances des machines virtuelles.

1. À l’**Étape 1 : Fournir les informations d’identification vCenter Server**, cliquez sur **Ajouter des informations d’identification** pour attribuer un nom convivial aux informations d’identification, puis ajoutez un **nom d’utilisateur** et un **mot de passe** pour le compte vCenter Server dont se servira l’appliance pour détecter les machines virtuelles sur l’instance vCenter Server.
    - Vous devez avoir configuré un compte avec les autorisations nécessaires dans le tutoriel précédent.
    - Si vous souhaitez étendre la découverte à des objets VMware spécifiques (centres de données vCenter Server, clusters, dossier de clusters, hôtes, dossier d’hôtes ou machines virtuelles individuelles), passez en revue les instructions contenues dans [cet article](set-discovery-scope.md) pour restreindre le compte utilisé par Azure Migrate.
1. À l’**Étape 2 : Fournir les détails de vCenter Server**, cliquez sur **Ajouter une source de découverte** pour sélectionner le nom convivial des informations d’identification dans la liste déroulante, puis spécifiez l’**adresse IP/nom de domaine complet** de l’instance vCenter Server. Vous pouvez laisser le **port** par défaut (443) ou spécifier un port personnalisé sur lequel vCenter Server est à l’écoute, puis cliquez sur **Enregistrer**.
1. Dès que vous cliquez sur Enregistrer, l’appliance tente de valider la connexion à vCenter Server avec les informations d’identification indiquées et affiche l’**état de validation** dans le tableau pour l’adresse IP/le nom de domaine complet de vCenter Server.
1. Vous pouvez **revalider** la connectivité à vCenter Server à tout moment avant de lancer la découverte.
1. À l’**Étape 3 : Fournir les informations d’identification de la machine virtuelle pour découvrir les applications installées et pour effectuer un mappage des dépendances sans agent**, cliquez sur **Ajouter des informations d’identification** et spécifiez le système d’exploitation pour lequel les informations d’identification sont fournies, le nom convivial des informations d’identification ainsi que le **nom d’utilisateur** et le **mot de passe**. Cliquez ensuite sur **Enregistrer**.

    - Vous pouvez ajouter des informations d’identification ici si vous avez créé un compte à utiliser pour la [fonctionnalité de découverte d’applications](how-to-discover-applications.md) ou la [fonctionnalité d’analyse de dépendances sans agent](how-to-create-group-machine-dependencies-agentless.md).
    - Si vous ne souhaitez pas utiliser ces fonctionnalités, vous pouvez cliquer sur le curseur pour passer l’étape. Vous pouvez par la suite revenir sur votre décision à tout moment.
    - Passez en revue les informations d’identification nécessaires à la [découverte d’applications](migrate-support-matrix-vmware.md#application-discovery-requirements) ou à l’[analyse des dépendances sans agent](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless).

5. Cliquez sur **Démarrer la découverte** pour lancer la découverte de machines virtuelles. Une fois la découverte lancée, vous pouvez vérifier l’état de la découverte dans le tableau pour l’adresse IP/le nom de domaine complet de vCenter Server.

La découverte fonctionne comme ceci :
- Environ 15 minutes sont nécessaires pour que les métadonnées des machines virtuelles découvertes apparaissent dans le portail.
- La découverte des applications, rôles et fonctionnalités installés prend un certain temps. Sa durée dépend du nombre de machines virtuelles découvertes. Pour 500 machines virtuelles, il faut environ une heure pour que l’inventaire des applications s’affiche dans le portail Azure Migrate.


## <a name="next-steps"></a>Étapes suivantes

- [Évaluer des machines virtuelles VMware](./tutorial-assess-vmware-azure-vm.md) en vue de leur migration vers des machines virtuelles Azure
- [Passer en revue les données](migrate-appliance.md#collected-data---vmware) que l’appliance collecte pendant la découverte
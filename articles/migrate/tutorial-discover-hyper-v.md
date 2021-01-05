---
title: Découvrir des machines virtuelles Hyper-V avec Azure Migrate Server Assessment
description: Apprenez à découvrir les machines virtuelles Hyper-V locales avec l’outil Azure Migrate Server Assessment
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: tutorial
ms.date: 09/14/2020
ms.custom: mvc
ms.openlocfilehash: 1b860c739ab9ed9737f9f946cb13c731fa4722db
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753057"
---
# <a name="tutorial-discover-hyper-v-vms-with-server-assessment"></a>Tutoriel : Découvrir les machines virtuelles Hyper-V avec Server Assessment

Dans le cadre de votre migration vers Azure, vous devez découvrir vos charges de travail locales et réaliser votre inventaire. 

Ce tutoriel vous montre comment découvrir des machines virtuelles Hyper-V locales avec l’outil Azure Migrate Server Assessment, à l’aide d’une appliance Azure Migrate légère. Vous allez déployer l’appliance comme une machine virtuelle Hyper-V afin de découvrir de façon continue les métadonnées relatives aux machines et aux performances.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer un compte Azure
> * Préparer l’environnement Hyper-V pour la découverte
> * Créer un projet Azure Migrate.
> * Configurer l’appliance Azure Migrate
> * Démarrer la découverte en continu

> [!NOTE]
> Les tutoriels enseignent le moyen le plus rapide de tester un scénario et d’utiliser les options par défaut.  

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, vérifiez les prérequis.


**Prérequis** | **Détails**
--- | ---
**Hôte Hyper-V** | Les hôtes Hyper-V sur lesquels les machines virtuelles sont situées peuvent être autonomes ou appartenir à un cluster.<br/><br/> L’hôte doit exécuter Windows Server 2019, Windows Server 2016 ou Windows Server 2012 R2.<br/><br/> Vérifiez que les connexions entrantes sont autorisées sur le port 5985 WinRM (HTTP), afin de permettre à l’appliance de se connecter en vue de tirer (pull) les données de performances et les métadonnées de machines virtuelles à l’aide d’une session CIM (Common Information Model).
**Étapes de déploiement d’appliance** | L’hôte Hyper-v doit allouer des ressources à une machine virtuelle pour l’appliance :<br/><br/> - Windows Server 2016<br/><br/> \- 16 Go de RAM<br/><br/> - 8 processeurs virtuels<br/><br/> - Environ 80 Go de stockage sur disque<br/><br/> - Un commutateur virtuel externe<br/><br/> - Un accès à Internet pour la machine virtuelle, directement ou via un proxy
**Machines virtuelles** | Les machines virtuelles peuvent exécuter le système d’exploitation Windows ou Linux. 

Avant de commencer, vous pouvez [examiner les données](migrate-appliance.md#collected-data---hyper-v) que l’appliance collecte pendant la découverte.

## <a name="prepare-an-azure-user-account"></a>Préparer un compte de stockage Azure

Pour créer un projet Azure Migrate et inscrire l’appliance Azure Migrate, vous avez besoin d’un compte avec :
- Des autorisations de niveau Contributeur ou Propriétaire sur un abonnement Azure
- Des autorisations permettant d’inscrire des applications Azure Active Directory

Si vous venez de créer un compte Azure gratuit, vous êtes le propriétaire de votre abonnement. Si vous n’êtes pas le propriétaire de l’abonnement, demandez à celui-ci de vous attribuer les autorisations de la façon suivante :


1. Dans le portail Azure, recherchez « abonnements », puis sous **Services**, sélectionnez **Abonnements**.

    ![Zone de recherche pour rechercher l’abonnement Azure](./media/tutorial-discover-hyper-v/search-subscription.png)

2. Dans la page **Abonnements**, sélectionnez l’abonnement dans lequel vous souhaitez créer un projet Azure Migrate. 
3. Dans l’abonnement, sélectionnez **Contrôle d’accès (IAM)**  > **Vérifier l’accès**.
4. Dans **Vérifier l’accès**, recherchez le compte d’utilisateur correspondant.
5. Dans **Ajouter une attribution de rôle**, cliquez sur **Ajouter**.

    ![Rechercher un compte d’utilisateur pour vérifier l’accès et attribuer un rôle](./media/tutorial-discover-hyper-v/azure-account-access.png)

6. Dans **Ajouter une attribution de rôle**, sélectionnez le rôle Contributeur ou Propriétaire, puis sélectionnez le compte (azmigrateuser dans notre exemple). Ensuite, cliquez sur **Enregistrer**.

    ![Ouvre la page Ajouter une attribution de rôle pour attribuer un rôle au compte.](./media/tutorial-discover-hyper-v/assign-role.png)

7. Dans le portail, recherchez des utilisateurs, puis, sous **Services**, sélectionnez **Utilisateurs**.
8. Dans **Paramètres utilisateur**, vérifiez que les utilisateurs Azure AD peuvent inscrire des applications (défini sur **Oui** par défaut).

    ![Vérifier dans les paramètres utilisateur que les utilisateurs peuvent inscrire des applications Active Directory](./media/tutorial-discover-hyper-v/register-apps.png)

9. L’administrateur général ou le locataire peuvent également attribuer à un compte le rôle **Développeur d’applications** pour permettre l’inscription d’applications AAD. [Plus d’informations](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md)

## <a name="prepare-hyper-v-hosts"></a>Préparer les hôtes Hyper-V

Configurez un compte disposant d’un accès Administrateur sur les hôtes Hyper-V. L’appliance utilise ce compte pour la découverte.

- Option 1 : Préparez un compte disposant d’un accès Administrateur sur l’hôte Hyper-V.
- Option n°2 : Préparez un compte d’administrateur local ou un compte d’administrateur de domaine, puis ajoutez le compte à ces groupes : Utilisateurs de gestion à distance, Administrateurs Hyper-V et Utilisateurs de l’Analyseur de performances.


## <a name="set-up-a-project"></a>Configuration d’un projet

Configurez un nouveau projet Azure Migrate.

1. Dans le portail Azure, sélectionnez **Tous les services**, puis recherchez **Azure Migrate**.
2. Sous **Services**, sélectionnez **Azure Migrate**.
3. Dans **Vue d’ensemble**, sélectionnez **Créer un projet**.
5. Dans **Créer un projet**, sélectionnez votre abonnement Azure et votre groupe de ressources. Créez un groupe de ressources si vous n’en avez pas.
6. Dans **Détails du projet**, spécifiez le nom du projet ainsi que la zone géographique où vous souhaitez le créer. Passez en revue les zones géographiques prises en charge pour les clouds [publics](migrate-support-matrix.md#supported-geographies-public-cloud) et du [secteur public](migrate-support-matrix.md#supported-geographies-azure-government).

   ![Zones pour le nom et la région du projet](./media/tutorial-discover-hyper-v/new-project.png)

7. Sélectionnez **Create** (Créer).
8. Attendez quelques minutes, le temps nécessaire au déploiement du projet Azure Migrate.

L’outil **Azure Migrate Server Assessment** est ajouté par défaut au nouveau projet.

![Page montrant l’outil Server Assessment ajouté par défaut](./media/tutorial-discover-hyper-v/added-tool.png)


## <a name="set-up-the-appliance"></a>Configurer l’appliance

Ce tutoriel configure l’appliance sur une machine virtuelle Hyper-V, de la façon suivante :

- Fournissez un nom d'appliance et générez une clé de projet Azure Migrate sur le portail.
- Téléchargez un disque dur virtuel Hyper-V compressé à partir du portail Azure.
- Créez l’appliance et vérifiez qu’elle peut se connecter à Azure Migrate Server Assessment.
- Configurez l’appliance pour la première fois, puis inscrivez-la auprès du projet Azure Migrate en utilisant la clé de projet Azure Migrate.
> [!NOTE]
> Si, pour une raison quelconque, vous ne pouvez pas configurer l’appliance à l’aide du modèle, vous pouvez la configurer à l’aide d’un script PowerShell. [Plus d’informations](deploy-appliance-script.md#set-up-the-appliance-for-hyper-v)


### <a name="generate-the-azure-migrate-project-key"></a>Générer la clé de projet Azure Migrate

1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Assessment**, sélectionnez **Découvrir**.
2. Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec Hyper-V**.
3. Dans **1 : Générer une clé de projet Azure Migrate**, attribuez un nom à l'appliance Azure Migrate que vous allez configurer pour la détection de machines virtuelles Hyper-V. Il doit s'agir d'un nom alphanumérique de 14 caractères maximum.
1. Cliquez sur **Générer une clé** pour lancer la création des ressources Azure nécessaires. Ne fermez pas la page Découvrir des machines pendant la création des ressources.
1. Une fois les ressources Azure créées, une **clé de projet Azure Migrate** est générée.
1. Copiez la clé car vous en aurez besoin pour terminer l'inscription de l'appliance lors de sa configuration.

### <a name="download-the-vhd"></a>Télécharger le disque dur virtuel

Dans **2 : Télécharger l'appliance Azure Migrate**, sélectionnez le fichier .VHD et cliquez sur **Télécharger**. 


### <a name="verify-security"></a>Vérifier la sécurité

Vérifiez que le fichier compressé est sécurisé avant de le déployer.

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.

2. Exécutez la commande PowerShell suivante pour générer le code de hachage du fichier ZIP
    - ```C:\>Get-FileHash -Path <file_location> -Algorithm [Hashing Algorithm]```
    - Exemple d’utilisation : ```C:\>Get-FileHash -Path ./AzureMigrateAppliance_v3.20.09.25.zip -Algorithm SHA256```

3.  Vérifiez les toutes dernières versions de l’appliance et les valeurs de hachage :

    - Pour le cloud public Azure :

        **Scénario** | **Télécharger** | **SHA256**
        --- | --- | ---
        Hyper-V (8,91 Go) | [Version la plus récente](https://go.microsoft.com/fwlink/?linkid=2140422) |  40aa037987771794428b1c6ebee2614b092e6d69ac56d48a2bbc75eeef86c99a

    - Pour Azure Government :

        **Scénario** _ | _ *Télécharger** | **SHA256**
        --- | --- | ---
        Hyper-V (85,8 Mo) | [Version la plus récente](https://go.microsoft.com/fwlink/?linkid=2140424) |  cfed44bb52c9ab3024a628dc7a5d0df8c624f156ec1ecc3507116bae330b257f

### <a name="create-the-appliance-vm"></a>Créer la machine virtuelle de l’appliance

Importez le fichier téléchargé, puis créez la machine virtuelle.

1. Extrayez le fichier de disque dur virtuel compressé dans un dossier sur l’hôte Hyper-V qui hébergera la machine virtuelle de l’appliance. Trois dossiers sont extraits.
2. Ouvrez le Gestionnaire Hyper-V. Dans **Actions**, cliquez sur **Importer la machine virtuelle**.
2. Dans l’Assistant Importation de machine virtuelle > **Avant de commencer**, cliquez sur **Suivant**.
3. Dans **Localiser le dossier**, spécifiez le dossier contenant le disque dur virtuel extrait. Cliquez ensuite sur **Suivant**.
1. Dans **Sélectionner une machine virtuelle**, cliquez sur **Suivant**.
2. Dans **Choisir le type d’importation**, cliquez sur **Copier la machine virtuelle (créer un identifiant unique)** . Cliquez ensuite sur **Suivant**.
3. Dans **Choisir la destination**, laissez la valeur par défaut. Cliquez sur **Suivant**.
4. Dans **Dossiers de stockage**, laissez la valeur par défaut. Cliquez sur **Suivant**.
5. Dans **Choisir un réseau**, spécifiez le commutateur virtuel qui sera utilisé par la machine virtuelle. Le commutateur nécessite une connexion à Internet pour envoyer des données à Azure.
6. Dans **Récapitulatif**, passez en revue les paramètres. Puis, cliquez sur **Terminer**.
7. Dans Gestionnaire Hyper-V > **Machines virtuelles**, démarrez la machine virtuelle.


### <a name="verify-appliance-access-to-azure"></a>Vérifier l’accès de l’appliance à Azure

Vérifiez que la machine virtuelle de l’appliance peut se connecter aux URL Azure pour les clouds [publics](migrate-appliance.md#public-cloud-urls) et du [secteur public](migrate-appliance.md#government-cloud-urls).

### <a name="configure-the-appliance"></a>Configurer l’appliance

Configurez l’appliance pour la première fois.

> [!NOTE]
> Si vous configurez l’appliance à l’aide d’un [script PowerShell](deploy-appliance-script.md) au lieu du disque dur virtuel téléchargé, ne tenez pas compte des deux premières étapes de cette procédure.

1. Dans Gestionnaire Hyper-V> **Machines virtuelles**, cliquez avec le bouton droit sur la machine virtuelle > **Se connecter**.
2. Spécifiez la langue, le fuseau horaire et le mot de passe pour l’appliance.
3. Ouvrez un navigateur sur une machine qui peut se connecter à la machine virtuelle, puis ouvrez l’URL de l’application web de l’appliance : **https://*nom ou adresse IP de l’appliance* : 44368**.

   Vous pouvez aussi ouvrir l’application à partir du Bureau de l’appliance en cliquant sur le raccourci de l’application.
1. Acceptez les **termes du contrat de licence** et lisez les informations relatives aux tiers.
1. Dans l’application web > **Configurer les prérequis**, procédez comme suit :
    - **Connectivité** : L’application vérifie que la machine virtuelle a accès à Internet. Si la machine virtuelle utilise un proxy :
      - Cliquez sur **Configurer le proxy**, puis spécifiez l'adresse du proxy (sous la forme http://ProxyIPAddress ou http://ProxyFQDN) ) et le port d'écoute.
      - Spécifiez les informations d’identification si le proxy nécessite une authentification.
      - Seuls les proxys HTTP sont pris en charge.
      - Si vous avez ajouté des détails de proxy ou désactivé le proxy et/ou l’authentification, cliquez sur **Enregistrer** pour relancer la vérification de la connectivité.
    - **Synchronisation de l’heure** : L’heure est vérifiée. L’heure de l’appliance doit être synchronisée avec l’heure Internet pour que la découverte des machines virtuelles fonctionne correctement.
    - **Installer les mises à jour** : Azure Migrate Server Assessment vérifie que les dernières mises à jour sont installées sur l'appliance. Au terme de la vérification, vous pouvez cliquer sur **Afficher les services de l'appliance** pour voir l'état et les versions des composants exécutés sur l'appliance.

### <a name="register-the-appliance-with-azure-migrate"></a>Inscrire l’appliance auprès d’Azure Migrate

1. Collez la **clé de projet Azure Migrate** copiée à partir du portail. Si vous n’avez pas la clé, accédez à **Évaluation de serveur > Découvrir > Gérer les appliances existantes**, sélectionnez le nom d’appliance que vous avez indiqué au moment de générer la clé, puis copiez la clé correspondante.
1. Cliquez sur **Connexion**. Une invite de connexion Azure s’ouvre dans un nouvel onglet du navigateur. S’il n’apparaît pas, vérifiez que vous avez désactivé le bloqueur de fenêtres publicitaires dans le navigateur.
1. Sous le nouvel onglet, connectez-vous avec votre nom d’utilisateur et votre mot de passe Azure.
   
   La connexion avec un code PIN n’est pas prise en charge.
3. Une fois la connexion établie, revenez à l’application web. 
4. Si le compte d’utilisateur Azure utilisé pour la connexion dispose des autorisations adéquates sur les ressources Azure créées au moment de la génération de la clé, l’inscription de l’appliance est lancée.
1. Une fois l'appliance inscrite, vous pouvez consulter les détails de l'inscription en cliquant sur **Afficher les détails**.



### <a name="delegate-credentials-for-smb-vhds"></a>Déléguer des informations d’identification pour les disques durs virtuels sur SMB

Si vous utilisez des disques durs virtuels sur des SMB, vous devez activer la délégation des informations d’identification de l’appliance aux hôtes Hyper-V. Pour effectuer cette opération à partir de l’appliance :

1. Sur la machine virtuelle de l’appliance, exécutez cette commande. HyperVHost1/HyperVHost2 sont des exemples de noms d’hôte.

    ```
    Enable-WSManCredSSP -Role Client -DelegateComputer HyperVHost1.contoso.com, HyperVHost2.contoso.com, HyperVHost1, HyperVHost2 -Force
    ```

2. Vous pouvez également effectuer cette opération dans l’éditeur d’objets de stratégie de groupe sur l’appliance :
    - Dans **Stratégie de l’ordinateur local** > **Configuration de l’ordinateur**, cliquez sur **Modèles d’administration** > **Système** > **Délégation d’informations d’identification**.
    - Double-cliquez sur **Autoriser la délégation de nouvelles informations d’identification**, puis sélectionnez **Activé**.
    - Dans **Options**, cliquez sur **Afficher**, puis ajoutez chaque hôte Hyper-V que vous voulez découvrir à la liste, avec **wsman/** comme préfixe.
    - Dans **Délégation d’informations d’identification**, double-cliquez sur **Autoriser la délégation de nouvelles informations d’identification avec l’authentification de serveur NTLM uniquement**. À nouveau, ajoutez chaque hôte Hyper-V que vous voulez découvrir à la liste, avec **wsman/** comme préfixe.

## <a name="start-continuous-discovery"></a>Démarrer la découverte en continu

Connectez-vous de l’appliance à des hôtes ou des clusters Hyper-V, et démarrez la découverte des machines virtuelles.

1. À l’**Étape 1 : Fournir les informations d'identification de l'hôte Hyper-V**, cliquez sur **Ajouter les informations d'identification** afin de spécifier un nom convivial pour les informations d'identification, puis ajoutez un **Nom d'utilisateur** et un **Mot de passe** pour l'hôte/le cluster Hyper-V que l'appliance utilisera afin de détecter les machines virtuelles. Cliquez sur **Save**(Enregistrer).
1. Si vous souhaitez ajouter plusieurs informations d'identification à la fois, cliquez sur **Ajouter** pour enregistrer et ajouter d'autres informations d'identification. Plusieurs informations d'identification peuvent être prises en charge pour la détection de machines virtuelles Hyper-V.
1. À l’**Étape 2 : Fournir les détails de l'hôte/du cluster Hyper-V**, cliquez sur **Ajouter une source de détection** afin de spécifier l'**adresse IP/nom de domaine complet** de l'hôte/du cluster Hyper-V ainsi que le nom convivial des informations d'identification à utiliser pour la connexion à l'hôte/au cluster.
1. Vous pouvez soit **Ajouter un seul élément**  à la fois, soit **Ajouter plusieurs éléments**  en une seule fois. Une option permet également de connaître les détails de l'hôte/du cluster Hyper-V via **Importer CSV**.


    - Si vous choisissez **Ajouter un seul élément**, vous devez spécifier un nom convivial pour les informations d'identification ainsi que l'**adresse IP/nom de domaine complet** de l'hôte/du cluster Hyper-V, puis cliquer sur **Enregistrer**.
    - Si vous choisissez **Ajouter plusieurs éléments** _(sélectionné par défaut)_ , vous pouvez ajouter plusieurs enregistrements à la fois en spécifiant l'**adresse IP/nom de domaine complet** de l'hôte/du cluster Hyper-V avec le nom convivial des informations d'identification dans la zone de texte. **Vérifiez** les enregistrements ajoutés et cliquez sur **Enregistrer**.
    - Si vous choisissez **Importer CSV**, vous pouvez télécharger un fichier de modèle CSV, puis renseigner le fichier avec l'**adresse IP/nom de domaine complet** de l'hôte/du cluster Hyper-V ainsi qu'un nom convivial pour les informations d'identification. Importez ensuite le fichier dans l'appliance, **vérifiez** les enregistrements dans le fichier, puis cliquez sur **Enregistrer**.

1. Dès que vous cliquerez sur Enregistrer, l'appliance essaiera de valider la connexion aux hôtes/clusters Hyper-V ajoutés et affichera l'**État de validation** dans le tableau pour chaque hôte/cluster.
    - Pour les hôtes/clusters validés avec succès, vous pouvez afficher plus de détails en cliquant sur leur adresse IP/nom de domaine complet.
    - Si la validation d'un hôte échoue, examinez l'erreur en cliquant sur **Échec de validation** dans la colonne État du tableau. Réglez le problème et recommencez la validation.
    - Pour supprimer des hôtes ou des clusters, cliquez sur **Supprimer**.
    - Vous ne pouvez pas supprimer un hôte spécifique d’un cluster. Vous pouvez supprimer seulement la totalité du cluster.
    - Vous pouvez ajouter un cluster, même s’il existe des problèmes avec des hôtes spécifiques dans le cluster.
1. Vous pouvez **revalider** la connectivité aux hôtes/clusters à tout moment avant de lancer la détection.
1. Cliquez sur **Démarrer la détection** pour lancer la détection des machines virtuelles à partir des hôtes/clusters validés. Une fois la détection lancée, vous pouvez vérifier son état pour chaque hôte/cluster du tableau.

Ceci démarre la découverte. Il faut environ 2 minutes par hôte pour que les métadonnées des serveurs détectés apparaissent sur le portail Azure.

## <a name="verify-vms-in-the-portal"></a>Vérifier les machines virtuelles dans le portail

Une fois la découverte terminée, vous pouvez vérifier que les machines virtuelles apparaissent dans le portail.

1. Ouvrez le tableau de bord Azure Migrate.
2. Dans la page **Azure Migrate - Serveurs** > **Azure Migrate : Server Assessment**, cliquez sur l’icône qui affiche le nombre de **Serveurs découverts**.

## <a name="next-steps"></a>Étapes suivantes

- [Évaluer les machines virtuelles Hyper-V](tutorial-assess-hyper-v.md) en vue de leur migration vers des machines virtuelles Azure
- [Passer en revue les données](migrate-appliance.md#collected-data---hyper-v) que l’appliance collecte pendant la découverte
---
title: Configurer une appliance Azure Migrate pour Hyper-V
description: Découvrez comment configurer une appliance Azure Migrate pour évaluer et migrer des machines virtuelles Hyper-V.
author: vikram1988
ms.author: vibansa
ms.manager: abhemraj
ms.topic: how-to
ms.date: 03/23/2020
ms.openlocfilehash: 5c5fdd1423d806bcc4d2f124310112a3e407e416
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751119"
---
# <a name="set-up-an-appliance-for-hyper-v-vms"></a>Configurer une appliance pour les machines virtuelles Hyper-V

Suivez cet article pour configurer l’appliance Azure Migrate pour l’évaluation des machines virtuelles Hyper-V à l’aide de l’outil [Azure Migrate : Évaluation de serveurs](migrate-services-overview.md#azure-migrate-server-assessment-tool).

L'[appliance Azure Migrate](migrate-appliance.md) est une appliance légère utilisée par Azure Migrate : Évaluation/Migration de serveurs pour découvrir les machines virtuelles Hyper-V locales et envoyer les métadonnées/données de performances des machines virtuelles à Azure.

Vous pouvez déployer l’appliance à l’aide de deux méthodes :

- Configuration sur une machine virtuelle Hyper-V au moyen d’un disque dur virtuel Hyper-V téléchargé. Il s’agit de la méthode décrite dans cet article.
- Configurez-la sur une machine virtuelle Hyper-V ou machine physique avec un script d’installation PowerShell. [Cette méthode](deploy-appliance-script.md) doit être utilisée si vous ne pouvez pas configurer une machine virtuelle à l’aide d’un disque dur virtuel, ou si vous êtes dans Azure Government.

Après avoir créé l’appliance, vérifiez qu’elle peut se connecter à Azure Migrate Server Assessment, configurez-la pour la première fois, puis inscrivez-la auprès du projet Azure Migrate.

## <a name="appliance-deployment-vhd"></a>Déploiement d'appliance (VHD)

Pour configurer l'appliance à l'aide d'un modèle VHD :

- Fournissez un nom d'appliance et générez une clé de projet Azure Migrate sur le portail.
- Téléchargez un disque dur virtuel Hyper-V compressé à partir du portail Azure.
- Créez l’appliance et vérifiez qu’elle peut se connecter à Azure Migrate Server Assessment.
- Configurez l'appliance pour la première fois, puis inscrivez-la auprès du projet Azure Migrate à l'aide de la clé de projet Azure Migrate.

### <a name="generate-the-azure-migrate-project-key"></a>Générer la clé de projet Azure Migrate

1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Assessment**, sélectionnez **Découvrir**.
2. Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec Hyper-V**.
3. Dans **1 : Générer une clé de projet Azure Migrate**, attribuez un nom à l'appliance Azure Migrate que vous allez configurer pour la détection de machines virtuelles Hyper-V. Il doit s'agir d'un nom alphanumérique de 14 caractères maximum.
1. Cliquez sur **Générer une clé** pour lancer la création des ressources Azure nécessaires. Ne fermez pas la page Détecter des machines pendant la création des ressources.
1. Une fois les ressources Azure créées, une **clé de projet Azure Migrate** est générée.
1. Copiez la clé car vous en aurez besoin pour terminer l'inscription de l'appliance lors de sa configuration.

### <a name="download-the-vhd"></a>Télécharger le disque dur virtuel

Dans **2 : Télécharger l'appliance Azure Migrate**, sélectionnez le fichier .VHD et cliquez sur **Télécharger**. 

   ![Sélections relatives à Détecter des machines](./media/tutorial-assess-hyper-v/servers-discover.png)


   ![Sélections relatives à Générer une clé](./media/tutorial-assess-hyper-v/generate-key-hyperv.png)


### <a name="verify-security"></a>Vérifier la sécurité

Vérifiez que le fichier compressé est sécurisé avant de le déployer.

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.
2. Exécutez la commande suivante pour générer le code de hachage du disque dur virtuel
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Exemple d’utilisation : ```C:\>Get-FileHash -Path ./AzureMigrateAppliance_v3.20.09.25.zip -Algorithm SHA256```





## <a name="create-the-appliance-vm"></a>Créer la machine virtuelle de l’appliance

Importez le fichier téléchargé, puis créez la machine virtuelle.

1. Extrayez le fichier de disque dur virtuel compressé dans un dossier sur l’hôte Hyper-V qui hébergera la machine virtuelle de l’appliance. Trois dossiers sont extraits.
2. Ouvrez le Gestionnaire Hyper-V. Dans **Actions**, cliquez sur **Importer la machine virtuelle**.

    ![Déployer le disque dur virtuel](./media/how-to-set-up-appliance-hyper-v/deploy-vhd.png)

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
4. Si le compte d’utilisateur Azure utilisé pour la connexion dispose des [autorisations](./tutorial-discover-hyper-v.md#prepare-an-azure-user-account) adéquates sur les ressources Azure créées au moment de la génération de la clé, l’inscription de l’appliance est lancée.
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

    ![Sélections relatives à l'ajout d'une source de détection](./media/tutorial-assess-hyper-v/add-discovery-source-hyperv.png)

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

Essayez l’[évaluation d’Hyper-V](tutorial-assess-hyper-v.md) avec Azure Migrate Server Assessment.
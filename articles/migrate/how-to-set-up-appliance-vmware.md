---
title: Configurer une appliance Azure Migrate pour VMware
description: Découvrez comment configurer une appliance Azure Migrate pour évaluer et migrer des machines virtuelles VMware.
author: vikram1988
ms.author: vibansa
ms.manager: abhemraj
ms.topic: how-to
ms.date: 04/16/2020
ms.openlocfilehash: cc479f8e4dc87155721b25fab2ba522880b0ac4f
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751055"
---
# <a name="set-up-an-appliance-for-vmware-vms"></a>Configurer une appliance pour les machines virtuelles VMware

Suivez cet article pour configurer l’appliance Azure Migrate pour l’évaluation avec l’outil [Azure Migrate : Évaluation de serveurs](migrate-services-overview.md#azure-migrate-server-assessment-tool) et pour la migration sans agent en utilisant l’outil [Azure Migrate : Migration de serveurs](migrate-services-overview.md#azure-migrate-server-migration-tool).

L’[appliance Azure Migrate](migrate-appliance.md) est un appareil léger utilisé par Azure Migrate : Évaluation de serveurs et Migration de serveurs pour découvrir les machines virtuelles VMware locales, pour envoyer des données de métadonnées/de performances des machines virtuelles à Azure et pour la réplication des machines virtuelles VMware pendant la migration sans agent.

Vous pouvez déployer l’appliance à l’aide de deux méthodes :

- Configuration sur une machine virtuelle VMware au moyen d’un modèle OVA téléchargé. Il s’agit de la méthode décrite dans cet article.
- Configuration sur une machine virtuelle VMware ou une machine physique avec un script d’installation PowerShell. Vous devez utiliser [cette méthode](deploy-appliance-script.md) si vous ne pouvez pas configurer une machine virtuelle à l’aide d’un modèle OVA, ou si vous êtes dans Azure Government.

Après avoir créé l’appliance, vérifiez qu’elle peut se connecter à Azure Migrate Server Assessment, configurez-la pour la première fois, puis inscrivez-la auprès du projet Azure Migrate.


## <a name="appliance-deployment-ova"></a>Déploiement de l’appliance (OVA)

Pour configurer l’appliance avec un modèle OVA, vous :
- Fournissez un nom d’appliance et générez une clé de projet Azure Migrate sur le portail.
- Téléchargez un fichier de modèle OVA, puis importez-le dans vCenter Server.
- Créez l’appliance et vérifiez qu’elle peut se connecter à Azure Migrate Server Assessment.
- Configurez l’appliance pour la première fois, puis inscrivez-la auprès du projet Azure Migrate en utilisant la clé de projet Azure Migrate.

### <a name="generate-the-azure-migrate-project-key"></a>Générer la clé de projet Azure Migrate

1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Assessment**, sélectionnez **Découvrir**.
2. Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Oui, avec l’hyperviseur vSphere VMware**.
3. Dans **1 : Générer une clé de projet Azure Migrate**, attribuez un nom à l’appliance Azure Migrate que vous allez configurer pour la découverte de machines virtuelles VMware. Il doit s’agir d’un nom alphanumérique d’au maximum 14 caractères.
1. Cliquez sur **Générer une clé** pour lancer la création des ressources Azure nécessaires. Ne fermez pas la page Détecter des machines pendant la création des ressources.
1. Une fois les ressources Azure créées, une **clé de projet Azure Migrate** est générée.
1. Copiez la clé, car vous en aurez besoin pour terminer l’inscription de l’appliance pendant sa configuration.

### <a name="download-the-ova-template"></a>Télécharger le modèle OVA
Dans **2 : Télécharger l’appliance Azure Migrate**, sélectionnez le fichier .OVA et cliquez sur **Télécharger**. 


   ![Sélections pour la découverte de machines](./media/tutorial-assess-vmware/servers-discover.png)


   ![Sélections pour la génération de la clé](./media/tutorial-assess-vmware/generate-key-vmware.png)

### <a name="verify-security"></a>Vérifier la sécurité

Vérifiez que le fichier .OVA est sécurisé avant de le déployer.

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.
2. Exécutez la commande suivante pour générer le code de hachage du fichier OVA :
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Exemple d’utilisation : ```C:\>C:\>CertUtil -HashFile C:\Users\Administrator\Desktop\MicrosoftAzureMigration.ova SHA256```
3. Pour la dernière version de l’appliance, le code de hachage généré doit correspondre à ces [paramètres](./tutorial-discover-vmware.md#verify-security).



## <a name="create-the-appliance-vm"></a>Créer la machine virtuelle de l’appliance

Importez le fichier téléchargé, puis créez une machine virtuelle.

1. Dans la console du client vSphere, cliquez sur **File** (Fichier) > **Deploy OVF Template** (Déployer le modèle OVF).
![Commande de menu pour le déploiement d’un modèle OVF](./media/tutorial-assess-vmware/deploy-ovf.png)

2. Dans l’Assistant de déploiement du modèle OVF > **Source**, spécifiez l’emplacement du fichier .OVA.
3. Dans **Name** (Nom) et **Location** (Emplacement), spécifiez un nom convivial pour la machine virtuelle. Sélectionnez l’objet d’inventaire dans lequel la machine virtuelle doit être hébergée.
5. Dans **Host/Cluster** (Hôte/Cluster), spécifiez l’hôte ou le cluster sur lequel s’exécute la machine virtuelle.
6. Dans **Storage** (Stockage), spécifiez la destination du stockage de la machine virtuelle.
7. Dans **Disk Format** (Format de disque), spécifiez le type de disque et la taille.
8. Dans **Network Mapping** (Mappage réseau), spécifiez le réseau auquel se connectera la machine virtuelle. Le réseau a besoin d’une connexion à Internet pour envoyer des métadonnées à Azure Migrate Server Assessment.
9. Passez en revue les paramètres, confirmez-les, puis cliquez sur **Finish** (Terminer).


## <a name="verify-appliance-access-to-azure"></a>Vérifier l’accès de l’appliance à Azure

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
4. Si le compte d’utilisateur Azure utilisé pour la connexion dispose des [autorisations](./tutorial-discover-vmware.md#prepare-an-azure-user-account) adéquates sur les ressources Azure créées au moment de la génération de la clé, l’inscription de l’appliance est lancée.
1. Une fois l’inscription de l’appliance terminée, vous pouvez consulter les détails de l’inscription en cliquant sur **Afficher les détails**.


## <a name="start-continuous-discovery"></a>Démarrer la découverte en continu

L’appliance doit se connecter à vCenter Server pour découvrir les données de configuration et de performances des machines virtuelles.

1. À l’**Étape 1 : Fournir les informations d’identification vCenter Server**, cliquez sur **Ajouter des informations d’identification** pour attribuer un nom convivial aux informations d’identification, puis ajoutez un **nom d’utilisateur** et un **mot de passe** pour le compte vCenter Server dont se servira l’appliance pour détecter les machines virtuelles sur l’instance vCenter Server.
    - Vous devez avoir configuré un compte avec les autorisations nécessaires dans le [tutoriel précédent](./tutorial-discover-vmware.md#create-an-account-to-access-vcenter).
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

Passez en revue les tutoriels pour l'[évaluation VMware](./tutorial-assess-vmware-azure-vm.md) et la [migration sans agent](tutorial-migrate-vmware.md).
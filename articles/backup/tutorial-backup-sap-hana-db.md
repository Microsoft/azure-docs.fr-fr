---
title: Tutoriel - Sauvegarder des bases de données SAP HANA dans des machines virtuelles Azure
description: Dans ce tutoriel, découvrez comment sauvegarder des bases de données SAP HANA s’exécutant sur une machine virtuelle Azure dans un coffre Recovery Services de Sauvegarde Azure.
ms.topic: tutorial
ms.date: 02/24/2020
ms.openlocfilehash: 31a0a773096ec0f69e87bfd4a05f8ba98185e6cf
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94695212"
---
# <a name="tutorial-back-up-sap-hana-databases-in-an-azure-vm"></a>Tutoriel : Sauvegarder des bases de données SAP HANA dans une machine virtuelle Azure

Ce tutoriel vous explique comment sauvegarder des bases de données SAP HANA s’exécutant sur des machines virtuelles Azure dans un coffre Recovery Services de Sauvegarde Azure. Cet article porte sur les points suivants :

> [!div class="checklist"]
>
> * Créer et configurer un coffre
> * Découvrir des bases de données
> * Configurer des sauvegardes

[Voici](sap-hana-backup-support-matrix.md#scenario-support) tous les scénarios actuellement pris en charge.

>[!NOTE]
>Depuis le 1er août 2020, la sauvegarde SAP HANA pour RHEL (7.4, 7.6, 7.7 et 8.1) est en disponibilité générale.

## <a name="prerequisites"></a>Prérequis

Avant de configurer les sauvegardes, prenez soin d’effectuer les opérations suivantes :

* Identifiez ou créez un [coffre Recovery Services](backup-sql-server-database-azure-vms.md#create-a-recovery-services-vault) dans la même région et avec le même abonnement que la machine virtuelle qui exécute SAP HANA.
* Autorisez la connectivité de la machine virtuelle à Internet pour lui permettre d’atteindre Azure comme décrit dans la procédure [Configurer la connectivité réseau](#set-up-network-connectivity) ci-dessous.
* Vérifiez que la longueur combinée du nom de la machine virtuelle SAP HANA Server et du nom du groupe de ressources ne dépasse pas 84 caractères pour Azure Resource Manager (machines virtuelles ARM_) (et 77 caractères pour les machines virtuelles classiques). Cette limitation est due au fait que certains caractères sont réservés par le service.
* Le **hdbuserstore** doit inclure une clé qui respecte les critères suivants :
  * Elle doit être présente dans le **hdbuserstore** par défaut. Par défaut, il s’agit du compte `<sid>adm` sous lequel SAP HANA est installé.
  * Pour MDC, la clé doit pointer vers le port SQL de **NAMESERVER**. Pour SDC, elle doit pointer vers le port SQL de **INDEXSERVER**.
  * Elle doit disposer des informations d’identification nécessaires pour ajouter et supprimer des utilisateurs.
  * Notez que cette clé peut être supprimée après l’exécution du script de préinscription
* Exécutez le script de configuration de sauvegarde SAP HANA (script de préinscription) dans la machine virtuelle où HANA est installé en tant qu’utilisateur racine. [Ce script](https://aka.ms/scriptforpermsonhana) prépare le système HANA pour la sauvegarde. Pour en savoir plus sur le script de préinscription, reportez-vous à la section [Ce que fait le script de préinscription](#what-the-pre-registration-script-does).
* Si votre configuration HANA utilise des points de terminaison privés, exécutez le [script de préinscription](https://aka.ms/scriptforpermsonhana) avec le paramètre *-sn* ou *--skip-network-checks*.

>[!NOTE]
>Le script de préinscription installe **compat-unixODBC234** pour les charges de travail SAP HANA s’exécutant sur RHEL (7.4, 7.6 et 7.7) et **unixODBC** pour RHEL 8.1. [Ce package se trouve dans le dépôt des services de mise à jour de RHEL for SAP HANA (pour RHEL 7 Server) pour les solutions SAP (RPM)](https://access.redhat.com/solutions/5094721).  Pour une image RHEL de la Place de marché Azure, le dépôt est **rhui-rhel-sap-hana-for-rhel-7-server-rhui-e4s-rpms**.

## <a name="set-up-network-connectivity"></a>Configurer la connectivité réseau

Pour toutes les opérations, une base de données SAP HANA s’exécutant sur une machine virtuelle Azure nécessite une connectivité avec le service Sauvegarde Azure, Stockage Azure et Azure Active Directory. Pour ce faire, vous pouvez utiliser des points de terminaison privés ou autoriser l’accès aux IP publiques ou aux noms de domaine complets (FQDN) requis. Le fait de ne pas permettre une connectivité appropriée aux services Azure requis peut entraîner l’échec d’opérations telles que la détection de base de données, la configuration de la sauvegarde, l’exécution de sauvegardes et la restauration de données.

Le tableau suivant répertorie les différentes alternatives que vous pouvez utiliser pour établir la connectivité :

| **Option**                        | **Avantages**                                               | **Inconvénients**                                            |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Instances Private Endpoint                 | Autorisent les sauvegardes sur des adresses IP privées au sein du réseau virtuel  <br><br>   Fournissent un contrôle approfondi du côté du réseau et du coffre | Engendre des [coûts](https://azure.microsoft.com/pricing/details/private-link/) de point de terminaison privé standard |
| Balises de service NSG                  | Plus faciles à gérer car les modifications apportées à la plage sont fusionnées automatiquement   <br><br>   Aucun coût supplémentaire | Peut être utilisé uniquement avec les groupes de sécurité réseau  <br><br>    Fournit l’accès à l’ensemble du service |
| Balises FQDN de Pare-feu Azure          | Plus faciles à gérer, car les FQDN requis sont gérés automatiquement | Utilisabes avec Pare-feu Azure uniquement                         |
| Autoriser l’accès aux FQDN/adresses IP du service | Aucun coût supplémentaire   <br><br>  Fonctionne avec toutes les appliances de sécurité réseau et tous les pare-feu | Il peut être nécessaire d’accéder à un large éventail d’adresses IP ou de FQDN   |
| Utiliser un proxy HTTP                 | Un seul point d’accès Internet aux machines virtuelles                       | Frais supplémentaires d’exécution de machine virtuelle avec le logiciel de serveur proxy         |

De plus amples informations sur l’utilisation de ces options sont disponibles ci-dessous :

### <a name="private-endpoints"></a>Instances Private Endpoint

Les points de terminaison privés vous permettent de vous connecter en toute sécurité à votre coffre Recovery Services à partir de serveurs situés dans un réseau virtuel. Le point de terminaison privé utilise une adresse IP de l’espace d’adressage du réseau virtuel pour votre coffre. Le trafic réseau entre vos ressources dans le réseau virtuel et le coffre transite via votre réseau virtuel et une liaison privée sur le réseau principal de Microsoft. Cela élimine l’exposition de l’Internet public. Pour en savoir plus sur les points de terminaison privés pour Sauvegarde Azure, cliquez [ici](./private-endpoints.md).

### <a name="nsg-tags"></a>Balises NSG

Si vous utilisez des groupes de sécurité réseau (NSG), utilisez la balise de service *AzureBackup* pour autoriser l’accès sortant vers Sauvegarde Azure. En plus de l’étiquette pour Sauvegarde Azure, vous devez également autoriser la connectivité pour l’authentification et le transfert de données en créant des [règles NSG](../virtual-network/network-security-groups-overview.md#service-tags) similaires pour Azure AD (*AzureActiveDirectory*) et Stockage Azure (*Storage*). Les étapes suivantes décrivent le processus de création d’une règle pour la balise de Sauvegarde Azure :

1. Dans **Tous les services**, accédez à **Groupes de sécurité réseau** et sélectionnez le groupe de sécurité réseau.

1. Sous **PARAMÈTRES**, sélectionnez **Règles de sécurité de trafic sortant**.

1. Sélectionnez **Ajouter**. Entrez toutes les informations nécessaires à la création d’une nouvelle règle, comme décrit dans [paramètres de règle de sécurité](../virtual-network/manage-network-security-group.md#security-rule-settings). Vérifiez que l’option **Destination** est définie sur *Balise de service* et l’option **Balise de service de destination** sur *AzureBackup*.

1. Sélectionnez **Ajouter** pour enregistrer la règle de sécurité de trafic sortant que vous venez de créer.

De même, vous pouvez créer des [règles de sécurité de trafic sortant NSG](../virtual-network/network-security-groups-overview.md#service-tags) pour Stockage Azure et Azure AD. Pour plus d’informations sur les balises de service, consultez [cet article](../virtual-network/service-tags-overview.md).

### <a name="azure-firewall-tags"></a>Balises Pare-feu Azure

Si vous utilisez Pare-feu Azure, créez une règle d’application en utilisant la [balise FQDN de Pare-feu Azure](../firewall/fqdn-tags.md) *AzureBackup*. Cela autorise tout accès sortant vers Sauvegarde Azure.

### <a name="allow-access-to-service-ip-ranges"></a>Autoriser l’accès aux plages d’adresses IP du service

Si vous choisissez d’autoriser l’accès aux adresses IP du service, reportez-vous aux plages d’adresses IP répertoriées dans le fichier JSON accessible [ici](https://www.microsoft.com/download/confirmation.aspx?id=56519). Vous devez autoriser l’accès aux adresses IP correspondant à Sauvegarde Azure, Stockage Azure et Azure Active Directory.

### <a name="allow-access-to-service-fqdns"></a>Autoriser l’accès aux FQDN du service

Vous pouvez également utiliser les FQDN suivants pour autoriser l’accès aux services requis à partir de vos serveurs :

| Service    | Noms de domaine auxquels accéder                             |
| -------------- | ------------------------------------------------------------ |
| Sauvegarde Azure  | `*.backup.windowsazure.com`                             |
| Stockage Azure | `*.blob.core.windows.net` <br><br> `*.queue.core.windows.net` |
| Azure AD      | Autoriser l’accès aux FQDN en vertu des sections 56 et 59 conformément à [cet article](/office365/enterprise/urls-and-ip-address-ranges#microsoft-365-common-and-office-online) |

### <a name="use-an-http-proxy-server-to-route-traffic"></a>Utiliser un serveur proxy HTTP pour acheminer le trafic

Lorsque vous sauvegardez une base de données SAP HANA qui s’exécute sur une machine virtuelle Azure, l’extension de sauvegarde sur la machine virtuelle utilise les API HTTPS pour envoyer des commandes de gestion à Sauvegarde Azure et des données à Stockage Azure. L’extension de sauvegarde utilise également Azure AD pour l’authentification. Acheminez le trafic de l’extension de sauvegarde pour ces trois services via le proxy HTTP. Utilisez la liste des adresses IP et des FQDN ci-dessus pour autoriser l’accès aux services requis. Les serveurs proxy authentifiés ne sont pas pris en charge.

## <a name="what-the-pre-registration-script-does"></a>Ce que fait le script de préinscription

Le script de préinscription assure les fonctions suivantes :

* En fonction de votre distribution Linux, le script installe ou met à jour tous les packages nécessaires à l’agent de Sauvegarde Azure sur votre distribution.
* Il effectue les vérifications de connectivité réseau sortante avec les serveurs de Sauvegarde Azure et les services dépendants comme Azure Active Directory et Stockage Azure.
* Il se connecte à votre système HANA à l’aide de la clé utilisateur figurant dans les [prérequis](#prerequisites). La clé utilisateur permet de créer un utilisateur de sauvegarde (AZUREWLBACKUPHANAUSER) dans le système HANA et **peut être supprimée dès lors que le script de préinscription a été correctement exécuté**.
* AZUREWLBACKUPHANAUSER reçoit les rôles et autorisations nécessaires suivants :
  * Pour MDC : DATABASE ADMIN et BACKUP ADMIN (depuis HANA 2.0 SPS05 et versions ultérieures) : pour créer des bases de données lors de la restauration.
  * Pour SDC : BACKUP ADMIN : pour créer des bases de données lors de la restauration.
  * CATALOG READ : permet de lire le catalogue de sauvegarde.
  * SAP_INTERNAL_HANA_SUPPORT : permet d’accéder à certaines tables privées. Obligatoire uniquement pour les versions SDC et MDC antérieures à HANA 2.0 SPS04 rév. 46. Cela n’est pas obligatoire pour HANA 2.0 SPS04 rév. 46 et versions ultérieures, car nous obtenons maintenant les informations requises des tables publiques avec le correctif de l’équipe HANA.
* Le script ajoute une clé à **hdbuserstore** pour AZUREWLBACKUPHANAUSER afin que le plug-in de sauvegarde HANA gère toutes les opérations (requêtes de base de données, opérations de restauration, configuration et exécution de la sauvegarde).

>[!NOTE]
> Vous pouvez transmettre explicitement la clé utilisateur indiquée dans le cadre des [prérequis](#prerequisites) en tant que paramètre au script de pré-inscription : `-sk SYSTEM_KEY_NAME, --system-key SYSTEM_KEY_NAME` <br><br>
>Pour connaître les autres paramètres acceptés par le script, utilisez la commande `bash msawb-plugin-config-com-sap-hana.sh --help`

Pour confirmer la création de la clé, exécutez la commande HDBSQL sur la machine HANA avec les informations d’identification SIDADM :

```hdbsql
hdbuserstore list
```

La sortie de commande doit afficher la clé {SID} {DBNAME} avec l’utilisateur en tant que AZUREWLBACKUPHANAUSER.

>[!NOTE]
> Vérifiez que vous disposez d’un ensemble unique de fichiers SSFS sous `/usr/sap/{SID}/home/.hdb/`. Vous ne devez trouver qu’un seul dossier dans ce chemin d’accès.

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Un coffre Recovery Services est une entité qui stocke les sauvegardes et les points de récupération créés au fil du temps. Le coffre Recovery Services contient également les stratégies de sauvegarde associées aux machines virtuelles protégées.

Pour créer un archivage de Recovery Services :

1. Connectez-vous à votre abonnement sur le [portail Azure](https://portal.azure.com/).

2. Dans le menu de gauche, sélectionnez **Tous les services**

   ![Sélectionner Tous les services](./media/tutorial-backup-sap-hana-db/all-services.png)

3. Dans la boîte de dialogue **Tous les services**, entrez **Recovery Services**. Liste des filtres de ressources variant en fonction de votre entrée. Dans la liste des ressources, sélectionnez **Coffres Recovery Services**.

   ![Sélectionner Coffres Recovery Services](./media/tutorial-backup-sap-hana-db/recovery-services-vaults.png)

4. Dans le tableau de bord **Coffres Recovery Services**, sélectionnez **Ajouter**.

   ![Ajouter un coffre Recovery Services](./media/tutorial-backup-sap-hana-db/add-vault.png)

   La boîte de dialogue **Coffre Recovery Services** s’ouvre. Attribuez des valeurs aux champs **Nom, Abonnement, Groupe de ressources** et **Emplacement**

   ![Créer un coffre Recovery Services](./media/tutorial-backup-sap-hana-db/create-vault.png)

   * **Name** : Le nom est utilisé pour identifier le coffre Recovery Services et doit être propre à l’abonnement Azure. Spécifiez un nom composé d’au moins deux caractères, mais sans dépasser 50 caractères. Il doit commencer par une lettre et ne peut être constitué que de lettres, chiffres et traits d’union. Pour ce tutoriel, nous avons utilisé le nom **SAPHanaVault**.
   * **Abonnement**: choisissez l’abonnement à utiliser. Si vous êtes membre d’un seul abonnement, son nom s’affiche. Si vous ne savez pas quel abonnement utiliser, utilisez l’abonnement par défaut (suggéré). Vous ne disposez de plusieurs choix que si votre compte professionnel ou scolaire est associé à plusieurs abonnements Azure. Ici, nous avons utilisé l’abonnement de **laboratoire de solution SAP HANA**.
   * **Groupe de ressources** : Utilisez un groupe de ressources existant ou créez-en un. Ici, nous avons utilisé **SAPHANADemo**.<br>
   Pour afficher la liste des groupes de ressources disponibles dans votre abonnement, sélectionnez **Utiliser existant**, puis sélectionnez une ressource dans la zone de liste déroulante. Pour créer un groupe de ressources, sélectionnez **Créer** et entrez le nom. Pour obtenir des informations complètes sur les groupes de ressources, consultez [Vue d’ensemble d’Azure Resource Manager](../azure-resource-manager/management/overview.md).
   * **Emplacement** : sélectionnez la région géographique du coffre. Le coffre doit se trouver dans la même région que la machine virtuelle exécutant SAP HANA. Nous avons utilisé **USA Est 2**.

5. Sélectionnez **Vérifier + créer**.

   ![Sélectionner Vérifier + créer](./media/tutorial-backup-sap-hana-db/review-create.png)

Le coffre Recovery Services est maintenant créé.

## <a name="discover-the-databases"></a>Détecter les bases de données

1. Dans le coffre, dans **Démarrer**, sélectionnez **Sauvegarde**. Dans la zone **Où s’exécute votre charge de travail ?** , sélectionnez **SAP HANA dans les machines virtuelles Azure**.
2. Sélectionnez **Démarrer la découverte**. Cette opération lance la détection des machines virtuelles Linux non protégées dans la région du coffre. Vous verrez la machine virtuelle Azure que vous souhaitez protéger.
3. Dans **Sélectionner les machines virtuelles**, sélectionnez le lien pour télécharger le script qui donne les autorisations au service Sauvegarde Azure d’accéder aux machines virtuelles SAP HANA pour la découverte des bases de données.
4. Exécutez le script sur la machine virtuelle hébergeant les bases de données SAP HANA que vous souhaitez sauvegarder.
5. Après avoir exécuté le script sur la machine virtuelle, dans **Sélectionner les machines virtuelles**, sélectionnez la machine virtuelle. Sélectionnez ensuite **Découvrir les bases de données**.
6. Le service Sauvegarde Azure détecte toutes les bases de données SAP HANA résidant sur la machine virtuelle. Lors de la détection, le service Sauvegarde Azure inscrit la machine virtuelle auprès du coffre et y installe une extension. Aucun agent n’est installé sur la base de données.

   ![Détecter les bases de données](./media/tutorial-backup-sap-hana-db/database-discovery.png)

## <a name="configure-backup"></a>Configurer une sauvegarde

Maintenant que les bases de données à sauvegarder sont découvertes, nous allons activer la sauvegarde.

1. Sélectionnez **Configurer la sauvegarde**.

   ![Configurer une sauvegarde](./media/tutorial-backup-sap-hana-db/configure-backup.png)

2. Dans **Sélectionner les éléments à sauvegarder**, sélectionnez une ou plusieurs bases de données à protéger, puis sélectionnez **OK**.

   ![Sélectionner les éléments à sauvegarder](./media/tutorial-backup-sap-hana-db/select-items-to-backup.png)

3. Dans **Stratégie de sauvegarde > Choisir une stratégie de sauvegarde**, créez une stratégie de sauvegarde pour les bases de données en suivant les instructions de la section suivante.

   ![Choisir une stratégie de sauvegarde](./media/tutorial-backup-sap-hana-db/backup-policy.png)

4. Après avoir créé la stratégie, dans le menu **Sauvegarde**, sélectionnez **Activer la sauvegarde**.

   ![Sélectionner Activer la sauvegarde](./media/tutorial-backup-sap-hana-db/enable-backup.png)

5. Vous pouvez suivre la progression de la configuration de la sauvegarde dans la zone **Notifications** du portail.

## <a name="creating-a-backup-policy"></a>Création d’une stratégie de sauvegarde

Une stratégie de sauvegarde définit le moment auquel les sauvegardes sont effectuées, ainsi que leur durée de rétention.

* Une stratégie est créée au niveau du coffre.
* Plusieurs coffres peuvent utiliser la même stratégie de sauvegarde, mais vous devez appliquer la stratégie de sauvegarde à chaque coffre.

Spécifiez les paramètres de stratégie comme suit :

1. Dans **Nom de la stratégie**, entrez le nom de la nouvelle stratégie. Dans le cas présent, entrez **SAPHANA**.

   ![Entrer le nom de la nouvelle stratégie](./media/tutorial-backup-sap-hana-db/new-policy.png)

2. Dans **Stratégie de sauvegarde complète**, sélectionnez une **Fréquence de sauvegarde**. Vous pouvez choisir **Quotidienne** ou **Hebdomadaire**. Pour ce tutoriel, nous avons choisi la sauvegarde **Quotidienne**.

   ![Sélectionner une fréquence de sauvegarde](./media/tutorial-backup-sap-hana-db/backup-frequency.png)

3. Dans **Durée de rétention**, configurez les paramètres de rétention pour la sauvegarde complète.
   * Par défaut, toutes les options sont sélectionnées. Désactivez les limites de durée de rétention que vous ne souhaitez pas utiliser et définissez celles qui vous intéressent.
   * La période de rétention minimale est de sept jours pour tous les types de sauvegardes (complète/différentielle/fichier journal).
   * Des points de récupération sont marqués pour la rétention et varient selon la durée de rétention. Par exemple, si vous sélectionnez une sauvegarde complète quotidienne, seule une sauvegarde complète est déclenchée chaque jour.
   * La sauvegarde d’un jour spécifique est marquée et conservée conformément à la durée de rétention hebdomadaire et aux paramètres.
   * Les durées de rétention mensuelle et annuelle ont le même comportement.
4. Dans le menu de **stratégie Sauvegarde complète**, cliquez sur **OK** pour accepter les paramètres.
5. Sélectionnez ensuite **Sauvegarde différentielle** pour ajouter une stratégie différentielle.
6. Dans la stratégie **Sauvegarde différentielle**, sélectionnez **Activer** pour ouvrir les contrôles de fréquence et de rétention. Nous avons activé une sauvegarde différentielle chaque **dimanche** à **2:00**, qui est conservée pendant **30 jours**.

   ![Stratégie de sauvegarde différentielle](./media/tutorial-backup-sap-hana-db/differential-backup-policy.png)

   >[!NOTE]
   >Les sauvegardes incrémentielles sont désormais disponibles en préversion publique. Vous pouvez choisir une sauvegarde différentielle ou incrémentielle comme sauvegarde quotidienne, mais pas les deux.
   >
7. Dans **Stratégie de sauvegarde incrémentielle**, sélectionnez **Activer** pour ouvrir les contrôles de fréquence et de conservation.
    * Vous pouvez déclencher au plus une sauvegarde incrémentielle par jour.
    * Les sauvegardes incrémentielles peuvent être conservées jusqu’à 180 jours. Si vous avez besoin d’une durée de rétention supérieure, vous devez utiliser des sauvegardes complètes.

    ![Stratégie de sauvegarde incrémentielle](./media/backup-azure-sap-hana-database/incremental-backup-policy.png)

8. Sélectionnez **OK** pour enregistrer la stratégie et revenir au menu principal **Stratégie de sauvegarde**.
9. Sélectionnez **Sauvegarde de fichier journal** pour ajouter une stratégie de sauvegarde de fichier journal.
   * L’option **Sauvegarde de fichier journal** est définie par défaut sur **Activer**. Cette option ne peut pas être désactivée, car SAP HANA gère toutes les sauvegardes de fichiers journaux.
   * Nous avons défini **2 heures** comme planification de sauvegarde et **15 jours** de période de rétention.

    ![Stratégie de sauvegarde de fichier journal](./media/tutorial-backup-sap-hana-db/log-backup-policy.png)

   >[!NOTE]
   > Les sauvegardes de fichiers journaux ne commencent à se produire qu’en cas de réussite d’une sauvegarde complète.
   >

10. Sélectionnez **OK** pour enregistrer la stratégie et revenir au menu principal **Stratégie de sauvegarde**.
11. Après avoir défini la stratégie de sauvegarde, sélectionnez **OK**.

Vous avez maintenant configuré les sauvegardes de vos bases de données SAP HANA.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [exécuter des sauvegardes à la demande sur des bases de données SAP HANA qui s’exécutent sur des machines virtuelles Azure](backup-azure-sap-hana-database.md#run-an-on-demand-backup)
* Découvrez comment [restaurer des bases de données SAP HANA qui s’exécutent sur des machines virtuelles Azure](sap-hana-db-restore.md)
* Découvrez comment [gérer des bases de données SAP HANA sauvegardées à l’aide de Sauvegarde Azure](sap-hana-db-manage.md)
* Découvrez comment [résoudre les problèmes courants lors de la sauvegarde de bases de données SAP HANA](backup-azure-sap-hana-database-troubleshoot.md)
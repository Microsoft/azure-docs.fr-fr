---
title: Sauvegarder une base de données SAP HANA sur Azure avec Sauvegarde Azure
description: Dans cet article, découvrez comment sauvegarder des bases de données SAP HANA sur des machines virtuelles Azure avec le service Sauvegarde Azure.
ms.topic: conceptual
ms.date: 11/12/2019
ms.openlocfilehash: f7957670b3ba98c640ebc53c6427273ca75a4e6d
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94682845"
---
# <a name="back-up-sap-hana-databases-in-azure-vms"></a>Sauvegarder des bases de données SAP HANA dans des machines virtuelles Azure

Les bases de données SAP HANA sont des charges de travail critiques nécessitant un faible objectif de point de récupération (RPO) et une conservation à long terme. Vous pouvez sauvegarder les bases de données SAP HANA qui s’exécutent sur les machines virtuelles Azure en utilisant [Sauvegarde Azure](backup-overview.md).

Cet article explique comment sauvegarder des bases de données SAP HANA s’exécutant sur des machines virtuelles Azure pour un coffre Recovery Services de la Sauvegarde Azure.

Dans cet article, vous allez apprendre à :
> [!div class="checklist"]
>
> * Créer et configurer un coffre
> * Découvrir des bases de données
> * Configurer des sauvegardes
> * Exécuter un travail de sauvegarde à la demande

>[!NOTE]
>Depuis le 1er août 2020, la sauvegarde SAP HANA pour RHEL (7.4, 7.6, 7.7 et 8.1) est en disponibilité générale.

>[!NOTE]
>**La suppression réversible pour SQL Server dans une machine virtuelle Azure et la suppression réversible pour SAP HANA dans les charges de travail de machine virtuelle Azure** sont maintenant disponibles en préversion.<br>
>Pour vous inscrire à la préversion, écrivez-nous à l’adresse suivante : [AskAzureBackupTeam@microsoft.com](mailto:AskAzureBackupTeam@microsoft.com).

## <a name="prerequisites"></a>Prérequis

Reportez-vous aux sections [Conditions préalables](tutorial-backup-sap-hana-db.md#prerequisites) et [Ce que fait le script de préinscription](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does) pour configurer la base de données pour la sauvegarde.

### <a name="establish-network-connectivity"></a>Établir la connectivité réseau

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

#### <a name="private-endpoints"></a>Instances Private Endpoint

Les points de terminaison privés vous permettent de vous connecter en toute sécurité à votre coffre Recovery Services à partir de serveurs situés dans un réseau virtuel. Le point de terminaison privé utilise une adresse IP de l’espace d’adressage du réseau virtuel pour votre coffre. Le trafic réseau entre vos ressources dans le réseau virtuel et le coffre transite via votre réseau virtuel et une liaison privée sur le réseau principal de Microsoft. Cela élimine l’exposition de l’Internet public. Pour en savoir plus sur les points de terminaison privés pour Sauvegarde Azure, cliquez [ici](./private-endpoints.md).

#### <a name="nsg-tags"></a>Balises NSG

Si vous utilisez des groupes de sécurité réseau (NSG), utilisez la balise de service *AzureBackup* pour autoriser l’accès sortant vers Sauvegarde Azure. En plus de l’étiquette pour Sauvegarde Azure, vous devez également autoriser la connectivité pour l’authentification et le transfert de données en créant des [règles NSG](../virtual-network/network-security-groups-overview.md#service-tags) similaires pour Azure AD (*AzureActiveDirectory*) et Stockage Azure (*Storage*).  Les étapes suivantes décrivent le processus de création d’une règle pour la balise de Sauvegarde Azure :

1. Dans **Tous les services**, accédez à **Groupes de sécurité réseau** et sélectionnez le groupe de sécurité réseau.

1. Sous **PARAMÈTRES**, sélectionnez **Règles de sécurité de trafic sortant**.

1. Sélectionnez **Ajouter**. Entrez toutes les informations nécessaires à la création d’une nouvelle règle, comme décrit dans [paramètres de règle de sécurité](../virtual-network/manage-network-security-group.md#security-rule-settings). Vérifiez que l’option **Destination** est définie sur *Balise de service* et l’option **Balise de service de destination** sur *AzureBackup*.

1. Sélectionnez **Ajouter** pour enregistrer la règle de sécurité de trafic sortant que vous venez de créer.

De même, vous pouvez créer des règles de sécurité de trafic sortant NSG pour Stockage Azure et Azure AD. Pour plus d’informations sur les balises de service, consultez [cet article](../virtual-network/service-tags-overview.md).

#### <a name="azure-firewall-tags"></a>Balises Pare-feu Azure

Si vous utilisez Pare-feu Azure, créez une règle d’application en utilisant la [balise FQDN de Pare-feu Azure](../firewall/fqdn-tags.md) *AzureBackup*. Cela autorise tout accès sortant vers Sauvegarde Azure.

#### <a name="allow-access-to-service-ip-ranges"></a>Autoriser l’accès aux plages d’adresses IP du service

Si vous choisissez d’autoriser l’accès aux adresses IP du service, reportez-vous aux plages d’adresses IP répertoriées dans le fichier JSON accessible [ici](https://www.microsoft.com/download/confirmation.aspx?id=56519). Vous devez autoriser l’accès aux adresses IP correspondant à Sauvegarde Azure, Stockage Azure et Azure Active Directory.

#### <a name="allow-access-to-service-fqdns"></a>Autoriser l’accès aux FQDN du service

Vous pouvez également utiliser les FQDN suivants pour autoriser l’accès aux services requis à partir de vos serveurs :

| Service    | Noms de domaine auxquels accéder                             |
| -------------- | ------------------------------------------------------------ |
| Sauvegarde Azure  | `*.backup.windowsazure.com`                             |
| Stockage Azure | `*.blob.core.windows.net` <br><br> `*.queue.core.windows.net` |
| Azure AD      | Autoriser l’accès aux FQDN en vertu des sections 56 et 59 conformément à [cet article](/office365/enterprise/urls-and-ip-address-ranges#microsoft-365-common-and-office-online) |

#### <a name="use-an-http-proxy-server-to-route-traffic"></a>Utiliser un serveur proxy HTTP pour acheminer le trafic

Lorsque vous sauvegardez une base de données SAP HANA qui s’exécute sur une machine virtuelle Azure, l’extension de sauvegarde sur la machine virtuelle utilise les API HTTPS pour envoyer des commandes de gestion à Sauvegarde Azure et des données à Stockage Azure. L’extension de sauvegarde utilise également Azure AD pour l’authentification. Acheminez le trafic de l’extension de sauvegarde pour ces trois services via le proxy HTTP. Utilisez la liste des adresses IP et des FQDN ci-dessus pour autoriser l’accès aux services requis. Les serveurs proxy authentifiés ne sont pas pris en charge.

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="discover-the-databases"></a>Détecter les bases de données

1. Dans le coffre, dans **Démarrer**, sélectionnez **Sauvegarde**. Dans la zone **Où s’exécute votre charge de travail ?** , sélectionnez **SAP HANA dans les machines virtuelles Azure**.
2. Sélectionnez **Démarrer la découverte**. Cette opération lance la détection des machines virtuelles Linux non protégées dans la région du coffre.

   * Après la détection, les machines virtuelles non protégées apparaissent dans le portail en étant répertoriées par nom et groupe de ressources.
   * Si une machine virtuelle n’est pas répertoriée alors qu’elle devrait l’être, vérifiez si elle a déjà été sauvegardée dans un coffre.
   * Plusieurs machines virtuelles peuvent avoir le même nom, mais appartenir à différents groupes de ressources.

3. Dans **Sélectionner les machines virtuelles**, sélectionnez le lien pour télécharger le script qui donne les autorisations au service Sauvegarde Azure d’accéder aux machines virtuelles SAP HANA pour la découverte des bases de données.
4. Exécutez le script sur chacune des machines virtuelles hébergeant les bases de données SAP HANA que vous souhaitez sauvegarder.
5. Après avoir exécuté le script sur les machines virtuelles, dans **Sélectionner les machines virtuelles**, sélectionnez les machines souhaitées. Sélectionnez ensuite **Découvrir les bases de données**.
6. Le service Sauvegarde Azure détecte toutes les bases de données SAP HANA résidant sur la machine virtuelle. Lors de la détection, le service Sauvegarde Azure inscrit la machine virtuelle auprès du coffre et y installe une extension. Aucun agent n’est installé sur la base de données.

    ![Détection des bases de données SAP HANA](./media/backup-azure-sap-hana-database/hana-discover.png)

## <a name="configure-backup"></a>Configurer une sauvegarde  

Maintenant, activez la sauvegarde.

1. À l’étape 2, sélectionnez **Configurer la sauvegarde**.

    ![Configurez une sauvegarde](./media/backup-azure-sap-hana-database/configure-backup.png)
2. Dans **Sélectionner les éléments à sauvegarder**, sélectionnez toutes les bases de données que vous souhaitez protéger, puis choisissez **OK**.

    ![Sélectionner les éléments à sauvegarder](./media/backup-azure-sap-hana-database/select-items.png)
3. Dans **Stratégie de sauvegarde** > **Choisir une stratégie de sauvegarde**, créez une stratégie de sauvegarde pour les bases de données en suivant les instructions ci-dessous.

    ![Choisir une stratégie de sauvegarde](./media/backup-azure-sap-hana-database/backup-policy.png)
4. Après avoir créé la stratégie, dans le menu **Sauvegarde**, sélectionnez **Activer la sauvegarde**.

    ![Activer la sauvegarde](./media/backup-azure-sap-hana-database/enable-backup.png)
5. Vous pouvez suivre la progression de la configuration de la sauvegarde dans la zone **Notifications** du portail.

### <a name="create-a-backup-policy"></a>Créer une stratégie de sauvegarde

Une stratégie de sauvegarde définit le moment auquel les sauvegardes sont effectuées, ainsi que leur durée de rétention.

* Une stratégie est créée au niveau du coffre.
* Plusieurs coffres peuvent utiliser la même stratégie de sauvegarde, mais vous devez appliquer la stratégie de sauvegarde à chaque coffre.

>[!NOTE]
>Sauvegarde Azure ne s’ajuste pas automatiquement au changement d’heure lorsque vous sauvegardez une base de données SAP HANA s’exécutant sur une machine virtuelle Azure.
>
>Modifiez la stratégie manuellement en fonction des besoins.

Spécifiez les paramètres de stratégie comme suit :

1. Dans **Nom de la stratégie**, entrez le nom de la nouvelle stratégie.

   ![Saisir un nom de stratégie](./media/backup-azure-sap-hana-database/policy-name.png)
1. Dans **Stratégie de sauvegarde complète**, comme **Fréquence de sauvegarde**, sélectionnez **Tous les jours** ou **Toutes les semaines**.
   * **Daily (Quotidienne)** : sélectionnez l’heure et le fuseau horaire de début du travail de sauvegarde.
       * Vous devez exécuter une sauvegarde complète. Vous ne pouvez pas désactiver cette option.
       * Sélectionnez **Sauvegarde complète** pour afficher la stratégie.
       * Si vous choisissez des sauvegardes complètes quotidiennes, vous ne pouvez pas créer de sauvegardes différentielles.
   * **Hebdomadaire** : sélectionnez le jour de la semaine, l’heure et le fuseau horaire de début du travail de sauvegarde.

   ![Sélectionner la fréquence de sauvegarde](./media/backup-azure-sap-hana-database/backup-frequency.png)

1. Dans **Durée de rétention**, configurez les paramètres de rétention pour la sauvegarde complète.
    * Par défaut, toutes les options sont sélectionnées. Désactivez les limites de durée de rétention que vous ne souhaitez pas utiliser et définissez celles qui vous intéressent.
    * La période de rétention minimale est de sept jours pour tous les types de sauvegardes (complète/différentielle/fichier journal).
    * Des points de récupération sont marqués pour la rétention et varient selon la durée de rétention. Par exemple, si vous sélectionnez une sauvegarde complète quotidienne, seule une sauvegarde complète est déclenchée chaque jour.
    * La sauvegarde d’un jour spécifique est marquée et conservée conformément à la durée de rétention hebdomadaire et aux paramètres.
    * Les durées de rétention mensuelle et annuelle ont le même comportement.

1. Dans le menu de **stratégie Sauvegarde complète**, cliquez sur **OK** pour accepter les paramètres.
1. Sélectionnez **Sauvegarde différentielle** pour ajouter une stratégie différentielle.
1. Dans la stratégie **Sauvegarde différentielle**, sélectionnez **Activer** pour ouvrir les contrôles de fréquence et de rétention.
    * Vous pouvez déclencher au plus une sauvegarde différentielle par jour.
    * Les sauvegardes différentielles peuvent être conservées jusqu’à 180 jours. Si vous avez besoin d’une durée de rétention supérieure, vous devez utiliser des sauvegardes complètes.

    ![Stratégie de sauvegarde différentielle](./media/backup-azure-sap-hana-database/differential-backup-policy.png)

    > [!NOTE]
    > Les sauvegardes incrémentielles sont désormais prises en charge dans une version préliminaire publique. Vous pouvez choisir une sauvegarde différentielle ou incrémentielle comme sauvegarde quotidienne, mais pas les deux.
1. Dans la stratégie **Sauvegarde incrémentielle**, sélectionnez **Activer** pour ouvrir les contrôles de fréquence et de rétention.
    * Vous pouvez déclencher au plus une sauvegarde incrémentielle par jour.
    * Les sauvegardes incrémentielles peuvent être conservées jusqu’à 180 jours. Si vous avez besoin d’une durée de rétention supérieure, vous devez utiliser des sauvegardes complètes.

    ![Stratégie de sauvegarde incrémentielle](./media/backup-azure-sap-hana-database/incremental-backup-policy.png)

1. Sélectionnez **OK** pour enregistrer la stratégie et revenir au menu principal **Stratégie de sauvegarde**.
1. Sélectionnez **Sauvegarde de fichier journal** pour ajouter une stratégie de sauvegarde de fichier journal.
    * Dans **Sauvegarde de fichier journal**, sélectionnez **Activer**.  Cette option ne peut pas être désactivée, car SAP HANA gère toutes les sauvegardes de fichiers journaux.
    * Définissez les contrôles de fréquence et de rétention.

    > [!NOTE]
    > Les sauvegardes de fichiers journaux ne commencent à s’effectuer qu’en cas de réussite d’une sauvegarde complète.

1. Sélectionnez **OK** pour enregistrer la stratégie et revenir au menu principal **Stratégie de sauvegarde**.
1. Après avoir défini la stratégie de sauvegarde, sélectionnez **OK**.

> [!NOTE]
> Chaque sauvegarde de fichier journal est chaînée à la sauvegarde complète précédente pour former une chaîne de récupération. Cette sauvegarde complète est conservée jusqu’à la fin de la durée de conservation de la dernière sauvegarde de fichier journal. Il est donc possible que la sauvegarde complète soit conservée pour une durée supplémentaire afin que tous les journaux puissent être récupérés. Supposons que l’utilisateur effectue une sauvegarde complète hebdomadaire, une sauvegarde différentielle par jour, et que les journaux enregistrent deux heures d’activité. Tous sont conservés 30 jours. Cependant, la sauvegarde complète hebdomadaire ne peut être réellement nettoyée/supprimée que lorsque la sauvegarde complète suivante est disponible, à savoir après 30 + 7 jours. Par exemple, une sauvegarde complète hebdomadaire a lieu le 16 novembre. Conformément à la stratégie de conservation, elle doit être conservée jusqu’au 16 décembre. La dernière sauvegarde de fichier journal de cette sauvegarde complète a lieu avant la prochaine sauvegarde complète planifiée, le 22 novembre. Tant que ce journal n’est pas disponible, jusqu’au 22 décembre, la sauvegarde complète du 16 novembre ne peut pas être supprimée. La sauvegarde complète du 16 novembre est donc conservée jusqu’au 22 décembre.

## <a name="run-an-on-demand-backup"></a>Exécuter une sauvegarde à la demande

Les sauvegardes s’exécutent conformément à la planification de la stratégie. Vous pouvez exécuter une sauvegarde à la demande en procédant comme suit :

1. Dans le menu du coffre, sélectionnez **Éléments de sauvegarde**.
2. Dans **Éléments de sauvegarde**, sélectionnez la machine virtuelle exécutant la base de données SAP HANA, puis **Sauvegarder maintenant**.
3. Dans **Sauvegarder maintenant**, choisissez le type de sauvegarde que vous souhaitez effectuer. Sélectionnez ensuite **OK**. Cette sauvegarde sera conservée en fonction de la stratégie associée à cet élément de sauvegarde.
4. Surveiller les notifications du portail. Vous pouvez surveiller la progression du travail dans le tableau de bord du coffre > **Travaux de sauvegarde** > **En cours d’exécution**. Selon la taille de votre base de données, la création de la sauvegarde initiale peut prendre un certain temps.

Par défaut, la période de rétention des sauvegardes à la demande est de 45 jours.

## <a name="run-sap-hana-studio-backup-on-a-database-with-azure-backup-enabled"></a>Exécuter une sauvegarde SAP HANA Studio d’une base de données pour laquelle le service Sauvegarde Azure est activé

Si vous souhaitez effectuer une sauvegarde locale (à l’aide de HANA Studio) d’une base de données en cours de sauvegarde par le service Sauvegarde Azure, procédez comme suit :

1. Attendez la fin de toute sauvegarde complète ou de fichier journal de la base de données. Vérifiez l’état dans SAP HANA Studio/Cockpit.
1. Désactivez les sauvegardes de fichiers journaux, puis définissez le catalogue de sauvegarde sur le système de fichiers pour la base de données appropriée.
1. Pour ce faire, double-cliquez sur **systemdb** > **Configuration (Configuration)**  > **Select Database (Sélectionner la base de données)**  > **Filter (Log) (Filtrer [journal])** .
1. Définissez **enable_auto_log_backup** sur **No (Non)** .
1. Définissez **log_backup_using_backint** sur **False (Faux)** .
1. Définissez **catalog_backup_using_backint** sur **False**.
1. Effectuez une sauvegarde complète à la demande de la base de données.
1. Attendez la fin de la sauvegarde complète et de la sauvegarde du catalogue.
1. Rétablissez les paramètres précédents sur les valeurs pour Azure :
    * Définissez **enable_auto_log_backup** sur **Yes (Oui)** .
    * Définissez **log_backup_using_backint** sur **True (Vrai)** .
    * Définissez **catalog_backup_using_backint** sur **True**.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [restaurer des bases de données SAP HANA qui s’exécutent sur des machines virtuelles Azure](./sap-hana-db-restore.md)
* Découvrez comment [gérer des bases de données SAP HANA sauvegardées à l’aide de Sauvegarde Azure](./sap-hana-db-manage.md)

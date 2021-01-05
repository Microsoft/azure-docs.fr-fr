---
title: Sauvegarder Azure Database pour PostgreSQL
description: Découvrir la sauvegarde Azure Database pour PostgreSQL avec conservation à long terme (préversion)
ms.topic: conceptual
ms.date: 09/08/2020
ms.custom: references_regions
ms.openlocfilehash: 32837e5bb871f7db5024569efc27940047aeacd7
ms.sourcegitcommit: f7084d3d80c4bc8e69b9eb05dfd30e8e195994d8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97734638"
---
# <a name="azure-database-for-postgresql-backup-with-long-term-retention-preview"></a>Sauvegarde Azure Database pour PostgreSQL avec conservation à long terme (préversion)

Sauvegarde Azure et les services de base de données Azure constituent ensemble une solution de sauvegarde d’entreprise pour les serveurs Azure Database pour PostgreSQL qui conserve les sauvegardes jusqu’à 10 années.

Outre la conservation à long terme, la solution a également de nombreuses autres fonctionnalités, comme indiqué ci-dessous :

- Contrôle d’accès en fonction du rôle Azure (Azure RBAC) à la base de données à l’aide de l’authentification Azure Active Directory et Managed Service Identity (MSI).
- Sauvegarde planifiée et à la demande contrôlée par le client au niveau de la base de données individuelle.
- Restaurations au niveau de la base de données sur un serveur Postgres ou directement dans le stockage d’objets blob.
- Conservation à long terme.
- Surveillance centralisée de toutes les opérations et de tous les travaux.
- Les sauvegardes sont stockées dans des domaines de sécurité et d’erreur distincts. Ainsi, même si le serveur source devait être compromis ou même rendu inutilisable, les sauvegardes restent en sécurité dans le [coffre de sauvegarde](backup-vault-overview.md).
- L’utilisation de **pg_dump** permet une plus grande flexibilité dans les restaurations, car vous pouvez restaurer des versions d’une base de données ou même restaurer seulement une partie de la sauvegarde.

Vous pouvez utiliser cette solution indépendamment ou en plus de la solution de sauvegarde native offerte par Azure PostgreSQL, qui permet une conservation jusqu’à 35 jours. La solution native est adaptée aux récupérations d’exploitation, par exemple quand vous voulez récupérer à partir des sauvegardes les plus récentes. La solution Sauvegarde Azure vous aide à répondre à vos besoins en matière de conformité, et offre une sauvegarde et une restauration plus flexibles et plus précises.

## <a name="support-matrix"></a>Matrice de prise en charge

|Support  |Détails  |
|---------|---------|
|Déploiements pris en charge   |  [Azure Database pour PostgreSQL - Serveur unique](https://docs.microsoft.com/azure/postgresql/overview#azure-database-for-postgresql---single-server)     |
|Régions Azure prises en charge |  USA Est, USA Est 2, USA Centre, USA Centre Sud, USA Ouest, USA Ouest 2, USA Centre Ouest, Brésil Sud, Canada Centre, Europe Nord, Europe Ouest, Royaume-Uni Sud, Royaume-Uni Ouest, Allemagne Centre-Ouest, Suisse Nord, Suisse Ouest, Asie Est, Asie Sud-Est, Japon Est, Japon Ouest, Corée Centre, Corée Sud, Inde Centre, Australie Est, Australie Centre, Australie Centre 2, Émirats arabes unis Nord  |
|Versions d’Azure PostgreSQL prises en charge    |   9.5, 9.6, 10, 11      |

## <a name="feature-considerations-and-limitations"></a>Considérations et limitations relatives aux fonctionnalités

- Toutes les opérations sont prises en charge seulement depuis le portail Azure.
- La limite recommandée pour la taille maximale de la base de données est de 400 Go.
- La sauvegarde inter-régions n’est pas prise en charge. Cela signifie que vous ne pouvez pas sauvegarder un serveur Azure PostgreSQL dans un coffre qui se trouve dans une autre région. De même, vous pouvez restaurer une sauvegarde seulement sur un serveur situé dans la même région que le coffre.
- Seules les données sont récupérées lors de la restauration. Les « rôles » ne sont pas restaurés.
- Avec la préversion, nous vous recommandons d’exécuter la solution uniquement sur votre environnement de test.

## <a name="backup-process"></a>Processus de sauvegarde

1. Cette solution utilise **pg_dump** pour effectuer des sauvegardes de vos bases de données Azure PostgreSQL.

2. Une fois que vous avez spécifié les bases de données Azure PostgreSQL à sauvegarder, le service vérifie s’il dispose de l’ensemble d’autorisations approprié et de l’accès nécessaire pour effectuer l’opération de sauvegarde sur le serveur et la base de données.

3. Sauvegarde Azure utilise un rôle Worker avec une extension de sauvegarde installée dans celui-ci. Cette extension communique avec le serveur Postgres.

4. Cette extension se compose d’un coordinateur et d’un plug-in Azure Postgres. Si le coordinateur est responsable du déclenchement des workflows pour différentes opérations, comme la configuration de la sauvegarde, la sauvegarde et la restauration, le plug-in est quant à lui responsable du flux de données réel.
  
5. Quand vous déclenchez la configuration de la protection sur les bases de données sélectionnées, le service de sauvegarde configure le coordinateur avec les planifications de sauvegarde et d’autres détails de la stratégie.

6. Au moment planifié, le coordinateur communique avec le plug-in et démarre le streaming des données de sauvegarde depuis le serveur Postgres en utilisant **pg_dump**.

7. Le plug-in envoie les données directement au coffre de sauvegarde, ce qui élimine la nécessité d’un emplacement intermédiaire. Les données sont chiffrées avec des clés gérées par Microsoft et stockées par le service Sauvegarde Azure dans des comptes de stockage.

8. Une fois le transfert de données terminé, le coordinateur confirme la validation auprès du service de sauvegarde.

    ![Processus de sauvegarde](./media/backup-azure-database-postgresql/backup-process.png)

## <a name="configure-backup-on-azure-postgresql-databases"></a>Configurer la sauvegarde sur des bases de données Azure PostgreSQL

Vous pouvez configurer une sauvegarde sur plusieurs bases de données sur plusieurs serveurs Azure PostgreSQL. Vérifiez que les [autorisations préalables](#prerequisite-permissions-for-configure-backup-and-restore) requises par le service pour sauvegarder les serveurs Postgres sont déjà configurées.

Les instructions suivantes constituent un guide pas à pas de la configuration de la sauvegarde sur les bases de données Azure PostgreSQL avec Sauvegarde Azure :

1. Il existe deux façons de commencer le processus :

    1. Accédez à [Centre de sauvegarde](backup-center-overview.md) -> **Vue d’ensemble** -> **Sauvegarder**.

        ![Accéder au Centre de sauvegarde](./media/backup-azure-database-postgresql/backup-center.png)

        Sous **Lancer : Configurer la sauvegarde**, sélectionnez le **Type de source de données** **Azure Database pour PostgreSQL**.

        ![Dans Lancer : Configurer la sauvegarde, sélectionner le type de source de données](./media/backup-azure-database-postgresql/initiate-configure-backup.png)

    1. Vous pouvez aussi accéder directement à [Coffres de sauvegarde](backup-vault-overview.md) -> **Sauvegarder**.

        ![Accéder à Coffres de sauvegarde](./media/backup-azure-database-postgresql/backup-vaults.png)

        ![Sélectionner Sauvegarder dans Coffre de sauvegarde](./media/backup-azure-database-postgresql/backup-backup-vault.png)

1. Sous **Configurer la sauvegarde**, sélectionnez le **Coffre de sauvegarde** où vous voulez sauvegarder vos bases de données Postgres. Ces informations sont préremplies si vous êtes déjà dans le contexte du coffre.

    ![Sélectionner Coffre de sauvegarde dans Configurer la sauvegarde](./media/backup-azure-database-postgresql/configure-backup.png)

1. Sélectionnez ou créez une **Stratégie de sauvegarde**.

    ![Choisir une stratégie de sauvegarde](./media/backup-azure-database-postgresql/backup-policy.png)

1. Sélectionnez des ressources ou des bases de données Postgres à sauvegarder.

    ![Sélectionner les ressources à sauvegarder](./media/backup-azure-database-postgresql/select-resources.png)

1. Vous pouvez choisir dans la liste de tous les serveurs Azure PostgreSQL dans les différents abonnements s’ils se trouvent dans la même région que le coffre. Développez la flèche pour voir la liste des bases de données au sein d’un serveur.

    ![Choisir les serveurs](./media/backup-azure-database-postgresql/choose-servers.png)

1. Le service effectue ces contrôles sur les bases de données sélectionnées pour vérifier si le coffre dispose des autorisations nécessaires pour sauvegarder les bases de données et les serveurs Postgres sélectionnés.
    1. **Préparation de la sauvegarde** pour toutes les bases de données doit indiquer **Réussite** pour pouvoir continuer.
    1. En cas d’erreur, **corrigez l’erreur**, puis **revalidez** ou supprimez la base de données dans les sélections.

    ![Erreurs de validation à corriger](./media/backup-azure-database-postgresql/validation-errors.png)

1. Vérifiez tous les détails sous **Examiner et configurer**, puis sélectionnez **Configurer la sauvegarde** pour soumettre l’opération.

    ![Vérifier les détails dans Examiner et configurer](./media/backup-azure-database-postgresql/review-and-configure.png)

1. Une fois déclenchée, l’opération **Configurer la sauvegarde** va créer une instance de sauvegarde. Vous pouvez suivre l’état de l’opération sous le volet [Instances de sauvegarde](backup-center-monitor-operate.md#backup-instances) dans le Centre de sauvegarde ou la vue du coffre.

    ![Instances de sauvegarde](./media/backup-azure-database-postgresql/backup-instances.png)

1. Les sauvegardes sont déclenchées en fonction de la planification de sauvegarde définie dans la stratégie. Les travaux peuvent faire l’objet d’un suivi sous [Tâches de sauvegarde](backup-center-monitor-operate.md#backup-jobs). Actuellement, vous pouvez afficher les travaux des sept derniers jours.

    ![Travaux de sauvegarde](./media/backup-azure-database-postgresql/backup-jobs.png)

## <a name="create-backup-policy"></a>Créer la stratégie de sauvegarde

1. Accédez à **Centre de sauvegarde** -> **Stratégies de sauvegarde** -> **Ajouter**. Vous pouvez aussi accéder directement à **Coffre de sauvegarde** -> **Stratégie de sauvegarde** -> **Ajouter**.

    ![Ajouter une stratégie de sauvegarde](./media/backup-azure-database-postgresql/add-backup-policy.png)

1. Entrez un **nom** pour la nouvelle stratégie.

    ![Saisir un nom de stratégie](./media/backup-azure-database-postgresql/enter-policy-name.png)

1. Définissez la planification de sauvegarde. Nous prenons actuellement en charge **Hebdomadaire**. Vous pouvez planifier les sauvegardes à un ou plusieurs jours de la semaine.

    ![Définir la planification de sauvegarde](./media/backup-azure-database-postgresql/define-backup-schedule.png)

1. Définissez les paramètres de **Conservation**. Vous pouvez ajouter une ou plusieurs règles de conservation. Chaque règle de conservation nécessite des entrées pour des sauvegardes spécifiques, et le magasin de données et la durée de conservation de ces sauvegardes.

1. Vous pouvez choisir de stocker vos sauvegardes dans un des deux magasins de données (ou niveaux) : **Magasin de données des sauvegardes** (niveau chaud) ou **Magasin de données des archives** (en préversion). Vous pouvez choisir entre **deux options de hiérarchisation** pour définir quand les sauvegardes sont hiérarchisées dans les deux magasins de données :

    - Choisissez de copier **Immédiatement** si vous préférez avoir une copie de sauvegarde simultanément dans le magasin de données des sauvegardes et dans le magasin de données des archivages.
    - Choisissez de déplacer **À l’expiration** si vous préférez déplacer la sauvegarde vers le magasin de données des archivages quand elle expire dans le magasin de données des sauvegardes.

1. La **règle de conservation par défaut** est appliquée en l’absence d’une autre règle de conservation, et sa valeur par défaut est de trois mois.

    - Les durées de conservation vont de 7 jours à 10 ans dans le **Magasin de données des sauvegardes**.
    - Les durées de conservation vont de 6 mois à 10 ans dans le **Magasin de données des archives**.

    ![Modifier la durée de conservation](./media/backup-azure-database-postgresql/edit-retention.png)

>[!NOTE]
>Les règles de conservation sont évaluées selon un ordre de priorité prédéterminé. La priorité est la plus élevée pour la règle **annuelle**, suivie de la règle **mensuelle**, puis de la règle **hebdomadaire**. Les paramètres de conservation par défaut sont appliqués quand aucune autre règle n’est éligible. Par exemple, le même point de récupération peut être la première sauvegarde réussie effectuée chaque semaine ainsi que la première sauvegarde réussie effectuée chaque mois. Cependant, comme la priorité de la règle mensuelle est supérieure à celle de la règle hebdomadaire, la conservation correspondant à la première sauvegarde réussie effectuée chaque mois s’applique.

## <a name="restore"></a>Restaurer

Vous pouvez restaurer une base de données sur n’importe quel serveur Azure PostgreSQL au sein du même abonnement, si le service a l’ensemble d’autorisations approprié sur le serveur cible. Vérifiez que les [autorisations préalables](#prerequisite-permissions-for-configure-backup-and-restore) requises par le service pour sauvegarder les serveurs Postgres sont déjà configurées.

Suivez ce guide pas à pas pour déclencher une restauration :

1. Il existe deux façons de démarrer le processus de restauration :
    1. Accédez à [Centre de sauvegarde](backup-center-overview.md) -> **Vue d’ensemble** -> **Restaurer**.

    ![Sélectionner Restaurer dans le Centre de sauvegarde](./media/backup-azure-database-postgresql/backup-center-restore.png)

    Sous **Lancer : Restaurer**, sélectionnez le **Type de source de données** **Azure Database pour PostgreSQL**. Sélectionnez l’**Instance de sauvegarde**.

    ![Sélectionner Type de source de données dans Lancer : Restaurer](./media/backup-azure-database-postgresql/initiate-restore.png)

    1. Vous pouvez aussi accéder directement à **Coffre de sauvegarde** -> **Instances de sauvegarde**. Sélectionnez l’**Instance de sauvegarde** correspondant à la base de données que vous voulez restaurer.

    ![Instances de sauvegarde pour la restauration](./media/backup-azure-database-postgresql/backup-instances-restore.png)

    ![Liste des instances de sauvegarde](./media/backup-azure-database-postgresql/list-backup-instances.png)

    ![Sélectionnez Restaurer](./media/backup-azure-database-postgresql/select-restore.png)

1. **Sélectionnez le point de récupération** dans la liste de toutes les sauvegardes complètes disponibles pour l’instance de sauvegarde sélectionnée. Le dernier point de récupération est sélectionné par défaut.

    ![Sélectionner un point de récupération](./media/backup-azure-database-postgresql/select-recovery-point.png)

    ![Liste de points de récupération](./media/backup-azure-database-postgresql/list-recovery-points.png)

1. Entrez les **Paramètres de restauration**. À ce stade, vous pouvez choisir parmi deux types de restaurations : **Restaurer en tant que base de données** et **Restaurer en tant que fichiers**.

1. **Restaurer en tant que base de données** :  Restaurez les données de sauvegarde pour créer une nouvelle base de données sur le serveur PostgreSQL cible.

    - Le serveur cible peut être le même que le serveur source. Cependant, le remplacement de la base de données d’origine n’est pas pris en charge.
    - Vous pouvez choisir le serveur parmi tous les abonnements, mais dans la même région que le coffre.
    - Sélectionnez **Vérifier + restaurer**. Ceci va déclencher une validation pour vérifier si le service dispose des autorisations de restauration appropriées sur le serveur cible.

    ![Restaurer en tant que base de données](./media/backup-azure-database-postgresql/restore-as-database.png)

1. **Restaurer en tant que fichiers** : Copiez les fichiers de la sauvegarde vers le compte de stockage cible (objets blob).

    - Vous pouvez choisir un compte de stockage parmi tous les abonnements, mais dans la même région que le coffre.
    - Sélectionnez le conteneur cible dans la liste des conteneurs filtrée pour le compte de stockage sélectionné.
    - Sélectionnez **Vérifier + restaurer**. Ceci va déclencher une validation pour vérifier si le service dispose des autorisations de restauration appropriées sur le serveur cible.

    ![Restaurer sous forme de fichiers](./media/backup-azure-database-postgresql/restore-as-files.png)

1. Vérifiez les informations, puis sélectionnez **Restaurer**. Ceci va déclencher un travail de restauration correspondant qui peut être suivi sous **Travaux de sauvegarde**.

## <a name="prerequisite-permissions-for-configure-backup-and-restore"></a>Autorisations prérequises pour configurer la sauvegarde et la restauration

Sauvegarde Azure suit des recommandations strictes en matière de sécurité. Bien qu’il s’agisse d’un service Azure natif, les autorisations sur la ressource ne sont pas accordées par défaut : elles doivent être explicitement données par l’utilisateur.  De même, les informations d’identification pour la connexion à la base de données ne sont pas stockées. Ceci est important pour protéger vos données. Au lieu de cela, nous utilisons l’authentification Azure Active Directory.

[Téléchargez ce document](https://download.microsoft.com/download/7/4/d/74d689aa-909d-4d3e-9b18-f8e465a7ebf5/OSSbkpprep_automated.docx) pour obtenir un script automatisé et les instructions associées. Il va accorder un ensemble d’autorisations approprié à un serveur Azure PostgreSQL pour la sauvegarde et la restauration.

## <a name="manage-the-backed-up-azure-postgresql-databases"></a>Gérer les bases de données Azure PostgreSQL sauvegardées

Voici les opérations de gestion que vous pouvez effectuer sur les **instances de sauvegarde** :

### <a name="on-demand-backup"></a>Sauvegarde à la demande

Pour déclencher une sauvegarde qui n’est pas dans la planification spécifiée dans la stratégie, accédez à **Instances de sauvegarde** -> **Sauvegarder maintenant**.
Choisissez dans la liste des règles de conservation qui ont été définies dans la stratégie de sauvegarde associée.

![Déclencher une sauvegarde maintenant](./media/backup-azure-database-postgresql/backup-now.png)

![Choisir dans la liste des règles de conservation](./media/backup-azure-database-postgresql/retention-rules.png)

### <a name="stop-protection"></a>Arrêter la protection

Vous pouvez arrêter la protection sur un élément de sauvegarde. Ceci va également supprimer les points de récupération associés à cet élément de sauvegarde. Nous ne fournissons pas encore la possibilité d’arrêter la protection tout en conservant les points de récupération existants.

![Arrêter la protection](./media/backup-azure-database-postgresql/stop-protection.png)

### <a name="change-policy"></a>Changer la stratégie

Vous pouvez changer la stratégie associée à une instance de sauvegarde.

1. Sélectionnez l’**instance de sauvegarde** -> **Changer la stratégie**.

    ![Changer la stratégie](./media/backup-azure-database-postgresql/change-policy.png)

1. Sélectionnez la nouvelle stratégie que vous souhaitez appliquer à la base de données.

    ![Réaffecter une stratégie](./media/backup-azure-database-postgresql/reassign-policy.png)

## <a name="troubleshooting"></a>Dépannage

Cette section fournit des informations de dépannage pour la sauvegarde des bases de données Azure PostgreSQL avec Sauvegarde Azure.

### <a name="usererrormsimissingpermissions"></a>UserErrorMSIMissingPermissions

Donnez à l’identité MSI du coffre de sauvegarde un accès en **Lecture** sur le serveur Postgres que vous voulez sauvegarder ou restaurer :

Pour établir une connexion sécurisée à la base de données PostgreSQL, Sauvegarde Azure utilise le modèle d’authentification [ Managed Service Identity (MSI)](../active-directory/managed-identities-azure-resources/overview.md). Cela signifie que le coffre de sauvegarde aura accès seulement aux ressources auxquelles l’utilisateur a explicitement accordé une autorisation.

Une identité MSI système est automatiquement affectée au coffre au moment de la création. Vous devez donner à cette identité MSI du coffre l’accès aux serveurs PostgreSQL dont vous prévoyez de sauvegarder des bases de données.

Étapes :

1. Sur le serveur Postgres, accédez au volet **Contrôle d’accès (IAM)** .

    ![Volet Contrôle d’accès](./media/backup-azure-database-postgresql/access-control-pane.png)

1. Sélectionnez **Ajouter une attribution de rôle**.

    ![Ajouter une attribution de rôle](./media/backup-azure-database-postgresql/add-role-assignment.png)

1. Dans le volet contextuel droit qui s’ouvre, entrez les informations suivantes :<br>

    **Rôle :** Lecteur<br>
    **Attribuer l’accès à :** Choisissez **Coffre de sauvegarde**<br>
    Si vous ne trouvez pas l’option **Coffre de sauvegarde** dans la liste déroulante, choisissez l’option **Utilisateur, groupe ou principal du service Azure AD**<br>

    ![Sélectionner un rôle](./media/backup-azure-database-postgresql/select-role.png)

    **Sélectionnez :** Entrez le nom du coffre de sauvegarde où vous voulez sauvegarder ce serveur et ses bases de données.<br>

    ![Entrer le nom du coffre de sauvegarde](./media/backup-azure-database-postgresql/enter-backup-vault-name.png)

### <a name="usererrorbackupuserauthfailed"></a>UserErrorBackupUserAuthFailed

Créez un utilisateur de sauvegarde de base de données qui peut s’authentifier auprès d’Azure Active Directory :

Cette erreur peut provenir de l’absence d’un administrateur Azure Active Directory pour le serveur PostgreSQL ou de l’absence d’un utilisateur de sauvegarde pouvant s’authentifier avec Azure Active Directory.

Étapes :

Ajoutez un administrateur Active Directory au serveur OSS :

Cette étape est nécessaire pour se connecter à la base de données via un utilisateur qui peut s’authentifier avec Azure Active Directory au lieu d’un mot de passe. L’utilisateur Administrateur Azure AD dans Azure Database pour PostgreSQL aura le rôle **azure_ad_admin**. Seul un rôle **azure_ad_admin** peut créer des utilisateurs de base de données qui peuvent s’authentifier avec Azure AD.

1. Accédez à l’onglet Administrateur Active Directory dans le volet de navigation gauche de la vue du serveur, puis ajoutez-vous (ou une autre personne) en tant qu’administrateur Active Directory.

    ![Définir un administrateur Active Directory](./media/backup-azure-database-postgresql/set-admin.png)

1. Veillez à **Enregistrer** la définition de l’utilisateur Administrateur Active Directory.

    ![Enregistrer la définition de l’utilisateur Administrateur Active Directory](./media/backup-azure-database-postgresql/save-admin-setting.png)

Reportez-vous à [ce document](https://download.microsoft.com/download/7/4/d/74d689aa-909d-4d3e-9b18-f8e465a7ebf5/OSSbkpprep_automated.docx) pour obtenir la liste des étapes à suivre pour effectuer les étapes d’octroi des autorisations.

### <a name="usererrormissingnetworksecuritypermissions"></a>UserErrorMissingNetworkSecurityPermissions

Établissez une visibilité du réseau en activant l’indicateur **Autoriser l’accès aux services Azure** dans la vue du serveur. Dans la vue du serveur, sous le volet **Sécurité de la connexion**, définissez l’indicateur **Autoriser l’accès aux services Azure** sur **Oui**.

![Autoriser l’accès aux services Azure](./media/backup-azure-database-postgresql/allow-access-to-azure-services.png)

### <a name="usererrorcontainernotaccessible"></a>UserErrorContainerNotAccessible

#### <a name="permission-to-restore-to-a-storage-account-container-when-restoring-as-files"></a>Autorisation de restaurer sur un conteneur d’un compte de stockage lors de la restauration en tant que fichiers

1. Donnez à l’identité MSI du coffre de sauvegarde l’autorisation d’accéder aux conteneurs du compte de stockage en utilisant le portail Azure.
    1. Accédez à **Compte de stockage** -> **Contrôle d’accès** -> **Ajouter une attribution de rôle**.
    1. Affectez le rôle **Contributeur aux données Blob du stockage** à l’identité MSI du coffre de sauvegarde.

    ![Affecter le rôle Contributeur aux données Blob du stockage](./media/backup-azure-database-postgresql/assign-storage-blog-data-contributor-role.png)

1. Vous pouvez aussi accorder des autorisations précises sur le conteneur spécifique vers lequel vous effectuez la restauration en utilisant la commande Azure CLI [az role assignment create](/cli/azure/role/assignment).

    ```azurecli
    az role assignment create --assignee $VaultMSI_AppId  --role "Storage Blob Data Contributor"   --scope $id
    ```

    1. Remplacez le paramètre du destinataire de l’affectation par l’**ID d’application** de l’identité MSI du coffre et le paramètre d’étendue de façon à référencer votre conteneur spécifique.
    1. Pour obtenir l’**ID d’application** de l’identité MSI du coffre, sélectionnez **Toutes les applications** sous **Type d’application** :

        ![Sélectionner Toutes les applications](./media/backup-azure-database-postgresql/select-all-applications.png)

    1. Recherchez le nom du coffre et copiez l’ID d’application :

        ![Rechercher le nom du coffre](./media/backup-azure-database-postgresql/search-for-vault-name.png)

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des coffres de sauvegarde](backup-vault-overview.md)

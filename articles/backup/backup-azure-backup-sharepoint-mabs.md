---
title: Sauvegarder une batterie de serveurs SharePoint dans Azure avec MABS
description: Le serveur de sauvegarde Azure vous permet de sauvegarder et de restaurer vos données SharePoint. Cet article fournit des informations vous permettant de configurer votre batterie de serveurs SharePoint, afin de pouvoir stocker les données souhaitées dans Azure. Vous pouvez restaurer des données SharePoint protégées à partir d’un disque ou d’Azure.
ms.topic: conceptual
ms.date: 04/26/2020
ms.openlocfilehash: 837aabf739431eebaa6406770620329fe6345eb7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89375395"
---
# <a name="back-up-a-sharepoint-farm-to-azure-with-mabs"></a>Sauvegarder une batterie de serveurs SharePoint dans Azure avec MABS

Les batteries de serveurs SharePoint peuvent être sauvegardées sur Microsoft Azure à l’aide du serveur de sauvegarde Azure et ce, de la même façon que d’autres sources de données. Azure Backup offre une flexibilité dans la planification d’une sauvegarde pour créer des points de sauvegarde quotidiens, hebdomadaires, mensuels ou annuels, et vous offre des options de stratégie de conservation pour les différents points de sauvegarde. MABS vous permet de stocker des copies sur disque local pour bénéficier d’objectifs de temps de récupération (RTO) rapides, ainsi que de stocker des copies sur Azure pour une conservation à long terme économique.

La sauvegarde de SharePoint dans Azure avec MABS est un processus similaire à la sauvegarde locale de SharePoint dans DPM (Data Protection Manager). Cet article comprend des points particuliers à prendre en compte pour Azure.

## <a name="sharepoint-supported-versions-and-related-protection-scenarios"></a>Versions SharePoint prises en charge et scénarios de protection associés

Pour obtenir la liste des versions de SharePoint prises en charge et des versions de MABS nécessaires pour les sauvegarder, consultez [la matrice de protection MABS](./backup-mabs-protection-matrix.md).

## <a name="before-you-start"></a>Avant de commencer

Quelques points doivent être confirmés avant de sauvegarder une batterie de serveurs SharePoint sur Azure.

### <a name="whats-not-supported"></a>Ce qui n'est pas pris en charge

* MABS qui protège une batterie de serveurs SharePoint ne protège pas les index de recherche ni les bases de données de service d’application. Vous devrez configurer la protection de ces bases de données séparément.

* MABS ne fournit aucune sauvegarde des bases de données SQL Server SharePoint hébergées sur des partages de type SOFS (Scale-out File Server).

### <a name="prerequisites"></a>Prérequis

Avant de continuer, assurez-vous que vous respectez toutes les [conditions préalables à l’utilisation de Sauvegarde Microsoft Azure](backup-azure-dpm-introduction.md#prerequisites-and-limitations) pour protéger les charges de travail. Certaines tâches liées aux conditions préalables incluent : créer un coffre de sauvegarde, télécharger les informations d’identification du coffre, installer l’agent Sauvegarde Azure et inscrire le serveur de sauvegarde Azure auprès du coffre.

Conditions préalables et limitations supplémentaires :

* Par défaut, lorsque vous protégez SharePoint, toutes les bases de données de contenu (et les bases de données SharePoint_Config et SharePoint_AdminContent*) sont protégées. Si vous souhaitez ajouter des personnalisations, telles que des index de recherche, des modèles ou des bases de données de service d’application, ou le service de profil utilisateur, vous devrez les configurer séparément pour la protection. Veillez à activer la protection pour tous les dossiers qui incluent ces types de fonctionnalités ou fichiers de personnalisation.

* Vous ne pouvez pas protéger les bases de données SharePoint en tant que source de données SQL Server. Vous pouvez récupérer des bases de données individuelles à partir d’une sauvegarde de batterie de serveurs.

* N’oubliez pas que MABS s’exécute en tant que **système local** et, pour sauvegarder les bases de données SQL Server, il nécessite les privilèges d’administrateur système sur ce compte pour le serveur SQL. Sur le serveur SQL que vous souhaitez sauvegarder, définissez NT AUTHORITY\SYSTEM sur **sysadmin**.

* Pour chaque tranche de 10 millions d’éléments dans la batterie de serveurs, l’espace disponible doit être de 2 Go minimum sur le volume hébergeant le dossier du serveur de sauvegarde Azure. Cet espace est nécessaire pour la génération du catalogue. Pour vous permettre d’utiliser MABS afin de restaurer des éléments spécifiques (collections de sites, sites, listes, bibliothèques de documents, dossiers, documents et éléments de liste), la génération du catalogue crée une liste de toutes les URL contenues dans chaque base de données de contenu. Vous pouvez afficher la liste des URL dans le volet des éléments récupérables de la zone de tâches Récupération de la console administrateur de MABS.

* Dans la batterie de serveurs SharePoint, si des bases de données SQL Server ont été configurées à l’aide d’alias SQL Server, installez les composants clients SQL Server sur le serveur web frontal devant être protégé par MABS.

* La protection d’éléments du magasin d’applications n’est pas prise en charge avec SharePoint 2013.

* MABS ne prend pas en charge la protection à distance de FILESTREAM. L’objet FILESTREAM doit faire partie de la base de données.

## <a name="configure-backup"></a>Configurer une sauvegarde

Pour sauvegarder la batterie de serveurs SharePoint, configurez la protection pour SharePoint à l’aide de ConfigureSharePoint.exe, puis créez un groupe de protection dans MABS.

1. **Exécutez ConfigureSharePoint.exe** : cet outil configure le service Enregistreur VSS SharePoint \(WSS\) et fournit à l’agent de protection les informations d’identification de la batterie de serveurs SharePoint. Une fois que vous avez déployé l’agent de protection, le fichier ConfigureSharePoint.exe se trouve dans le dossier `<MABS Installation Path\>\bin` sur le serveur web frontal.  Si vous avez plusieurs serveurs WFE, il vous suffit de l’installer sur l’un d’eux. Exécutez comme suit :

    * Sur le serveur WFE, sur une invite de commandes, accédez à `\<MABS installation location\>\\bin\\` et exécutez `ConfigureSharePoint \[\-EnableSharePointProtection\] \[\-EnableSPSearchProtection\] \[\-ResolveAllSQLAliases\] \[\-SetTempPath <path>\]`, où :

        * **EnableSharePointProtection** active la protection de la batterie de serveurs SharePoint, active l’enregistreur VSS et inscrit l’identité de l’application DCOM WssCmdletsWrapper pour qu’elle s’exécute en tant qu’utilisateur dont les informations d’identification sont entrées avec cette option. Ce compte doit être un administrateur de batterie de serveurs ainsi qu’un administrateur local sur le serveur web frontal.

        * **EnableSPSearchProtection** permet la protection de WSS 3.0 SP Search à l’aide de la clé de Registre SharePointSearchEnumerationEnabled située sous HKLM\\Software\\Microsoft\\Microsoft Data Protection Manager\\Agent\\2.0\\ sur le serveur web frontal et inscrit l’identité de l’application DCOM WssCmdletsWrapper pour qu’elle s’exécute en tant qu’utilisateur dont les informations d’identification sont entrées avec cette option. Ce compte doit être un administrateur de batterie de serveurs ainsi qu’un administrateur local sur le serveur web frontal.

        * **ResolveAllSQLAliases** affiche tous les alias signalés par l’enregistreur VSS SharePoint et les résout dans le serveur SQL correspondant. Cette option affiche également les noms d’instance résolus et les serveurs mis en miroir, le cas échéant. En outre, elle signale tous les alias qui n’ont pas été résolus à un serveur SQL.

        * **SetTempPath** définit les variables d’environnement TEMP et TMP pour le chemin d’accès spécifié. La récupération au niveau de l’élément échouera si l’objet de la récupération est une collection de sites, un site, une liste ou un élément de taille importante, alors que l’espace disponible dans le dossier temporaire de l’administrateur de la batterie de serveur est insuffisant. Cette option vous permet de déplacer le dossier des fichiers temporaires vers un volume disposant de suffisamment d’espace pour stocker la collection de sites ou le site en cours de récupération.

    * Entrez les informations d'identification de l’administrateur de la batterie de serveurs. Ce compte doit être membre du groupe administrateur local sur le serveur Web frontal (WFE). Si l’administrateur de la batterie de serveurs n’est pas un administrateur local, accordez les autorisations suivantes sur le serveur web frontal (WFE) :

        * Accordez au groupe **WSS_Admin_WPG** le contrôle total du dossier MABS (`%Program Files%\Data Protection Manager\DPM\`).

        * Accordez au groupe **WSS_Admin_WPG** un accès en lecture à la clé de Registre MABS (`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Data Protection Manager`).

        Après l’exécution de ConfigureSharePoint.exe, vous devrez le réexécuter à chaque modification des informations d’identification de l’administrateur de la batterie de serveurs SharePoint.

1. Pour créer un groupe de protection, sélectionnez **Protection** > **Actions** > **Créer un groupe de protection** pour ouvrir l’Assistant **Création d’un nouveau groupe de protection** dans la console MABS.

1. Dans **Sélectionner le type de groupe de protection**, sélectionnez **Serveurs**.

1. Dans **Sélectionnez les membres du groupe**, développez le serveur qui détient le rôle WFE. S’il y a plusieurs serveurs WFE, sélectionnez celui sur lequel vous avez installé ConfigureSharePoint.exe.

    Lorsque vous développez le serveur SharePoint, MABS interroge VSS pour savoir quelles données MABS peut protéger.  Si la base de données SharePoint est distante, MABS s’y connecte. Si les sources de données SharePoint ne s’affichent pas, vérifiez que l’enregistreur VSS est en cours d’exécution sur le serveur SharePoint et un serveur SQL distant, et assurez-vous que l’agent MABS est bien installé à la fois sur le serveur SharePoint et le serveur SQL distant. En outre, assurez-vous que les bases de données SharePoint ne sont pas protégées ailleurs en tant que bases de données SQL Server.

1. Dans **Sélectionner la méthode de protection des données**, spécifiez la façon dont vous souhaitez gérer les sauvegardes à court et à long terme. La sauvegarde à court terme se fait toujours sur le disque en premier, avec la possibilité de sauvegarder du disque vers le cloud Azure à l’aide de Sauvegarde Azure \(à court ou long terme\).

1. Dans **Sélectionner les objectifs à court terme**, spécifiez la manière dont vous voulez sauvegarder dans le stockage à court terme sur disque.   Dans **Durée de rétention**, spécifiez la durée pendant laquelle vous souhaitez conserver les données sur disque. Dans **Fréquence de synchronisation**, spécifiez la fréquence à laquelle vous souhaitez exécuter une sauvegarde incrémentielle sur disque. Si vous ne souhaitez pas définir d’intervalle de sauvegarde, vous pouvez activer l’option « Juste avant un point de récupération » afin que MABS exécute une sauvegarde complète rapide juste avant chaque point de récupération planifié.

1. Dans la page Vérifier l’allocation de disque, vérifiez l’espace disque de pool de stockage alloué pour le groupe de protection.

    **Taille totale des données** correspond à la taille des données que vous souhaitez sauvegarder et **Disk space to be provisioned on MABS** (Espace disque à provisionner sur MABS) correspond à l’espace recommandé par MABS pour le groupe de protection. MABS choisit le volume de sauvegarde idéal en fonction des paramètres. Toutefois, vous pouvez modifier les choix de volume de sauvegarde dans **Détails de l’allocation de disque**. Pour les charges de travail, sélectionnez le stockage préféré dans le menu déroulant. Vos modifications changent les valeurs de **Stockage Total** et **Stockage libre** dans le volet **Stockage sur disque disponible**. L’espace sous-provisionné est la quantité de stockage que MABS vous suggère d’ajouter au volume pour continuer à procéder à des sauvegardes de façon fluide à l’avenir.

1. Sous **Choisir la méthode de création de réplica**, sélectionnez la façon dont vous souhaitez gérer la réplication initiale complète des données.  Si vous décidez de répliquer sur le réseau, nous vous recommandons de choisir une heure creuse. Pour de grandes quantités de données ou des conditions de réseau peu optimales, envisagez de répliquer les données hors connexion à l’aide d’un support amovible.

1. Sous **Sélectionner les options de vérification de cohérence**, indiquez comment vous voulez automatiser les vérifications de cohérence. Vous pouvez activer une vérification pour qu’elle s’exécute uniquement lorsque les données de réplication deviennent incohérentes ou selon une planification. Si vous ne souhaitez pas configurer la vérification de cohérence automatique, vous pouvez exécuter une vérification manuelle à tout moment en cliquant avec le bouton droit sur le groupe de protection dans la zone **Protection** de la console MABS et en sélectionnant **Effectuer une vérification de cohérence**.

1. Si vous avez choisi de sauvegarder dans le cloud à l’aide de Sauvegarde Azure, sur la page **Indiquer les données de protection en ligne**, veillez à sélectionner les charges de travail que vous souhaitez sauvegarder dans Azure.

1. Sous **Spécifier la planification de sauvegarde en ligne**, indiquez la fréquence à laquelle doivent avoir lieu les sauvegardes incrémentielles dans Azure. Vous pouvez planifier des sauvegardes quotidiennes, hebdomadaires, mensuelles et annuelles, ainsi que sélectionner la date et l’heure de leur exécution. Les sauvegardes peuvent avoir lieu jusqu’à deux fois par jour. Chaque fois qu’une sauvegarde s’exécute, un point de récupération de données est créé dans Azure à partir de la copie des données sauvegardées stockées sur le disque MABS.

1. Sous **Spécifier une stratégie de rétention en ligne**, vous pouvez préciser la façon dont les points de récupération créés à partir des sauvegardes quotidiennes, hebdomadaires, mensuelles et annuelles sont conservés dans Azure.

1. Sous **Choisir la réplication en ligne**, sélectionnez la façon dont la réplication initiale complète des données aura lieu. Vous pouvez répliquer sur le réseau ou effectuer une sauvegarde en mode hors connexion (essaimage hors connexion). La sauvegarde en mode hors connexion utilise la fonctionnalité Azure Import. [En savoir plus](./backup-azure-backup-import-export.md).

1. Dans la page **Résumé**, vérifiez vos paramètres. Après que vous avez sélectionné **Créer un groupe**, la réplication initiale des données est effectuée. Une fois l’opération terminée, l’état du groupe de protection s’affiche comme **OK** dans la page **État**. La sauvegarde a alors lieu conformément aux paramètres du groupe de protection.

## <a name="monitoring"></a>Surveillance

Une fois le groupe de protection créé, la réplication initiale se produit et MABS commence à sauvegarder et synchroniser les données SharePoint. MABS analyse la synchronisation initiale et les sauvegardes ultérieures.  Vous pouvez analyser les données SharePoint de deux manières :

* L’analyse MABS par défaut vous permet de configurer des notifications pour l’analyse proactive et la publication d’alertes. Vous pouvez envoyer des notifications par e-mail pour les alertes critiques, d’avertissement ou d’information, ainsi que pour l’état des récupérations instanciées.

* Si vous utilisez Operations Manager, vous pouvez publier les alertes de façon centralisée.

### <a name="set-up-monitoring-notifications"></a>Configurer des notifications d’analyse

1. Dans la Console Administrateur MABS, sélectionnez **Analyse** > **Action** > **Options**.

2. Sélectionnez **Serveur SMTP**, saisissez le nom du serveur, le port et l’adresse e-mail à partir desquels les notifications seront envoyées. L’adresse doit être valide.

3. Dans **Serveur SMTP authentifié**, saisissez un nom d’utilisateur et un mot de passe. Le nom d’utilisateur et le mot de passe doivent correspondre au nom du compte de domaine de la personne dont l’adresse « De » est décrite à l’étape précédente. Sinon, la remise de notification échoue.

4. Pour tester les paramètres du serveur SMTP, sélectionnez **Envoyer un message de test**, saisissez l’adresse e-mail à laquelle MABS doit envoyer le message de test, puis sélectionnez **OK**. Sélectionnez **Options** > **Notifications**, puis sélectionnez les types d’alertes dont les destinataires souhaitent être avertis. Dans **Destinataires**, saisissez l’adresse e-mail de chaque destinataire auquel MABS doit envoyer des copies des notifications.

### <a name="publish-operations-manager-alerts"></a>Publier les alertes Operations Manager

1. Dans la Console Administrateur MABS, sélectionnez **Analyse** > **Action** > **Options** > **Publication d’alertes** > **Publier des alertes actives**.

2. Un fois l’option **Publication d’alertes** activée, toutes les alertes MABS existantes pouvant nécessiter une action de l’utilisateur sont publiées dans le journal des événements **Alertes MABS**. L’agent Operations Manager installé sur le serveur MABS publie ensuite ces alertes dans Operations Manager, puis continue à mettre à jour la console à mesure que de nouvelles alertes sont générées.

## <a name="restore-a-sharepoint-item-from-disk-by-using-mabs"></a>Restauration d’un élément SharePoint à partir du disque à l’aide du serveur de sauvegarde Azure

Dans l’exemple ci-dessous, *l’élément de récupération SharePoint* a été supprimé accidentellement et doit être récupéré.
![MABS SharePoint Protection4](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection5.png)

1. Ouvrez la **console administrateur MABS**. Toutes les batteries de serveurs SharePoint protégées par MABS apparaissent dans l’onglet **Protection**.

    ![MABS SharePoint Protection3](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection4.png)
2. Pour commencer la récupération de l’élément, sélectionnez l’onglet **Récupération** .

    ![MABS SharePoint Protection5](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection6.png)
3. Vous pouvez rechercher *l’élément de récupération SharePoint* dans SharePoint à l’aide d’une recherche avec des caractères génériques dans une plage de points de récupération.

    ![MABS SharePoint Protection6](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection7.png)
4. Sélectionnez le point de récupération approprié dans les résultats de la recherche, cliquez avec le bouton droit sur l’élément, puis sélectionnez **Restaurer**.
5. Vous pouvez également parcourir les différents points de récupération et sélectionner une base de données ou un élément à restaurer. Sélectionnez **Date > Temps de récupération**, puis sélectionnez **Base de données > Batterie de serveurs SharePoint > Point de récupération > Élément**.

    ![MABS SharePoint Protection7](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection8.png)
6. Cliquez avec le bouton droit sur l’élément, puis sélectionnez **Restaurer** pour ouvrir **l’Assistant Récupération**. Sélectionnez **Suivant**.

    ![Vérifier la sélection à restaurer](./media/backup-azure-backup-sharepoint/review-recovery-selection.png)
7. Sélectionnez le type de récupération que vous souhaitez effectuer, puis sélectionnez **Suivant**.

    ![Type de récupération](./media/backup-azure-backup-sharepoint/select-recovery-type.png)

   > [!NOTE]
   > La sélection de l’option **Récupérer sur le site d’origine** dans l’exemple récupère l’élément sur le site SharePoint d’origine.
   >
   >
8. Sélectionnez le **processus de récupération** que vous souhaitez utiliser.

   * Sélectionnez **Récupérer sans utiliser de batterie de serveurs de récupération** si la batterie de serveurs SharePoint n’a pas changé et qu’elle est la même que le point de récupération en cours de restauration.
   * Sélectionnez l’option de **récupération avec une batterie de serveurs** si la batterie de serveurs SharePoint a changé depuis la création du point de restauration.

     ![processus de récupération](./media/backup-azure-backup-sharepoint/recovery-process.png)
9. Spécifiez l’emplacement intermédiaire de l’instance SQL Server pour la restauration temporaire de la base de données, ainsi qu’un partage de fichiers intermédiaire sur le serveur de sauvegarde Azure et le serveur qui exécute SharePoint afin de restaurer l’élément.

    ![Emplacement intermédiaire1](./media/backup-azure-backup-sharepoint/staging-location1.png)

    Le serveur de sauvegarde Azure connecte la base de données de contenu, qui héberge l’élément SharePoint, à l’instance SQL temporaire. À partir de la base de données de contenu, l’élément est restauré et stocké à l’emplacement du fichier intermédiaire sur le serveur de sauvegarde Azure. L’élément récupéré situé à l’emplacement intermédiaire doit maintenant être exporté vers l’emplacement intermédiaire sur la batterie de serveurs SharePoint.

    ![Emplacement intermédiaire2](./media/backup-azure-backup-sharepoint/staging-location2.png)
10. Sélectionnez **Spécifier les options de récupération**et appliquez des paramètres de sécurité à la batterie de serveurs SharePoint, ou appliquez les paramètres de sécurité du point de récupération. Sélectionnez **Suivant**.

    ![Options de récupération](./media/backup-azure-backup-sharepoint/recovery-options.png)

    > [!NOTE]
    > Vous pouvez choisir de limiter l'utilisation de la bande passante réseau. Cela réduit l'impact sur le serveur de production pendant les heures de production.
    >
    >
11. Passez en revue les informations de résumé et sélectionnez **Récupérer** pour commencer la récupération du fichier.

    ![Résumé de la récupération](./media/backup-azure-backup-sharepoint/recovery-summary.png)
12. Sélectionnez maintenant l’onglet **Surveillance** dans la **console administrateur du serveur de sauvegarde Azure** pour afficher **l’état** de la récupération.

    ![État de la récupération](./media/backup-azure-backup-sharepoint/recovery-monitoring.png)

    > [!NOTE]
    > Le fichier est maintenant restauré. Vous pouvez actualiser le site SharePoint pour vérifier le fichier restauré.
    >
    >

## <a name="restore-a-sharepoint-database-from-azure-by-using-mabs"></a>Restaurer une base de données SharePoint à partir d’Azure à l’aide de MABS

1. Pour récupérer une base de données de contenu SharePoint, parcourez les différents points de récupération (comme indiqué ci-dessus), puis sélectionnez le point de récupération à restaurer.

    ![MABS SharePoint Protection8](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection9.png)
2. Double-cliquez sur le point de récupération SharePoint pour afficher les informations du catalogue SharePoint disponibles.

   > [!NOTE]
   > Comme la batterie de serveurs SharePoint est protégée pour une conservation à long terme dans Azure, aucune information du catalogue (métadonnées) n’est disponible sur le serveur MABS. Par conséquent, chaque fois qu’une base de données contenu SharePoint ponctuelle doit être récupérée, vous devez de nouveau cataloguer la batterie de serveurs SharePoint.
   >
   >
3. Sélectionnez **Recataloguer**.

    ![MABS SharePoint Protection10](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection12.png)

    La fenêtre d’état **Recataloguer le cloud** s’ouvre.

    ![MABS SharePoint Protection11](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection13.png)

    Une fois le catalogage terminé, l’état passe à *Succès*. Sélectionnez **Fermer**.

    ![MABS SharePoint Protection12](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection14.png)
4. Sélectionnez l’objet SharePoint affiché dans l’onglet **Récupération** du serveur de sauvegarde Azure pour obtenir la structure de base de données de contenu. Cliquez avec le bouton droit sur l’élément, puis sélectionnez **Récupérer**.

    ![MABS SharePoint Protection13](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection15.png)
5. À ce stade, suivez les étapes de récupération mentionnées précédemment dans cet article pour la récupération d’une base de données de contenu SharePoint à partir du disque.

## <a name="switching-the-front-end-web-server"></a>Changement de serveur web front-end

Si vous avez plusieurs serveurs web frontaux et que vous voulez changer celui que MABS utilise pour protéger la batterie de serveurs, suivez les instructions suivantes :

La procédure suivante utilise l’exemple d’une batterie de serveurs avec deux serveurs web frontaux, *Server1* et *Server2*. MABS utilise *Server1* pour protéger la batterie de serveurs. Modifiez le serveur web frontal utilisé que MABS utilise pour *Server2* afin de pouvoir supprimer *Server1* de la batterie de serveurs.

> [!NOTE]
> Si le serveur web frontal que MABS utilise pour protéger la batterie de serveurs n’est pas disponible, utilisez la procédure suivante pour changer de serveur web frontal en commençant à l’étape 4.

### <a name="to-change-the-front-end-web-server-that-mabs-uses-to-protect-the-farm"></a>Pour changer le serveur web frontal que MABS utilise pour protéger la batterie de serveurs

1. Arrêtez le service Enregistreur VSS SharePoint sur *Server1* en exécutant la commande suivante sur une invite de commandes :

    ```CMD
    stsadm -o unregisterwsswriter
    ```

1. Sur *Server1*, ouvrez l’Éditeur du Registre, puis accédez à la clé suivante :

   **HKLM\System\CCS\Services\VSS\VssAccessControl**

1. Vérifiez toutes les valeurs répertoriées dans la sous-clé VssAccessControl. Si la donnée de valeur d’une entrée est 0 et qu’un autre enregistreur VSS est en cours d’exécution sous les informations d’identification du compte associé, remplacez-la par 1.

1. Installez un agent de protection sur *Server2*.

   > [!WARNING]
   > Vous pouvez intervertir des serveurs web frontaux uniquement si les deux serveurs se trouvent sur le même domaine.

1. Sur *Server2*, sur une invite de commandes, remplacez le répertoire par `_MABS installation location_\bin\` et exécutez **ConfigureSharepoint**. Pour plus d’informations sur ConfigureSharePoint, consultez [Configurer la sauvegarde](#configure-backup).

1. Sélectionnez le groupe de protection auquel appartient la batterie de serveurs, puis sélectionnez **Modifier le groupe de protection**.

1. Dans l’Assistant modifier le groupe, sur la page **Sélectionner les membres du groupe**, développez *Server2* et sélectionnez la batterie de serveurs, puis terminez l’Assistant.

   Une vérification de cohérence commencera.

1. Si vous avez effectué l’étape 6, vous pouvez maintenant supprimer le volume du groupe de protection.

## <a name="next-steps"></a>Étapes suivantes

Consultez l’article [Serveur Exchange de sauvegarde](backup-azure-exchange-mabs.md).
Consultez l’article [Serveur SQL de sauvegarde](backup-azure-sql-mabs.md).

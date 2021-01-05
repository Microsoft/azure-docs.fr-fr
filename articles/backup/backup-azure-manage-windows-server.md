---
title: Gérer les serveurs et les coffres Azure Recovery Services
description: Dans cet article, découvrez comment utiliser le tableau de bord Vue d’ensemble du coffre Recovery Services pour gérer et effectuer le monitorage de vos coffres Recovery Services.
ms.topic: conceptual
ms.date: 07/08/2019
ms.openlocfilehash: 74351d781287d863db8be0fc7d20517e0479106c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89002128"
---
# <a name="monitor-and-manage-recovery-services-vaults"></a>Surveiller et gérer les coffres Recovery Services

Cet article explique comment utiliser le tableau de bord **Vue d’ensemble** du coffre Recovery Services pour surveiller et gérer vos coffres Recovery Services. Lorsque vous ouvrez un coffre Recovery Services à partir de la liste, le tableau de bord **Vue d’ensemble** du coffre sélectionné s’ouvre. Le tableau de bord fournit des informations détaillées sur le coffre. Il s’y trouve des *vignettes* qui montrent l’état des alertes d’avertissement et des alertes critiques, les travaux de sauvegarde en cours et ceux qui ont échoué, ainsi que la quantité de stockage localement redondant (LRS) et la quantité de stockage géoredondant (GRS) utilisée. Si vous sauvegardez des machines virtuelles Azure dans le coffre, la vignette [**État de la prévérification de sauvegarde** affiche toutes les alertes d’avertissement et toutes les alertes critiques](#backup-pre-check-status). L’image suivante représente le tableau de bord **Vue d’ensemble** pour **Contoso-vault**. La vignette **Éléments de sauvegarde** montre que neuf éléments sont inscrits dans le coffre.

![Tableau de bord du coffre Recovery Services](./media/backup-azure-manage-windows-server/rs-vault-blade.png)

Les prérequis de cet article sont : un abonnement Azure, un coffre Recovery Services, et au moins un élément de sauvegarde configuré pour le coffre.

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="open-a-recovery-services-vault"></a>Ouvrez un coffre Recovery Services

Pour surveiller les alertes ou afficher les données de gestion concernant un coffre Recovery Services, ouvrez celui-ci.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide de votre abonnement Azure.

2. Dans le portail Azure, sélectionnez **Tous les services**.

   ![Ouvrir une liste de Recovery Services - Étape 1](./media/backup-azure-manage-windows-server/open-rs-vault-list.png)

3. Dans la boîte de dialogue **Tous les services**, tapez **Recovery Services**. Au fur et à mesure de la saisie, la liste est filtrée. Lorsque l’option **Coffres Recovery Services** s’affiche, sélectionnez-la pour ouvrir la liste des coffres Recovery Services de votre abonnement.

    ![Créer un coffre Recovery Services - Étape 1](./media/backup-azure-manage-windows-server/list-of-rs-vaults.png) <br/>

4. Dans la liste des coffres, sélectionnez un coffre pour ouvrir son tableau de bord **Vue d’ensemble**.

    ![Tableau de bord du coffre Recovery Services](./media/backup-azure-manage-windows-server/rs-vault-blade.png) <br/>

    Le tableau de bord Vue d’ensemble utilise des vignettes pour fournir des alertes et des données de travaux de sauvegarde.

## <a name="monitor-backup-jobs-and-alerts"></a>Surveiller les travaux et les alertes de sauvegarde

Le tableau de bord **Vue d’ensemble** du coffre Recovery Services fournit des vignettes qui contiennent des informations relatives à la surveillance et à l’utilisation. Les vignettes de la section Surveillance affichent les alertes d’avertissement, les alertes critiques, ainsi que les travaux en cours et les travaux ayant échoué. Sélectionnez une alerte ou un travail pour ouvrir le menu Alertes de sauvegarde ou Travaux de sauvegarde, filtré selon ce travail ou cette alerte.

![Tâches du tableau de bord Backup](./media/backup-azure-manage-windows-server/monitor-dashboard-tiles-warning.png)

La section Surveillance montre les résultats des requêtes prédéfinies **Alertes de sauvegarde** et **Travaux de sauvegarde**. Les vignettes de la section Surveillance fournissent des informations à jour concernant :

* Les alertes critiques et d’avertissement pour les travaux de sauvegarde (au cours des dernières 24 heures)
* État de la prévérification de sauvegarde pour les machines virtuelles Azure. Pour obtenir des informations complètes sur l’état de la prévérification, consultez [État de prévérification de la sauvegarde](#backup-pre-check-status).
* Les travaux de sauvegarde en cours et ceux ayant échoué (au cours des dernières 24 heures).

Les vignettes de la section Utilisation fournissent les informations suivantes :

* Le nombre d’éléments de sauvegarde configurés pour le coffre
* Le stockage Azure (réparti entre LRS et le GRS) consommé par le coffre

Sélectionnez les vignettes (à l’exception de Stockage de sauvegarde) pour ouvrir le menu associé. Dans l’image ci-dessus, la vignette Alertes de sauvegarde montre trois alertes critiques. Quand vous sélectionnez la ligne des alertes critiques dans la vignette Alertes de sauvegarde, seules les alertes critiques s’affichent.

![Menu Alertes de sauvegarde filtré sur les alertes critiques](./media/backup-azure-manage-windows-server/critical-backup-alerts.png)

Le menu Alertes de sauvegarde, dans l’image ci-dessus, est filtré par : l’état est actif, la gravité est critique et la période est sur les dernières 24 heures.

### <a name="backup-pre-check-status"></a>État de la prévérification de sauvegarde

Les vérifications préalables de la sauvegarde vérifient la configuration de vos machines virtuelles et recherchent les problèmes susceptibles de nuire aux sauvegardes. Elles agrègent ces informations pour que vous puissiez les consulter directement dans le tableau de bord du coffre Recovery Services et fournissent des mesures correctives recommandées afin de garantir la réussite des sauvegardes des applications et des fichiers. Elles ne nécessitent aucune infrastructure et n’entraînent aucun coût supplémentaire.  

Les prévérifications de sauvegarde s’exécutent dans le cadre des opérations de sauvegarde planifiée pour vos machines virtuelles Azure. Elles concluent à un des états suivants :

* **Réussite** : cet état indique que la configuration de votre machine virtuelle doit déboucher sur des sauvegardes réussies et qu’aucune action corrective n’est nécessaire.
* **Avertissement** : cet état indique un ou plusieurs problèmes dans la configuration de la machine virtuelle qui *peuvent* entraîner des échecs de la sauvegarde. Il fournit des étapes *recommandées* pour garantir la réussite des sauvegardes. Par exemple, si vous n’avez pas installé le dernier agent de machine virtuelle, les sauvegardes peuvent échouer par intermittence. Cette situation débouchera sur l’état Avertissement.
* **Critique** : Cet état indique un ou plusieurs problèmes critiques dans la configuration de la machine virtuelle qui *vont* entraîner des échecs de la sauvegarde et fournit des étapes *obligatoires* pour garantir la réussite des sauvegardes. Par exemple, un problème réseau provoqué par une mise à jour des règles du groupe de sécurité réseau d’une machine virtuelle entraîne l’échec des sauvegardes, car il empêche la machine virtuelle de communiquer avec le service Azure Backup. Cette situation peut déboucher sur un état critique.

Suivez les étapes ci-dessous pour commencer à résoudre les problèmes signalés par les vérifications préalables à la sauvegarde pour les sauvegardes de machines virtuelles sur votre coffre Recovery Services.

* Sélectionnez la mosaïque **État des vérifications préalables de sauvegarde (machines virtuelles Azure)** sur le tableau de bord du coffre Recovery Services.
* Sélectionnez une machine virtuelle dont l’état de vérification de la sauvegarde est **Critique** ou **Avertissement**. Cette action ouvre le volet des **détails de la machine virtuelle**.
* Sélectionnez le volet de notification en haut du volet pour afficher la description du problème de configuration et les étapes de résolution.

## <a name="manage-backup-alerts"></a>Gérer les alertes de sauvegarde

Pour accéder au menu Alertes de sauvegarde, dans le menu du coffre Recovery Services, sélectionnez **Alertes de sauvegarde**.

![Alertes de sauvegarde](./media/backup-azure-manage-windows-server/backup-alerts-menu.png)

Le rapport Alertes de sauvegarde répertorie les alertes concernant le coffre.

![Rapport Alertes de sauvegarde](./media/backup-azure-manage-windows-server/backup-alerts.png)

### <a name="alerts"></a>Alertes

La liste Alertes de sauvegarde affiche les informations sélectionnées pour les alertes filtrées. Dans le menu Alertes de sauvegarde, vous pouvez appliquer un filtre pour n’afficher que les alertes critiques ou les alertes d’avertissement.

| Niveau d’alerte | Événements qui génèrent des alertes |
| ----------- | ----------- |
| Critique | Vous recevez des alertes critiques quand : Des travaux de sauvegarde échouent, des travaux de récupération échouent, et quand vous suspendez la protection sur un serveur mais conservez les données.|
| Avertissement | Vous recevez des alertes d’avertissement quand : Les travaux de sauvegarde se terminent par des avertissements. Par exemple, lorsque moins de 100 fichiers n’ont pas été sauvegardés en raison de problèmes d’altération ou lorsque plus de 1 000 000 fichiers ont été correctement sauvegardés. |
| Informationnel | Actuellement, les alertes d’information ne sont pas utilisées. |

### <a name="viewing-alert-details"></a>Affichage des détails de l’alerte

Le rapport Alertes de sauvegarde signale huit informations pour chaque alerte. Utilisez le bouton **Choisir des colonnes** pour modifier les informations du rapport.

![Alertes de sauvegarde Bouton Choisir des colonnes](./media/backup-azure-manage-windows-server/backup-alerts.png)

Par défaut, toutes les informations, à l’exception de **l’heure de la dernière occurrence**, sont contenues dans le rapport.

* Alerte
* Élément de sauvegarde
* Serveur protégé
* severity
* Duration
* Heure de création
* Statut
* Heure de la dernière occurrence

### <a name="change-the-details-in-alerts-report"></a>Modifier les informations du rapport d’alertes

1. Pour modifier les informations du rapport, dans le menu **Alertes de sauvegarde**, sélectionnez **Choisir des colonnes**.

   ![Sélectionner Choisir des colonnes](./media/backup-azure-manage-windows-server/alerts-menu-choose-columns.png)

   Le menu **Choisir des colonnes** s’ouvre.

2. Dans le menu **Choisir des colonnes**, choisissez les informations que vous souhaitez inclure dans le rapport.

    ![Menu Choisir des colonnes](./media/backup-azure-manage-windows-server/choose-columns-menu.png)

3. Pour enregistrer vos modifications et fermer le menu Choisir des colonnes, sélectionnez **Terminé**.

   Si vous apportez des modifications, mais ne souhaitez pas les conserver, sélectionnez **Rétablir** pour retourner à la dernière configuration enregistrée.

### <a name="change-the-filter-in-alerts-report"></a>Modifier le filtre du rapport d’alertes

Utilisez le menu **Filtrer** pour changer le niveau de gravité, l’état, ainsi que l’heure de début et l’heure de fin des alertes.

> [!NOTE]
> Le fait de modifier le filtre des alertes de sauvegarde n’a aucun impact sur les alertes critiques ou d’avertissement qui s’affichent dans le tableau de bord Vue d’ensemble du coffre.
>  

1. Pour modifier le filtre des alertes de sauvegarde, dans le menu Alertes de sauvegarde, sélectionnez **Filtrer**.

   ![Menu Filtrer](./media/backup-azure-manage-windows-server/alerts-menu-choose-filter.png)

   Le menu Filtrer s’affiche.

   ![Menu Filtrer les alertes](./media/backup-azure-manage-windows-server/filter-alert-menu.png)

2. Modifiez le niveau de gravité, l’état, l’heure de début ou l’heure de fin, puis sélectionnez **Terminé** pour enregistrer vos modifications.

## <a name="configuring-notifications-for-alerts"></a>Configuration des notifications pour les alertes

Configurez des notifications afin de générer des e-mails lorsqu’une alerte d’avertissement ou une alerte critique est déclenchée. Vous pouvez choisir d’envoyer des e-mails toutes les heures, ou chaque fois qu’une alerte est déclenchée.

   ![Filtrer les alertes](./media/backup-azure-manage-windows-server/configure-notification.png)

Par défaut, les notifications par e-mail sont **activées**. Pour arrêter les notifications par e-mail, sélectionnez **Désactiver**.

Sous **Notifier**, choisissez **Par alerte** si vous ne souhaitez pas les regrouper ou si vous n’avez pas beaucoup d’éléments qui peuvent générer des alertes. Chaque alerte se traduit par une notification (paramètre par défaut), et un e-mail de résolution est immédiatement envoyé.

Si vous sélectionnez **Synthèse horaire**, un e-mail est envoyé aux destinataires, dans lequel sont indiquées les alertes non résolues qui ont été générées au cours de la dernière heure. Un e-mail de résolution est envoyé à la fin de l’heure.

Choisissez la gravité d’alerte (critique ou avertissement) à utiliser pour générer un e-mail. Actuellement, il n’existe pas d’alertes d’information.

## <a name="manage-backup-items"></a>Gérer les éléments de sauvegarde

Un coffre Recovery Services peut contenir de nombreux types de données de sauvegarde. [En savoir plus](backup-overview.md#what-can-i-back-up) sur ce que vous pouvez sauvegarder. Pour gérer les différents serveurs, ordinateurs, bases de données et charges de travail, sélectionnez la vignette **Éléments de sauvegarde** pour afficher le contenu du coffre.

![Mosaïque Éléments de sauvegarde](./media/backup-azure-manage-windows-server/backup-items.png)

La liste des éléments de sauvegarde, classés par type de gestion de sauvegarde, s’ouvre.

![Liste des éléments de sauvegarde](./media/backup-azure-manage-windows-server/list-backup-items.png)

Pour explorer un type d’instance protégée, sélectionnez l’élément correspondant dans la colonne Type de gestion des sauvegardes. Par exemple, dans l’illustration ci-dessus, le coffre contient deux machines virtuelles Azure protégées. En sélectionnant **Machine virtuelle Azure**, vous ouvrez la liste des machines virtuelles protégées dans ce coffre.

![Liste des machines virtuelles protégées](./media/backup-azure-manage-windows-server/list-of-protected-virtual-machines.png)

La liste des machines virtuelles comprend des données utiles : le groupe de ressources associé, la [prévérification de sauvegarde](#backup-pre-check-status) précédente, l’état de la dernière sauvegarde, et la date du point de restauration le plus récent. Dans la dernière colonne, les points de suspension permettent d’ouvrir le menu pour déclencher les tâches courantes. Les données utiles fournies dans les colonnes sont différentes pour chaque type de sauvegarde.

![Ouvrir le menu de sélection pour les tâches courantes](./media/backup-azure-manage-windows-server/ellipsis-menu.png)

## <a name="manage-backup-jobs"></a>Gérer les travaux de sauvegarde

Dans le tableau de bord du coffre, la vignette **Travaux de sauvegarde** montre plusieurs travaux qui sont en cours ou qui ont échoué au cours des dernières 24 heures. La vignette donne un aperçu du menu Travaux de sauvegarde.

![Vignette Travaux de sauvegarde](./media/backup-azure-manage-windows-server/backup-jobs-tile.png)

Pour afficher des informations détaillées sur les travaux, sélectionnez **En cours** ou **Échec** afin d’ouvrir le menu Travaux de sauvegarde filtré selon l’état choisi.

### <a name="backup-jobs-menu"></a>Menu Travaux de sauvegarde

Le menu **Travaux de sauvegarde** affiche des informations sur le type de l’élément, l’opération, l’état, l’heure de début et la durée.  

Pour ouvrir le menu Travaux de sauvegarde, dans le menu principal du coffre, sélectionnez **Travaux de sauvegarde**.

![Sélectionner des travaux de sauvegarde](./media/backup-azure-manage-windows-server/backup-jobs-menu-item.png)

La liste des travaux de sauvegarde s’ouvre.

![Liste des travaux de sauvegarde](./media/backup-azure-manage-windows-server/backup-jobs-list.png)

Le menu Travaux de sauvegarde affiche l’état de toutes les opérations de tous les types de sauvegarde, au cours des dernières 24 heures. Utilisez le menu **Filtrer** pour modifier les filtres. Les filtres sont abordés dans les sections suivantes.

Pour modifier les filtres :

1. Dans le menu Travaux de sauvegarde du coffre, sélectionnez **Filtrer**.

   ![Sélectionner un filtre pour les travaux de sauvegarde](./media/backup-azure-manage-windows-server/vault-backup-job-menu-filter.png)

    Le menu Filtrer s’ouvre.

   ![Le menu Filtrer s’ouvre pour les travaux de sauvegarde](./media/backup-azure-manage-windows-server/filter-menu-backup-jobs.png)

2. Choisissez les paramètres de filtre, puis sélectionnez **Terminé**. La liste filtrée est actualisée selon les nouveaux paramètres.

#### <a name="item-type"></a>Type d’élément

Le type d’élément correspond au type de gestion des sauvegardes de l’instance protégée. Il existe quatre types, qui sont cités dans la liste suivante. Vous pouvez afficher tous les types d’éléments ou un seul type d’élément. Vous ne pouvez pas sélectionner deux ou trois types d’éléments. Les types d’éléments disponibles sont les suivants :

* Tous les types d’éléments
* Machine virtuelle Azure
* Fichiers et dossiers
* Stockage Azure
* Charge de travail Azure

#### <a name="operation"></a>Opération

Vous pouvez afficher une seule opération ou toutes les opérations. Vous ne pouvez pas sélectionner deux ou trois opérations. Les opérations disponibles sont les suivantes :

* Toutes les opérations
* Inscrire
* Configurer une sauvegarde
* Sauvegarde
* Restaurer
* Désactiver la sauvegarde
* Supprimer les données de sauvegarde

#### <a name="status"></a>Statut

Vous pouvez afficher tous les états ou un seul état. Vous ne pouvez pas sélectionner deux ou trois états. Les états disponibles sont les suivants :

* Tous les états
* Completed
* En cours
* Échec
* Opération annulée
* Terminé avec des avertissements

#### <a name="start-time"></a>Heure de début

Date et heure de début de la requête. La valeur par défaut est de 24 heures.

#### <a name="end-time"></a>Heure de fin

Date et heure de fin de la requête.

### <a name="export-jobs"></a>Exporter les travaux

Utilisez l’option **Exporter les travaux** pour créer une feuille de calcul contenant des informations sur tous les travaux. La feuille de calcul comprend une feuille qui contient un récapitulatif de tous les travaux, ainsi qu’une feuille pour chaque travail.

Pour exporter les informations relatives aux travaux vers une feuille de calcul, sélectionnez **Exporter les travaux**. Le service crée une feuille de calcul en utilisant le nom du coffre et la date actuelle, mais vous pouvez modifier ce nom.

## <a name="monitor-backup-usage"></a>Surveiller l’utilisation de la sauvegarde

Dans le tableau de bord, la vignette Stockage de sauvegarde indique le stockage consommé dans Azure. Les données d’utilisation du stockage incluent :

* L’utilisation du stockage cloud LRS associée au coffre
* L’utilisation du stockage cloud GRS associée au coffre

## <a name="troubleshooting-monitoring-issues"></a>Résolution des problèmes de surveillance

**Problème :** Les travaux ou les alertes de l’agent Sauvegarde Azure n’apparaissent pas dans le portail.

**Procédure de résolution :** le processus ```OBRecoveryServicesManagementAgent``` envoie au service de sauvegarde Azure les données relatives à l’alerte et au travail. Il peut arriver que ce processus se bloque ou s’arrête.

1. Pour vérifier que le processus n’est pas en cours d’exécution, ouvrez **Gestionnaire des tâches** et vérifiez si ```OBRecoveryServicesManagementAgent``` est actuellement exécuté.

2. Si le processus n’est pas en cours d’exécution, ouvrez le **Panneau de configuration** et parcourez la liste des services. Démarrez ou redémarrez l’**Agent de gestion Microsoft Azure Recovery Services**.

    Pour plus d’informations, consultez les journaux d’activité à l’emplacement suivant :<br/>
   `<AzureBackup_agent_install_folder>\Microsoft Azure Recovery Services Agent\Temp\GatewayProvider*` Par exemple :<br/>
   `C:\Program Files\Microsoft Azure Recovery Services Agent\Temp\GatewayProvider0.errlog`

## <a name="next-steps"></a>Étapes suivantes

* [Restaurer un serveur Windows Server ou un client Windows à partir d’Azure](backup-azure-restore-windows-server.md)
* Pour en savoir plus sur Azure Backup, consultez la [vue d’ensemble d’Azure Backup](./backup-overview.md)

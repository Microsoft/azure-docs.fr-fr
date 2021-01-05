---
title: Sauvegarder un serveur Exchange avec le serveur de sauvegarde Azure
description: Découvrez comment sauvegarder un serveur Exchange dans une sauvegarde Microsoft Azure avec le serveur de sauvegarde Azure.
ms.reviewer: kasinh
ms.topic: conceptual
ms.date: 03/24/2017
ms.openlocfilehash: 86389ef89ced74905db0fc8bfeaabcea6140762a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91262992"
---
# <a name="back-up-an-exchange-server-to-azure-with-azure-backup-server"></a>Sauvegarder un serveur Exchange dans Azure avec le serveur de sauvegarde Azure

Cet article explique comment configurer un serveur de sauvegarde Azure pour sauvegarder un serveur Microsoft Exchange dans une sauvegarde Microsoft Azure.  

## <a name="prerequisites"></a>Prérequis

Avant de continuer, assurez-vous que le serveur de sauvegarde Azure est [installé et prêt](backup-azure-microsoft-azure-backup.md).

## <a name="mabs-protection-agent"></a>Agent de protection du serveur de sauvegarde Azure

Pour installer l’agent de protection du serveur de sauvegarde Azure sur le serveur Exchange, procédez comme suit :

1. Assurez-vous que les pare-feux sont correctement configurés. Consultez la page [Configuration d’exceptions de pare-feu pour l’agent](/system-center/dpm/configure-firewall-settings-for-dpm).
2. Installez l’agent sur le serveur Exchange en sélectionnant **Gestion > Agents > Installer** dans la console administrateur du serveur de sauvegarde Azure. Pour obtenir des instructions détaillées, consultez la page [Installation de l’agent de protection du serveur de sauvegarde Azure](/system-center/dpm/deploy-dpm-protection-agent).

## <a name="create-a-protection-group-for-the-exchange-server"></a>Créer un groupe de protection pour le serveur Exchange

1. Dans la console administrateur du serveur de sauvegarde Azure, sélectionnez **Protection**, puis **Nouveau** dans le ruban d’outils pour ouvrir l’assistant **Créer un groupe de protection**.
2. Dans l’écran d’**accueil** de l’Assistant, sélectionnez **Suivant**.
3. Dans l’écran **Sélectionner le type de groupe de protection**, sélectionnez **Serveurs**, puis **Suivant**.
4. Sélectionnez la base de données du serveur Exchange que vous souhaitez protéger, puis **Suivant**.

   > [!NOTE]
   > Si vous protégez Exchange 2013, vérifiez les [Conditions préalables pour Exchange 2013](/system-center/dpm/back-up-exchange).
   >
   >

    Dans l’exemple suivant, la base de données Exchange 2010 est sélectionnée.

    ![Sélectionner les membres du groupe](./media/backup-azure-backup-exchange-server/select-group-members.png)
5. Sélectionnez la méthode de protection des données.

    Attribuez un nom au groupe de protection, puis sélectionnez les deux options suivantes :

   * Je souhaite une protection à court terme à l’aide de Disque.
   * Je voudrais une protection en ligne.
6. Sélectionnez **Suivant**.
7. Sélectionnez l’option **Exécuter Eseutil pour vérifier l’intégrité des données** si vous souhaitez vérifier l’intégrité des bases de données Exchange Server.

    Une fois cette option sélectionnée, une vérification de la cohérence de sauvegarde s’exécute sur le serveur de sauvegarde Azure, afin d’éviter le trafic d’E/S généré lors de l’exécution de la commande **eseutil** sur le serveur Exchange.

   > [!NOTE]
   > Pour utiliser cette option, vous devez copier les fichiers Ese.dll et Eseutil.exe dans le répertoire C:\Program Files\Microsoft Azure Backup\DPM\DPM\bin sur le serveur de sauvegarde Azure. Dans le cas contraire, l’erreur suivante est déclenchée :  
   > ![erreur eseutil](./media/backup-azure-backup-exchange-server/eseutil-error.png)
   >
   >
8. Sélectionnez **Suivant**.
9. Sélectionnez la base de données pour **Sauvegarde de copie**, puis **Suivant**.

   > [!NOTE]
   > Si vous ne sélectionnez pas « Sauvegarde complète » pour au moins une copie DAG d’une base de données, les journaux ne seront pas tronqués.
   >
   >
10. Configurez les objectifs de **Sauvegarde à court terme**, puis **Suivant**.
11. Vérifiez l’espace disque disponible, puis sélectionnez **Suivant**.
12. Sélectionnez l’heure à laquelle le serveur de sauvegarde Azure doit créer la réplication initiale, puis sélectionnez **Suivant**.
13. Sélectionnez les options de vérification de cohérence, puis **Suivant**.
14. Choisissez la base de données que vous souhaitez sauvegarder sur Azure, puis sélectionnez **Suivant**. Par exemple :

    ![Spécifier les données de protection en ligne](./media/backup-azure-backup-exchange-server/specify-online-protection-data.png)
15. Définissez la planification pour **Sauvegarde Azure**, puis sélectionnez **Suivant**. Par exemple :

    ![Spécifier la planification de sauvegarde en ligne](./media/backup-azure-backup-exchange-server/specify-online-backup-schedule.png)

    > [!NOTE]
    > Notez que les points de récupération en ligne sont basés sur des points de récupération complète express. Par conséquent, vous devez planifier le point de récupération en ligne après l’heure spécifiée pour le point de récupération complète express.
    >
    >
16. Configurez la stratégie de rétention pour **Sauvegarde Azure**, puis sélectionnez **Suivant**.
17. Choisissez une option de réplication en ligne, puis sélectionnez **Suivant**.

    Si vous disposez d’une base de données volumineuse, la création de la sauvegarde initiale sur le réseau peut prendre un long moment. Pour éviter ce problème, vous pouvez créer une sauvegarde hors connexion.  

    ![Spécifier la stratégie de rétention en ligne](./media/backup-azure-backup-exchange-server/specify-online-retention-policy.png)
18. Confirmez les paramètres, puis sélectionnez **Créer un groupe**.
19. Sélectionnez **Fermer**.

## <a name="recover-the-exchange-database"></a>Récupérer la base de données Exchange

1. Pour récupérer une base de données Exchange, sélectionnez **Récupération** dans la console administrateur du serveur de sauvegarde Azure.
2. Localisez la base de données Exchange que vous souhaitez récupérer.
3. Sélectionnez un point de récupération en ligne dans la liste déroulante *Heure de récupération* .
4. Cliquez sur **Récupérer** pour lancer l’**Assistant Récupération**.

Pour les points de récupération en ligne, il existe cinq types de récupération :

* **Récupérer à l’emplacement d’origine du serveur Exchange :** les données seront récupérées sur le serveur Exchange d’origine.
* **Récupérer vers une autre base de données sur un serveur Exchange :** les données seront récupérées dans une autre base de données sur un autre serveur Exchange.
* **Récupérer dans une base de données de récupération :** les données seront récupérées dans une base de données de récupération Exchange (RDB).
* **Copier dans un dossier réseau :** les données seront récupérées dans un dossier réseau.
* **Copier sur bande :** si une bibliothèque de bandes ou un lecteur de bandes autonome est connecté et configuré sur MABS, le point de récupération est copié sur une bande disponible.

    ![Choisir la réplication en ligne](./media/backup-azure-backup-exchange-server/choose-online-replication.png)

## <a name="next-steps"></a>Étapes suivantes

* [Azure Backup - Forum Aux Questions](backup-azure-backup-faq.md)

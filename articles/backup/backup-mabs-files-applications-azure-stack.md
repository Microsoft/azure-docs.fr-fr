---
title: Sauvegarder des fichiers dans des machines virtuelles Azure Stack
description: Utilisez Sauvegarde Azure pour sauvegarder et restaurer des fichiers et applications Azure Stack dans votre environnement Azure Stack.
ms.topic: conceptual
ms.date: 06/05/2018
ms.openlocfilehash: cae95c10c510969cc0553a54a506789d6be427d7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89180981"
---
# <a name="back-up-files-and-applications-on-azure-stack"></a>Sauvegarder des fichiers et applications sur Azure Stack

Vous pouvez utiliser Sauvegarde Azure pour protéger (ou sauvegarder) des fichiers et applications sur Azure Stack. Pour sauvegarder des fichiers et applications, installez un serveur Sauvegarde Microsoft Azure en tant que machine virtuelle s’exécutant sur Azure Stack. Vous pouvez protéger les fichiers sur n’importe quel serveur Azure Stack du même réseau virtuel. Après avoir installé un serveur de sauvegarde Azure, ajoutez des disques Azure pour augmenter le stockage local disponible pour les données de sauvegarde à court terme. Un serveur de sauvegarde Azure utilise un stockage Azure pour la rétention à long terme.

> [!NOTE]
> Bien qu’un serveur de sauvegarde Azure et Data Protection Manager (DPM) System Center soient similaires, DPM n’est pas pris en charge pour une utilisation avec Azure Stack.
>

Cet article ne couvre pas l’installation du serveur de sauvegarde Azure dans l’environnement Azure Stack. Pour installer un serveur de sauvegarde Azure sur Azure Stack, voir l’article [Installer un serveur de sauvegarde Azure](backup-mabs-install-azure-stack.md).

## <a name="back-up-files-and-folders-in-azure-stack-vms-to-azure"></a>Sauvegarder des fichiers et des dossiers dans des machines virtuelles Azure Stack sur Azure

Pour configurer le serveur de sauvegarde Azure afin de protéger les fichiers dans des machines virtuelles Azure Stack, ouvrez la console du serveur de sauvegarde Azure. Vous allez utiliser la console pour configurer les groupes de protection et pour protéger les données sur vos machines virtuelles.

1. Dans la console du serveur de sauvegarde Azure, sélectionnez **Protection**, puis **Nouveau** dans la barre d’outils pour ouvrir l’Assistant **Création d’un nouveau groupe de protection**.

   ![Configurer la protection dans la console du serveur de sauvegarde Azure](./media/backup-mabs-files-applications-azure-stack/1-mabs-menu-create-protection-group.png)

    L’ouverture de l’assistant peut prendre quelques secondes. Une fois l’Assistant ouvert, sélectionnez **Suivant** pour passer à l’écran **Sélectionner le type de groupe de protection**.

   ![L’assistant Création d’un nouveau groupe de protection s’ouvre](./media/backup-mabs-files-applications-azure-stack/2-create-new-protection-group-wiz.png)

2. Dans l’écran **Sélectionner le type de groupe de protection**, choisissez **Serveurs**, puis sélectionnez **Suivant**.

    ![Sélectionner le type de groupe de protection](./media/backup-mabs-files-applications-azure-stack/3-select-protection-group-type.png)

    L’écran **Sélectionner les membres du groupe** s’ouvre.

    ![Sélectionner les membres du groupe](./media/backup-mabs-files-applications-azure-stack/4-opening-screen-choose-servers.png)

3. Sur l’écran **Sélectionner les membres du groupe**, sélectionnez **+** pour développer la liste des sous-éléments. Cochez les cases correspondant à tous les éléments que vous souhaitez protéger. Une fois tous les éléments sélectionnés, sélectionnez **Suivant**.

    ![Sélectionnez chaque élément que vous souhaitez protéger](./media/backup-mabs-files-applications-azure-stack/5-select-group-members.png)

    Microsoft recommande de placer toutes les données qui partageront la même stratégie de protection dans un seul groupe de protection. Pour des informations complètes sur la planification et le déploiement des groupes de protection, consultez l’article System Center DPM, [Déployer des groupes de protection](/system-center/dpm/create-dpm-protection-groups).

4. Dans l’écran **Sélectionner la méthode de protection des données**, entrez le nom du groupe de protection. Cochez la case **I want short-term protection using:** (Je souhaite une protection à court terme avec :) et **I want online protection** (Je souhaite une protection en ligne). Sélectionnez **Suivant**.

    ![Sélectionner la méthode de protection des données](./media/backup-mabs-files-applications-azure-stack/6-select-data-protection-method.png)

    Pour sélectionner **I want online protection** (Je souhaite une protection en ligne), vous devez d'abord sélectionner **I want short-term protection using:** (Je souhaite une protection à court terme avec :) disque. Le serveur de sauvegarde Azure ne protège pas sur bande. Le disque est donc le seul choix possible pour la protection à court terme.

5. Sur l’écran **Spécifier les objectifs à court terme**, choisissez la durée pendant laquelle conserver les points de récupération enregistrés sur le disque et quand enregistrer les sauvegardes incrémentielles. Sélectionnez **Suivant**.

    > [!IMPORTANT]
    > Vous ne devez **pas** conserver des données de récupération (sauvegarde) opérationnelle sur des disques attachés à un serveur de sauvegarde Azure pendant plus de cinq jours.
    >

    ![Spécifier les objectifs à court terme](./media/backup-mabs-files-applications-azure-stack/7-select-short-term-goals.png)

    Au lieu de sélectionner un intervalle pour les sauvegardes incrémentielles, pour exécuter une sauvegarde complète rapide juste avant chaque point de récupération planifié, sélectionnez **Juste avant un point de récupération**. Si vous protégez des charges de travail d’application, le serveur de sauvegarde Azure crée des points de récupération selon la fréquence de synchronisation planifiée (à condition que l’application prenne en charge les sauvegardes incrémentielles). Si l’application ne prend pas en charge les sauvegardes incrémentielles, le serveur de sauvegarde Azure exécute une sauvegarde complète rapide.

    Pour les **points de récupération de fichiers**, spécifiez à quel moment créer des points de récupération. Sélectionnez **Modifier** pour définir les heures et jours de la semaine auxquels des points de récupération doivent être créés.

6. Sur l’écran **Vérifier l’allocation de disque**, vérifiez l’espace disque de pool de stockage alloué pour le groupe de protection.

    La **taille totale des données** correspond à la taille des données que vous souhaitez sauvegarder et l’**espace disque à provisionner** sur le serveur de sauvegarde Azure correspond à l’espace recommandé pour le groupe de protection. Le serveur de sauvegarde Azure choisit le volume de sauvegarde idéal en fonction des paramètres. Toutefois, vous pouvez modifier les choix de volume de sauvegarde dans les détails de l’allocation de disque. Pour les charges de travail, sélectionnez le stockage préféré dans le menu déroulant. Vos modifications changent les valeurs de Stockage Total et Stockage libre dans le volet Stockage sur disque disponible. L’espace sous-approvisionné est la quantité de stockage que le serveur de sauvegarde Azure suggère que vous ajoutiez au volume pour continuer à procéder à des sauvegardes à l’avenir.

7. Sous **Choisir la méthode de création de réplica**, sélectionnez la façon dont vous souhaitez gérer la réplication initiale complète des données. Si vous choisissez de répliquer sur le réseau, Azure vous recommande de choisir une heure creuse. Pour de grandes quantités de données ou des conditions de réseau peu optimales, envisagez de répliquer les données à l’aide d’un support amovible.

8. Sous **Sélectionner les options de vérification de cohérence**, indiquez comment vous voulez automatiser les vérifications de cohérence. Autorisez les vérifications de cohérence à s’exécuter uniquement quand la réplication des données devient incohérente ou selon une planification. Si vous ne voulez pas configurer une vérification de cohérence automatique, effectuez une vérification manuelle à tout moment :
    * Dans la zone **Protection** de la console du serveur de sauvegarde Azure, cliquez avec le bouton droit sur le groupe de protection et sélectionnez **Effectuer une vérification de cohérence**.

9. Si vous avez choisi de sauvegarder dans Azure, dans la page **Indiquer les données de protection en ligne**, veillez à sélectionner les charges de travail que vous souhaitez sauvegarder dans Azure.

10. Sous **Spécifier la planification de sauvegarde en ligne**, indiquez quand doivent avoir lieu les sauvegardes incrémentielles dans Azure.

    Vous pouvez planifier des sauvegardes quotidiennes, hebdomadaires, mensuelles et annuelles, ainsi que sélectionner la date et l’heure de leur exécution. Les sauvegardes peuvent avoir lieu jusqu’à deux fois par jour. Chaque fois qu’une sauvegarde de travail s’exécute, un point de récupération de données est créé dans Azure à partir de la copie des données de sauvegarde stockées sur le disque du serveur de sauvegarde Azure.

11. Sous **Spécifier la stratégie de rétention en ligne**, sélectionnez la façon dont les points de récupération créés à partir des sauvegardes quotidiennes, hebdomadaires, mensuelles et annuelles sont conservés dans Azure.

12. Sous **Choisir la réplication en ligne**, sélectionnez la façon dont la réplication initiale complète des données doit se produire.

13. Sous **Résumé**, vérifiez vos paramètres. Lorsque vous sélectionnez **Créer un groupe**, la réplication initiale des données se produit. Une fois la réplication des données terminée, dans la page **État**, l’état du groupe de protection est **OK**. Le travail de sauvegarde initiale a lieu conformément aux paramètres du groupe de protection.

## <a name="recover-file-data"></a>Récupérer des données de fichier

Utilisez la console du serveur de sauvegarde Azure pour récupérer des données sur votre machine virtuelle.

1. Dans la barre de navigation de la console du serveur de sauvegarde Azure, sélectionnez **Récupération** et recherchez les données que vous souhaitez récupérer. Dans le volet de résultats, sélectionnez les données.

2. Dans le calendrier de la section des points de récupération, les dates en gras indiquent les points de récupération disponibles. Sélectionnez la date pour la récupération.

3. Dans le volet **Recoverable item** (Élément récupérable), sélectionnez l’élément que vous souhaitez récupérer.

4. Dans le volet **Actions**, sélectionnez **Récupérer** pour ouvrir l’Assistant Récupération.

5. Vous pouvez récupérer les données comme suit :

    * **Recover to the original location** (Récupérer à l’emplacement d’origine) : si l’ordinateur client est connecté via VPN, cette option ne fonctionne pas. Utilisez un autre emplacement, puis copiez les données à partir de cet emplacement.
    * **Récupérer à un autre emplacement**

6. Spécifiez les options de récupération :

    * Pour **Existing version recovery behavior** (Comportement de récupération de la version existante), sélectionnez **Créer une copie**, **Ignorer**, ou **Remplacer**. Remplacer est disponible uniquement pour la récupération vers l’emplacement d’origine.
    * Pour **Restore security** (Sécurité de restauration), choisissez **Apply settings of the destination computer** (Appliquer les paramètres de l’ordinateur de destination) ou **Apply the security settings of the recovery point version** (Appliquer les paramètres de la version du point de récupération).
    * Pour **Limitation de l’utilisation de la bande passante réseau**, sélectionnez **Modifier** pour activer la limitation de l’utilisation de la bande passante réseau.
    * Pour **Notification**, sélectionnez **Envoyer un e-mail lorsque la récupération est terminée**, et indiquez les destinataires qui recevront la notification. Séparez les adresses de messagerie par des virgules.
    * Après avoir fait vos choix, sélectionnez **Suivant**.

7. Vérifiez vos paramètres de récupération, puis sélectionnez **Récupérer**.

    >[!Note]
    >Pendant l’exécution du travail de récupération, tous les travaux de synchronisation des éléments de récupération sélectionnés sont annulés.

Si vous utilisez le stockage de sauvegarde moderne (MBS), la récupération par l’utilisateur final (EUR) du serveur de fichiers n’est pas prise en charge. L’EUR du serveur de fichiers comporte une dépendance sur le service VSS (Volume Shadow Copy), que le stockage de sauvegarde moderne n’utilise pas. Si l’EUR est activé, utilisez les étapes suivantes pour récupérer des données :

1. Accédez aux fichiers protégés, puis cliquez avec le bouton droit de la souris sur le nom du fichier et sélectionnez **Propriétés**.

2. Dans le menu **Propriétés**, sélectionnez **Versions précédentes** et choisissez la version que vous souhaitez récupérer.

## <a name="view-azure-backup-server-with-a-vault"></a>Afficher le serveur de sauvegarde Azure avec un coffre

Pour voir les entités du serveur de sauvegarde Azure sur le portail Azure, vous pouvez suivre les étapes ci-dessous :

1. Ouvrir le coffre Recovery Services.
2. Sélectionner Infrastructure de sauvegarde.
3. Afficher la liste des serveurs de gestion des sauvegardes.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation d’un serveur de sauvegarde Azure pour protéger d’autres charges de travail, voir les articles suivants :

* [Sauvegarder un batterie de serveurs SharePoint](./backup-mabs-sharepoint-azure-stack.md)
* [Sauvegarder SQL Server](./backup-mabs-sql-azure-stack.md)

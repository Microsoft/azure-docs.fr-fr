---
title: À propos de l’agent MARS
description: Découvrir comment l’agent MARS prend en charge les scénarios de sauvegarde
ms.topic: conceptual
ms.date: 08/04/2020
ms.openlocfilehash: 8e4ace0c17dbe75e989981db56583ed9477b3716
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87562597"
---
# <a name="about-the-microsoft-azure-recovery-services-mars-agent"></a>À propos de l’agent Microsoft Azure Recovery Services (MARS)

Cet article décrit comment le service Sauvegarde Azure utilise l’agent Microsoft Azure Recovery Services (MARS) pour sauvegarder et restaurer des fichiers, des dossiers et le volume ou l’état du système, depuis un ordinateur local sur Azure.

## <a name="backup-scenarios"></a>Scénarios de sauvegarde

L’agent MARS prend en charge les scénarios de sauvegarde suivants :

![Scénarios de sauvegarde MARS](./media/backup-try-azure-backup-in-10-mins/backup-scenarios.png)

- **Fichiers et dossiers** : protection sélective des fichiers et dossiers Windows.
- **Au niveau du volume** : protection de l’ensemble d’un volume Windows de votre machine.
- **Au niveau du système** : protection de l’ensemble de l’état du système Windows.

### <a name="additional-scenarios"></a>Autres cas de figure

- **Sauvegarder des fichiers et des dossiers spécifiques au sein de machines virtuelles Azure** : la méthode principale pour sauvegarder des machines virtuelles Azure consiste à utiliser une extension Sauvegarde Azure sur la machine virtuelle. L’extension sauvegarde la totalité de la machine virtuelle. Si vous souhaitez sauvegarder des fichiers et des dossiers spécifiques au sein d’une machine virtuelle, vous pouvez installer l’agent MARS sur les machines virtuelles Azure. Pour plus d’informations, consultez [Architecture : Sauvegarde de machine virtuelle Azure prédéfinie](./backup-architecture.md#architecture-built-in-azure-vm-backup).

- **Amorçage hors connexion** : les sauvegardes complètes initiales des données dans Azure transfèrent généralement de grandes quantités de données et nécessitent plus de bande passante réseau. Les sauvegardes suivantes transfèrent uniquement le volume de données delta (ou incrémentiel). La sauvegarde Azure compresse les sauvegardes initiales. Par l’intermédiaire du processus d’*amorçage hors connexion*, la sauvegarde Azure peut utiliser des disques pour charger les données de sauvegarde initiale compressées hors connexion dans Azure. Pour plus d’informations, consultez [Sauvegarde Azure hors connexion avec Azure Data Box](offline-backup-azure-data-box.md).

## <a name="restore-scenarios"></a>Scénarios de restauration

L’agent MARS prend en charge les scénarios de restauration suivants :

![Scénarios de récupération MARS](./media/backup-try-azure-backup-in-10-mins/restore-scenarios.png)

- **Même serveur** : le serveur sur lequel la sauvegarde a été créée à l’origine.
  - **Fichiers et dossiers** : choisissez les fichiers et dossiers individuels que vous souhaitez restaurer.
  - **Au niveau du volume** : choisissez le volume et le point de récupération que vous souhaitez restaurer. Ensuite, restaurez-le dans le même emplacement ou dans un autre emplacement sur la même machine.  Créez une copie des fichiers existants, remplacez les fichiers existants ou ignorez la récupération des fichiers existants.
  - **Au niveau du système** : choisissez l’état du système et le point de récupération à restaurer sur la même machine à un emplacement spécifié.

- **Serveur alternatif** : serveur autre que celui sur lequel la sauvegarde a été effectuée.
  - **Fichiers et dossiers** : choisissez les fichiers et dossiers individuels dont vous souhaitez restaurer le point de récupération sur une machine cible.
  - **Au niveau du volume** : choisissez le volume et le point de récupération que vous souhaitez restaurer à un autre emplacement. Créez une copie des fichiers existants, remplacez les fichiers existants ou ignorez la récupération des fichiers existants.
  - **Au niveau du système** : choisissez l’état du système et le point de récupération à restaurer en tant que fichier d’état système sur une autre machine.

## <a name="backup-process"></a>Processus de sauvegarde

1. Dans le portail Azure, créez un [coffre Recovery Services](install-mars-agent.md#create-a-recovery-services-vault) et choisissez des fichiers, des dossiers et l’état du système dans les **objectifs de sauvegarde**.
2. [Téléchargez les informations d’identification du coffre Recovery Services et le programme d’installation de l’agent](./install-mars-agent.md#download-the-mars-agent) sur une machine locale.

3. [Installez l’agent](./install-mars-agent.md#install-and-register-the-agent) et utilisez les informations d’identification du coffre téléchargées pour inscrire la machine auprès du coffre Recovery Services.
4. Dans la console de l’agent sur le client, [configurez la sauvegarde](./backup-windows-with-mars-agent.md#create-a-backup-policy) pour spécifier ce que vous voulez sauvegarder, le moment de la sauvegarde (planification), la durée de conservation les sauvegardes dans Azure (stratégie de rétention) et démarrer la protection.

![Diagramme de l’agent Sauvegarde Azure](./media/backup-try-azure-backup-in-10-mins/backup-process.png)

### <a name="additional-information"></a>Informations supplémentaires

- La **sauvegarde initiale** (première sauvegarde) s’exécute conformément à vos paramètres de sauvegarde.  L’agent MARS utilise le service VSS pour prendre un instantané des volumes sélectionnés pour la sauvegarde. L’agent n’utilise l’opération d’enregistreur de système de Windows que pour capturer l’instantané. Il n’utilise aucun enregistreur VSS d’application, et ne capture pas d’instantané cohérent au niveau de l’application. Après avoir pris l’instantané avec VSS, l’agent MARS crée un disque dur virtuel dans le dossier de cache que vous avez spécifié quand vous avez configuré la sauvegarde. L’agent stocke également des sommes de contrôle pour chaque bloc de données.

- Les **sauvegardes incrémentielles** (sauvegardes suivantes) s’exécutent conformément à la planification que vous spécifiez. Durant des sauvegardes incrémentielles, les fichiers modifiés sont identifiés et un nouveau disque dur virtuel est créé. Le disque dur virtuel est compressé et chiffré, puis envoyé au coffre. Une fois la sauvegarde incrémentielle terminée, le nouveau disque dur virtuel est fusionné avec le disque dur virtuel créé après la réplication initiale. Ce disque dur virtuel fusionné fournit le dernier état à utiliser pour la comparaison dans le cadre de la sauvegarde en cours.

- L’agent MARS peut exécuter le travail de sauvegarde en **mode optimisé** en utilisant le journal des modifications USN (nombre de séquences de mise à jour), ou en **mode non optimisé** en vérifiant les modifications apportées aux répertoires ou aux fichiers via une analyse du volume entier. Le mode non optimisé est plus lent parce que l’agent doit analyser chaque fichier sur le volume et effectuer une comparaison avec les métadonnées afin de déterminer les fichiers modifiés.  La **sauvegarde initiale** s’exécutera toujours en mode non optimisé. Si la sauvegarde précédente a échoué, la tâche de sauvegarde planifiée suivante s’exécute en mode non optimisé. Pour plus d’informations sur ces modes et leur vérification, consultez [cet article](backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-backup-job-running-in-unoptimized-mode).

## <a name="next-steps"></a>Étapes suivantes

[Matrice de prise en charge de l’agent MARS](./backup-support-matrix-mars-agent.md)

[Forum aux questions sur l’agent MARS](./backup-azure-file-folder-backup-faq.md)

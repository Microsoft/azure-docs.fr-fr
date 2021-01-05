---
title: Agent Microsoft Azure Recovery Services (MARS) – FAQ
description: Cette section répond aux questions courantes liées à la sauvegarde des fichiers et des dossiers avec Sauvegarde Microsoft Azure.
ms.topic: conceptual
ms.date: 07/29/2019
ms.openlocfilehash: 1edfaed99e60409774496c5ae75df8be99a8fe1f
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94917356"
---
# <a name="frequently-asked-questions---microsoft-azure-recovery-services-mars-agent"></a>Foire aux questions : agent Microsoft Azure Recovery Services (MARS)

Cet article répond aux questions courantes sur la sauvegarde de données à l’aide de l’agent MARS (Microsoft Azure Recovery Services) dans le service [Sauvegarde Microsoft Azure](backup-overview.md).

## <a name="configure-backups"></a>Configurer des sauvegardes

### <a name="where-can-i-download-the-latest-version-of-the-mars-agent"></a>Où puis-je télécharger la dernière version de l’agent MARS ?

Vous pouvez [télécharger ici](https://aka.ms/azurebackup_agent) la dernière version de l’agent MARS utilisé lors de la sauvegarde de machines Windows Server et System Center DPM, et du serveur de Sauvegarde Microsoft Azure.

### <a name="where-can-i-download-the-vault-credentials-file"></a>Où puis-je télécharger le fichier des informations d’identification du coffre ?

Dans le portail Azure, accédez à **Propriétés** pour votre coffre. Sous **Informations d’identification de la sauvegarde**, cochez la case **La dernière version de l’agent Recovery Services est déjà utilisée**. Sélectionnez **Télécharger**.

![Télécharger les informations d’identification](./media/backup-azure-file-folder-backup-faq/download-credentials.png)

### <a name="how-long-are-vault-credentials-valid"></a>Combien de temps les informations d’identification restent-elles valides ?

Les informations d’identification du coffre expirent au bout de 10 jours. Si le fichier d’informations d’identification arrive à expiration, retéléchargez-le à partir du Portail Microsoft Azure.

### <a name="what-characters-are-allowed-for-the-passphrase"></a>Quels sont les caractères autorisés pour la phrase secrète ?

La phrase secrète doit utiliser des caractères du jeu de caractères ASCII, avec des [valeurs ASCII inférieures ou égales à 127](/office/vba/language/reference/user-interface-help/character-set-0127).

### <a name="from-what-drives-can-i-back-up-files-and-folders"></a>Sur quels types de lecteurs puis-je sauvegarder des fichiers et des dossiers ?

Vous ne pouvez pas sauvegarder de données sur les types de lecteurs et volumes suivants :

* Médias amovibles : toutes les sources d’éléments de sauvegarde doivent être déclarées fixes.
* Volumes en lecture seule : le volume doit être accessible en écriture pour que le service VSS puisse fonctionner.
* Volumes hors connexion : le volume doit être en ligne pour que le service VSS puisse fonctionner.
* Partages réseau : le volume doit être local sur le serveur à sauvegarder à l’aide de la sauvegarde en ligne.
* Volumes protégés par BitLocker : le volume doit être déverrouillé pour que la sauvegarde soit possible.
* Identification du système de fichiers : NTFS est le seul système de fichiers pris en charge.

### <a name="what-file-and-folder-types-are-supported"></a>Quels sont les types de fichiers et dossiers pris en charge ?

[En savoir plus](backup-support-matrix-mars-agent.md#supported-file-types-for-backup) sur les types de fichiers et dossiers pris en charge pour la sauvegarde.

### <a name="can-i-use-the-mars-agent-to-back-up-files-and-folders-on-an-azure-vm"></a>Puis-je utiliser l’agent MARS pour sauvegarder des fichiers et des dossiers sur une machine virtuelle Azure ?  

Oui. La Sauvegarde Azure assure la sauvegarde au niveau de la machine virtuelle des machines virtuelles Azure à l’aide de l’extension de machine virtuelle de l’agent de machine virtuelle Azure. Si vous souhaitez sauvegarder des fichiers et des dossiers sur le système d’exploitation Windows invité de la machine virtuelle, vous pouvez installer l’agent MARS.

### <a name="can-i-use-the-mars-agent-to-back-up-files-and-folders-on-temporary-storage-for-the-azure-vm"></a>Puis-je utiliser l’agent MARS pour sauvegarder des fichiers et des dossiers dans un espace de stockage temporaire de la machine virtuelle Azure ?

Oui. Installez l’agent MARS et sauvegardez les fichiers et dossiers sur le système d’exploitation Windows invité, dans un espace de stockage temporaire.

* Les opérations de sauvegarde échouent lorsque les données du stockage temporaire sont effacées.
* Si les données du stockage temporaire sont supprimées, vous pouvez uniquement restaurer le stockage non volatile.

### <a name="how-do-i-register-a-server-to-another-region"></a>Comment faire pour inscrire un serveur dans une autre région ?

Les données de sauvegarde sont envoyées au centre de données du coffre dans lequel le serveur est inscrit. Le moyen le plus simple pour changer de centre de données consiste à désinstaller, puis réinstaller l’agent, et à inscrire la machine auprès d’un nouveau coffre, dans la région requise.

### <a name="does-the-mars-agent-support-windows-server-2012-deduplication"></a>L’agent MARS prend-il en charge la déduplication de Windows Server 2012 ?

Oui. Cet agent convertit les données dédupliquées en données normales lorsqu’il prépare l’opération de sauvegarde. Ensuite, il optimise les données pour la sauvegarde, les chiffre, puis les envoie au coffre.

### <a name="do-i-need-administrator-permissions-to-install-and-configure-the-mars-agent"></a>Ai-je besoin d’autorisations d’administrateur pour installer et configurer l’agent MARS ?

Oui, l’installation de l’agent MARS et la configuration des sauvegardes à l’aide de la console MARS impliquent que l’utilisateur soit administrateur local sur le serveur protégé.

### <a name="what-is-the-impact-on-mars-agent-backups-of-transferring-the-vault-subscription-to-a-different-azure-ad-directory"></a>Lorsque l'abonnement au coffre est transféré vers une autre instance d'Azure AD Directory, quel est l'impact sur les sauvegardes de l'agent MARS ?

Le changement d'instance d'Azure AD Directory n'a aucun impact sur les sauvegardes de l'agent MARS. 

## <a name="manage-backups"></a>Gestion des sauvegardes

### <a name="what-happens-if-i-rename-a-windows-machine-configured-for-backup"></a>Que se passe-t-il si je renomme un ordinateur Windows configuré pour la sauvegarde ?

Lorsque vous renommez un ordinateur Windows, toutes les sauvegardes actuellement configurées sont arrêtées.

* Vous devez enregistrer le nouveau nom de l’ordinateur dans le coffre de sauvegarde.
* Lorsque vous enregistrez le nouveau nom dans le coffre, la première opération de sauvegarde est une sauvegarde *complète*.
* Si vous devez récupérer des données sauvegardées dans le coffre avec le nom de l’ancien serveur, utilisez l’option permettant de restaurer les données à un autre emplacement dans l’Assistant Récupération de données. [Plus d’informations](backup-azure-restore-windows-server.md#use-instant-restore-to-restore-data-to-an-alternate-machine)

### <a name="what-is-the-maximum-file-path-length-for-backup"></a>Quelle est la longueur maximale autorisée pour le chemin d’accès au fichier de sauvegarde ?

L’agent MARS s’appuie sur NTFS et utilise la spécification de longueur de chemin d’accès définie par [l’API Windows](/windows/win32/FileIO/naming-a-file#fully-qualified-vs-relative-paths). Si les fichiers que vous souhaitez protéger sont plus longs que la valeur autorisée, sauvegardez le dossier parent ou le lecteur de disque.  

### <a name="what-characters-are-allowed-in-file-paths"></a>Quels sont les caractères autorisés dans les chemins d’accès aux fichiers ?

L’agent MARS s’appuie sur NTFS et autorise les [caractères pris en charge](/windows/win32/FileIO/naming-a-file#naming-conventions) dans les noms/chemins d’accès aux fichiers.

### <a name="the-warning-azure-backups-have-not-been-configured-for-this-server-appears"></a>Un avertissement s’affiche, signalant que les sauvegardes Azure n’ont pas été configurées pour ce serveur.

Cet avertissement, qui peut s’afficher même si vous avez configuré une stratégie de sauvegarde, apparaît lorsque les paramètres de planification de la sauvegarde stockés sur le serveur local diffèrent de ceux qui figurent dans le coffre de sauvegarde.

* Lorsque le serveur ou les paramètres ont été restaurés à un état correct connu, les planifications de sauvegarde peuvent se désynchroniser.
* Si vous recevez cet avertissement, [reconfigurez](backup-azure-manage-windows-server.md) la stratégie de sauvegarde, puis exécutez une sauvegarde à la demande pour resynchroniser les données du serveur local avec Azure.

## <a name="manage-the-backup-cache-folder"></a>Gérer le dossier du cache de sauvegarde

### <a name="whats-the-minimum-size-requirement-for-the-cache-folder"></a>Quelle est la taille minimale requise du dossier du cache ?

La taille du dossier du cache détermine la quantité de données que vous sauvegardez.

* Les volumes de dossier de cache doivent disposer d’un espace disponible correspondant à 5 à 10 % minimum de la taille totale des données de sauvegarde.
* Si le volume dispose de moins de 5 % de l’espace libre, augmentez la taille du volume, ou déplacez le dossier de cache vers un volume ayant suffisamment d’espace disponible en suivant [ces étapes](#how-do-i-change-the-cache-location-for-the-mars-agent).
* Si vous sauvegardez l’état du système Windows, il vous faut 30 à 35 Go supplémentaires d’espace libre sur le volume contenant le dossier du cache.

### <a name="how-to-check-if-scratch-folder-is-valid-and-accessible"></a>Comment vérifier si le dossier de travail est valide et accessible ?

1. Par défaut, le dossier de travail se trouve ici : `\Program Files\Microsoft Azure Recovery Services Agent\Scratch`
2. Vérifiez que le chemin de l’emplacement de votre dossier de travail correspond aux valeurs des entrées de clé de Registre ci-dessous :

    | Chemin d’accès au Registre | Clé de Registre | Valeur |
    | --- | --- | --- |
    | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config` |ScratchLocation |*Emplacement du nouveau dossier de cache* |
    | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider` |ScratchLocation |*Emplacement du nouveau dossier de cache* |

### <a name="how-do-i-change-the-cache-location-for-the-mars-agent"></a>Comment faire pour modifier l’emplacement du cache pour l’agent MARS ?

1. Pour arrêter le moteur de sauvegarde, exécutez cette commande dans une invite de commandes avec élévation de privilèges :

    ```Net stop obengine```
2. Si vous avez configuré la sauvegarde de l’état du système, ouvrez Gestion des disques et démontez le ou les disques dont le nom est au format `"CBSSBVol_<ID>"`.
3. Par défaut, le dossier de travail se trouve ici : `\Program Files\Microsoft Azure Recovery Services Agent\Scratch`
4. Copiez la totalité du dossier `\Scratch` sur un autre lecteur disposant d’un espace suffisant. Assurez-vous que le contenu est copié et non déplacé.
5. Mettez à jour les entrées de registre suivantes en utilisant le chemin du dossier de travail qui vient d’être déplacé.

    | Chemin d’accès au Registre | Clé de Registre | Valeur |
    | --- | --- | --- |
    | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config` |ScratchLocation |*Nouvel emplacement du dossier de travail* |
    | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider` |ScratchLocation |*Nouvel emplacement du dossier de travail* |

6. Redémarrez le moteur de sauvegarde via une invite de commandes avec élévation de privilèges :

    ```command
    Net stop obengine

    Net start obengine
    ```

7. Exécutez une sauvegarde à la demande. Une fois la sauvegarde terminée au nouvel emplacement, vous pouvez supprimer le dossier du cache d’origine.

### <a name="where-should-the-cache-folder-be-located"></a>Où le dossier du cache doit-il se trouver ?

Les emplacements suivants pour le dossier du cache ne sont pas recommandés :

* Partage réseau ou média amovible : le dossier du cache doit être local sur le serveur nécessitant une sauvegarde à l’aide de la sauvegarde en ligne. Les emplacements réseau ou les médias amovibles comme les lecteurs USB ne sont pas pris en charge.
* Volumes hors connexion : le dossier du cache doit être en ligne pour la sauvegarde attendue avec l’agent de sauvegarde Azure

### <a name="are-there-any-attributes-of-the-cache-folder-that-arent-supported"></a>Existe-t-il des attributs du dossier du cache qui ne sont pas pris en charge ?

Les attributs suivants ou leurs combinaisons ne sont pas pris en charge pour le dossier du cache :

* Chiffré
* Dédupliqué
* Compressé
* Partiellement alloué
* Point d’analyse

Le dossier du cache et les métadonnées du disque dur virtuel ne possèdent les attributs nécessaires pour l’agent de sauvegarde Azure.

### <a name="is-there-a-way-to-adjust-the-amount-of-bandwidth-used-for-backup"></a>Existe-t-il un moyen d’adapter la quantité de bande passante utilisée pour la sauvegarde ?

Oui. Vous pouvez utiliser l’option **Modifier les propriétés** de l’agent MARS pour régler la bande passante et la durée. [Plus d’informations](backup-windows-with-mars-agent.md#enable-network-throttling)

## <a name="restore"></a>Restaurer

### <a name="manage"></a>Gérer

#### <a name="can-i-recover-if-i-forgot-my-passphrase"></a>Puis-je effectuer la récupération si j’ai oublié ma phrase secrète ?

L’agent de sauvegarde Azure a besoin d’une phrase secrète (que vous avez fournie lors de l’inscription) pour déchiffrer les données sauvegardées lors de la restauration. Passez en revue les scénarios ci-dessous pour comprendre les options de gestion d’une phrase secrète perdue :

| Ordinateur d’origine <br> *(ordinateur source sur lequel les sauvegardes ont été effectuées)* | Passphrase | Options disponibles |
| --- | --- | --- |
| Disponible |Perdue |Si votre ordinateur d’origine (où les sauvegardes ont été effectuées) est disponible et toujours inscrit auprès du même coffre Recovery Services, vous pouvez régénérer la phrase secrète en suivant ces [étapes](./backup-azure-manage-mars.md#re-generate-passphrase).  |
| Perdue |Perdue |Impossible de récupérer les données ou les données ne sont pas disponibles |

Tenez compte des conditions suivantes :

* Si vous désinstallez puis réinscrivez l’agent sur le même ordinateur d’origine avec
  * la *même phrase secrète*, vous serez en mesure de restaurer vos données sauvegardées.
  * une *phrase secrète différente*, vous ne serez pas en mesure de restaurer vos données sauvegardées.
* Si vous installez l’agent sur un *autre ordinateur* avec
  * la *même phrase secrète* (utilisée sur l’ordinateur d’origine), vous serez en mesure de restaurer vos données sauvegardées.
  * une *phrase secrète différente*, vous ne serez pas en mesure de restaurer vos données sauvegardées.
* Si votre ordinateur d’origine est endommagé (vous empêchant de régénérer la phrase secrète via la console MARS) mais que vous pouvez restaurer le dossier de travail d’origine utilisé par l’agent MARS ou y accéder, vous pourrez peut-être effectuer la restauration (si vous avez oublié le mot de passe). Pour obtenir de l’aide, contactez le service clientèle.

#### <a name="how-do-i-recover-if-i-lost-my-original-machine-where-backups-were-taken"></a>Comment effectuer la récupération si j’ai perdu mon ordinateur d’origine (sur lequel les sauvegardes ont été effectuées) ?

Si vous avez la même phrase secrète (que celle que vous avez fournie lors de l’inscription) de l’ordinateur d’origine, vous pouvez restaurer les données sauvegardées sur un autre ordinateur. Passez en revue les scénarios ci-dessous pour comprendre vos options de restauration.

| Ordinateur d’origine | Passphrase | Options disponibles |
| --- | --- | --- |
| Perdue |Disponible |Vous pouvez installer et inscrire l’agent MARS sur un autre ordinateur avec la même phrase secrète que celle que vous avez fournie lors de l’inscription de l’ordinateur d’origine. Choisissez **Option de récupération** > **Autre emplacement** pour effectuer la restauration. Pour plus d’informations, consultez cet [article](./backup-azure-restore-windows-server.md#use-instant-restore-to-restore-data-to-an-alternate-machine).
| Perdue |Perdue |Impossible de récupérer les données ou les données ne sont pas disponibles |

### <a name="my-backup-jobs-have-been-failing-or-not-running-for-a-long-time-im-past-the-retention-period-can-i-still-restore"></a>Mes tâches de sauvegarde ont échoué ou n’ont pas été exécutées pendant une longue période. Ma période de rétention a expiré. Puis-je encore à restaurer ?

Par mesure de sécurité, Sauvegarde Azure conserve le point de récupération le plus récent, même s’il est ultérieur à la période de rétention. Une fois que les sauvegardes sont relancées et que les nouveaux points de récupération deviennent disponibles, le point de récupération plus ancien est supprimé conformément à la rétention spécifiée.

### <a name="what-happens-if-i-cancel-an-ongoing-restore-job"></a>Que se passe-t-il si j’annule un travail de restauration en cours ?

Si un travail de restauration en cours est annulé, le processus de restauration s’arrête. Tous les fichiers restaurés avant l’annulation restent dans la destination configurée (emplacement d’origine ou autre), sans aucun restauration.

### <a name="does-the-mars-agent-back-up-and-restore-acls-set-on-files-folders-and-volumes"></a>L’agent MARS sauvegarde-t-il et restaure-t-il les listes de contrôle d’accès définies sur les fichiers, les dossiers et les volumes ?

* L’agent MARS sauvegarde les listes de contrôle d’accès définies sur les fichiers, les dossiers et les volumes
* Pour l’option de récupération Restaurer le volume, l’agent MARS fournit une option permettant d’ignorer la restauration des autorisations de liste de contrôle d’accès au fichier ou dossier en cours de récupération.
* Pour l’option de récupération de fichiers et dossiers individuels, l’agent MARS effectue une restauration avec les autorisations de liste de contrôle d’accès (il n’existe aucune option permettant d’ignorer la restauration de liste de contrôle d’accès).

## <a name="next-steps"></a>Étapes suivantes

[Apprenez](tutorial-backup-windows-server-to-azure.md) comment sauvegarder une machine Windows.

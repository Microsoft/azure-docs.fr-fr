---
title: Détecter un problème de sauvegarde lente de fichiers et de dossiers
description: Apporte des conseils visant à vous aider à diagnostiquer la cause des problèmes de performances d’Azure Backup
ms.topic: troubleshooting
ms.date: 07/05/2019
ms.openlocfilehash: b3f2ac343ef4a703f347ec8a57f242a636bb32d2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88824013"
---
# <a name="troubleshoot-slow-backup-of-files-and-folders-in-azure-backup"></a>Résolution des problèmes de sauvegarde lente de fichiers et de dossiers dans Azure Backup

Cet article apporte des conseils visant à vous aider à diagnostiquer la cause du ralentissement des performances de sauvegarde des fichiers et des dossiers avec Azure Backup. Lorsque vous utilisez l’agent Azure Backup pour sauvegarder des fichiers, le processus de sauvegarde peut prendre plus longtemps que prévu. Ce ralentissement peut être dû à une ou plusieurs des raisons suivantes :

* [Des goulots d’étranglement de performances ralentissent l’ordinateur en cours de sauvegarde.](#cause1)
* [Un autre processus ou logiciel antivirus interfère avec le processus Azure Backup.](#cause2)
* [L’agent de sauvegarde est en cours d’exécution sur une machine virtuelle Azure](#cause3)  
* [Vous sauvegardez un grand nombre de fichiers (plusieurs millions).](#cause4)

Avant de commencer à résoudre le problème, nous vous recommandons de télécharger et d’installer le [dernier agent Azure Backup](https://aka.ms/azurebackup_agent). Nous apportons des mises à jour fréquentes à l’agent de sauvegarde afin de résoudre différents problèmes, d’ajouter des fonctionnalités et d’améliorer les performances.

Par ailleurs, nous vous recommandons vivement de passer en revue le [Forum aux questions sur le service Azure Backup](backup-azure-backup-faq.md) pour vous assurer que vous ne rencontrez pas des problèmes de configuration courants.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="cause-backup-job-running-in-unoptimized-mode"></a>Cause : Travail de sauvegarde en cours d’exécution en mode non optimisé

* L’agent MARS peut exécuter le travail de sauvegarde en **mode optimisé** en utilisant le journal des modifications USN (nombre de séquences de mise à jour) ou en **mode non optimisé** en vérifiant les modifications apportées aux répertoires ou aux fichiers en analysant l’ensemble du volume.
* Le mode non optimisé est lent, car l’agent doit analyser chaque fichier sur le volume et effectuer une comparaison avec les métadonnées afin de déterminer les fichiers modifiés.
* Pour effectuer cette vérification, ouvrez **Détails du travail** à partir de la console de l’agent MARS et vérifiez si l’état indique **Transfert de données (non optimisé, peut prendre plus de temps)** comme indiqué ci-dessous :

    ![Exécution en mode non optimisé](./media/backup-azure-troubleshoot-slow-backup-performance-issue/unoptimized-mode.png)

* Les conditions suivantes peuvent entraîner l’exécution du travail de sauvegarde en mode non optimisé :
  * La première sauvegarde (également appelée réplication initiale) s’exécute toujours en mode non optimisé
  * Si le travail de sauvegarde précédent échoue, la tâche de sauvegarde planifiée suivante s’exécute en mode non optimisée.

<a id="cause1"></a>

## <a name="cause-performance-bottlenecks-on-the-computer"></a>Cause : goulots d'étranglement au niveau des performances sur l'ordinateur

La présence de goulots d’étranglement sur l’ordinateur en cours de sauvegarde peut engendrer des retards. Par exemple, la capacité de l’ordinateur à lire le disque ou à écrire dessus ou encore la disponibilité de la bande passante pour l’envoi de données peuvent créer des goulots d’étranglement.

Windows fournit un outil intégré appelé [Analyseur de performances](https://techcommunity.microsoft.com/t5/ask-the-performance-team/windows-performance-monitor-overview/ba-p/375481) (Performance Monitor ou Perfmon) pour détecter les goulots d’étranglement.

Voici quelques compteurs de performances et plages qui peuvent être utiles pour diagnostiquer les goulots d’étranglement et permettre des sauvegardes optimales.

| Compteur | Statut |
| --- | --- |
| Disque logique (disque physique)--% temps d’inactivité |<li> De 100 à 50 % d'inactivité = sain</br><li> De 49 à 20 % d'inactivité = avertissement ou analyse</br><li> De 19 à 0 % d'inactivité = critique ou hors spécifications |
| Disque logique (disque physique)--% moy. disque s/lecture ou disque s/écriture |<li> De 0,001 à 0,015 ms = sain</br><li> De 0,015 à 0,025 ms = avertissement ou analyse</br><li> 0,026 ou plus = critique ou hors spécifications |
| Disque logique (disque physique)--Taille de file d’attente du disque actuelle (pour toutes les instances) |80 demandes pendant plus de 6 minutes |
| Mémoire--Octets de réserve non paginée |<li> Moins de 60 % du pool consommé = sain<br><li> De 61 à 80 % du pool consommé = avertissement ou analyse</br><li> Plus de 80 % du pool consommé = critique ou hors spécifications |
| Mémoire--Octets de réserve paginée |<li> Moins de 60 % du pool consommé = sain</br><li> De 61 à 80 % du pool consommé = avertissement ou analyse</br><li> Plus de 80 % du pool consommé = critique ou hors spécifications |
| Mémoire--Mégaoctets disponibles |<li> 50 % de mémoire disponible ou plus = sain</br><li> 25 % de mémoire disponible = analyse</br><li>10 % de mémoire disponible = avertissement</br><li> Moins de 100 Mo ou 5 % de mémoire disponible = critique ou hors spécifications |
| Processeur--\%temps processeur (toutes les instances) |<li> Moins de 60 % consommés = sain</br><li> De 61 à 90 % consommés = analyse ou attention</br><li> De 91 à 100 % consommés = critique |

> [!NOTE]
> Si vous déterminez que l’infrastructure est la cause du problème, nous vous recommandons de défragmenter les disques régulièrement pour de meilleures performances.
>
>

<a id="cause2"></a>

## <a name="cause-another-process-or-antivirus-software-interfering-with-azure-backup"></a>Cause : un autre processus ou logiciel antivirus interfère avec Sauvegarde Azure

Nous avons vu plusieurs exemples dans lesquels d’autres processus au sein du système Windows ont dégradé les performances du processus de l’agent Azure Backup. Par exemple, si vous utilisez l’agent Azure Backup et un autre programme pour sauvegarder des données ou si un logiciel antivirus est en cours d’exécution et qu’il verrouille les fichiers à sauvegarder, plusieurs verrouillages sur les fichiers peuvent provoquer des conflits. Dans ce cas, la sauvegarde risque d’échouer ou l’opération peut prendre plus longtemps que prévu.

La meilleure recommandation dans ce scénario consiste à désactiver l’autre programme de sauvegarde pour déterminer si le temps de sauvegarde de l’agent Azure Backup évolue. En règle générale, s’assurer que plusieurs opérations de sauvegarde n’interfèrent pas l’une sur l’autre est suffisant pour empêcher les conflits.

Pour les programmes antivirus, nous vous recommandons d’exclure les fichiers et emplacements suivants :

* C:\Program Files\Microsoft Azure Recovery Services Agent\bin\cbengine.exe as a process
* C:\Program Files\Microsoft Azure Recovery Services Agent\ folders
* Nouvel emplacement (si l’emplacement standard n’est pas utilisé)

<a id="cause3"></a>

## <a name="cause-backup-agent-running-on-an-azure-virtual-machine"></a>Cause : l'agent de sauvegarde est en cours d'exécution sur une machine virtuelle Azure

Si vous exécutez l’agent de sauvegarde sur une machine virtuelle, les performances seront plus lentes par rapport à une exécution sur un ordinateur physique. Cela s’explique par les limitations liées aux opérations d’E/S par seconde.  Toutefois, vous pouvez optimiser les performances en basculant les lecteurs de données sauvegardées vers un stockage premium Azure. Nous travaillons à la résolution de ce problème. Le correctif sera disponible dans une version ultérieure.

<a id="cause4"></a>

## <a name="cause-backing-up-a-large-number-millions-of-files"></a>Cause : vous sauvegardez un grand nombre de fichiers (plusieurs millions)

Le déplacement d’un volume important de données prend plus de temps que le déplacement d’un volume plus restreint de données. Dans certains cas, la durée de la sauvegarde dépend non seulement de la taille des données, mais également du nombre de fichiers ou de dossiers. Cela se vérifie plus particulièrement lorsque des millions de petits fichiers (de quelques octets à quelques kilo-octets) sont sauvegardés.

Ce comportement se produit parce qu’Azure catalogue vos fichiers pendant que vous sauvegardez les données et les déplacez dans Azure. Dans certains cas rares, l’opération de catalogage peut prendre plus longtemps que prévu.

Les indicateurs suivants peuvent vous aider à identifier le goulot d’étranglement et appliquer en conséquence les étapes suivantes :

* **L’interface utilisateur affiche la progression du transfert de données**. Les données sont toujours en cours de transfert. La bande passante réseau ou la taille des données engendrent peut-être un ralentissement.
* **L’interface utilisateur n’affiche pas la progression du transfert de données**. Ouvrez les journaux d’activité sous C:\Program Files\Microsoft Azure Recovery Services Agent\Temp, puis recherchez l’entrée FileProvider::EndData dans les journaux d’activité. Cette entrée signifie que le transfert de données est terminé et que l’opération de catalogage est en cours. N’annulez pas les tâches de sauvegarde. Attendez un peu plus longtemps que l’opération de catalogage soit terminée. Si le problème persiste, contactez le [support Azure](https://portal.azure.com/#create/Microsoft.Support).

Si vous essayez de sauvegarder des disques volumineux, il est recommandé d’utiliser [Azure Data Box](./offline-backup-azure-data-box.md) pour la première sauvegarde (réplication initiale).  Si vous ne pouvez pas utiliser Data Box, tous les problèmes réseau temporaires qui se produisent dans votre environnement pendant des transferts de données longs sur le réseau peuvent entraîner des échecs de sauvegarde.  Pour éviter ceux-ci, vous pouvez ajouter quelques dossiers à votre sauvegarde initiale et continuer à ajouter des dossiers de manière incrémentielle jusqu’à ce que tous les dossiers soient dûment sauvegardés dans Azure.  Les sauvegardes incrémentielles suivantes seront relativement plus rapides.

## <a name="next-steps"></a>Étapes suivantes

* [Questions courantes concernant la sauvegarde des fichiers et des dossiers](backup-azure-file-folder-backup-faq.md)

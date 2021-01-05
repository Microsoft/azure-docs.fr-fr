---
title: Configurer, optimiser et résoudre les problèmes de AzCopy avec le Stockage Azure | Microsoft Docs
description: Configurer, optimiser et résoudre les problèmes de AzCopy avec le Stockage Azure. Supprimer ou changer l’emplacement des fichiers journaux et de plan Modifier le niveau de consignation par défaut.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 07/27/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: dineshm
ms.openlocfilehash: a54c77844498beb4fd052153cbcf8cbe498cdb75
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132208"
---
# <a name="configure-optimize-and-troubleshoot-azcopy"></a>Configurer, optimiser et dépanner AzCopy

AzCopy est un utilitaire de ligne de commande que vous pouvez utiliser pour copier des blobs ou des fichiers vers ou depuis un compte de stockage. Cet article vous aide à effectuer les tâches de configuration avancées et vous aide à résoudre les problèmes pouvant survenir lorsque vous utilisez AzCopy.

> [!NOTE]
> Si vous recherchez du contenu pour vous aider à bien démarrer avec AzCopy, consultez les articles suivants :
> - [Bien démarrer avec AzCopy](storage-use-azcopy-v10.md)
> - [Transférer des données avec AzCopy et le Stockage Blob](storage-use-azcopy-blobs.md)
> - [Transférer des données avec AzCopy et le stockage de fichiers](storage-use-azcopy-files.md)
> - [Transférer des données avec AzCopy et des compartiments Amazon S3](storage-use-azcopy-s3.md)

## <a name="configure-proxy-settings"></a>Configuration des paramètres de proxy

Pour configurer les paramètres de proxy pour AzCopy, définissez la variable d’environnement `https_proxy`. Si vous exécutez AzCopy sur Windows, AzCopy détecte automatiquement les paramètres de proxy. Vous n’avez donc pas besoin d’utiliser ce paramètre dans Windows. Si vous choisissez d’utiliser ce paramètre dans Windows, il remplace la détection automatique.

| Système d’exploitation | Commande  |
|--------|-----------|
| **Windows** | Dans une invite de commandes, tapez : `set https_proxy=<proxy IP>:<proxy port>`<br> Pour PowerShell, tapez : `$env:https_proxy="<proxy IP>:<proxy port>"`|
| **Linux** | `export https_proxy=<proxy IP>:<proxy port>` |
| **macOS** | `export https_proxy=<proxy IP>:<proxy port>` |

Actuellement, AzCopy ne prend en charge les serveurs proxy qui requièrent une authentification avec NTLM ou Kerberos.

### <a name="bypassing-a-proxy"></a>Contournement d’un proxy ###

Si vous exécutez AzCopy sur Windows et souhaitez lui demander de n’utiliser aucun _proxy_ (au lieu de détecter automatiquement les paramètres), utilisez ces commandes. Avec ces paramètres, AzCopy ne recherche pas ou ne tente pas d’utiliser un proxy.

| Système d’exploitation | Environnement | Commandes  |
|--------|-----------|----------|
| **Windows** | Invite de commandes (CMD) | `set HTTPS_PROXY=dummy.invalid` <br>`set NO_PROXY=*`|
| **Windows** | PowerShell | `$env:HTTPS_PROXY="dummy.invalid"` <br>`$env:NO_PROXY="*"`<br>|

Sur d’autres systèmes d’exploitation, laissez simplement la variable HTTPS_PROXY non définie si vous ne souhaitez pas utiliser de proxy.

## <a name="optimize-performance"></a>Optimiser les performances

Vous pouvez effectuer un test d’évaluation des performances, puis utiliser des commandes et des variables d’environnement pour trouver un compromis optimal entre les performances et la consommation des ressources.

Cette section vous aidera à effectuer les tâches d'optimisation suivantes :

> [!div class="checklist"]
> * Exécuter des tests d’évaluation
> * Optimiser le débit
> * Optimiser l’utilisation de la mémoire 
> * Optimiser la synchronisation des fichiers

### <a name="run-benchmark-tests"></a>Exécuter des tests d’évaluation

Vous pouvez exécuter un test d’évaluation des performances sur des conteneurs d’objets blob ou des partages de fichiers spécifiques pour afficher des statistiques générales sur les performances et identifier des goulots d’étranglement des performances. Vous pouvez exécuter le test en chargeant ou en téléchargeant des données de test générées. 

Utilisez la commande suivante pour exécuter un test d’évaluation des performances.

|    |     |
|--------|-----------|
| **Syntaxe** | `azcopy benchmark 'https://<storage-account-name>.blob.core.windows.net/<container-name>'` |
| **Exemple** | `azcopy benchmark 'https://mystorageaccount.blob.core.windows.net/mycontainer/myBlobDirectory?sv=2018-03-28&ss=bjqt&srs=sco&sp=rjklhjup&se=2019-05-10T04:37:48Z&st=2019-05-09T20:37:48Z&spr=https&sig=%2FSOVEFfsKDqRry4bk3qz1vAQFwY5DDzp2%2B%2F3Eykf%2FJLs%3D'` |

> [!TIP]
> Cet exemple englobe les arguments de chemin d’accès avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

Cette commande exécute un test d’évaluation des performances en chargeant les données de test dans une destination spécifiée. Les données de test sont générées en mémoire, chargées dans la destination, puis supprimées de la destination une fois le test terminé. Vous pouvez spécifier le nombre de fichiers à générer et leur taille souhaitée à l’aide de paramètres de commande facultatifs.

Si vous préférez exécuter ce test en téléchargeant des données, définissez le paramètre `mode` sur `download`. Pour obtenir des informations de référence détaillées, consultez [benchmark azcopy](storage-ref-azcopy-bench.md). 

### <a name="optimize-throughput"></a>Optimiser le débit

Vous pouvez utiliser l’indicateur `cap-mbps` dans vos commandes pour plafonner le débit de données. Par exemple, la commande suivante reprend un travail et plafonne le débit à `10` mégabits (Mb) par seconde. 

```azcopy
azcopy jobs resume <job-id> --cap-mbps 10
```

Le débit peut diminuer pendant le transfert de petits fichiers. Vous pouvez augmenter le débit en définissant la variable d’environnement `AZCOPY_CONCURRENCY_VALUE`. Cette variable spécifie le nombre de demandes pouvant être effectuées simultanément.  

Si votre ordinateur dispose de moins de 5 unités centrales, la valeur de cette variable est définie sur `32`. Sinon, la valeur par défaut est égale à 16 multiplié par le nombre d’unités centrales. La valeur maximale par défaut de cette variable est `3000`, mais vous pouvez l’augmenter ou la diminuer manuellement. 

| Système d’exploitation | Commande  |
|--------|-----------|
| **Windows** | `set AZCOPY_CONCURRENCY_VALUE=<value>` |
| **Linux** | `export AZCOPY_CONCURRENCY_VALUE=<value>` |
| **macOS** | `export AZCOPY_CONCURRENCY_VALUE=<value>` |

Utilisez `azcopy env` pour vérifier la valeur actuelle de cette variable. Si la valeur est vide, vous pouvez lire la valeur utilisée en examinant le début de tout fichier journal AzCopy. La valeur sélectionnée et la raison pour laquelle elle a été sélectionnée sont signalées ici.

Avant de définir cette variable, nous vous recommandons d’exécuter un test d’évaluation. Le processus de test d’évaluation signalera la valeur de concurrence recommandée. En guise d’alternative, si vos conditions de réseau et charges utiles varient, affectez le mot `AUTO` à cette variable plutôt qu’un nombre particulier. AzCopy exécutera alors toujours le même processus de réglage automatique que celui qu’il utilise dans les tests d’évaluation.

### <a name="optimize-memory-use"></a>Optimiser l’utilisation de la mémoire

Définissez la variable d’environnement `AZCOPY_BUFFER_GB` pour spécifier la quantité maximale de mémoire système qu’AzCopy doit utiliser lors du téléchargement et du chargement des fichiers.
Exprimez cette valeur en gigaoctets (Go).

| Système d’exploitation | Commande  |
|--------|-----------|
| **Windows** | `set AZCOPY_BUFFER_GB=<value>` |
| **Linux** | `export AZCOPY_BUFFER_GB=<value>` |
| **macOS** | `export AZCOPY_BUFFER_GB=<value>` |

### <a name="optimize-file-synchronization"></a>Optimiser la synchronisation des fichiers

La commande [sync](storage-ref-azcopy-sync.md) identifie tous les fichiers à l'emplacement de destination, puis compare les noms de fichiers et les derniers horodatages modifiés avant de démarrer l'opération de synchronisation. Si vous disposez d'un grand nombre de fichiers, vous pouvez améliorer les performances en éliminant ce traitement préalable. 

Pour ce faire, utilisez plutôt la commande [azcopy copy](storage-ref-azcopy-copy.md) et définissez l'indicateur `--overwrite` sur `ifSourceNewer`. AzCopy comparera les fichiers tels qu'ils sont copiés sans effectuer d'analyses et de comparaison préalables. Cela offre un avantage en termes de performances lorsque le nombre de fichiers à comparer est important.

La commande [azcopy copy](storage-ref-azcopy-copy.md) ne supprime pas les fichiers de l'emplacement de destination. Par conséquent, si vous souhaitez supprimer les fichiers à l'emplacement de destination lorsqu'ils n'existent plus à l'emplacement source, utilisez la commande [azcopy sync](storage-ref-azcopy-sync.md) avec l'indicateur `--delete-destination` défini sur la valeur `true` ou `prompt`. 

## <a name="troubleshoot-issues"></a>Problèmes de dépannage

AzCopy crée des fichiers journaux et de plan pour chaque travail. Vous pouvez utiliser les journaux d’activité pour examiner et résoudre les problèmes potentiels. 

Les journaux d’activité contiennent l’état de la défaillance (`UPLOADFAILED`, `COPYFAILED`et `DOWNLOADFAILED`), le chemin complet et la raison de la défaillance.

Par défaut, les fichiers journaux et de plan sont situés dans le répertoire `%USERPROFILE%\.azcopy` sur Windows ou dans le répertoire `$HOME$\.azcopy` sur Mac et Linux, mais vous pouvez changer cet emplacement si vous le souhaitez.

L’erreur correspondante n’est pas nécessairement la première erreur qui apparaît dans le fichier. Pour les erreurs telles que les erreurs réseau, les délais d’expiration et les erreurs de serveur occupé, AzCopy effectue jusqu’à 20 nouvelles tentatives. En général, le processus aboutit.  La première erreur que vous voyez peut être un problème anodin qui a fait l’objet d’une nouvelle tentative réussie.  Ainsi, au lieu de vous concentrer sur la première erreur du fichier, recherchez les erreurs à proximité de `UPLOADFAILED`, `COPYFAILED` ou `DOWNLOADFAILED`. 

> [!IMPORTANT]
> Lorsque vous soumettez requête au support Microsoft (ou que vous résolvez le problème impliquant un tiers), partagez la version rédigée de la commande que vous souhaitez exécuter. Cela garantit que la SAP n’est pas accidentellement partagée avec tout le monde. Vous trouverez la version expurgée au début du fichier journal.

### <a name="review-the-logs-for-errors"></a>Passer en revue les journaux d’activité pour détecter la présence d’erreurs

La commande suivante obtient toutes les erreurs avec l’état `UPLOADFAILED` à partir du journal `04dc9ca9-158f-7945-5933-564021086c79` :

**Windows (PowerShell)**

```
Select-String UPLOADFAILED .\04dc9ca9-158f-7945-5933-564021086c79.log
```

**Linux**

```
grep UPLOADFAILED .\04dc9ca9-158f-7945-5933-564021086c79.log
```

### <a name="view-and-resume-jobs"></a>Afficher et reprendre des travaux

Chaque opération de transfert crée un travail AzCopy. Utilisez la commande suivante pour afficher l’historique des travaux :

```
azcopy jobs list
```

Pour afficher les statistiques du travail, utilisez la commande suivante :

```
azcopy jobs show <job-id>
```

Pour filtrer les transferts par état, utilisez la commande suivante :

```
azcopy jobs show <job-id> --with-status=Failed
```

Utilisez la commande suivante pour reprendre un travail annulé/ayant échoué. Cette commande utilise son identificateur ainsi que le jeton SAP, car il n’est pas persistant pour des raisons de sécurité :

```
azcopy jobs resume <job-id> --source-sas="<sas-token>"
azcopy jobs resume <job-id> --destination-sas="<sas-token>"
```

> [!TIP]
> Placez les arguments de chemin d’accès tels que le jeton SAS avec des guillemets simples (' '). Utilisez des guillemets simples dans tous les interpréteurs de commandes, à l’exception de l’interface de commande Windows (cmd. exe). Si vous utilisez une interface de commande Windows (cmd. exe), placez les arguments de chemin d’accès entre guillemets doubles (" ") au lieu de guillemets simples (' ').

Lorsque vous reprenez un travail, AzCopy examine le fichier de plan de travail. Le fichier de plan répertorie tous les fichiers identifiés pour le traitement lors de la création du travail. Lorsque vous reprenez un travail, AzCopy essaiera de transférer tous les fichiers répertoriés dans le fichier de plan et qui n’ont pas été déjà transférés.

## <a name="change-the-location-of-the-plan-and-log-files"></a>Changer l’emplacement des fichiers journaux et de plan

Par défaut, les fichiers journaux et de plan sont situés dans le répertoire `%USERPROFILE%\.azcopy` sur Windows ou dans le répertoire `$HOME$\.azcopy` sur Mac et Linux. Vous pouvez changer cet emplacement.

### <a name="change-the-location-of-plan-files"></a>Changer l’emplacement des fichiers de plan

Utilisez l’une de ces commandes.

| Système d’exploitation | Commande  |
|--------|-----------|
| **Windows** | PowerShell :`$env:AZCOPY_JOB_PLAN_LOCATION="<value>"` <br> Dans une invite de commandes, tapez : `set AZCOPY_JOB_PLAN_LOCATION=<value>` |
| **Linux** | `export AZCOPY_JOB_PLAN_LOCATION=<value>` |
| **macOS** | `export AZCOPY_JOB_PLAN_LOCATION=<value>` |

Utilisez `azcopy env` pour vérifier la valeur actuelle de cette variable. Si la valeur est vide, les fichiers de plan sont écrits à l’emplacement par défaut.

### <a name="change-the-location-of-log-files"></a>Changer l’emplacement des fichiers journaux

Utilisez l’une de ces commandes.

| Système d’exploitation | Commande  |
|--------|-----------|
| **Windows** | PowerShell :`$env:AZCOPY_LOG_LOCATION="<value>"` <br> Dans une invite de commandes, tapez : `set AZCOPY_LOG_LOCATION=<value>`|
| **Linux** | `export AZCOPY_LOG_LOCATION=<value>` |
| **macOS** | `export AZCOPY_LOG_LOCATION=<value>` |

Utilisez `azcopy env` pour vérifier la valeur actuelle de cette variable. Si la valeur est vide, les journaux sont écrits à l’emplacement par défaut.

## <a name="change-the-default-log-level"></a>Modifier le niveau de consignation par défaut

Par défaut, le niveau de consignation d’AzCopy est défini sur `INFO`. Si vous souhaitez réduire la verbosité du journal afin d’économiser de l’espace disque, écrasez ce paramètre en utilisant l’option ``--log-level``. 

Les niveaux d’enregistrement disponibles sont : `NONE`, `DEBUG`, `INFO`, `WARNING`, `ERROR`, `PANIC` et `FATAL`.

## <a name="remove-plan-and-log-files"></a>Supprimer les fichiers de plan et les fichiers journaux

Si vous souhaitez supprimer tous les fichiers journaux et de plan de votre ordinateur local pour économiser de l’espace disque, utilisez la commande `azcopy jobs clean`.

Pour supprimer les fichiers journaux et de plan associés à un seul travail, utilisez `azcopy jobs rm <job-id>`. Remplacez l’espace réservé `<job-id>` dans cet exemple par l’ID du travail.



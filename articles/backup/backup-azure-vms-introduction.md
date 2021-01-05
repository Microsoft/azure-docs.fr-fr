---
title: À propos de la sauvegarde de machine virtuelle Azure
description: Dans cet article, découvrez la manière dont le service Sauvegarde Azure sauvegarde les machines virtuelles Azure, et comment suivre les meilleures pratiques.
ms.topic: conceptual
ms.date: 09/13/2019
ms.openlocfilehash: 7fa47b83eb8fa06c028079cf47ea0cb46df31860
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325228"
---
# <a name="an-overview-of-azure-vm-backup"></a>Vue d’ensemble de la sauvegarde de machines virtuelles Azure

Cet article décrit la manière dont le [service Sauvegarde Azure](./backup-overview.md) sauvegarde les machines virtuelles Azure.

Le service Sauvegarde Azure propose des sauvegardes indépendantes et isolées pour éviter une destruction involontaire des données de vos machines virtuelles. Les sauvegardes sont stockées dans un coffre Recovery Services avec gestion intégrée des points de récupération. La configuration et la mise à l’échelle sont simples, les sauvegardes sont optimisées, et vous pouvez facilement restaurer en fonction des besoins.

Pendant le processus de sauvegarde, une [capture instantanée est créée](#snapshot-creation) et les données sont transférées vers le coffre Recovery Services sans que cela n’impacte les charges de travail de production. La capture instantanée offre différents niveaux de cohérence, comme décrit [ici](#snapshot-consistency).

Le service Sauvegarde Azure propose également des offres spécialisées pour les charges de travail de base de données comme [SQL Server](backup-azure-sql-database.md) et [SAP HANA](sap-hana-db-about.md) qui prennent en charge les charges de travail, offrent un objectif de point de récupération (RPO) de 15 minutes et autorisent la sauvegarde et la restauration des bases de données individuelles.

## <a name="backup-process"></a>Processus de sauvegarde

Voici comment Sauvegarde Azure effectue une sauvegarde de machines virtuelles Azure :

1. Pour des machines virtuelles Azure sélectionnés pour la sauvegarde, Sauvegarde Azure démarre un travail de sauvegarde conformément à la planification de sauvegarde que vous spécifiez.
1. Lors de la première sauvegarde, une extension de sauvegarde est installée sur la machine virtuelle si celle-ci est en cours d’exécution.
    - Pour les machines virtuelles Windows, l’[extension VMSnapshot](../virtual-machines/extensions/vmsnapshot-windows.md) est installée.
    - Pour les machines virtuelles Linux, l’[extension VMSnapshotLinux](../virtual-machines/extensions/vmsnapshot-linux.md) est installée.
1. Pour les machines virtuelles Windows en cours d’exécution, le service Sauvegarde se coordonne avec le service VSS (Volume Shadow Copy Service) Windows pour prendre un instantané de cohérence d’application de la machine virtuelle.
    - Par défaut, Sauvegarde Azure effectue des sauvegardes VSS complètes.
    - Si la sauvegarde ne peut pas prendre d’instantané de cohérence d’application, elle prend un instantané cohérent au niveau fichier du stockage sous-jacent (parce qu’aucune écriture d’application n’a lieu quand la machine virtuelle est arrêtée).
1. Pour les machines virtuelles Linux, Sauvegarde Azure effectue une sauvegarde cohérente au niveau fichier. Pour les instantanés de cohérence d’application, vous devez personnaliser manuellement le pré-script et le post-script.
1. Une fois que le service Sauvegarde a pris l’instantané, il transfère les données au coffre.
    - La sauvegarde est optimisée grâce à la sauvegarde de chaque disque de machine virtuelle en parallèle.
    - Pour chaque disque sauvegardé, Sauvegarde Azure lit les blocs sur le disque, puis identifie et transfère uniquement les blocs de données ayant changé (le delta) depuis la sauvegarde précédente.
    - Les données d’instantanés peuvent ne pas être immédiatement copiées dans le coffre. Aux heures de pointe, cela peut prendre quelques heures. La durée de sauvegarde totale d’une machine virtuelle est inférieure à 24 heures pour les stratégies de sauvegarde quotidienne.
1. Les modifications apportées à une machine virtuelle Windows une fois la Sauvegarde Azure activée sur celle-ci sont les suivantes :
    - Le composant redistribuable de Microsoft Visual C++ 2013 (x64) – 12.0.40660 est installé sur la machine virtuelle
    - Le type de démarrage du Service VSS est passé de manuel à automatique
    - Le service Windows IaaSVmProvider est ajouté

1. Une fois le transfert de données terminé, l’instantané est supprimé et un point de récupération est créé.

![Architecture de la sauvegarde des machines virtuelles Azure](./media/backup-azure-vms-introduction/vmbackup-architecture.png)

## <a name="encryption-of-azure-vm-backups"></a>Chiffrement des sauvegardes de machines virtuelles Azure

Quand vous sauvegardez des machines virtuelles Azure avec Sauvegarde Azure, ces machines sont chiffrées au repos avec SSE (Storage Service Encryption). Sauvegarde Azure peut également sauvegarder des machines virtuelles Azure chiffrées avec Azure Disk Encryption.

**Chiffrement** | **Détails** | **Support**
--- | --- | ---
**SSE** | Avec SSE, Stockage Azure chiffre automatiquement les données au repos avant de les stocker. Stockage Azure déchiffre aussi les données avant de les récupérer. Sauvegarde Azure prend en charge les sauvegardes de machines virtuelles avec deux types de Storage Service Encryption :<li> **SSE avec clés gérées par la plateforme** : ce chiffrement est utilisé par défaut pour tous les disques de vos machines virtuelles. [En savoir plus](../virtual-machines/disk-encryption.md#platform-managed-keys).<li> **SSE avec clés gérées par le client** : utiliser des CMK signifie que vous gérez les clés utilisées pour chiffrer les disques. [En savoir plus](../virtual-machines/disk-encryption.md#customer-managed-keys). | Sauvegarde Azure utilise SSE pour chiffrer au repos les machines virtuelles Azure.
**Azure Disk Encryption** | Azure Disk Encryption chiffre les disques du système d’exploitation et de données pour les machines virtuelles Azure.<br/><br/> Azure Disk Encryption s’intègre avec les clés de chiffrement de lecteur BitLocker qui sont sauvegardées dans un coffre de clés en tant que secrets. Azure Disk Encryption s’intègre également avec les clés de chiffrement Azure Key Vault. | Sauvegarde Azure prend en charge la sauvegarde des machines virtuelles Azure managées et non managées, chiffrées uniquement avec des clés de chiffrement de lecteur BitLocker, ou avec des clés de chiffrement de lecteur BitLocker et des clés de chiffrement Azure Key Vault.<br/><br/> Les clés des deux types sont sauvegardées et chiffrées.<br/><br/> Ces clés étant sauvegardées, les utilisateurs disposant des autorisations nécessaires peuvent restaurer les clés et secrets dans le coffre de clés si nécessaire. Ces utilisateurs peuvent également récupérer la machine virtuelle chiffrée.<br/><br/> Ni Azure ni des utilisateurs non autorisés ne peuvent lire les clés et secrets chiffrés.

Pour les machines virtuelles Azure managées et non managées, le service Sauvegarde prend en charge les machines virtuelles chiffrées uniquement avec des clés de chiffrement de lecteur BitLocker, ou avec des clés de chiffrement de lecteur BitLocker et des clés de chiffrement Azure Key Vault.

Les clés de chiffrement de lecteur BitLocker (secrets) et les clés de chiffrement Azure Key Vault (clés) sauvegardées sont chiffrées. Ces données ne peuvent être lues et utilisées que si elles sont restaurées dans le coffre de clés par des utilisateurs autorisés. Ni Azure ni des utilisateurs non autorisés ne peuvent lire ou utiliser des clés ou secrets sauvegardés.

Les clés de chiffrement de lecteur BitLocker sont également sauvegardées. Ainsi, en cas de perte, des utilisateurs autorisés peuvent restaurer les clés de chiffrement de lecteur BitLocker dans le coffre de clés pour récupérer les machines virtuelles chiffrées. Seuls les utilisateurs disposant du niveau d’autorisations nécessaire peuvent sauvegarder et restaurer des machines virtuelles chiffrées ou des clés et secrets.

## <a name="snapshot-creation"></a>Création d’instantanés

Sauvegarde Azure prend des captures instantanées en fonction de la planification de sauvegarde.

- **Machines virtuelles Windows :** pour les machines virtuelles Windows, le service Sauvegarde se coordonne avec le service VSS afin de prendre un instantané de cohérence d’application des disques de machine virtuelle.  Par défaut, Sauvegarde Azure effectue une sauvegarde VSS complète (il tronque les journaux de l’application, par exemple SQL Server, au moment de la sauvegarde pour obtenir une sauvegarde cohérente au niveau de l’application).  Si vous utilisez une base de données SQL Server sur la sauvegarde de machine virtuelle Azure, vous pouvez modifier le paramètre pour effectuer une copie de sauvegarde VSS (pour conserver les journaux). Pour plus d’informations, consultez [cet article](./backup-azure-vms-troubleshoot.md#troubleshoot-vm-snapshot-issues).

- **Machines virtuelles Linux :** pour prendre des instantanés de cohérence d’application de machines virtuelles Linux, utilisez le framework pré-script et post-script de Linux pour écrire vos propres scripts afin de garantir la cohérence.

  - Sauvegarde Azure appelle uniquement les pré-scripts et post-scripts que vous avez écrits.
  - Si les pré-scripts et post-scripts s’exécutent correctement, Sauvegarde Azure marque le point de récupération comme étant cohérent dans les applications. Toutefois, lors de l’utilisation de scripts personnalisés, vous êtes responsable de la cohérence des applications .
  - [Apprenez-en davantage](backup-azure-linux-app-consistent.md) sur la configuration des scripts.

## <a name="snapshot-consistency"></a>Cohérence des instantanés

Le tableau suivant explique les différents types de cohérence de capture instantanée :

**Instantané** | **Détails** | **Récupération** | **Considération**
--- | --- | --- | ---
**Cohérence des applications** | Les sauvegardes cohérentes dans les applications capturent le contenu et les opérations d’E/S en attente de la mémoire. Les instantanés de cohérence d’application utilisent l’enregistreur VSS (ou un pré/post-script pour Linux) pour vérifier la cohérence des données d’application avant une sauvegarde. | Lors de la récupération d’une machine virtuelle avec un instantané de cohérence d’application, la machine virtuelle démarre. Il n’y a aucune altération ni perte des données. Les applications démarrent dans un état cohérent. | Windows : Tous les enregistreurs VSS ont réussi<br/><br/> Linux : Les pré/post-scripts sont configurés et ont réussi
**Cohérence du système de fichiers** | Les sauvegardes cohérentes de système de fichiers assurent la cohérence en prenant une capture instantanée de tous les fichiers au même moment.<br/><br/> | Lorsque vous récupérez une machine virtuelle avec un instantané cohérent du système de fichiers, la machine virtuelle démarre. Il n’y a aucune altération ni perte des données. Les applications ont besoin d’implémenter leur propre mécanisme de « correction » pour s’assurer que les données restaurées sont cohérentes. | Windows : Certains enregistreurs VSS ont échoué <br/><br/> Linux : Par défaut (si les pré/post-scripts ne sont pas configurés ou ont échoué)
**Cohérence en cas d’incident** | Des instantanés de cohérence des incidents sont pris généralement si une machine virtuelle Azure s’arrête au moment de la sauvegarde. Seules les données déjà présentes sur le disque au moment de la sauvegarde sont capturées et sauvegardées. | Le mécanisme commence par le processus de démarrage de la machine virtuelle suivi d’une vérification du disque afin de corriger les erreurs dues à une altération. Toutes les données en mémoire ou opérations d’écriture qui n’ont pas été transférées sur disque avant l’incident sont perdues. Les applications implémentent leur propre vérification des données. Par exemple, une application de base de données peut utiliser son journal des transactions pour la vérification. Si le journal des transactions comporte des entrées qui ne figurent pas dans la base de données, le logiciel de base de données effectue alors une restauration des transactions jusqu’à ce que les données soient cohérentes. | La machine virtuelle est à l’état d’arrêt (stoppée/désallouée).

>[!NOTE]
> Si l’état d’approvisionnement est **Réussi**, Sauvegarde Azure effectue des sauvegardes cohérentes du système de fichiers. Si l’état d’approvisionnement est **Non disponible** ou **Échec**, des sauvegardes cohérentes en cas d’incident sont effectuées. Si l’état d’approvisionnement est **Création** ou **Suppression**, cela signifie que Sauvegarde Azure tente à nouveau d’effectuer les opérations.

## <a name="backup-and-restore-considerations"></a>Considérations relatives à la sauvegarde et à la restauration

**Considération** | **Détails**
--- | ---
**Disque** | Les disques d’une machine virtuelle sont sauvegardés en parallèle. Par exemple, si une machine virtuelle comprend quatre disques, le service Sauvegarde tente de les sauvegarder simultanément. La sauvegarde est incrémentielle (seules les données ayant changé sont sauvegardées).
**Planification** |  Pour réduire le trafic de sauvegarde, sauvegardez les machines virtuelles à des moments différents de la journée en veillant à éviter les chevauchements. La sauvegarde simultanée de machines virtuelles provoque des embouteillages.
**Préparation des sauvegardes** | Gardez à l’esprit le temps nécessaire pour préparer la sauvegarde. Le temps de préparation inclut l’installation ou la mise à jour de l’extension de sauvegarde, et le déclenchement d’une capture instantanée en fonction de la planification de sauvegarde.
**Transfert de données** | Tenez compte du temps dont le service Sauvegarde Azure a besoin pour identifier les modifications incrémentielles par rapport à la sauvegarde précédente.<br/><br/> Dans le cadre d’une sauvegarde incrémentielle, le service Sauvegarde Azure détermine les modifications en calculant la somme de contrôle du bloc. Si un bloc est modifié, il est marqué pour transfert vers le coffre. Le service analyse les blocs identifiés pour tenter de réduire encore davantage la quantité de données à transférer. Après avoir évalué tous les blocs modifiés, Sauvegarde Azure transfère les modifications vers le coffre.<br/><br/> Il peut y avoir un décalage entre la prise de l’instantané et sa copie dans le coffre. Aux heures de pointe, le transfert des captures instantanées vers le coffre peut prendre jusqu’à huit heures. La durée de la sauvegarde quotidienne d’une machine virtuelle est inférieure à 24 heures.
**Sauvegarde initiale** | Si le temps de sauvegarde total des sauvegardes incrémentielles est inférieur à 24 heures, cela peut ne pas être le cas pour la première sauvegarde. Le temps nécessaire pour la sauvegarde initiale dépend du volume des données et de l’heure à laquelle la sauvegarde a lieu.
**File d’attente de restauration** | Sauvegarde Azure traite les travaux de restauration de plusieurs comptes de stockage en même temps, et met les demandes de restauration en file d’attente.
**Copie pendant la restauration** | Durant le processus de restauration, les données sont copiées du coffre vers le compte de stockage.<br/><br/> Le temps de restauration total varie selon les opérations d’E/S par seconde (IOPS) et le débit du compte de stockage.<br/><br/> Pour réduire le temps de copie, sélectionnez un compte de stockage non chargé avec d’autres lectures et écritures d’application.

### <a name="backup-performance"></a>Performances de sauvegarde

Les scénarios courants ci-dessous peuvent affecter la durée de sauvegarde totale :

- **Ajout d’un nouveau disque à une machine virtuelle Azure protégée :** si une machine virtuelle est en cours de sauvegarde incrémentielle et qu’un nouveau disque est ajouté, le temps de sauvegarde augmente. La durée totale de sauvegarde peut dépasser 24 heures en raison de la réplication initiale du nouveau disque, ainsi que la réplication delta des disques existants.
- **Disques fragmentés :** les opérations de sauvegarde sont plus rapides quand les changements de disque sont contigus. Si les changements sont dispersés et fragmentés sur un disque, la sauvegarde est plus lente.
- **Évolution du disque :** si l’évolution quotidienne des disques protégés faisant l’objet d’une sauvegarde incrémentielle dépasse les 200 Go, la sauvegarde peut prendre du temps (plus de huit heures).
- **Versions de Sauvegarde :** la dernière version de Sauvegarde, appelée Restauration instantanée, utilise un processus optimisé par rapport à la comparaison de la somme de contrôle pour identifier les changements. En revanche, si vous utilisez la Restauration instantanée et avez supprimé un instantané de sauvegarde, la sauvegarde bascule vers une comparaison de somme de contrôle. Dans ce cas, l’opération de sauvegarde prend plus de 24 heures (ou échoue).

### <a name="restore-performance"></a>Performances de la restauration

Les scénarios courants ci-dessous peuvent affecter la durée totale de la restauration :

- La durée totale de la restauration varie selon les opérations d'entrée/sortie par seconde (IOPS) et le débit du compte de stockage.
- La durée totale de la restauration peut être affectée si le compte de stockage cible est chargé avec d'autres opérations de lecture et d'écriture de l'application. Pour améliorer l'opération de restauration, sélectionnez un compte de stockage qui n'est pas chargé avec d'autres données d'application.

## <a name="best-practices"></a>Meilleures pratiques

Lors de la configuration des sauvegardes de machines virtuelles, nous vous suggérons de suivre les pratiques suivantes :

- Modifiez les heures de planification par défaut définies dans une stratégie. Par exemple, si l’heure par défaut d’une stratégie est minuit, incrémentez-la de quelques minutes pour optimiser l’utilisation des ressources.
- Si vous restaurez des machines virtuelles à partir d’un seul coffre, nous vous recommandons vivement d’utiliser différents [comptes de stockage v2 à usage général](../storage/common/storage-account-upgrade.md) pour faire en sorte que le compte de stockage cible ne soit pas limité. Par exemple, chaque machine virtuelle doit avoir un compte de stockage distinct. Par exemple, si 10 machines virtuelles sont restaurées, utilisez 10 comptes de stockage distincts.
- Pour la sauvegarde de machines virtuelles qui utilisent le stockage Premium, avec la restauration instantanée, nous vous recommandons d’allouer *50 %* d’espace libre de l’espace de stockage total alloué, qui est nécessaire **uniquement** pour la première sauvegarde. L’espace libre de 50 % n’est plus une obligation une fois la première sauvegarde terminée.
- La limite du nombre de disques par compte de stockage dépend de la lourdeur des disques auxquels accèdent les applications s’exécutant sur une machine virtuelle IaaS. En règle générale, si plus de 5 disques sont présents sur un compte de stockage, équilibrez la charge en déplaçant certains disques vers des comptes de stockage distincts.

## <a name="backup-costs"></a>Coûts de sauvegarde

Les machines virtuelles Azure sauvegardées avec Sauvegarde Azure sont soumises aux [tarifs de Sauvegarde Azure](https://azure.microsoft.com/pricing/details/backup/).

La facturation ne commence pas avant la fin de la première sauvegarde réussie. À partir de ce moment, la facturation du stockage et des machines virtuelles protégées commence. La facturation continue tant que des données de sauvegarde sont stockées dans un coffre pour la machine virtuelle. Si vous arrêtez la protection d’une machine virtuelle alors que les données de sauvegarde de cette machine sont présentes dans un coffre, la facturation continue.

La facturation pour une machine virtuelle spécifiée ne s’interrompt que si la protection est levée et que toutes les données de sauvegarde sont supprimées. Quand la protection s’arrête et qu’il n’y a aucun travail de sauvegarde actif, la taille de la dernière sauvegarde de machine virtuelle réussie devient la taille de l’instance protégée utilisée pour la facture mensuelle.

Le calcul de la taille d’instance protégée est basé sur la taille *réelle* de la machine virtuelle. La taille de la machine virtuelle correspond à la somme de toutes les données stockées sur celles-ci, à l’exclusion du stockage temporaire. La tarification est basée sur les données réelles stockées sur les disques de données, pas sur la valeur maximale de taille prise en charge pour chaque disque de données attaché à la machine virtuelle.

De même, la facture du stockage de sauvegarde est basée sur la quantité de données stockées sur le service Sauvegarde Azure, qui est la somme des données réelles contenues dans chaque point de récupération.

Prenons l’exemple d’une machine virtuelle de taille standard A2 avec deux disques de données supplémentaires, d’une taille maximale de 32 To chacun. Le tableau suivant montre les données réelles stockées sur chacun de ces disques :

**Disque** | **Taille maximale** | **Données réelles présentes**
--- | --- | ---
Disque de système d’exploitation | 32 To | 17 Go
Disque local/temporaire | 135 Go | 5 Go (non inclus pour la sauvegarde)
Disque de données 1 | 32 To| 30 Go
Disque de données 2 | 32 To | 0 Go

La taille réelle de la machine virtuelle est dans ce cas 17 Go + 30 Go + 0 Go= 47 Go. Cette taille d’instance protégée (47 Go) devient la base de la facture mensuelle. La taille de l’instance protégée utilisée pour la facturation augmente proportionnellement à la quantité de données dans la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes

- [Préparer une sauvegarde de machine virtuelle Azure](backup-azure-arm-vms-prepare.md).
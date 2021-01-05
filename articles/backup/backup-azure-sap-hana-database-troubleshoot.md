---
title: Résoudre les erreurs de sauvegarde de bases de données SAP HANA
description: Décrit comment résoudre les erreurs courantes qui peuvent survenir lorsque vous utilisez le service Sauvegarde Azure pour sauvegarder des bases de données SAP HANA.
ms.topic: troubleshooting
ms.date: 11/7/2019
ms.openlocfilehash: b9fa73ee38e337a547816432212bc68d419f40bb
ms.sourcegitcommit: 1d366d72357db47feaea20c54004dc4467391364
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95411323"
---
# <a name="troubleshoot-backup-of-sap-hana-databases-on-azure"></a>Résoudre les problèmes de sauvegarde des bases de données SAP HANA sur Azure

Cet article fournit des informations de dépannage pour la sauvegarde des bases de données SAP HANA sur les machines virtuelles Azure. Pour en savoir plus sur les scénarios de sauvegarde de bases de données SAP HANA actuellement pris en charge, consultez [Prise en charge des scénarios](sap-hana-backup-support-matrix.md#scenario-support).

## <a name="prerequisites-and-permissions"></a>Prérequis et autorisations

Reportez-vous aux sections [Conditions préalables](tutorial-backup-sap-hana-db.md#prerequisites) et [Ce que fait le script de préinscription](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does) avant de configurer les sauvegardes.

## <a name="common-user-errors"></a>Erreurs utilisateur courantes

### <a name="usererrorhanainternalrolenotpresent"></a>UserErrorHANAInternalRoleNotPresent

| **Message d’erreur**      | <span style="font-weight:normal">Sauvegarde Azure ne dispose pas des privilèges de rôle nécessaires pour effectuer la sauvegarde</span>    |
| ---------------------- | ------------------------------------------------------------ |
| **Causes possibles**    | Le rôle a peut-être été modifié.                          |
| **Action recommandée** | Pour résoudre le problème, exécutez le script à partir du volet **Discover DB** ou téléchargez-le [ici](https://aka.ms/scriptforpermsonhana). Vous pouvez également ajouter le rôle « SAP_INTERNAL_HANA_SUPPORT » à l’utilisateur de sauvegarde de charge de travail (AZUREWLBACKUPHANAUSER). |

### <a name="usererrorinopeninghanaodbcconnection"></a>UserErrorInOpeningHanaOdbcConnection

| Message d’erreur      | <span style="font-weight:normal">Échec de la connexion au système HANA</span>                        |
| ------------------ | ------------------------------------------------------------ |
| **Causes possibles**    | L’instance SAP HANA est peut-être inactive.<br/>Les autorisations permettant à Sauvegarde Azure d’interagir avec la base de données HANA ne sont pas définies. |
| **Action recommandée** | Vérifiez que la base de données SAP HANA est active. Si la base de données est opérationnelle, vérifiez que toutes les autorisations requises sont définies. Si une autorisation est absente, exécutez le [script de préinscription](https://aka.ms/scriptforpermsonhana) pour l’ajouter. |

### <a name="usererrorhanainstancenameinvalid"></a>UserErrorHanaInstanceNameInvalid

| Message d’erreur      | <span style="font-weight:normal">L’instance SAP HANA spécifiée n’est pas valide ou est introuvable</span>  |
| ------------------ | ------------------------------------------------------------ |
| **Causes possibles**    | Il est impossible de sauvegarder plusieurs instances SAP HANA sur une seule machine virtuelle Azure. |
| **Action recommandée** | Exécutez le [script de préinscription](https://aka.ms/scriptforpermsonhana) sur l’instance SAP HANA que vous voulez sauvegarder. Si le problème persiste, contactez le support technique Microsoft. |

### <a name="usererrorhanaunsupportedoperation"></a>UserErrorHanaUnsupportedOperation

| Message d’erreur      | <span style="font-weight:normal">L’opération SAP HANA spécifiée n’est pas prise en charge</span>              |
| ------------------ | ------------------------------------------------------------ |
| **Causes possibles**    | Sauvegarde Azure pour SAP HANA ne prend pas en charge la sauvegarde incrémentielle et les actions effectuées sur des clients natifs SAP HANA (Studio/ Cockpit/ DBA Cockpit) |
| **Action recommandée** | Pour plus d’informations, consultez [cette page](./sap-hana-backup-support-matrix.md#scenario-support). |

### <a name="usererrorhanapodoesnotsupportbackuptype"></a>UserErrorHANAPODoesNotSupportBackupType

| Message d’erreur      | <span style="font-weight:normal">Cette base de données SAP HANA ne prend pas en charge le type de sauvegarde demandé</span>  |
| ------------------ | ------------------------------------------------------------ |
| **Causes possibles**    | Sauvegarde Azure ne prend pas en charge la sauvegarde incrémentielle et les sauvegardes utilisant des instantanés |
| **Action recommandée** | Pour plus d’informations, consultez [cette page](./sap-hana-backup-support-matrix.md#scenario-support). |

### <a name="usererrorhanalsnvalidationfailure"></a>UserErrorHANALSNValidationFailure

| Message d’erreur      | <span style="font-weight:normal">La séquence de journaux de transactions consécutifs de sauvegarde est rompue</span>                                    |
| ------------------ | ------------------------------------------------------------ |
| **Causes possibles**    | La destination de sauvegarde des journaux a peut-être été modifiée depuis l’opération backint sur le système de fichiers ou le fichier exécutable backint a été modifié |
| **Action recommandée** | Déclenchez une sauvegarde complète pour résoudre le problème                   |

### <a name="usererrorsdctomdcupgradedetected"></a>UserErrorSDCtoMDCUpgradeDetected

| Message d’erreur      | <span style="font-weight:normal">Mise à niveau de SDC vers MDC détectée</span>                                   |
| ------------------ | ------------------------------------------------------------ |
| **Causes possibles**    | L’instance SAP HANA a été mise à niveau de SDC vers MDC. Les sauvegardes échoueront après la mise à jour. |
| **Action recommandée** | Pour résoudre le problème, procédez de la manière décrite dans [Mise à niveau de SDC vers MDC détectée](#sdc-to-mdc-upgrade-with-a-change-in-sid) |

### <a name="usererrorinvalidbackintconfiguration"></a>UserErrorInvalidBackintConfiguration

| Message d’erreur      | <span style="font-weight:normal">Configuration backint non valide détectée</span>                       |
| ------------------ | ------------------------------------------------------------ |
| **Causes possibles**    | Les paramètres de sauvegarde ne sont pas spécifiés correctement pour Sauvegarde Azure |
| **Action recommandée** | Vérifiez que les paramètres suivants (backint) sont définis :<br/>\* [catalog_backup_using_backint:true]<br/>\* [enable_accumulated_catalog_backup:false]<br/>\* [parallel_data_backup_backint_channels:1]<br/>\* [log_backup_timeout_s:900)]<br/>\* [backint_response_timeout:7200]<br/>Si les paramètres basés sur backint sont présents au niveau de l’HÔTE, supprimez-les. Si les paramètres ne sont pas présents au niveau de l’HÔTE, mais ont été modifiés manuellement au niveau de la base de données, rétablissez les valeurs appropriées comme décrit précédemment. Sinon, sur le Portail Microsoft Azure, exécutez [Arrêter la protection et conserver les données de sauvegarde](./sap-hana-db-manage.md#stop-protection-for-an-sap-hana-database), puis sélectionnez **Reprendre la sauvegarde**. |

### <a name="usererrorincompatiblesrctargetsystemsforrestore"></a>UserErrorIncompatibleSrcTargetSystemsForRestore

|Message d’erreur  |Les systèmes source et cible pour la restauration ne sont pas compatibles  |
|---------|---------|
|Causes possibles   | Les systèmes source et cible pour la restauration ne sont pas compatibles        |
|Action recommandée   |   Assurez-vous que votre scénario de restauration ne figure pas dans la liste suivante des restaurations potentiellement incompatibles : <br><br>   **Cas 1 :** SYSTEMDB ne peut pas être renommé lors de la restauration.  <br><br> **Cas 2 :** Source – SDC et Cible – MDC : La base de données source ne peut pas être restaurée en tant que SYSTEMDB ou base de données de locataire sur la cible. <br><br> **Cas 3 :** Source – MDC et Cible – SDC : Impossible de restaurer la base de données source (SYSTEMDB ou base de données de locataire) sur la cible. <br><br>  Pour plus d’informations, consultez la note **1642148** dans [SAP ONE Support Launchpad](https://launchpad.support.sap.com). |

## <a name="restore-checks"></a>Restaurer les vérifications

### <a name="single-container-database-sdc-restore"></a>Restauration d’une base de données à conteneur unique (SDC)

Prenez en charge les entrées lors de la restauration d’une base de données à conteneur unique (SDC) pour HANA sur un autre ordinateur SDC. Le nom de la base de données doit être indiqué en minuscules et « sdc » ajouté entre crochets. L’instance HANA s’affiche en majuscules.

Supposez qu’une instance SDC HANA « H21 » est sauvegardée. La page Éléments de sauvegarde affiche le nom de l’élément de sauvegarde sous la forme **« H21 (SDC) »** . Si vous tentez de restaurer cette base de données sur un autre SDC cible, par exemple H11, les entrées suivantes doivent être fournies.

![Nom de la base de données SDC restaurée](media/backup-azure-sap-hana-database/hana-sdc-restore.png)

Notez les points suivants :

- Par défaut, le nom de la base de données restaurée est renseigné avec le nom de l’élément de sauvegarde. Dans ce cas, il s’agit de h21(sdc).
- La sélection de la cible en tant que H11 ne modifie pas automatiquement le nom de la base de données restaurée. **Elle doit être modifiée en H11 (SDC)** . Concernant SDC, le nom de la base de données restaurée sera l’ID d’instance cible avec des lettres minuscules suivi de « sdc » entre crochets.
- Étant donné que SDC ne peut avoir qu’une seule base de données, vous devez aussi cocher la case pour autoriser le remplacement des données de la base de données existantes par les données du point de récupération.
- Linux respecte la casse. Par conséquent, veillez à la conserver.

### <a name="multiple-container-database-mdc-restore"></a>Restauration de la base de données à conteneurs multiples (MDC)

Dans plusieurs bases de données de conteneur pour HANA, la configuration standard est SYSTEMDB + 1 ou plusieurs bases de données de locataire. La restauration d’une instance de SAP HANA entière signifie qu’il faut restaurer les bases de données SYSTEMDB et de locataire. L’une restaure d’abord SYSTEMDB, puis procède à la restauration de la base de données du locataire. La base de données système signifie essentiellement remplacer les informations système sur la cible sélectionnée. Cette restauration remplace également les informations relatives à BackInt dans l’instance cible. Par conséquent, une fois la base système restaurée vers une instance cible, vous devez réexécuter le script de préinscription. C’est uniquement à ce moment-là que les restaurations ultérieures de la base de données locataire réussiront.

## <a name="back-up-a-replicated-vm"></a>Sauvegarder une machine virtuelle répliquée

### <a name="scenario-1"></a>Scénario 1

La machine virtuelle d’origine a été répliquée à l’aide d’Azure Site Recovery ou d’une Sauvegarde des machines virtuelles Azure. La nouvelle machine virtuelle a été créée pour simuler l’ancienne . Autrement dit, les paramètres sont exactement les mêmes (cela est dû au fait que la machine virtuelle d’origine a été supprimée et que la restauration a été effectuée à partir de la sauvegarde de machine virtuelle ou d’Azure Site Recovery).

Ce scénario peut inclure deux cas. Découvrez comment sauvegarder la machine virtuelle répliquée dans les deux cas ci-dessous :

1. La nouvelle machine virtuelle créée porte le même nom et se trouve dans les mêmes groupe de ressources et abonnement que la machine virtuelle supprimée.

    - L’extension est déjà présente sur la machine virtuelle, mais n’est visible pour aucun des services.
    - Réexécutez le script de préinscription.
    - Réinscrivez l’extension pour la même machine dans le portail Azure (**Sauvegarde** -> **Afficher les détails** > sélectionner la machine virtuelle Azure appropriée -> Réinscrire).
    - La sauvegarde des bases de données sauvegardées (à partir de la machine virtuelle supprimée) doit ensuite démarrer correctement.

2. La nouvelle machine virtuelle créée présente l’une des caractéristiques suivantes :

    - Elle porte un nom différent de celui de la machine virtuelle supprimée.
    - Elle porte le même nom que la machine virtuelle supprimée, mais se trouve dans un groupe de ressources ou un abonnement différents de ceux de la machine virtuelle supprimée.

    Si c’est le cas, procédez comme suit :

    - L’extension est déjà présente sur la machine virtuelle, mais n’est visible pour aucun des services.
    - Exécutez le script de préinscription.
    - Si vous découvrez et protégez les nouvelles bases de données, vous commencez à voir des bases de données actives en double dans le portail. Pour éviter cela, vous devez [Arrêter la protection avec données conservées](sap-hana-db-manage.md#stop-protection-for-an-sap-hana-database) pour les anciennes bases de données. Ensuite, passez aux étapes restantes.
    - Détecter les bases de données pour activer la sauvegarde.
    - Activez les sauvegardes sur ces bases de données.
    - Les bases de données déjà sauvegardées (à partir de la machine virtuelle supprimée) continueront d’être stockées dans le coffre (avec leurs sauvegardes conservées conformément à la stratégie).

### <a name="scenario-2"></a>Scénario 2

La machine virtuelle d’origine a été répliquée à l’aide d’Azure Site Recovery ou d’une sauvegarde de machine virtuelle Azure. La nouvelle machine virtuelle a été créée à partir du contenu, pour être utilisée comme modèle. Il s’agit d’une nouvelle machine virtuelle avec un nouveau SID.

Procédez comme suit pour activer les sauvegardes sur la nouvelle machine virtuelle :

- L’extension est déjà présente sur la machine virtuelle, mais n’est visible pour aucun des services.
- Exécutez le script de préinscription. En fonction du SID de la nouvelle machine virtuelle, deux scénarios peuvent se produire :
  - La machine virtuelle d’origine et la nouvelle machine virtuelle ont le même SID. Le script de pré-inscription s’exécutera correctement.
  - La machine virtuelle d’origine et la nouvelle machine virtuelle ont des SID différents. Le script de pré-inscription échouera. Contactez le support technique pour obtenir de l’aide dans ce scénario.
- Détectez les bases de données que vous souhaitez sauvegarder.
- Activez les sauvegardes sur ces bases de données.

## <a name="sdc-version-upgrade-or-mdc-version-upgrade-on-the-same-vm"></a>Mise à niveau de la version de SDC ou de MDC sur la même machine virtuelle

Les mises à niveau du système d’exploitation et les changements de version de SDC ou de MDC qui n’entraînent pas de modification du SID peuvent être gérées comme suit :

- Assurez-vous que les nouvelles versions du système d’exploitation, de SDC ou de MDC sont actuellement [prises en charge par Sauvegarde Azure](sap-hana-backup-support-matrix.md#scenario-support).
- [Arrêtez la protection avec données conservées](sap-hana-db-manage.md#stop-protection-for-an-sap-hana-database) pour la base de données.
- Effectuez la mise à niveau ou la mise à jour.
- Réexécutez le script de préinscription. Souvent, le processus de mise à niveau peut supprimer [les rôles nécessaires](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does). L’exécution du script de préinscription permet de vérifier tous les rôles requis.
- Réactivez la protection de la base de données.

## <a name="sdc-to-mdc-upgrade-with-no-change-in-sid"></a>Mise à niveau de SDC vers MDC sans modification du SID

Vous pouvez gérer les mises à niveau de SDC vers MDC qui n’entraînent pas de modification du SID comme suit :

- Assurez-vous que la nouvelle version de MDC est actuellement [prise en charge par Sauvegarde Azure](sap-hana-backup-support-matrix.md#scenario-support).
- [Arrêtez la protection avec données conservées](sap-hana-db-manage.md#stop-protection-for-an-sap-hana-database) pour l’ancienne base de données SDC.
- Effectuez la mise à niveau. À l’issue de l’opération, le système HANA est maintenant MDC avec une base de données système et une ou plusieurs bases de données de locataire.
- Réexécuter le [script de préinscription](https://aka.ms/scriptforpermsonhana)
- Réinscrivez l’extension pour la même machine dans le portail Azure (**Sauvegarde** -> **Afficher les détails** > sélectionner la machine virtuelle Azure appropriée -> Réinscrire).
- Sélectionnez **Redécouvrir les bases de données** pour la même machine virtuelle. Cette action a pour effet de faire apparaître les nouvelles bases de données de l’étape 3 en tant que SYSTEMDB et Tenant DB, et non SDC.
- L’ancienne base de données SDC continue d’exister dans le coffre, et les anciennes données sauvegardées sont conservées conformément à la stratégie.
- Configurez la sauvegarde de ces bases de données.

## <a name="sdc-to-mdc-upgrade-with-a-change-in-sid"></a>Mise à niveau de SDC vers MDC avec modification du SID

Vous pouvez gérer les mises à niveau de SDC vers MDC qui entraînent une modification du SID comme suit :

- Assurez-vous que la nouvelle version de MDC est actuellement [prise en charge par Sauvegarde Azure](sap-hana-backup-support-matrix.md#scenario-support).
- **Arrêtez la protection avec données conservées** pour l’ancienne base de données SDC.
- Effectuez la mise à niveau. À l’issue de l’opération, le système HANA est maintenant MDC avec une base de données système et une ou plusieurs bases de données de locataire.
- Réexécutez le [script de préinscription](https://aka.ms/scriptforpermsonhana) avec les détails (nouveaux SID et MDC) corrects. En raison d’une modification du SID, vous risquez de rencontrer des problèmes avec l’exécution du script. Si vous rencontrez des problèmes, contactez le support de Sauvegarde Azure.
- Réinscrivez l’extension pour la même machine dans le portail Azure (**Sauvegarde** -> **Afficher les détails** > sélectionner la machine virtuelle Azure appropriée -> Réinscrire).
- Sélectionnez **Redécouvrir les bases de données** pour la même machine virtuelle. Cette action a pour effet de faire apparaître les nouvelles bases de données de l’étape 3 en tant que SYSTEMDB et Tenant DB, et non SDC.
- L’ancienne base de données SDC continue d’exister dans le coffre, et les anciennes données sauvegardées sont conservées conformément à la stratégie.
- Configurez la sauvegarde de ces bases de données.

## <a name="re-registration-failures"></a>Échecs de réinscription

Vérifiez la présence d’un ou plusieurs des symptômes suivants avant de déclencher l’opération de réinscription :

- Toutes les opérations (comme la sauvegarde, la restauration et la configuration de la sauvegarde) échouent sur la machine virtuelle avec un des codes d’erreur suivants : **WorkloadExtensionNotReachable, UserErrorWorkloadExtensionNotInstalled, WorkloadExtensionNotPresent, WorkloadExtensionDidntDequeueMsg**.
- Si la zone de l’**État de la sauvegarde** de l’élément de sauvegarde affiche **Inaccessible**, excluez toutes les autres causes susceptibles d’entraîner le même état :

  - Absence d’autorisation pour effectuer les opérations liées à la sauvegarde sur la machine virtuelle
  - La machine virtuelle étant arrêtée, les sauvegardes ne peuvent pas se produire
  - Problèmes de réseau

Ces symptômes peuvent survenir pour une ou plusieurs des raisons suivantes :

- Une extension a été supprimée ou désinstallée sur le portail.
- La machine virtuelle a été restaurée à un point dans le temps via une restauration de disques sur place.
- La machine virtuelle a été arrêtée pendant une période prolongée, entraînant l’expiration de la configuration d’extension qui s’y trouvait.
- La machine virtuelle a été supprimée et une autre machine virtuelle a été créée avec le même nom et le même groupe de ressources que la machine virtuelle supprimée.

Dans les scénarios précédents, nous vous recommandons de déclencher une opération de réinscription sur la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes

- Consultez les [questions fréquentes (FAQ)](./sap-hana-faq-backup-azure-vm.md) sur la sauvegarde des bases de données SAP HANA sur des machines virtuelles Azure.

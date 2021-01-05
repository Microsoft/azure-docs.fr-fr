---
title: Tableau de prise en charge de Sauvegarde Azure pour la sauvegarde de SQL Server dans les machines virtuelles Azure
description: Propose un résumé des limitations et des paramètres de prise en charge de la sauvegarde de SQL Server dans les machines virtuelles Azure avec le service Sauvegarde Azure.
ms.topic: conceptual
ms.date: 03/05/2020
ms.custom: references_regions
ms.openlocfilehash: b189eceb6b5a7f2e508387c0b91b238ff5fcb088
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92174061"
---
# <a name="support-matrix-for-sql-server-backup-in-azure-vms"></a>Tableau de prise en charge de la sauvegarde de SQL Server dans les machines virtuelles Azure

Vous pouvez utiliser le service Sauvegarde Azure pour sauvegarder des bases de données SQL Server dans des machines virtuelles Azure hébergées sur la plateforme cloud Microsoft Azure. Cet article résume les limitations et les paramètres généraux de prise en charge des scénarios et des déploiements de sauvegarde de SQL Server dans des machines virtuelles Azure.

## <a name="scenario-support"></a>Prise en charge du scénario

**Support** | **Détails**
--- | ---
**Déploiements pris en charge** | Les machines virtuelles Azure de la Place de marché SQL et les machines virtuelles autres que celles de la Place de marché (sur lesquelles SQL Server est installé manuellement) sont prises en charge.
**Régions prises en charge** | Australie Sud-Est (ASE), Australie Est (AE), Australie Centre (AC), Australie Centre 2 (AC) <br> Brésil Sud (BRS)<br> Canada Centre (CNC), Canada Est (CE)<br> Asie Sud-Est (SEA), Asie Est (EA) <br> USA Est (EUS), USA Est 2 (EUS2), West Central US (WCUS), USA Ouest (WUS), USA Ouest 2 (WUS 2), USA Centre Nord (NCUS), USA Centre (CUS), USA Centre Sud (SCUS) <br> Inde Centre (INC), Inde Sud (INS), Inde Ouest <br> Japon Est (JPE), Japon Ouest (JPW) <br> Corée Centre (KRC), Corée Sud (KRS) <br> Europe Nord (NE), Europe Ouest (WE) <br> Royaume-Uni Sud (UKS), Royaume-Uni Ouest (UKW) <br> US Gov Arizona, US Gov Virginie, US Gov Texas, US DoD Centre, US DoD Est <br> Allemagne Nord, Allemagne Centre-Ouest <br> Suisse Nord, Suisse Ouest <br> France Centre <br> Chine Est, Chine Est 2, Chine Nord, Chine Nord 2
**Systèmes d’exploitation pris en charge** | Windows Server 2019, Windows Server 2016, Windows Server 2012, Windows Server 2008 R2 SP1 <br/><br/> Linux n’est pas actuellement pris en charge.
**Versions de SQL Server prises en charge** | SQL Server 2019, SQL Server 2017 comme indiqué dans la [page Rechercher le cycle de vie des produits](https://support.microsoft.com/lifecycle/search?alpha=SQL%20server%202017), SQL Server 2016 et différents SP comme indiqué dans la [page Rechercher le cycle de vie des produits](https://support.microsoft.com/lifecycle/search?alpha=SQL%20server%202016%20service%20pack), SQL Server 2014, SQL Server 2012, SQL Server 2008 R2, SQL Server 2008 <br/><br/> Enterprise, Standard, Web, Developer, Express.<br><br>Les versions de base de données locales Express ne sont pas prises en charge.
**Versions .NET prises en charge** | .NET Framework 4.5.2 ou ultérieur installé sur la machine virtuelle

## <a name="feature-considerations-and-limitations"></a>Considérations et limitations relatives aux fonctionnalités

|Paramètre  |Limite maximale |
|---------|---------|
|Nombre de bases de données pouvant être protégées sur un serveur (et dans un coffre)    |   2000      |
|Taille de la base de données prise en charge (au-delà, des problèmes de performances peuvent survenir)   |   2 To      |
|Nombre de fichiers pris en charge dans une base de données    |   1 000      |

>[!NOTE]
> [Téléchargez le planificateur de ressources détaillé](https://download.microsoft.com/download/A/B/5/AB5D86F0-DCB7-4DC3-9872-6155C96DE500/SQL%20Server%20in%20Azure%20VM%20Backup%20Scale%20Calculator.xlsx) pour calculer le nombre approximatif de bases de données protégées recommandé par serveur en fonction des ressources de machine virtuelle, de la bande passante et de la stratégie de sauvegarde.

* La sauvegarde SQL Server peut être configurée dans le portail Azure ou **PowerShell**. CLI n’est pas pris en charge.
* La solution est prise en charge pour les deux types de [déploiements](../azure-resource-manager/management/deployment-models.md) : machines virtuelles Azure Resource Manager et machines virtuelles classiques.
* Tous les types de sauvegarde (complète/différentielle/journal) et les modes de récupération (simple/complet/journalisé en bloc) sont pris en charge.
* Les types de sauvegarde Complète et Copie complète sont pris en charge pour les bases de données **en lecture seule**.
* La compression native SQL est prise en charge si elle est explicitement activée par l’utilisateur dans la stratégie de sauvegarde. Sauvegarde Azure remplace les valeurs par défaut au niveau de l’instance par la clause COMPRESSION / NO_COMPRESSION en fonction de la valeur de ce contrôle, tel que défini par l’utilisateur.
* La sauvegarde des base de données compatibles avec TDE est prise en charge. Pour restaurer une base de données chiffrée avec TDE sur un autre serveur SQL Server, vous devez d’abord [restaurer le certificat sur le serveur de destination](/sql/relational-databases/security/encryption/move-a-tde-protected-database-to-another-sql-server). La compression de sauvegarde pour les bases de données compatibles TDE pour SQL Server 2016 et les versions plus récentes est disponible, mais à une taille de transfert inférieure, comme expliqué [ici](https://techcommunity.microsoft.com/t5/sql-server/backup-compression-for-tde-enabled-databases-important-fixes-in/ba-p/385593).
* Les opérations de sauvegarde et de restauration des bases de données miroirs et des instantanés de base de données ne sont pas prises en charge.
* L’**instance de cluster de basculement (FCI)** SQL Server n’est pas prise en charge.
* L’utilisation de plusieurs solutions de sauvegarde pour sauvegarder votre instance SQL Server autonome ou votre groupe de disponibilité SQL AlwaysOn peut entraîner l’échec de la sauvegarde. Évitez de le faire. La sauvegarde de deux nœuds d’un groupe de disponibilité individuellement avec les mêmes solutions ou des solutions différentes peut également entraîner l’échec de la sauvegarde.
* Quand des groupes de disponibilité sont configurés, les sauvegardes sont effectuées à partir des différents nœuds en fonction de plusieurs facteurs. Le comportement des sauvegardes pour un groupe de disponibilité est récapitulé ci-dessous.

### <a name="back-up-behavior-with-always-on-availability-groups"></a>Comportement de sauvegarde avec les groupes de disponibilité Always On

Il est recommandé de configurer la sauvegarde sur un seul nœud d’un groupe de disponibilité. Configurez toujours la sauvegarde dans la même région que le nœud principal. En d’autres termes, le nœud principal doit toujours se trouver dans la région où vous configurez la sauvegarde. Si tous les nœuds du groupe de disponibilité se trouvent dans la région où la sauvegarde est configurée, cela ne pose pas de problème.

#### <a name="for-cross-region-ag"></a>Groupe de disponibilité interrégional

* Quelle que soit la préférence de sauvegarde, les sauvegardes ne s’exécuteront que sur les nœuds qui se trouvent dans la région où la sauvegarde est configurée. Cela est dû au fait que les sauvegardes interrégionales ne sont pas prises en charge. Si vous ne disposez que de deux nœuds et que le nœud secondaire se trouve dans l’autre région, les sauvegardes continuent de s’exécuter sur le nœud principal (sauf si votre préférence de sauvegarde est « Secondaire uniquement »).
* Si un nœud bascule vers une région différente de celle où la sauvegarde est configurée, les sauvegardes échouent sur les nœuds de la région ayant basculé.

En fonction de la préférence de sauvegarde et des types de sauvegardes (complète/différentielle/de fichier journal/copie complète uniquement), les sauvegardes sont effectuées à partir d’un nœud particulier (principal/secondaire).

#### <a name="backup-preference-primary"></a>Préférence de sauvegarde : Principal

**Type de sauvegarde** | **Nœud**
--- | ---
Complète | Principal
Différentielle | Principal
Journal |  Principal
Copie complète uniquement |  Principal

#### <a name="backup-preference-secondary-only"></a>Préférence de sauvegarde : Secondaire uniquement

**Type de sauvegarde** | **Nœud**
--- | ---
Complète | Principal
Différentielle | Principal
Journal |  Secondary
Copie complète uniquement |  Secondary

#### <a name="backup-preference-secondary"></a>Préférence de sauvegarde : Secondary

**Type de sauvegarde** | **Nœud**
--- | ---
Complète | Principal
Différentielle | Principal
Journal |  Secondary
Copie complète uniquement |  Secondary

#### <a name="no-backup-preference"></a>Pas de préférence de sauvegarde

**Type de sauvegarde** | **Nœud**
--- | ---
Complète | Principal
Différentielle | Principal
Journal |  Secondary
Copie complète uniquement |  Secondary

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [sauvegarder une base de données SQL Server](backup-azure-sql-database.md) s'exécutant sur une machine virtuelle Azure.
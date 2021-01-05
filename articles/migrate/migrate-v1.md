---
title: Utiliser la version précédente d’Azure Migrate
description: Décrit comment utiliser la version précédente d’Azure Migrate.
author: ms-psharma
ms.author: panshar
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 11/19/2019
ms.custom: mvc
ms.openlocfilehash: 4fda6c51121838bfa1f3624759b1230d8554d573
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753941"
---
# <a name="work-with-the-previous-version-of-azure-migrate"></a>Utiliser la version précédente d’Azure Migrate

Cet article fournit des informations sur l’utilisation de la version précédente d’Azure Migrate. 


Il existe deux versions du service Azure Migrate :

- **Version actuelle** : utilisez cette version pour créer des projets Azure Migrate, découvrir des machines locales et orchestrer des évaluations et migrations. [Apprenez-en davantage](whats-new.md) sur les nouveautés de cette version.
- **Version précédente** : Si vous utilisez la version précédente d’Azure Migrate (seule l’évaluation des machines virtuelles VMware locales était prise en charge), vous devez désormais utiliser la version actuelle. Les projets de la version précédente sont appelés projets classiques dans cet article. Si vous avez encore besoin d’utiliser des projets Azure Migrate créés dans la version précédente, voici ce que vous pouvez et ne pouvez pas faire :
    - Vous ne pouvez plus créer de projets de migration.
    - Nous vous recommandons de ne pas effectuer de nouvelles découvertes.
    - Vous pouvez toujours accéder aux projets existants.
    - Vous pouvez toujours exécuter des évaluations.
    

## <a name="upgrade-between-versions"></a>Mise à niveau entre versions

Vous ne pouvez pas mettre à niveau des projets ou composants de la version précédente vers la nouvelle version. Vous devez [créer un projet Azure Migrate](create-manage-projects.md) et y [ajouter des outils d’évaluation et de migration](./create-manage-projects.md). Consultez les tutoriels pour comprendre comment utiliser les outils d’évaluation et de migration disponibles. Si un espace de travail Log Analytics est attaché à un projet classique, vous pouvez l’attacher à un projet de la version actuelle après avoir supprimé le projet classique.

## <a name="find-projects-from-previous-version"></a>Rechercher des projets de la version précédente

Recherchez des projets de la version précédente comme suit :

1. Dans le portail Azure, sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure Migrate**. 
2. Le tableau de bord Azure Migrate affiche une notification et un lien permettant d’accéder aux anciens projets Azure Migrate.
3. Cliquez sur le lien pour ouvrir des projets classiques.

## <a name="delete-projects-from-previous-version"></a>Supprimer des projets de la version précédente

Pour rechercher et supprimer des projets de la version précédente, procédez comme suit :

1. Dans le portail Azure, sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure Migrate**. 
2. Le tableau de bord Azure Migrate affiche une notification et un lien permettant d’accéder aux anciens projets Azure Migrate.
3. Cliquez sur le lien pour ouvrir des projets classiques.
4. Sélectionnez le projet à supprimer et supprimez-le. 


## <a name="create-an-assessment"></a>Créer une évaluation

Une fois les machines virtuelles découvertes dans le portail, vous pouvez les regrouper pour créer une évaluation.

- Vous pouvez créer des évaluations locales immédiatement après la découverte des machines virtuelles sur le portail.
- Il est recommandé d’attendre au moins un jour avant de créer des évaluations basées sur les performances afin d’obtenir des recommandations de taille fiables.

Créez une évaluation comme suit :

1. Dans la page **Vue d’ensemble** du projet, cliquez sur **Créer une évaluation**.
2. Cliquez sur **Tout afficher** pour passer en revue les propriétés de l’évaluation.
3. Créez le groupe et spécifiez un nom de groupe.
4. Sélectionnez les machines que vous souhaitez ajouter au groupe.
5. Cliquez sur **Créer une évaluation** pour créer le groupe et l’évaluation.
6. Une fois l’évaluation créée, affichez-la dans **Vue d’ensemble** > **Tableau de bord**.
7. Cliquez sur **Exporter l’évaluation** pour la télécharger sous la forme d’un fichier Excel.

Si vous souhaitez mettre à jour une évaluation existante avec les dernières données de performances, vous pouvez utiliser la commande **Recalculer** sur l’évaluation afin de la mettre à jour.

## <a name="review-an-assessment"></a>Réviser une évaluation 

Une évaluation comporte trois étapes :

- Une évaluation commence par une analyse d’adéquation visant à déterminer si les machines sont compatibles dans Azure.
- Estimations de dimensionnement.
- Estimation des coûts mensuels.

Une machine ne passe au stade suivant que si elle réussit le précédent. Par exemple, si la vérification d’adéquation Azure n’est pas concluante, la machine est marquée comme inadéquate pour Azure, et le dimensionnement et l’évaluation des coûts ne sont pas effectués.


### <a name="review-azure-readiness"></a>Examiner la préparation pour Azure

La vue Préparé pour Azure de l’évaluation indique l’état de préparation de chaque machine virtuelle.

**Préparation** | **State** | **Détails**
--- | --- | ---
Disponible pour Azure | Aucun problème de compatibilité. La machine peut être migrée telle quelle vers Azure. Elle démarrera dans Azure avec une prise en charge complète d’Azure. | Pour les machines virtuelles qui sont prêtes, Azure Migrate recommande une taille de machine virtuelle dans Azure.
Préparé pour Azure sous condition | La machine peut démarrer dans Azure, mais risque de ne pas être totalement prise en charge. Par exemple, une machine avec une version précédente de Windows Server qui n’est pas prise en charge dans Azure. | Azure Migrate explique les problèmes de préparation et fournit des étapes de correction.
Non préparé pour Azure |  La machine virtuelle ne démarrera pas dans Azure. Par exemple, si une machine virtuelle dispose d’un disque d’une capacité supérieure à 4 To, elle ne peut pas être hébergée sur Azure. | Azure Migrate explique les problèmes de préparation et fournit des étapes de correction.
Préparation inconnue | Azure Migrate ne peut pas identifier l’état de préparation pour Azure, généralement parce qu’il n’y pas de données disponibles.


#### <a name="azure-vm-properties"></a>Propriétés de machine virtuelle Azure
La préparation prend en compte un certain nombre de propriétés de machine virtuelle pour déterminer si la machine virtuelle peut s’exécuter dans Azure.


**Propriété** | **Détails** | **Préparation**
--- | --- | ---
**Type de démarrage** | BIOS pris en charge. UEFI non prise en charge. | Préparé sous condition si le type de démarrage est UEFI.
**Cœurs** | Cœurs de la machine <= nombre maximal de cœurs (128) pris en charge pour une machine virtuelle Azure.<br/><br/> Si l’historique des performances est disponible, Azure Migrate prend en considération les cœurs utilisés.<br/>Si un facteur de confort est spécifié dans les paramètres de l’évaluation, le nombre de cœurs utilisés est multiplié par le facteur de confort.<br/><br/> En l’absence d’historique des performances, Azure Migrate utilise les cœurs alloués, sans appliquer le facteur de confort. | Prêt si inférieur ou égal aux limites.
**Mémoire** | Taille de la mémoire de la machine <= mémoire maximale (3 892 Go sur la série Azure M Standard_M128m&nbsp;<sup>2</sup>) d’une machine virtuelle Azure. [Plus d’informations](../virtual-machines/sizes.md)<br/><br/> Si l’historique des performances est disponible, Azure Migrate prend en considération la mémoire utilisée.<br/><br/>Si un facteur de confort est spécifié, la mémoire utilisée est multipliée par le facteur de confort.<br/><br/> En l’absence d’historique, la mémoire allouée est utilisée sans appliquer le facteur de confort.<br/><br/> | Prêt si dans les limites.
**Disque de stockage** | La taille allouée d’un disque doit être inférieure ou égale à 4 To (4 096 Go).<br/><br/> Le nombre de disques attachés à la machine doit être inférieur ou égal à 65, disque du système d’exploitation compris. | Prêt si dans les limites.
**Mise en réseau** | Au maximum 32 cartes réseau doivent être attachées à une machine. | Prêt si dans les limites.

#### <a name="guest-operating-system"></a>Système d’exploitation invité

Outre les propriétés de la machine virtuelle, Azure Migrate examine le système d’exploitation invité de la machine virtuelle locale afin de déterminer si celle-ci peut s’exécuter dans Azure.

- Azure Migrate prend en considération le système d’exploitation spécifié dans vCenter Server.
- La détection effectuée par Azure Migrate étant basée sur l’appareil, il n’a aucun moyen de vérifier si le système d’exploitation en cours d’exécution sur la machine virtuelle est identique à celui spécifié dans vCenter Server.

La logique utilisée est la suivante.

**Système d’exploitation** | **Détails** | **Préparation**
--- | --- | ---
Windows Server 2016 avec tous les Service Packs | Azure assure un support complet. | Disponible pour Azure
Windows Server 2012 R2 et tous les Service Packs | Azure assure un support complet. | Disponible pour Azure
Windows Server 2012 et tous les Service Packs | Azure assure un support complet. | Disponible pour Azure
Windows Server 2008 R2 et tous les Service Packs | Azure assure un support complet.| Disponible pour Azure
Windows Server 2008 (32 bits et 64 bits) | Azure assure un support complet. | Disponible pour Azure
Windows Server 2003, 2003 R2 | Non pris en charge. La prise en charge dans Azure nécessite un contrat de support personnalisé ([Custom Support Agreement (CSA)](/troubleshoot/azure/virtual-machines/server-software-support)). | Préparé pour Azure sous condition. Envisagez une mise à niveau du système d’exploitation avant de migrer vers Azure.
Windows 2000, 98, 95, NT, 3.1, MS-DOS | Non pris en charge. La machine pourrait démarrer dans Azure, mais aucune prise en charge du système d’exploitation n’est fournie par Azure. | Préparé pour Azure sous condition. Nous vous recommandons d’effectuer une mise à niveau du système d’exploitation avant de migrer vers Azure.
Clients Windows 7, 8 et 10 | Azure assure un support avec [abonnement Visual Studio uniquement](../virtual-machines/windows/client-images.md). | Préparé pour Azure sous condition
Windows 10 Pro Desktop | Azure fournit la prise en charge avec les [droits d’hébergement multilocataire](../virtual-machines/windows/windows-desktop-multitenant-hosting-deployment.md). | Préparé pour Azure sous condition
Windows Vista, XP Professionnel | Non pris en charge. La machine pourrait démarrer dans Azure, mais aucune prise en charge du système d’exploitation n’est fournie par Azure. | Préparé pour Azure sous condition. Nous vous recommandons d’effectuer une mise à niveau du système d’exploitation avant de migrer vers Azure.
Linux | Azure approuve ces [systèmes d’exploitation Linux](../virtual-machines/linux/endorsed-distros.md). D’autres systèmes d’exploitation Linux pourraient démarrer dans Azure, mais nous recommandons d’effectuer une mise à niveau du système d’exploitation vers une version approuvée avant de migrer vers Azure. | Disponible pour Azure si la version est approuvée.<br/><br/>Préparé pour Azure sous condition si la version n’est pas approuvée.
Autres systèmes d’exploitation<br/><br/> Par exemple, Oracle Solaris, Apple macOS, FreeBSD, etc. | Azure n’approuve pas ces systèmes d’exploitation. La machine peut démarrer dans Azure, mais aucune prise en charge du système d’exploitation n’est fournie par Azure. | Préparé pour Azure sous condition. Nous vous recommandons d’installer un système d’exploitation pris en charge avant de migrer vers Azure.  
Système d’exploitation spécifié comme **Autre** dans vCenter Server | Azure Migrate ne peut pas identifier le système d’exploitation dans ce cas. | État de la préparation inconnu. Vérifiez que le système d’exploitation en cours d’exécution sur la machine virtuelle est pris en charge dans Azure.
Systèmes d’exploitation 32 bits | La machine peut démarrer dans Azure, mais il est possible qu’Azure ne fournisse pas une prise en charge complète. | Préparé pour Azure sous condition. Envisagez de mettre à niveau le système d’exploitation 32 bits de la machine vers un système d’exploitation 64 bits avant de migrer vers Azure.


### <a name="review-sizing"></a>Réviser le dimensionnement

 La recommandation de taille effectuée par Azure Migrate varie selon le critère de dimensionnement spécifié dans les propriétés de l’évaluation.

- Si le dimensionnement est basé sur les performances, la recommandation de taille tient compte de l’historique des performances des machines virtuelles (UC et mémoire) et des disques (IOPS et débit).
- Si le critère de dimensionnement est « Localement », la recommandation dans Azure est basée la taille de la machine virtuelle locale. Le dimensionnement du disque est basé sur le type de stockage spécifié dans les propriétés de l’évaluation (disques Premium par défaut). Azure Migrate ne tient pas compte des données de performances de la machine virtuelle et des disques.

### <a name="review-cost-estimates"></a>Réviser les estimations de coûts

Les estimations de coûts affichent le coût total du calcul et du stockage résultant de l’exécution des machines virtuelles dans Azure avec des détails pour chaque machine.

- Les estimations de coûts sont calculées à l’aide de la recommandation de taille pour une machine virtuelle et ses disques, ainsi que des propriétés de l’évaluation.
- L’estimation des coûts mensuels de calcul et de stockage est agrégée pour toutes les machines virtuelles dans le groupe.
- L’estimation de coût s’applique à l’exécution de la machine virtuelle locale en tant que machine virtuelle IaaS Azure. Azure Migrate ne tient pas compte des coûts PaaS ou SaaS.

### <a name="review-confidence-rating-performance-based-assessment"></a>Réviser la note de confiance (évaluation basée sur les performances)

Chaque évaluation basée sur les performances est associée à une note de confiance.

- Une évaluation de la certitude attribue un score de une à cinq étoiles (de la note la plus basse à la plus élevée).
- La note de confiance est attribue à une évaluation en fonction de la disponibilité des points de données nécessaires pour calculer l’évaluation.
- Le niveau de confiance d’une évaluation permet d’évaluer la fiabilité des recommandations de taille fournies par Azure Migrate.
- Une note de confiance n’est pas disponible pour les évaluations locales « en l’état ».

Pour le dimensionnement basé sur les performances, Azure Migrate a besoin des informations suivantes :
- Données sur l’utilisation de l’UC.
- Données sur la mémoire de la machine virtuelle.
- Pour chaque disque attaché à la machine virtuelle, les données d’E/S par seconde du disque et de débit sont nécessaires.
- Pour chaque carte réseau attachée à une machine virtuelle, Azure Migrate a besoin des données d’entrées/sorties du réseau.
- Si l’une des informations ci-dessus n’est pas disponible, les recommandations de taille (et donc les notes de confiance) risquent de ne pas être fiables.


Les notes de confiance possibles sont résumées dans le tableau en fonction du pourcentage de points de données disponibles.

**Disponibilité des points de données** | **Niveau de confiance**
--- | ---
0 %-20 % | 1 étoile
21 %-40 % | 2 étoiles
41 %-60 % | 3 étoiles
61 %-80 % | 4 étoiles
81 %-100 % | 5 étoiles


#### <a name="assessment-issues-affecting-confidence-ratings"></a>Problèmes d’évaluation affectant les notes de confiance

Il peut arriver qu’une évaluation ne dispose pas de tous les points de données pour diverses raisons :

- Vous n’avez pas profilé votre environnement pour la durée de l’évaluation. Par exemple, si vous créez l’évaluation d’une durée d’un jour, vous devez attendre au moins un jour après le début de la découverte que tous les points de données aient été collectés.
- Certaines machines virtuelles ont été arrêtées pendant la période de calcul de l’évaluation. Si des machines virtuelles ont été arrêtées pendant une partie de la durée d’évaluation, Azure Migrate ne peut pas collecter de données de performances pour cette période.
- Certaines machines virtuelles ont été créées pendant la période de calcul de l’évaluation. Par exemple, si vous créez une évaluation utilisant l’historique des performances du mois précédent, mais avez créé un certain nombre de machines virtuelles dans l’environnement il y a semaine, l’historique des performances des nouvelles machines virtuelles ne reflète pas toute la durée.

> [!NOTE]
> Si la note de confiance d’une évaluation est inférieure à 5 étoiles, attendez au moins un jour que l’appliance profile l’environnement, puis recalculez l’évaluation. À défaut, il se peut que le dimensionnement basé sur les performances ne soit pas fiable. Si vous ne souhaitez pas recalculer, nous vous recommandons de passer à un dimensionnement local en modifiant les propriétés de l’évaluation.



## <a name="create-groups-using-dependency-visualization"></a>Créer des groupes à l’aide de la visualisation des dépendances

En plus de créer des groupes manuellement, vous pouvez créer des groupes à l’aide de la visualisation des dépendances.
- Cette méthode est généralement utilisée pour évaluer des groupes avec des niveaux supérieurs de confiance en vérifiant par recoupement les dépendances de machine avant d’exécuter une évaluation.
- La visualisation des dépendances peut vous aider à planifier efficacement votre migration vers Azure. Elle vous permet de ne rien oublier et vous épargne les pannes inopinées pendant la migration vers Azure.
- Vous pouvez découvrir tous les systèmes interdépendants qui doivent migrer en même temps et déterminer si un système en cours d’exécution continue de servir les utilisateurs ou si une mise hors service peut être envisagée au lieu de la migration.
- Azure Migrate utilise la solution Service Map dans Azure Monitor pour activer la visualisation des dépendances.

> [!NOTE]
> La visualisation des dépendances n’est pas disponible dans Azure Government.

Pour configurer la visualisation des dépendances, vous associez un espace de travail Log Analytics à un projet Azure Migrate, installez des agents sur les machines dont vous souhaitez visualiser les dépendances, puis créez des groupes en utilisant les informations sur les dépendances. 



### <a name="associate-a-log-analytics-workspace"></a>Associer un espace de travail Log Analytics

Pour utiliser la visualisation des dépendances, vous associez un espace de travail Log Analytics à un projet de migration. Vous ne pouvez créer ou attacher un espace de travail que dans l’abonnement où le projet de migration est créé.

1. Pour attacher un espace de travail Log Analytics à un projet, dans **Vue d’ensemble**, accédez à **Essentials**, puis cliquez sur **Requiert une configuration**.
2. Vous pouvez créer un espace de travail ou attacher un espace de travail existant :
  - Pour créer un espace de travail, spécifiez un nom. L’espace de travail est créé dans une région appartenant à la même [zone géographique Azure](https://azure.microsoft.com/global-infrastructure/geographies/) que le projet de migration.
  - Quand vous attachez un espace de travail existant, vous pouvez choisir parmi tous les espaces de travail disponibles dans le même abonnement que le projet de migration. Seuls les espaces de travail créés dans une région [Service Map prise en charge](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions) sont répertoriés. Pour attacher un espace de travail, assurez-vous d’avoir accès en lecture à l’espace de travail.

> [!NOTE]
> Vous ne pouvez pas changer l’espace de travail associé à un projet de migration.

### <a name="download-and-install-vm-agents"></a>Télécharger et installer des agents de machine virtuelle

Après avoir configuré un espace de travail, vous devez télécharger et installer des agents sur chaque machine locale à évaluer. En outre, si certaines de vos machines sont dépourvues de connexion Internet, vous devez télécharger et installer la [passerelle Log Analytics](../azure-monitor/platform/gateway.md) sur ces machines.

1. Dans **Vue d’ensemble**, cliquez sur **Gérer** > **Machines** et sélectionnez la machine souhaitée.
2. Dans la colonne **Dépendances**, cliquez sur **Installer des agents**.
3. Sur la page **Dépendances**, téléchargez et installez Microsoft Monitoring Agent (MMA) et l’agent de dépendances sur chacune des machines virtuelles à évaluer.
4. Copiez l’ID et la clé de l’espace de travail. Vous en aurez besoin lorsque vous installerez MMA sur la machine locale.

> [!NOTE]
> Pour automatiser l’installation des agents, vous pouvez utiliser un outil de déploiement, comme Configuration Manager, ou un outil partenaire, comme [Intigua](https://www.intigua.com/intigua-for-azure-migration), qui offre une solution de déploiement d’agent pour Azure Migrate.


#### <a name="install-the-mma-agent-on-a-windows-machine"></a>Installer l’agent MMA sur une machine Windows

Pour installer l’agent sur une machine Windows :

1. Double-cliquez sur l’agent téléchargé.
2. Sur la page d’**accueil**, cliquez sur **Suivant**. Sur la page **Termes du contrat de licence**, cliquez sur **J’accepte** pour accepter la licence.
3. Dans **Dossier de destination**, conservez ou modifiez le dossier d’installation par défaut > **Suivant**.
4. Dans **Options d’installation de l’agent**, sélectionnez **Azure Log Analytics** > **Suivant**.
5. Cliquez sur **Ajouter** pour ajouter un nouvel espace de travail Log Analytics. Collez l’ID et la clé de l’espace de travail que vous avez copiés sur le portail. Cliquez sur **Suivant**.

Vous pouvez installer l’agent à partir de la ligne de commande ou en utilisant une méthode automatisée comme Configuration Manager. [En savoir plus](../azure-monitor/platform/log-analytics-agent.md#installation-options) sur l’utilisation de ces méthodes pour installer l’agent MMA.

#### <a name="install-the-mma-agent-on-a-linux-machine"></a>Installer l’agent MMA sur une machine Linux

Pour installer l’agent sur une machine Linux :

1. Transférez le groupe approprié (x86 ou x64) sur votre ordinateur Linux à l’aide de scp/sftp.
2. Installez le bundle avec l’argument --install.

    ```sudo sh ./omsagent-<version>.universal.x64.sh --install -w <workspace id> -s <workspace key>```

[Découvrez plus en détail](../azure-monitor/platform/agents-overview.md#supported-operating-systems) la liste des systèmes d’exploitation Linux pris en charge par MMA.

### <a name="install-the-mma-agent-on-a-machine-monitored-by-operations-manager"></a>Installer l’agent sur une machine surveillée par Operations Manager

Pour les machines surveillées par System Center Operations Manager 2012 R2 ou version ultérieure, il n'est pas nécessaire d'installer l'agent MMA. Service Map s’intègre avec l’agent MMA Operations Manager pour recueillir les données de dépendance nécessaires. [Plus d’informations](../azure-monitor/insights/service-map-scom.md#prerequisites) L’agent de dépendances doit être installé.

### <a name="install-the-dependency-agent"></a>Installer l’agent de dépendances

1. Pour installer l’agent de dépendances sur une machine Windows, double-cliquez sur le fichier d’installation et suivez l’Assistant.
2. Pour installer l’agent de dépendances sur une machine Linux en tant que racine, exécutez la commande suivante :

    ```sh InstallDependencyAgent-Linux64.bin```

- Apprenez-en davantage sur la [prise en charge de l’agent de dépendances](../azure-monitor/insights/vminsights-enable-overview.md#supported-operating-systems) pour les systèmes d’exploitation Windows et Linux.
- [En savoir plus](../azure-monitor/insights/vminsights-enable-hybrid.md#dependency-agent) sur la façon dont vous pouvez utiliser des scripts pour installer l’agent de dépendances.

>[!NOTE]
> L’article sur Azure Monitor pour machines virtuelles mentionné qui fournit une vue d’ensemble des exigences système et des méthodes de déploiement de l’agent de dépendances est également applicable à la solution Service Map.

### <a name="create-a-group-with-dependency-mapping"></a>Créer un groupe avec un mappage des dépendances

1. Après avoir installé les agents, accédez au portail et cliquez sur **Gérer** > **Machines**.
2. Recherchez la machine sur laquelle vous avez installé les agents.
3. La colonne **Dépendances** pour la machine doit maintenant apparaître comme **Afficher les dépendances**. Cliquez sur la colonne pour afficher les dépendances de la machine.
4. Le mappage de dépendances de la machine affiche les informations suivantes :
    - Connexions TCP entrantes (clients) et sortantes (serveurs) vers/depuis la machine
        - Les machines dépendantes sur lesquelles ne sont pas installés l’agent MMA et l’agent de dépendances sont regroupées par numéros de port.
        - Les machines dépendantes sur lesquelles sont installés l’agent MMA et l’agent de dépendances apparaissent sous forme de zones distinctes.
    - Processus en cours d’exécution dans la machine (vous pouvez développer chaque zone de machine pour afficher les processus correspondants)
    - Les propriétés de la machine, à savoir son nom de domaine complet, son système d’exploitation et son adresse MAC, sont affichées. Vous pouvez cliquer sur chaque machine pour en afficher les détails.

4. Vous pouvez examiner les dépendances pour différentes durées en cliquant sur la durée dans l’étiquette de l’intervalle de temps. Par défaut, il est fixé à une heure. Vous pouvez le modifier ou spécifier une date de début, une date de fin et une durée.

   > [!NOTE]
   >    L’intervalle de temps maximal pris en charge est de une heure. Utilisez les journaux Azure Monitor pour [interroger les données de dépendance](./how-to-create-group-machine-dependencies.md) sur une plus longue durée.

5. Quand vous avez identifié des machines dépendantes à grouper, utilisez la commande Ctrl+clic pour les sélectionner sur le mappage, puis cliquez sur **Grouper les machines**.
6. Spécifiez un nom de groupe. Vérifiez que les machines dépendantes sont découvertes par Azure Migrate.

    > [!NOTE]
    > Si une machine dépendante n’est pas découverte par Azure Migrate, vous ne pouvez pas l’ajouter au groupe. Pour ajouter ces machines au groupe, vous devez réexécuter le processus de découverte avec l’étendue adéquate dans vCenter Server et vérifier que les machines sont découvertes par Azure Migrate.  

7. Si vous souhaitez créer une évaluation pour ce groupe, cochez la case prévue à cet effet.
8. Cliquez sur **OK** pour enregistrer le groupe.

Une fois le groupe créé, nous vous recommandons d’installer les agents sur toutes les machines du groupe et d’affiner le groupe en visualisant la dépendance de l’ensemble du groupe.

## <a name="query-dependency-data-from-azure-monitor-logs"></a>Interroger les données de dépendance à partir des journaux d'activité Azure Monitor

Les données de dépendance capturées par Service Map peuvent être interrogées dans l'espace de travail Log Analytics associé à votre projet Azure Migrate. [Découvrez-en plus](../azure-monitor/insights/service-map.md#log-analytics-records) sur les tables de données Service Map à interroger dans les journaux d'activité Azure Monitor. 

Pour exécuter des requêtes Kusto :

1. Après avoir installé les agents, accédez au portail et cliquez sur **Vue d’ensemble**.
2. Dans **Vue d’ensemble**, accédez à la section **Essentials** du projet et cliquez sur le nom de l’espace de travail fourni en regard de **Espace de travail OMS**.
3. Sur la page de l’espace de travail Log Analytics, cliquez sur **Général** > **Journaux d’activité**.
4. Rédigez votre requête pour recueillir des données de dépendance à l'aide des journaux d'activité Azure Monitor. Accédez à des exemples de requêtes dans la section suivante.
5. Exécutez votre requête en cliquant sur Exécuter. 

[Découvrez-en plus](../azure-monitor/log-query/log-analytics-tutorial.md) sur la rédaction de requêtes Kusto. 

### <a name="sample-azure-monitor-logs-queries"></a>Exemples d'interrogation des journaux d'activité Azure Monitor

Voici des exemples de requêtes que vous pouvez utiliser pour extraire des données de dépendance. Vous pouvez modifier les requêtes pour extraire les points de données de votre choix. Une liste exhaustive des champs des enregistrements de données de dépendance est disponible [ici](../azure-monitor/insights/service-map.md#log-analytics-records). Accédez à d'autres exemples de requêtes [ici](../azure-monitor/insights/service-map.md#sample-log-searches).

#### <a name="summarize-inbound-connections-on-a-set-of-machines"></a>Résumer les connexions entrantes sur un ensemble de machines

Les enregistrements de la table des métriques de connexion, VMConnection, ne représentent pas des connexions réseau physiques individuelles. Différentes connexions réseau physiques sont regroupées au sein d'une connexion logique. [Découvrez-en plus](../azure-monitor/insights/service-map.md#connections) sur la manière dont les données de connexion réseau physique sont agrégées au sein d'un même enregistrement logique dans VMConnection. 

```
// the machines of interest
let ips=materialize(ServiceMapComputer_CL
| summarize ips=makeset(todynamic(Ipv4Addresses_s)) by MonitoredMachine=ResourceName_s
| mvexpand ips to typeof(string));
let StartDateTime = datetime(2019-03-25T00:00:00Z);
let EndDateTime = datetime(2019-03-30T01:00:00Z); 
VMConnection
| where Direction == 'inbound' 
| where TimeGenerated > StartDateTime and TimeGenerated  < EndDateTime
| join kind=inner (ips) on $left.DestinationIp == $right.ips
| summarize sum(LinksEstablished) by Computer, Direction, SourceIp, DestinationIp, DestinationPort
```

#### <a name="summarize-volume-of-data-sent-and-received-on-inbound-connections-between-a-set-of-machines"></a>Résumer le volume de données envoyées et reçues sur les connexions entrantes entre un ensemble de machines

```
// the machines of interest
let ips=materialize(ServiceMapComputer_CL
| summarize ips=makeset(todynamic(Ipv4Addresses_s)) by MonitoredMachine=ResourceName_s
| mvexpand ips to typeof(string));
let StartDateTime = datetime(2019-03-25T00:00:00Z);
let EndDateTime = datetime(2019-03-30T01:00:00Z); 
VMConnection
| where Direction == 'inbound' 
| where TimeGenerated > StartDateTime and TimeGenerated  < EndDateTime
| join kind=inner (ips) on $left.DestinationIp == $right.ips
| summarize sum(BytesSent), sum(BytesReceived) by Computer, Direction, SourceIp, DestinationIp, DestinationPort
```


## <a name="next-steps"></a>Étapes suivantes
[Découvrez](migrate-services-overview.md) la dernière version d’Azure Migrate.
---
title: Sauvegarder des machines virtuelles Hyper-V à l’aide de MABS
description: Cet article contient les procédures de sauvegarde et de récupération des machines virtuelles à l’aide du serveur de sauvegarde Microsoft Azure (MABS).
ms.topic: conceptual
ms.date: 07/18/2019
ms.openlocfilehash: fc4e34e11e2474521082b1c23f600e9a5ca7a9fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89377996"
---
# <a name="back-up-hyper-v-virtual-machines-with-azure-backup-server"></a>Sauvegarder des machines virtuelles Hyper-V avec le serveur de sauvegarde Azure

Cet article explique comment sauvegarder des machines virtuelles Hyper-V à l’aide du serveur de sauvegarde Microsoft Azure (MABS).

## <a name="supported-scenarios"></a>Scénarios pris en charge

MABS peut sauvegarder des machines virtuelles exécutées sur des serveurs hôtes Hyper-V dans les scénarios suivants :

- **Machines virtuelles avec stockage local ou direct** : sauvegardez des machines virtuelles hébergées sur des serveurs autonomes hôtes Hyper-V disposant d’un stockage en attachement direct ou local. Par exemple : un disque dur, un appareil de réseau SAN (Storage Area Network) ou un dispositif de stockage NAS (Network Attached Storage). L’agent de protection du serveur MABS doit être installé sur tous les hôtes.

- **Machines virtuelles dans un cluster avec stockage CSV** : sauvegardez des machines virtuelles hébergées sur un cluster Hyper-V avec un stockage sur un volume partagé de cluster (CSV). L’agent de protection du serveur MABS est installé sur chaque nœud du cluster.

## <a name="host-versus-guest-backup"></a>Sauvegarde des hôtes et des invités

Le serveur MABS peut effectuer une sauvegarde de machines virtuelles Hyper-V au niveau de l’hôte ou de l’invité. Au niveau de l’hôte, l’agent de protection du serveur MABS est installé sur le cluster ou le serveur hôte Hyper-V. Il protège l’ensemble des machines virtuelles et les fichiers de données en cours d’exécution sur cet hôte.   Au niveau de l’invité, l’agent est installé sur chaque machine virtuelle. Il protège la charge de travail présente sur cette machine.

Ces deux méthodes présentent des avantages et des inconvénients :

- Les sauvegardes au niveau de l’hôte sont flexibles, car elles fonctionnent indépendamment du type de système d’exploitation exécuté sur les ordinateurs invités et ne nécessitent pas l’installation de l’agent de protection MABS sur chaque machine virtuelle. Si vous déployez une sauvegarde au niveau de l’hôte, vous récupérez un ordinateur virtuel dans son ensemble, ou encore des fichiers et dossiers (récupération au niveau de l’élément).

- La sauvegarde au niveau de l’invité se révèle utile pour protéger des charges de travail spécifiques en cours d’exécution sur une machine virtuelle. Au niveau de l’hôte, vous pouvez récupérer une machine virtuelle dans son ensemble ou des fichiers spécifiques. Toutefois, cette récupération n’est pas assurée dans le contexte d’une application spécifique. Par exemple, pour récupérer des éléments SharePoint spécifiques à partir d’une machine virtuelle sauvegardée, vous devez effectuer une sauvegarde de cette machine virtuelle au niveau de l’invité. Utilisez la sauvegarde au niveau de l’invité pour protéger les données stockées sur des disques directs. Le transfert direct permet à la machine virtuelle d’accéder directement au dispositif de stockage. Il ne stocke pas les données de volume virtuel dans un fichier de disque dur virtuel.

## <a name="how-the-backup-process-works"></a>Fonctionnement du processus de sauvegarde

Le serveur MABS effectue la sauvegarde avec VSS comme suit. Les étapes de cette description sont numérotées pour plus de clarté.

1. Le moteur de synchronisation basé sur des blocs du serveur MABS effectue une copie initiale de la machine virtuelle protégée et s’assure que la copie de la machine virtuelle est complète et cohérente.

2. Une fois la copie initiale effectuée et vérifiée, MABS utilise l’enregistreur VSS Hyper-V pour capturer les sauvegardes. L’enregistreur VSS fournit un ensemble de blocs de disques cohérents au niveau des données qui sont synchronisés avec le serveur MABS. Cette approche offre l’avantage d’une « sauvegarde complète » avec le serveur MABS, tout en minimisant les données de sauvegarde qui doivent être transférées sur le réseau.

3. L’agent de protection MABS sur un serveur exécutant Hyper-V utilise les API Hyper-V existantes pour déterminer si une machine virtuelle protégée prend également en charge VSS.

   - Si une machine virtuelle est conforme aux conditions requises pour la sauvegarde en ligne et que le composant des services d’intégration Hyper-V est installé, l’enregistreur VSS Hyper-V transfère de manière récursive la requête VSS à tous les processus VSS sur la machine virtuelle. Cette opération se produit sans l’installation de l’agent de protection MABS sur la machine virtuelle. La requête VSS récursive permet à l’enregistreur VSS Hyper-V de s’assurer que les opérations d’écriture sur le disque sont synchronisées afin qu’un instantané VSS soit capturé sans perte de données.

     Le composant des services d’intégration Hyper-V appelle l’enregistreur VSS Hyper-V dans les services VSS (Volume Shadow Copy Services) sur les machines virtuelles pour s’assurer que leurs données d’application sont dans un état cohérent.

   - Si la machine virtuelle n’est pas conforme aux exigences de sauvegarde en ligne, MABS utilise automatiquement les API Hyper-V pour suspendre la machine virtuelle avant de capturer les fichiers de données.

4. Après la synchronisation de la copie de base de référence initiale de la machine virtuelle avec le serveur MABS, toutes les modifications apportées aux ressources de la machine virtuelle sont capturées dans un nouveau point de récupération. Le point de récupération représente l’état cohérent de la machine virtuelle à un moment donné. Les captures de points de récupération peuvent se produire au moins une fois par jour. Quand un point de récupération est créé, MABS utilise la réplication au niveau du bloc conjointement avec l’enregistreur VSS Hyper-V pour déterminer les blocs qui ont été modifiés sur le serveur qui exécute Hyper-V après la création du dernier point de récupération. Ces blocs de données sont ensuite transférés vers le serveur MABS et sont appliqués au réplica des données protégées.

5. Le serveur MABS utilise VSS sur les volumes qui hébergent les données de récupération hôtes, afin que plusieurs clichés instantanés soient disponibles. Chacun de ces clichés instantanés fournit une récupération distincte. Les points de récupération VSS sont stockés sur le serveur MABS. La copie temporaire effectuée sur le serveur exécutant Hyper-V est stockée uniquement pour la durée de la synchronisation MABS.

>[!NOTE]
>
>À compter de Windows Server 2016, les disques durs virtuels Hyper-V disposent d’un suivi des modifications intégré, connu sous le nom de « suivi de modifications durables » ou RCT. MABS utilise la fonctionnalité RCT (suivi des modifications natif d’Hyper-V), qui limite le nombre de longues vérifications de cohérence nécessaires dans des scénarios comme des plantages sur une machine virtuelle. RCT offre une meilleure résilience que le suivi des modifications proposé par les sauvegardes sur instantané VSS. MABS v3 optimise encore davantage la consommation de réseau et de stockage en transférant seulement les données modifiées pendant les vérifications de cohérence.

## <a name="backup-prerequisites"></a>Prérequis pour la sauvegarde

Voici les prérequis pour la sauvegarde des machines virtuelles Hyper-V avec MABS :

|Configuration requise|Détails|
|------------|-------|
|Prérequis pour MABS|- Si vous souhaitez effectuer une récupération au niveau de l’élément pour les machines virtuelles (récupérer des fichiers, des dossiers, des volumes), vous devez installer le rôle Hyper-V sur le serveur MABS.  Si vous souhaitez uniquement récupérer la machine virtuelle et non effectuer une récupération au niveau de l’élément, le rôle n’est pas obligatoire.<br />- Vous pouvez protéger jusqu’à 800 machines virtuelles de 100 Go chacune sur un serveur MABS et autoriser plusieurs serveurs MABS prenant en charge de plus grands clusters.<br />- MABS exclut le fichier d’échange des sauvegardes incrémentielles pour améliorer les performances de sauvegarde de la machine virtuelle.<br />- MABS peut sauvegarder un cluster ou un serveur Hyper-V dans le même domaine que le serveur MABS, ou dans un domaine enfant ou approuvé. Si vous souhaitez sauvegarder Hyper-V dans un groupe de travail ou un domaine non approuvé, vous devez configurer l’authentification. Pour un seul serveur Hyper-V, vous pouvez utiliser l’authentification NTLM ou par certificat. Pour un cluster, vous pouvez utiliser l’authentification par certificat uniquement.<br />- L’utilisation de la sauvegarde au niveau de l’hôte pour sauvegarder des données de machine virtuelle sur des disques directs n’est pas prise en charge. Dans ce scénario, nous vous recommandons d’utiliser une sauvegarde au niveau de l’hôte pour sauvegarder des fichiers VHD et une sauvegarde au niveau de l’invité pour sauvegarder les autres données qui ne sont pas visibles sur l’hôte.<br />   \- Vous pouvez sauvegarder des machines virtuelles stockées sur des volumes dédupliqués.|
|Prérequis pour les machines virtuelles Hyper-V|- La version des composants d’intégration qui s’exécute sur la machine virtuelle doit être la même que la version de l’hôte Hyper-V. <br />- Pour chaque sauvegarde de machine virtuelle, vous aurez besoin d’espace libre sur le volume hébergeant les fichiers de disque dur virtuel afin de donner à Hyper-V un espace suffisant pour les disques de différenciation (AVHD) lors de la sauvegarde. L’espace doit être au moins égal au temps de la fenêtre de calcul **Taille initiale du disque\*Taux de variation\*Sauvegarde**. Si vous exécutez plusieurs sauvegardes sur un cluster, vous aurez besoin d’une capacité de stockage suffisante pour accueillir les AVHD pour chacune des machines virtuelles à l’aide de ce calcul.<br />- Pour sauvegarder des machines virtuelles situées sur des serveurs hôtes Hyper-V exécutant Windows Server 2012 R2, un contrôleur SCSI doit être spécifié pour la machine virtuelle, même s’il n’est connecté à rien. (Dans la sauvegarde en ligne Windows Server 2012 R2, l’hôte Hyper-V monte un nouveau disque dur virtuel dans la machine virtuelle, puis le démonte par la suite. Seul le contrôleur SCSI peut prendre en charge cette opération et est par conséquent requis pour la sauvegarde en ligne de la machine virtuelle.  Sans ce paramètre, l’ID d’événement 10103 est émis lorsque vous tentez de sauvegarder la machine virtuelle.)|
|Composants requis Linux|- Vous pouvez sauvegarder des machines virtuelles Linux à l’aide de MABS. Seuls les instantanés cohérents au niveau des fichiers sont pris en charge.|
|Sauvegarder des machines virtuelles avec un stockage CSV|- Pour le stockage CSV, installez le fournisseur de matériel des services VSS (Volume Shadow Copy Services) sur le serveur Hyper-V. Contactez votre fournisseur de réseau SAN pour connaître le fournisseur de matériel VSS.<br />- Si un seul nœud s’arrête de manière inattendue dans un cluster CSV, MABS effectue une vérification de cohérence sur les machines virtuelles qui étaient en cours d’exécution sur ce nœud.<br />- Si vous devez redémarrer un serveur Hyper-V avec le chiffrement de lecteur BitLocker activé sur le cluster CSV, vous devez exécuter une vérification de cohérence pour les machines virtuelles Hyper-V.|
|Sauvegarder des machines virtuelles avec un stockage SMB|- Activez le montage automatique sur le serveur qui exécute Hyper-V pour activer la protection de la machine virtuelle.<br />   - Désactivez le déchargement TCP Chimney.<br />- Assurez-vous que tous les comptes machine$ Hyper-V ont des autorisations complètes sur les partages de fichiers SMB distants spécifiques.<br />- Vérifiez que le chemin de fichier pour tous les composants de la machine virtuelle pendant la récupération vers un autre emplacement comporte moins de 260 caractères. Si ce n’est pas le cas, la récupération peut réussir, mais Hyper-V ne pourra pas monter la machine virtuelle.<br />- Les scénarios suivants ne sont pas pris en charge :<br />     Les déploiements dans lesquels certains composants de la machine virtuelle se trouvent sur des volumes locaux et d’autres sur des volumes distants ; une adresse IPv4 ou IPv6 pour le serveur de fichiers d’emplacement de stockage et la récupération d’une machine virtuelle sur un ordinateur qui utilise des partages SMB distants.<br />- Vous devez activer le service Agent VSS du serveur de fichiers sur chaque serveur SMB. Ajoutez-le dans **Ajouter des rôles et fonctionnalités** > **Sélectionner des rôles de serveurs** > **Services de fichiers et de stockage** > **Services de fichiers** > **Service de fichiers** > **Service Agent VSS du serveur de fichiers**.|

## <a name="back-up-virtual-machines"></a>Sauvegarder des machines virtuelles

1. Configurez votre [serveur MABS](backup-azure-microsoft-azure-backup.md) et [votre stockage](backup-mabs-add-storage.md). Quand vous configurez votre stockage, utilisez ces instructions sur la capacité de stockage.
   - Taille moyenne des machines virtuelles : 100 Go
   - Nombre de machines virtuelles par serveur MABS : 800
   - Taille totale de 800 machines virtuelles : 80 To
   - Espace requis pour le stockage de sauvegarde : 80 To

2. Configurez l’agent de protection MABS sur le serveur Hyper-V ou les nœuds de cluster Hyper-V. Si vous effectuez une sauvegarde au niveau de l’invité, vous installez l’agent sur les machines virtuelles que vous souhaitez sauvegarder au niveau de l’invité.

3. Dans la console Administrateur du serveur de sauvegarde Microsoft Azure, sélectionnez **Protection** > **Créer un groupe de protection** pour ouvrir l’Assistant **Création d’un nouveau groupe de protection**.

4. Dans la page **Sélectionner les membres du groupe**, sélectionnez les machines virtuelles que vous souhaitez protéger à partir des serveurs hôtes Hyper-V sur lesquels elles se trouvent. Nous vous recommandons de placer toutes les machines virtuelles avec la même stratégie de protection dans un groupe de protection. Pour utiliser efficacement l’espace, activez la colocation. La colocation vous permet de localiser des données à partir de différents groupes de protection sur le même stockage sur disque ou sur bande, afin que plusieurs sources de données aient un seul réplica et un seul volume des points de récupération.

5. Dans la page **Sélectionner la méthode de protection des données**, indiquez le nom d’un groupe de protection. Sélectionnez **Je souhaite une protection à court terme à l’aide de Disque** et **Je voudrais une protection en ligne** si vous souhaitez sauvegarder des données sur Azure à l’aide du service Sauvegarde Azure.

6. Dans **Spécifier les objectifs à court terme** > **Durée de conservation**, spécifiez la durée pendant laquelle vous souhaitez conserver les données de disque. Dans **Fréquence de synchronisation**, spécifiez la fréquence à laquelle les sauvegardes incrémentielles des données doivent s’exécuter. Au lieu de sélectionner un intervalle pour les sauvegardes incrémentielles, vous pouvez également activer **Juste avant un point de récupération**. Quand ce paramètre est activé, MABS exécute une sauvegarde complète rapide juste avant chaque point de récupération planifié.

    > [!NOTE]
    >
    >Si vous protégez des charges de travail d’application, les points de récupération sont créés selon la fréquence de synchronisation, à condition que l’application prenne en charge les sauvegardes incrémentielles. Si ce n’est pas le cas, MABS exécute une sauvegarde complète rapide à la place d’une sauvegarde incrémentielle et crée des points de récupération conformément à la planification de la sauvegarde rapide.

7. Dans la page **Vérifier l’allocation de disque**, vérifiez l’espace disque de pool de stockage alloué pour le groupe de protection.

   **Taille totale des données** correspond à la taille des données que vous souhaitez sauvegarder et **Disk space to be provisioned on MABS** (Espace disque à provisionner sur MABS) correspond à l’espace recommandé par MABS pour le groupe de protection. MABS choisit le volume de sauvegarde idéal en fonction des paramètres. Toutefois, vous pouvez modifier les choix de volume de sauvegarde dans **Détails de l’allocation de disque**. Pour les charges de travail, sélectionnez le stockage préféré dans le menu déroulant. Vos modifications changent les valeurs de **Stockage Total** et **Stockage libre** dans le volet **Stockage sur disque disponible**. L’espace sous-provisionné est la quantité de stockage que MABS vous suggère d’ajouter au volume pour continuer à procéder à des sauvegardes de façon fluide à l’avenir.

8. Dans la page **Choisir la méthode de création d’un réplica**, spécifiez comment la réplication initiale des données dans le groupe de protection sera effectuée. Si vous choisissez de **répliquer automatiquement sur le réseau**, nous vous recommandons de choisir une heure creuse. Pour de grandes quantités de données ou des conditions réseau peu optimales, envisagez de sélectionner **Manuellement**, ce qui implique de répliquer les données hors connexion à l’aide d’un support amovible.

9. Dans la page **Options de vérification de cohérence**, indiquez comment vous voulez automatiser les vérifications de cohérence. Vous pouvez activer une vérification pour qu’elle s’exécute uniquement lorsque les données de réplication deviennent incohérentes ou selon une planification. Si vous ne souhaitez pas configurer une vérification de cohérence automatique, vous pouvez exécuter une vérification manuelle à tout moment en cliquant avec le bouton droit sur le groupe de protection et en sélectionnant **Effectuer une vérification de cohérence**.

    Une fois le groupe de protection créé, la réplication initiale des données se produit selon la méthode que vous avez sélectionnée. Après la réplication initiale, chaque sauvegarde a lieu conformément aux paramètres du groupe de protection. Si vous avez besoin de récupérer des données sauvegardées, notez les points suivants :

## <a name="back-up-replica-virtual-machines"></a>Sauvegarder des machines virtuelles de réplication

Si MABS s’exécute sur Windows Server 2012 R2 ou version ultérieure, vous pouvez sauvegarder des machines virtuelles de réplication. Cela est utile pour plusieurs raisons :

**Réduit l’impact des sauvegardes sur la charge de travail en cours d’exécution** : la sauvegarde d’une machine virtuelle entraîne une certaine surcharge au fur et à mesure de la création d’un instantané. En déchargeant le processus de sauvegarde vers un site distant secondaire, la charge de travail en cours d’exécution n’est plus affectée par l’opération de sauvegarde. Cela s’applique uniquement aux déploiements où la copie de sauvegarde est stockée sur un site distant. Par exemple, vous pouvez effectuer des sauvegardes quotidiennes et stocker des données localement pour garantir des temps de restauration rapides, mais effectuer des sauvegardes mensuelles ou trimestrielles à partir de machines virtuelles de réplication stockées à distance pour une conservation à long terme.

**Économise la bande passante** : dans un déploiement classique de succursale distante/siège, vous avez besoin d’une quantité appropriée de bande passante provisionnée pour transférer les données de sauvegarde entre les sites. Si vous créez une stratégie de réplication et basculement, en plus de votre stratégie de sauvegarde de données, vous pouvez réduire la quantité de données redondantes envoyées sur le réseau. En sauvegardant les données de la machine virtuelle de réplication plutôt que celles du serveur principal, vous économisez la surcharge liée à l’envoi des données sauvegardées sur le réseau.

**Active la sauvegarde de l’hébergeur** : vous pouvez utiliser un centre de données hébergé en tant que site de réplication, sans avoir besoin d’un centre de données secondaire. Dans ce cas, le contrat SLA de l’hébergeur exige une sauvegarde cohérente des machines virtuelles de réplication.

Une machine virtuelle de réplication est désactivée jusqu’à ce qu’un basculement soit lancé, et VSS ne peut pas garantir une sauvegarde cohérente au niveau des applications pour une machine virtuelle de réplication. La sauvegarde d’une machine virtuelle de réplication ne sera donc cohérente qu’en cas d’incident. Si la cohérence en cas d’incident ne peut pas être garantie, la sauvegarde échoue et cela peut se produire dans un certain nombre de conditions :

- La machine virtuelle de réplication n’est pas saine et se trouve dans un état critique.

- La machine virtuelle de réplication est en cours de resynchronisation (dans l’état Resynchronisation en cours ou Resynchronisation nécessaire).

- La réplication initiale entre le site principal et le site secondaire est en cours ou en attente pour la machine virtuelle.

- Les journaux .hrl sont appliqués à la machine virtuelle de réplication, ou une action précédente pour appliquer les journaux .hrl sur le disque virtuel a échoué, a été annulée ou a été interrompue.

- La migration ou le basculement de la machine virtuelle de réplication est en cours

## <a name="recover-backed-up-virtual-machines"></a>Récupérer des machines virtuelles sauvegardées

Lorsque vous pouvez récupérer une machine virtuelle sauvegardée, vous utilisez l’Assistant Récupération pour sélectionner la machine virtuelle et le point de récupération spécifique. Pour ouvrir l’Assistant Récupération et récupérer une machine virtuelle :

1. Dans la console Administrateur MABS, tapez le nom de la machine virtuelle ou développez la liste des éléments protégés et sélectionnez la machine virtuelle que vous souhaitez récupérer.

2. Dans le volet **Points de récupération pour**, sélectionnez dans le calendrier n’importe quelle date pour afficher les points de récupération disponibles. Ensuite, dans le volet **Chemin**, sélectionnez le point de récupération que vous souhaitez utiliser dans l’Assistant Récupération.

3. Dans le menu **Actions**, sélectionnez **Récupérer** pour ouvrir l’Assistant Récupération.

    La machine virtuelle et le point de récupération que vous avez sélectionnés s’affichent dans l’écran **Vérifier la récupération sélectionnée**. Sélectionnez **Suivant**.

4. Dans l’écran **Sélectionner le type de récupération**, sélectionnez l’emplacement où vous souhaitez restaurer les données, puis sélectionnez **Suivant**.

    - **Récupérer sur l’instance d’origine** : Lorsque vous récupérez sur l’instance d’origine, le disque dur virtuel d’origine est supprimé. MABS récupère le disque dur virtuel et d’autres fichiers de configuration à l’emplacement d’origine à l’aide de l’enregistreur VSS Hyper-V. À la fin du processus de récupération, les machines virtuelles sont toujours hautement disponibles.
        Le groupe de ressources doit être présent pour la récupération. S’il n’est pas disponible, effectuez la récupération à un autre emplacement, puis rendez la machine virtuelle hautement disponible.

    - **Récupérer l’ordinateur virtuel sur n’importe quel hôte** : MABS prend en charge un emplacement de récupération secondaire, qui permet de récupérer sans interruption une machine virtuelle Hyper-V protégée sur un hôte Hyper-V différent, quelle que soit l’architecture du processeur. Les machines virtuelles Hyper-V récupérées sur un nœud de cluster ne sont pas hautement disponibles. Si vous choisissez cette option, l’Assistant Récupération vous présente un écran supplémentaire pour identifier la destination et le chemin de destination.

    - **Copier dans un dossier réseau** : MABS prend en charge la récupération au niveau de l’élément, qui vous permet d’effectuer une récupération au niveau de l’élément de fichiers, dossiers, volumes et disques durs virtuels (VHD) à partir d’une sauvegarde au niveau de l’hôte de machines virtuelles Hyper-V vers un partage réseau ou un volume sur un serveur protégé par MABS. Il n’est pas nécessaire d’installer l’agent de protection MABS à l’intérieur de l’invité pour effectuer une récupération au niveau de l’élément. Si vous choisissez cette option, l’Assistant Récupération vous présente un écran supplémentaire pour identifier la destination et le chemin de destination.

5. Dans **Spécifier les options de récupération**, configurez les options de récupération, puis sélectionnez **Suivant** :

    - Si vous récupérez une machine virtuelle avec une bande passante faible, sélectionnez **Modifier** pour activer **Limitation de l’utilisation de la bande passante réseau**. Après avoir activé l’option de limitation, vous pouvez spécifier la quantité de bande passante que vous souhaitez rendre disponible et l’heure à laquelle cette bande passante est disponible.
    - Sélectionnez **Activer la récupération SAN au moyen d’instantanés matériels** si vous avez configuré votre réseau.
    - Sélectionnez **Send an e-mail when the recovery completes** (Envoyer un e-mail quand la récupération est terminée), puis indiquez les adresses e-mail si vous souhaitez que des notifications par e-mail soient envoyées une fois le processus de récupération terminé.

6. Dans l’écran de résumé, vérifiez que tous les détails sont corrects. Si les détails ne sont pas corrects ou si vous souhaitez apporter une modification, sélectionnez **Précédent**. Si vous êtes satisfait des paramètres, sélectionnez **Récupérer** pour démarrer le processus de récupération.

7. L’écran **État de la récupération** fournit des informations sur le travail de récupération.

## <a name="next-steps"></a>Étapes suivantes

[Récupérer des données à partir d’un serveur de sauvegarde Azure](./backup-azure-alternate-dpm-server.md)

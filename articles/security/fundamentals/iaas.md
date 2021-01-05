---
title: Meilleures pratiques de sécurité pour les charges de travail IaaS dans Azure | Microsoft Docs
description: " La migration des charges de travail vers Azure IaaS nous offre l’occasion de réévaluer nos conceptions "
services: security
documentationcenter: na
author: terrylanfear
manager: rkarlin
editor: TomSh
ms.assetid: 02c5b7d2-a77f-4e7f-9a1e-40247c57e7e2
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/28/2019
ms.author: terrylan
ms.openlocfilehash: 9b9a83cf71dfa7658c34c3c98f8d12a056adad0c
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94698782"
---
# <a name="security-best-practices-for-iaas-workloads-in-azure"></a>Meilleures pratiques de sécurité pour les charges de travail IaaS dans Azure
Cet article décrit les meilleures pratiques en matière de sécurité pour les machines virtuelles et les systèmes d’exploitation.

Ces meilleures pratiques font l’objet d’un consensus et sont compatibles avec les capacités et fonctionnalités actuelles de la plateforme Azure. Les opinions et technologies évoluant au fil du temps, cet article sera mis à jour de façon à refléter ces changements.

Dans la plupart des scénarios IaaS (Infrastructure en tant que service), les [machines virtuelles Azure](../../virtual-machines/index.yml) représentent la principale charge de travail pour les organisations qui utilisent l’informatique cloud. Cela est particulièrement vrai dans les [scénarios hybrides](https://social.technet.microsoft.com/wiki/contents/articles/18120.hybrid-cloud-infrastructure-design-considerations.aspx) où les organisations souhaitent migrer lentement les charges de travail vers le cloud. Dans ces cas, suivez les [considérations générales de sécurité pour IaaS](https://social.technet.microsoft.com/wiki/contents/articles/3808.security-considerations-for-infrastructure-as-a-service-iaas.aspx) et appliquez les meilleures pratiques de sécurité à toutes vos machines virtuelles.

## <a name="protect-vms-by-using-authentication-and-access-control"></a>Protéger les machines virtuelles à l’aide de l’authentification et du contrôle d’accès
La première étape dans la protection de vos machines virtuelles consiste à vous assurer que seuls les utilisateurs autorisés peuvent configurer de nouvelles machines virtuelles et y accéder.

> [!NOTE]
> Pour améliorer la sécurité des machines virtuelles Linux sur Azure, vous pouvez intégrer l’authentification Azure AD. Lorsque vous utilisez [l’authentification Azure AD pour les machines virtuelles Linux](../../virtual-machines/linux/login-using-aad.md), vous contrôlez et appliquez de façon centralisée des stratégies qui autorisent ou refusent l’accès aux machines virtuelles.
>
>

**Bonne pratique** : contrôler l’accès à la machine virtuelle.   
**Détail** : utilisez des [stratégies Azure](../../governance/policy/overview.md) pour établir des conventions pour les ressources de votre organisation et créer des stratégies personnalisées. Appliquez ces stratégies à vos ressources, telles que les [groupes de ressources](../../azure-resource-manager/management/overview.md). Les machines virtuelles qui appartiennent à un groupe de ressources héritent des stratégies de ce dernier.

Si votre organisation dispose de plusieurs abonnements, vous pouvez avoir besoin d’un moyen de gérer efficacement l’accès, les stratégies et la conformité de ces abonnements. [Les groupes d’administration Azure](../../governance/management-groups/overview.md) fournissent un niveau d’étendue au-delà des abonnements. Vous organisez les abonnements en groupes d’administration (conteneurs) et vous appliquez vos conditions de gouvernance à ces groupes. Tous les abonnements d’un groupe d’administration héritent automatiquement des conditions appliquées au groupe. Les groupes d’administration vous permettent une gestion de qualité professionnelle à grande échelle, quel que soit le type de vos abonnements.

**Bonne pratique** : réduire la variabilité au niveau de la configuration et du déploiement des machines virtuelles.   
**Détail** : utilisez des modèles [Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) pour renforcer vos choix de déploiement, mais aussi gérer et inventorier plus facilement les machines virtuelles de votre environnement.

**Bonne pratique** : Sécuriser l’accès privilégié.   
**Détail** : utilisez [l’approche des privilèges minimum](/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models) et des rôles Azure intégrés pour permettre aux utilisateurs d’accéder aux machines virtuelles et de les configurer :

- [Contributeur de machine virtuelle](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) : peut gérer les machines virtuelles, mais pas le réseau virtuel ni le compte de stockage auxquels elles sont connectées.
- [Contributeur de machine virtuelle Classic](../../role-based-access-control/built-in-roles.md#classic-virtual-machine-contributor) : peut gérer les machines virtuelles créées selon le modèle de déploiement Classic, mais pas le réseau virtuel ni le compte de stockage auxquels elles sont connectées.
- [Administrateur de la sécurité](../../role-based-access-control/built-in-roles.md#security-admin) : Seulement dans Security Center : peut afficher les stratégies de sécurité, les états de sécurité, les alertes et les recommandations, modifier les stratégies de sécurité et ignorer les alertes et les recommandations.
- [Utilisateur de DevTest Labs](../../role-based-access-control/built-in-roles.md#devtest-labs-user) : peut tout afficher et connecter, démarrer, redémarrer et arrêter les machines virtuelles.

Vos coadministrateurs et administrateurs d’abonnement peuvent modifier ce paramètre. Ils deviennent ainsi administrateurs de toutes les machines virtuelles d’un abonnement. Cela implique que vous autorisez tous vos coadministrateurs et administrateurs d’abonnement à se connecter à n’importe laquelle de vos machines.

> [!NOTE]
> Nous vous recommandons de consolider les machines virtuelles ayant le même cycle de vie dans le même groupe de ressources. Les groupes de ressources vous aident à déployer et surveiller vos ressources, tout en compilant leur coût.
>
>

Les organisations qui contrôlent l’accès aux machines virtuelles et leur configuration renforcent la sécurité globale de leurs machines virtuelles.

## <a name="use-multiple-vms-for-better-availability"></a>Utiliser plusieurs machines virtuelles pour améliorer la disponibilité
Si votre machine virtuelle exécute des applications critiques qui requièrent une haute disponibilité, il est vivement recommandé d’utiliser plusieurs machines virtuelles. Pour bénéficier d’une meilleure disponibilité, utilisez un [groupe à haute disponibilité](../../virtual-machines/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy) ou des [zones de disponibilité](../../availability-zones/az-overview.md).

Un groupe à haute disponibilité est un regroupement logique que vous pouvez utiliser dans Azure pour vous assurer que les ressources de machine virtuelle que vous y incluez sont isolées les unes des autres lors de leur déploiement dans un centre de données Azure. Azure veille à ce que les machines virtuelles que vous placez dans un groupe à haute disponibilité s’exécutent sur plusieurs serveurs physiques, racks de calcul, unités de stockage et commutateurs réseau. En cas de défaillance matérielle ou logicielle dans Azure, seul un sous-ensemble de vos machines virtuelles est affecté et votre application globale reste disponible pour vos clients. Les groupes à haute disponibilité sont une fonctionnalité essentielle pour créer des solutions cloud fiables.

## <a name="protect-against-malware"></a>Se protéger des programmes malveillants
Vous devez installer une solution de protection contre les programmes malveillants afin d’identifier et de supprimer les virus, logiciels espions et autres logiciels malveillants. Vous pouvez installer [Microsoft Antimalware](antimalware.md) ou une solution de protection des points de terminaison d’un partenaire de Microsoft ([Trend Micro](https://help.deepsecurity.trendmicro.com/Welcome.html), [Broadcom](https://www.broadcom.com/products), [McAfee](https://www.mcafee.com/us/products.aspx), [Windows Defender](https://www.microsoft.com/windows/comprehensive-security) ou [System Center Endpoint Protection](/configmgr/protect/deploy-use/endpoint-protection)).

Le logiciel Microsoft Antimalware inclut des fonctionnalités telles que la protection en temps réel, l’analyse planifiée, la correction des logiciels malveillants, la mise à jour des signatures, la mise à jour des moteurs, des exemples de création de rapport et la collecte d’événements d’exclusion. Pour les environnements hébergés séparément de votre environnement de production, vous pouvez utiliser une extension anti-programme malveillant pour protéger vos machines virtuelles et vos services cloud.

Vous pouvez intégrer Microsoft Antimalware et des solutions de partenaires avec [Azure Security Center](../../security-center/index.yml) pour bénéficier d’un déploiement simplifié et de fonctionnalités de détection intégrées (alertes et incidents).

**Bonne pratique** : installer une solution anti-programme malveillant.   
**Détail** : [Installer une solution partenaire Microsoft ou Microsoft Antimalware](../../security-center/security-center-services.md#supported-endpoint-protection-solutions-).

**Bonne pratique** : intégrer la solution anti-programme malveillant à Security Center pour surveiller l’état de la protection.   
**Détail** : [Gérer les problèmes de protection des points de terminaison avec Security Center](../../security-center/security-center-partner-integration.md).

## <a name="manage-your-vm-updates"></a>Gérer les sauvegardes des machines virtuelles
Les machines virtuelles Azure, comme toutes les machines virtuelles locales, sont destinées à être gérées par l’utilisateur. Azure ne leur envoie donc pas les mises à jour Windows. Vous devez gérer vous-même les mises à jour de vos machines virtuelles.

**Bonne pratique** : veiller à ce que les machines virtuelles soient toujours à jour.   
**Détail** : utilisez la solution [Update Management](../../automation/update-management/overview.md) d’Azure Automation pour gérer les mises à jour du système d’exploitation de vos ordinateurs Windows et Linux déployés dans Azure, dans des environnements locaux ou auprès d’autres fournisseurs de cloud. Vous pouvez rapidement évaluer l’état des mises à jour disponibles sur tous les ordinateurs d’agent et gérer le processus d’installation des mises à jour requises pour les serveurs.

Les ordinateurs gérés par Update Management utilisent les configurations suivantes pour effectuer l’évaluation et les déploiements de mises à jour :

- Microsoft Monitoring Agent (MMA) pour Windows ou Linux
- PowerShell DSC (Desired State Configuration, configuration d’état souhaité) pour Linux
- Runbook Worker hybride Automation
- Services Microsoft Update ou Windows Server Update (WSUS) pour ordinateurs Windows

Si vous utilisez Windows Update, veillez à ce que la configuration automatique de Windows Update reste activée.

**Bonne pratique** : faire en sorte, au moment du déploiement, que les images créées intègrent les dernières mises à jour Windows.   
**Détail** : recherchez et installez toutes les mises à jour Windows au début de chaque déploiement. Cette phase est particulièrement importante lorsque vous déployez les images que vous avez créées ou issues de votre propre bibliothèque. Bien que les images obtenues via la Place de marché Microsoft Azure soient automatiquement mises à jour par défaut, il peut y avoir un décalage (jusqu’à plusieurs semaines) après la publication d’une version publique.

**Bonne pratique** : redéployer régulièrement les machines virtuelles pour actualiser la version du système d’exploitation.   
**Détail** : définissez votre machine virtuelle à l’aide d’un [modèle Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) afin de faciliter son redéploiement. L’utilisation d’un modèle vous permet de bénéficier d’une machine virtuelle corrigée et sécurisée lorsque vous en avez besoin.

**Bonne pratique** : appliquer rapidement les mises à jour de sécurité pour les machines virtuelles.   
**Détail** : activer Azure Security Center (niveau Gratuit ou Standard) pour [identifier les mises à jour de sécurité manquantes et les appliquer](../../security-center/asset-inventory.md).

**Bonne pratique** : installer les dernières mises à jour de sécurité.   
**Détail** : Parmi les premières charges de travail que nos clients déplacent vers Azure figurent les labos et les systèmes accessibles de l’extérieur. Si vos machines virtuelles Azure hébergent des applications ou des services qui doivent être accessibles par Internet, soyez vigilant sur les mises à jour correctives. Installez les correctifs au-delà du système d’exploitation. Des vulnérabilités non corrigées sur des applications partenaires peuvent également entraîner des problèmes pouvant être facilement évités avec une gestion efficace des correctifs.

**Bonne pratique** : déployer et tester une solution de sauvegarde.   
**Détail** : gérez la sauvegarde de la même façon que toutes les autres opérations. Cela s’applique aux systèmes qui font partie de votre environnement de production étendu au cloud.

Les systèmes de développement et de test doivent suivre des stratégies de sauvegarde capables de fournir des capacités de restauration similaires à ce à quoi les utilisateurs se sont habitués au cours de leur expérience avec des environnements sur site. Les charges de travail migrées vers Azure doivent s’intégrer avec les solutions de sauvegarde existantes lorsque cela est possible. Vous pouvez également utiliser [Azure Backup](../../backup/backup-azure-vms-first-look-arm.md) pour répondre à vos exigences de sauvegarde.

Les organisations qui n’appliquent pas de stratégies de mise à jour logicielle s’exposent davantage aux menaces qui exploitent les vulnérabilités connues, déjà corrigées. Pour se conformer aux réglementations en vigueur, les entreprises doivent prouver leur diligence et la mise en place de contrôles appropriés visant à améliorer la sécurité de leurs charges de travail dans le cloud.

En matière de mise à jour logicielle, les meilleures pratiques sont relativement similaires entre un centre de données traditionnel et un environnement IaaS Azure. Nous vous recommandons d’évaluer vos stratégies actuelles de mise à jour logicielle afin d’y inclure les machines virtuelles hébergées dans Azure.

## <a name="manage-your-vm-security-posture"></a>Gérer l’état de sécurité des machines virtuelles
Les cybermenaces ne cessent d’évoluer. La protection de vos machines virtuelles requiert des fonctionnalités de supervision capables de détecter les menaces, d’empêcher les accès non autorisés à vos ressources, de déclencher des alertes et de limiter les faux positifs.

Pour surveiller l’état de la sécurité de vos [machines virtuelles Windows](../../security-center/security-center-introduction.md) et [Linux](../../security-center/security-center-introduction.md), utilisez [Azure Security Center](../../security-center/security-center-introduction.md). Dans Azure Security Center, protégez vos machines virtuelles grâce aux fonctionnalités suivantes :

- Appliquer les paramètres de sécurité du système d’exploitation avec les règles de configuration recommandées
- Rechercher et télécharger les mises à jour critiques et les mises à jour de sécurité qui peuvent être manquantes
- Mettre en œuvre les recommandations de protection contre les logiciels malveillants
- Valider le chiffrement des disques
- Évaluer et corriger les vulnérabilités
- Détecter les menaces

Security Center peut surveiller activement les menaces qui apparaissent alors sous la forme d’alertes de sécurité. Les menaces corrélées sont regroupées sous la forme d’un incident de sécurité.

Security Center stocke les données dans les [journaux Azure Monitor](../../azure-monitor/log-query/log-query-overview.md). Les journaux Azure Monitor fournissent un langage de requête et le moteur analytique vous donne des insights sur le fonctionnement de vos applications et de vos ressources. Les données sont également collectées à partir d’[Azure Monitor](../../batch/monitoring-overview.md), de solutions de gestion et d’agents installés sur des machines virtuelles hébergées dans le cloud ou localement. Cette fonctionnalité partagée vous permet de constituer une image complète de votre environnement.

Les organisations qui n’appliquent pas une sécurité renforcée à leurs machines virtuelles ne parviennent pas à détecter les actions des utilisateurs non autorisés visant à contourner les contrôles de sécurité mis en place.

## <a name="monitor-vm-performance"></a>Analyser les performances des machines virtuelles
Parfois, une machine virtuelle consomme trop de ressources, ce qui peut poser problème. Une machine virtuelle insuffisamment performante peut entraîner une interruption de service, réduisant ainsi la disponibilité. Cela s’applique particulièrement aux machines virtuelles qui hébergent IIS ou d’autres serveurs web, car une utilisation élevée du processeur ou de la mémoire peut indiquer une attaque par déni de service (DoS). Il est impératif de surveiller l’accès aux machines virtuelles de façon réactive (lorsqu’un problème survient) et de façon proactive (par rapport à des performances de référence mesurées pendant un fonctionnement normal).

Nous vous recommandons d’utiliser [Azure Monitor](../../azure-monitor/platform/data-platform.md) pour obtenir plus de visibilité sur l’intégrité de vos ressources. Fonctionnalités d’Azure Monitor :

- [Fichiers journaux de diagnostic des ressources](../../azure-monitor/platform/platform-logs-overview.md) : effectuent le monitoring des ressources de machines virtuelles et identifient les problèmes potentiels susceptibles de nuire à la disponibilité et aux performances.
- [Extension Diagnostics Azure](../../azure-monitor/platform/diagnostics-extension-overview.md) : offre des fonctionnalités de monitoring et de diagnostic pour les machines virtuelles Windows. Vous pouvez activer ces fonctionnalités en intégrant l’extension dans le [modèle Azure Resource Manager](../../virtual-machines/extensions/diagnostics-template.md).

Les organisations qui ne surveillent pas les performances des machines virtuelles ne peuvent pas déterminer si certaines variations de performances sont normales ou pas. Si une machine virtuelle consomme davantage de ressources que la normale, cela peut être le signe d’une attaque provenant d’une ressource externe ou d’un processus compromis en cours d’exécution sur la machine virtuelle.

## <a name="encrypt-your-virtual-hard-disk-files"></a>Chiffrer les fichiers de disque dur virtuel
Nous vous recommandons de chiffrer vos disques durs virtuels (VHD) afin de protéger votre volume de démarrage et les volumes de données au repos au niveau du stockage, ainsi que vos clés de chiffrement et vos secrets.

[Azure Disk Encryption](./azure-disk-encryption-vms-vmss.md) vous permet de chiffrer vos disques de machine virtuelle IaaS Windows et Linux. Azure Disk Encryption utilise la fonctionnalité standard [BitLocker](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc732774(v=ws.11)) de Windows et la fonctionnalité [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) de Linux pour fournir le chiffrement de volume du système d’exploitation et des disques de données. La solution est intégrée avec [Azure Key Vault](https://azure.microsoft.com/documentation/services/key-vault/), ce qui vous permet de contrôler et de gérer les clés et clés secrètes de chiffrement de disque dans votre abonnement Key Vault. Elle garantit également que toutes les données figurant sur les disques de vos machines virtuelles sont chiffrées au repos dans le Stockage Azure.

Voici les meilleures pratiques en matière d’utilisation d’Azure Disk Encryption :

**Bonne pratique** : activer le chiffrement sur les machines virtuelles.   
**Détail** : Azure Disk Encryption génère et écrit les clés de chiffrement dans votre coffre de clés. La gestion des clés de chiffrement dans votre coffre de clés nécessite l’authentification Azure AD. Créez une application Azure AD à cet effet. Pour l’authentification, vous pouvez utiliser soit l’authentification par clé secrète client, soit [l’authentification Azure AD par certificat client](../../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md).

**Bonne pratique** : utiliser une clé de chiffrement à clé pour renforcer la sécurité des clés de chiffrement, et ajouter une clé de chiffrement à clé à votre coffre de clés.   
**Détail** : Utilisez la cmdlet [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) pour créer une clé de chiffrement à clé dans le coffre de clés. Vous pouvez également importer une clé de chiffrement à clé à partir de votre module de sécurité matériel local de gestion des clés. Pour plus d’informations, consultez la [documentation relative à Key Vault](../../key-vault/keys/hsm-protected-keys.md). Quand une clé de chiffrement principale est spécifiée, Azure Disk Encryption utilise cette clé pour wrapper les secrets de chiffrement avant d’écrire dans Key Vault. En conservant une copie de cette clé dans un module de sécurité matériel local de gestion des clés, vous réduisez le risque de suppression accidentelle de clés.

**Bonne pratique** : prendre un [instantané](../../virtual-machines/windows/snapshot-copy-managed-disk.md) ou faire une sauvegarde avant de chiffrer les disques. Les sauvegardes offrent une possibilité de récupération en cas de défaillance inattendue au cours du chiffrement.   
**Détail** : Les machines virtuelles avec des disques managés imposent une sauvegarde avant que le chiffrement soit effectué. Une fois la sauvegarde effectuée, vous pouvez utiliser la cmdlet **Set-AzVMDiskEncryptionExtension** pour chiffrer des disques managés en spécifiant le paramètre *-skipVmBackup*. Pour plus d’informations sur la façon de sauvegarder et de restaurer des machines virtuelles chiffrées, consultez l’article [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md).

**Bonne pratique** : faire en sorte, pour Azure Disk Encryption, que le coffre de clés se trouve dans la même région que les machines virtuelles pour que les secrets de chiffrement ne franchissent pas les limites régionales.   
**Détail** : créez et utilisez un coffre de clés situé dans la même région que la machine virtuelle à chiffrer.

Azure Disk Encryption répond aux besoins métiers suivants :

- Les machines virtuelles IaaS sont sécurisées au repos via une technologie de chiffrement standard permettant de répondre aux exigences de sécurité et de conformité des organisations.
- Les machines virtuelles IaaS démarrent par le biais de stratégies et de clés contrôlées par les clients qui peuvent auditer leur utilisation dans le coffre de clés.

## <a name="restrict-direct-internet-connectivity"></a>Limiter la connectivité Internet directe
Surveillez et limitez la connectivité Internet directe des machines virtuelles. Les attaquants analysent en permanence des plages IP de cloud public pour détecter les ports de gestion ouverts et tentent des attaques « faciles » comme l’identification de mots de passe courants et des vulnérabilités non corrigées connues. Le tableau suivant répertorie les meilleures pratiques contribuant à protéger les utilisateurs contre ces attaques :

**Bonne pratique** : empêcher une exposition involontaire au routage et à la sécurité du réseau.   
**Détail** : utiliser Azure RBAC pour garantir que seul le groupe central de mise en réseau possède l’autorisation d’accès aux ressources réseau.

**Bonne pratique** : identifier les machines virtuelles exposées qui autorisent l’accès à partir de « n’importe quelle » adresse IP source et y remédier.   
**Détail** : utiliser Azure Security Center. Security Center vous recommande de restreindre l’accès via les points de terminaison accessibles sur Internet si l’un de vos Groupes de sécurité réseau possède une ou plusieurs règles de trafic entrant autorisant l’accès à partir de « n’importe quelle » adresse IP source. Security Center vous recommande de modifier ces règles de trafic entrant afin de [restreindre l’accès](../../security-center/security-center-network-recommendations.md) aux adresses IP source qui en ont réellement besoin.

**Bonne pratique** : restreindre les ports de gestion (RDP, SSH).   
**Détail** : [L’accès juste-à-temps (JAT) aux machines virtuelles](../../security-center/security-center-just-in-time.md) peut être utilisé pour verrouiller le trafic entrant vers vos machines virtuelles Azure, ce qui réduit l’exposition aux attaques et facilite la connexion aux machines virtuelles en cas de besoin. Lorsque l’accès JAT est activé, Security Center verrouille le trafic entrant vers vos machines virtuelles Azure en créant une règle de Groupe de sécurité réseau. Vous sélectionnez les ports de la machine virtuelle pour lesquels le trafic entrant sera verrouillé. Ces ports sont contrôlés par la solution JAT.

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article [Bonnes pratiques et tendances Azure relatives à la sécurité](best-practices-and-patterns.md) pour découvrir d’autres bonnes pratiques en matière de sécurité à appliquer dans le cadre de la conception, du déploiement et de la gestion de vos solutions cloud avec Azure.

Les ressources suivantes fournissent des informations générales sur la sécurité Azure et les services Microsoft associés :
* [Blog de l’équipe de sécurité Azure](/archive/blogs/azuresecurity/) : pour obtenir des informations à jour sur les dernières actualités sur la sécurité Azure
* [Centre de réponse aux problèmes de sécurité Microsoft](https://technet.microsoft.com/library/dn440717.aspx) : emplacement où les vulnérabilités de sécurité Microsoft, dont les problèmes rencontrés avec Azure, peuvent être rapportées ou signalées par e-mail à l’adresse secure@microsoft.com
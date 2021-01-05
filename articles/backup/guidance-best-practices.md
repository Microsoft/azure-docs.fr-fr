---
title: Aide et bonnes pratiques
description: Découvrez les meilleures pratiques et des conseils pour la sauvegarde de la charge de travail Cloud et locale dans le Cloud
ms.topic: conceptual
ms.date: 07/22/2020
ms.openlocfilehash: 522f7d2502a49b912f34f392c52e5046eba8d01f
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92092305"
---
# <a name="backup-cloud-and-on-premises-workloads-to-cloud"></a>Sauvegardez des charges de travail Cloud et locales vers le Cloud

## <a name="introduction"></a>Introduction

Sauvegarde Azure protège de manière complète vos ressources de données dans Azure grâce à une solution simple, sécurisée et économique qui requiert une infrastructure sans infrastructure. Il s’agit d’une solution de protection des données intégrée Azure pour un large éventail de charges de travail. Elle vous aide à protéger vos charges de travail stratégiques qui s’exécutent dans le Cloud et vous garantit que vos sauvegardes sont toujours disponibles et gérées à l’échelle de l’ensemble de votre espace de sauvegarde.

### <a name="intended-audience"></a>Public concerné

Le public cible principal de cet article est l’informatique et les administrateurs d’applications, ainsi que les responsables de l’implémentation d’organisations de grande taille et de taille moyenne, qui souhaitent en savoir plus sur les fonctionnalités de la technologie intégrée de protection des données Azure, la sauvegarde Azure, et comment implémenter efficacement des solutions qui protègent mieux vos déploiements. Cet article suppose que vous êtes familiarisé avec les principales technologies Azure, les concepts de protection des données et que vous avez l’habitude d’utiliser une solution de sauvegarde. Les instructions décrites dans cet article peuvent faciliter la conception de votre solution de sauvegarde sur Azure à l’aide de modèles établis et éviter les pièges connus.

### <a name="how-this-article-is-organized"></a>Organisation de cet article

Bien qu’il soit facile de commencer à protéger l’infrastructure et les applications sur Azure, lorsque vous vous assurez que les ressources Azure sous-jacentes sont correctement configurées et utilisées de manière optimale, vous pouvez accélérer votre temps de valeur. Cet article présente une brève présentation des considérations relatives à la conception et des conseils pour la configuration optimale de votre déploiement de sauvegarde Azure. Il examine les composants principaux (par exemple, coffre Recovery Services, stratégie de sauvegarde) et les concepts (par exemple la gouvernance) et comment les évaluer ainsi que leurs fonctionnalités, avec des liens vers la documentation détaillée du produit.

## <a name="architecture"></a>Architecture

![Architecture de Sauvegarde Azure](./media/guidance-best-practices/azure-backup-architecture.png)

### <a name="workloads"></a>Charges de travail

Sauvegarde Azure active la protection des données pour différentes charges de travail (locales et dans le Cloud). Il s’agit d’un mécanisme de protection des données intégré, sécurisé et fiable dans Azure. Il peut mettre à l’échelle en toute transparence sa protection sur plusieurs charges de travail sans aucune surcharge de gestion. Il existe également plusieurs canaux d’automatisation pour activer cette fonction (par le biais de PowerShell, de l’interface CLI, des modèles Azure Resource Manager et des API REST).

* **Solution de stockage évolutive, durable et sécurisée** , la sauvegarde Azure utilise un stockage d’objets BLOB fiable avec des fonctionnalités de sécurité et de haute disponibilité intégrées. Vous pouvez choisir des stockages LRS, GRS ou RA-GRS pour vos données de sauvegarde.  

* **Intégration de la charge de travail Native -** Sauvegarde Azure fournit une intégration native avec les charges de travail Azure (machines virtuelles, SAP HANA, SQL dans les machines virtuelles Azure et même Azure Files) sans que vous ayez besoin de gérer l’automatisation ou l’infrastructure pour déployer des agents, écrire de nouveaux scripts ou approvisionner le stockage.

### <a name="data-plane"></a>Plan de données

* **Gestion automatique du stockage** – Azure Backup automatise l’approvisionnement et la gestion des comptes de stockage pour les données de sauvegarde afin de s’assurer qu’elles évoluent à mesure que les données de sauvegarde augmentent.

* **Protection contre la suppression des programmes malveillants –** Protège contre les tentatives accidentelles et malveillantes de suppression de vos sauvegardes via la suppression réversible de sauvegardes. Les données de sauvegarde supprimées sont stockées pendant 14 jours gratuitement et peuvent être récupérées à partir de cet état.

* **Sauvegardes chiffrées sécurisées-** Sauvegarde Azure garantit que vos données de sauvegarde sont stockées de manière sécurisée, en tirant parti des fonctionnalités de sécurité intégrées de la plateforme Azure, telles qu’Azure RBAC et le chiffrement.

* **Gestion du cycle de vie des données de sauvegarde -** Sauvegarde Azure nettoie automatiquement les anciennes données de sauvegarde pour se conformer aux stratégies de rétention. Vous pouvez également hiérarchiser vos données du stockage opérationnel vers le stockage du coffre.

### <a name="management-plane"></a>Plan de gestion

* **Contrôle d’accès** : Les coffres (Recovery Services et coffres de sauvegarde) fournissent les fonctionnalités de gestion et sont accessibles via le Portail Azure, le Centre de sauvegarde, les tableaux de bord du coffre, le kit de développement logiciel (SDK), l’interface CLI et même les API REST. Il s’agit également d’une limite Azure RBAC, qui vous permet de limiter l’accès aux sauvegardes uniquement aux administrateurs de sauvegarde autorisés.

* **Gestion des stratégies** – Les stratégies de sauvegarde Azure dans chaque coffre définissent le moment où les sauvegardes doivent être déclenchées et la durée pendant laquelle elles doivent être conservées. Vous pouvez également gérer ces stratégies et les appliquer à plusieurs éléments.

* **Surveillance et création de rapports** – Azure Backup s’intègre à Log Analytics et permet également d’afficher des rapports via des classeurs.

* **Gestion des instantanés** – La sauvegarde Azure prend des instantanés pour certaines charges de travail Azure natives (machines virtuelles et Azure Files), gère ces captures instantanées et permet des restaurations rapides à partir de celles-ci. Cette option réduit considérablement le temps nécessaire pour récupérer vos données dans le stockage d’origine.

## <a name="vault-considerations"></a>Considérations sur le coffre

Sauvegarde Azure utilise des coffres (Recovery Services et Sauvegarde) pour orchestrer et gérer les sauvegardes. Elle utilise également ces coffres pour stocker les données sauvegardées. Une conception efficace des abonnements aide les organisations à établir une structure pour organiser et gérer les ressources dans Azure pour soutenir les priorités de l’entreprise. Respectez les consignes suivantes lorsque vous créez un coffre :  

### <a name="align-to-subscription-design-strategy"></a>Aligner sur la stratégie de conception d’abonnement

Étant donné que le coffre est étendu à un abonnement, adaptez la conception de votre coffre à la stratégie de conception d’abonnement, par exemple *stratégie de catégorie d’application* où les abonnements sont séparés selon des applications ou des services spécifiques, ou dans les lignes des archétypes d’application. Pour plus d’informations, consultez cet [article](/azure/cloud-adoption-framework/decision-guides/subscriptions/).

### <a name="single-or-multiple-vault"></a>Un ou plusieurs coffres

Vous pouvez utiliser un coffre unique ou plusieurs coffres pour organiser et gérer votre sauvegarde. Tenez compte des recommandations suivantes :

* Si vos charges de travail sont toutes gérées par un seul abonnement et une seule ressource, vous pouvez utiliser un coffre unique pour surveiller et gérer votre espace de sauvegarde.

* Si vos charges de travail sont réparties entre les abonnements, vous pouvez créer plusieurs coffres, un ou plusieurs par abonnement.
  * Le Centre de sauvegarde vous permet de disposer d’un seul volet pour gérer toutes les tâches liées à la sauvegarde. [En savoir plus ici]().
  * Vous pouvez personnaliser vos vues avec des modèles de classeur. L’Explorateur de sauvegarde est un modèle de ce type pour les machines virtuelles Azure. [En savoir plus ici](monitor-azure-backup-with-backup-explorer.md).
  * Si vous avez besoin d’une stratégie cohérente dans les coffres, vous pouvez utiliser une stratégie Azure pour propager la stratégie de sauvegarde dans plusieurs coffres. Vous pouvez écrire une [définition de stratégie Azure](../governance/policy/concepts/definition-structure.md) personnalisée qui utilise l’effet [« deployifnotexists »](../governance/policy/concepts/effects.md#deployifnotexists) pour propager une stratégie de sauvegarde dans plusieurs coffres. Vous pouvez [attribuer](../governance/policy/assign-policy-portal.md) cette définition Azure Policy à un domaine particulier (abonnement ou RG), de sorte qu’elle déploie une ressource de « politique de sauvegarde » dans tous les coffres Recovery services dans le cadre de l’attribution Azure Policy. Les paramètres de la stratégie de sauvegarde (par exemple, la fréquence de sauvegarde, la rétention, etc.) doivent être spécifiés par l’utilisateur en tant que paramètres dans l’affectation de la stratégie Azure.

* À mesure que l’encombrement de votre organisation augmente, vous souhaiterez peut-être déplacer les charges de travail entre les abonnements pour les raisons suivantes : aligner par stratégie de sauvegarde, consolider les coffres,compromis sur une redondance moindre pour économiser sur les coûts (passage de GRS à LRS).  Sauvegarde Azure permet de déplacer un coffre de Recovery Services entre les abonnements Azure, ou vers un autre groupe de ressources au sein du même abonnement. [En savoir plus ici](backup-azure-move-recovery-services-vault.md).

### <a name="review-default-settings"></a>Consulter les paramètres par défaut

Nous vous recommandons vivement de consulter les paramètres par défaut pour Type de réplication de stockage et Paramètres de sécurité pour vous assurer qu’ils correspondent à vos besoins avant de configurer les sauvegardes dans le coffre.

* *Le type de réplication de stockage* par défaut est défini sur Géoredondant (GRS). Une fois que vous avez configuré la sauvegarde, l’option de modification est désactivée. [Procédez comme suit](backup-create-rs-vault.md#set-storage-redundancy) pour évaluer et modifier les paramètres.

* *La suppression réversible* par défaut est activée sur les coffres nouvellement créés pour protéger les données de sauvegarde des suppressions accidentelles ou malveillantes. [Procédez comme suit](backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete) pour évaluer et modifier les paramètres.

* *La restauration interrégion* peut être utilisée pour restaurer des machines virtuelles Azure dans la région secondaire, qui est une région jumelée à Azure. Cette option vous permet d’effectuer des recherches pour répondre aux exigences d’audit ou de conformité, et de restaurer la machine virtuelle ou son disque en cas de sinistre dans la région primaire. CRR est une fonctionnalité d’abonnement pour tout coffre GRS. [En savoir plus ici](backup-create-rs-vault.md#set-cross-region-restore).

* Avant de finaliser la conception de votre coffre, passez en revue les [matrices de prise en charge du coffre](backup-support-matrix.md#vault-support) pour comprendre les facteurs qui peuvent influencer ou limiter vos choix de conception.

## <a name="backup-policy-considerations"></a>Remarques sur les stratégies de sauvegarde

La stratégie de sauvegarde Azure comporte deux composants : *Planifier* (quand effectuer la sauvegarde) et *Rétention* (durée de conservation de la sauvegarde). Vous pouvez définir la stratégie en fonction du type de données sauvegardées, des exigences RTO/RPO, des besoins opérationnels ou de conformité réglementaire et du type de charge de travail (par exemple, une machine virtuelle, une base de données, des fichiers). [En savoir plus ici](backup-architecture.md#backup-policy-essentials).

Respectez les consignes suivantes lorsque vous créez une stratégie de sauvegarde :

### <a name="schedule-considerations"></a>Remarques sur la planification

* Envisagez de regrouper les machines virtuelles qui requièrent les mêmes paramètres d’heure de début, de fréquence et de rétention au sein d’une même stratégie.

* Assurez-vous que l'heure de démarrage prévue de la sauvegarde se situe en dehors des heures de pointe de l'application de production.

* Pour réduire le trafic de sauvegarde, sauvegardez les machines virtuelles à des moments différents de la journée en veillant à éviter les chevauchements.

### <a name="retention-considerations"></a>Remarques sur la rétention

* La rétention à court terme peut être « minutes » ou « quotidienne ». La rétention pour les points de sauvegarde « hebdomadaire », « mensuelle » et « annuelle » est appelée rétention à long terme (LTR, Long Term Retention).

* Rétention à long terme :
  * Planifiée (exigences de conformité) - si vous savez à l’avance que les données seront nécessaires pour les années à venir, utilisez la rétention à long terme.
  * Non planifiée (exigence à la demande) - si vous ne le savez pas à l’avance, vous pouvez utiliser des paramètres de conservation personnalisés spécifiques (ces paramètres de conservation personnalisés ne sont pas affectés par les paramètres de stratégie).

* Sauvegarde à la demande avec rétention personnalisée - si vous devez effectuer une sauvegarde non planifiée à l’aide d’une stratégie de sauvegarde, vous pouvez utiliser une sauvegarde à la demande. Cela peut être utile pour effectuer des sauvegardes qui ne correspondent pas à votre sauvegarde planifiée ou pour une sauvegarde granulaire (par exemple, plusieurs sauvegardes de machines virtuelles IaaS par jour, puisque la sauvegarde planifiée n’autorise qu’une seule sauvegarde par jour). Il est important de noter que la stratégie de rétention définie dans la stratégie planifiée ne s’applique pas aux sauvegardes à la demande.

### <a name="optimize-backup-policy"></a>Optimiser la stratégie de sauvegarde

* À mesure que les besoins de votre entreprise évoluent, vous devrez peut-être étendre ou réduire la durée de rétention. Dans ce cas, vous pouvez vous attendre à ce qui suit :  
  * Si la conservation est prolongée, les points de récupération existants sont marqués et conservés conformément à la nouvelle stratégie.
  * Si la conservation est réduite, ils sont marqués de sorte à être nettoyés lors de la prochaine tâche de nettoyage, puis supprimés.
  * Les dernières règles de rétention s’appliquent à tous les points de rétention (sauf les points de rétention à la demande). Par conséquent, si la période de rétention est étendue (par exemple, à 100 jours), lorsque la sauvegarde est effectuée, puis réduite (par exemple, de 100 jours à 7 jours), toutes les données de sauvegarde sont conservées en fonction de la dernière période de rétention spécifiée (c’est-à-dire 7 jours).

* Le service Sauvegarde Azure vous offre la possibilité d’ *arrêter la protection et la gestion de vos sauvegardes*  :
  * *Arrêter la protection et conserver les données de sauvegarde* . Si vous mettez hors service ou désinstallez votre source de données (machine virtuelle, application), mais que vous devez conserver les données à des fins d’audit ou de conformité, vous pouvez utiliser cette option pour arrêter toutes les futures tâches de sauvegarde de la protection de votre source de données et conserver les points de récupération qui ont été sauvegardés. Vous pouvez ensuite restaurer ou reprendre la protection de la machine virtuelle.
  * *Arrêter la protection et supprimer les données de sauvegarde* . Cette option empêche toutes les futures tâches de sauvegarde de protéger votre machine virtuelle et supprime tous les points de récupération. Vous ne pourrez pas restaurer la machine virtuelle, ni utiliser l’option Reprendre la sauvegarde.

  * Si vous reprenez la protection (d’une source de données qui a été arrêtée lors de la conservation des données), les règles de rétention s’appliquent. Tous les points de récupération ayant expiré seront supprimés (à l’heure planifiée).

* Avant de terminer la conception de votre stratégie, il est important de prendre en compte les facteurs suivants susceptibles d’influer sur vos choix de conception.
  * Une stratégie de sauvegarde est étendue à un coffre.
  * Le nombre d’éléments par stratégie est limité (par exemple, 100 machines virtuelles). Pour mettre à l’échelle, vous pouvez créer des stratégies dupliquées avec des planifications identiques ou différentes.
  * Vous ne pouvez pas choisir des points de récupération spécifiques à supprimer.
  * Vous ne pouvez pas désactiver complètement la sauvegarde planifiée et conserver la source de données dans un état protégé. La sauvegarde la moins fréquente que vous pouvez configurer avec la stratégie consiste en une sauvegarde planifiée hebdomadaire. Une alternative serait d'arrêter la protection avec conservation des données et d'activer la protection chaque fois que vous voulez faire une sauvegarde, de faire une sauvegarde à la demande, puis de désactiver la protection mais de conserver les données sauvegardées. [En savoir plus ici](backup-azure-manage-vms.md#stop-protecting-a-vm).

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Pour vous aider à protéger vos données de sauvegarde et à répondre aux besoins de votre entreprise en matière de sécurité, la sauvegarde Azure fournit des garanties de confidentialité, d’intégrité et de disponibilité contre les attaques délibérées et l’utilisation abusive de vos données et systèmes importants. Tenez compte des recommandations de sécurité suivantes pour votre solution de sauvegarde Azure :

### <a name="authentication-and-authorization"></a>Authentification et autorisation

* Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) permet une gestion précise de l’accès, la répartition des tâches au sein de votre équipe et l’octroi de la quantité d’accès aux utilisateurs nécessaires à l’exécution de leurs tâches. [En savoir plus ici](backup-rbac-rs-vault.md).

* Le service Sauvegarde Azure fournit trois rôles intégrés pour contrôler les opérations de gestion des sauvegardes : Contributeurs, opérateurs et lecteurs de sauvegarde. [En savoir plus ici](backup-rbac-rs-vault.md#mapping-backup-built-in-roles-to-backup-management-actions).

* Le service Sauvegarde Azure dispose de plusieurs contrôles de sécurité intégrés pour empêcher, détecter et traiter les failles de sécurité (en savoir plus)

* Les comptes de stockage utilisés par les coffres Recovery Services sont isolés et ne sont pas accessibles aux utilisateurs à des fins malveillantes. L’accès est autorisé uniquement par le biais d’opérations de gestion de Sauvegarde Azure, telles que la restauration.

### <a name="encryption-of-data-in-transit-and-at-rest"></a>Chiffrement des données en transit et au repos

Le chiffrement protège vos données et vous aide à répondre aux engagements de votre entreprise en matière de sécurité et de conformité.

* Dans Azure, les données en transit entre le stockage Azure et le coffre sont protégées par HTTPS. Ces données restent sur le réseau Azure.

* Les données de sauvegarde sont automatiquement chiffrées à l’aide de clés gérées par Microsoft. Vous pouvez également utiliser vos propres clés, également appelées [clés gérées par le client](encryption-at-rest-with-cmk.md).

* Le service Sauvegarde Azure prend en charge la sauvegarde et la restauration des machines virtuelles Azure dont les disques de système d’exploitation ou de données sont chiffrés avec Azure Disk Encryption. [En savoir plus ici](backup-azure-vms-encryption.md).

### <a name="protection-of-backup-data-from-unintentional-deletes"></a>Protection des données de sauvegarde contre les suppressions involontaires

Le service Sauvegarde Azure fournit des fonctionnalités de sécurité pour vous aider à protéger les données de sauvegarde même après leur suppression. Avec la suppression réversible, si l’utilisateur supprime la sauvegarde (d’une machine virtuelle, d’une base de données SQL Server, d’un partage de fichiers Azure, de base de données SAP HANA), les données de sauvegarde sont conservées pendant 14 jours supplémentaires, ce qui permet la récupération de cette sauvegarde sans perte de données. La conservation des données de sauvegarde pendant 14 jours supplémentaires dans l’état « suppression réversible » n’occasionne pas de frais pour le client. [En savoir plus ici](backup-azure-security-feature-cloud.md).

### <a name="monitoring-and-alerts-of-suspicious-activity"></a>Surveillance et alertes d’activité suspecte

le service Sauvegarde Azure intègre des fonctionnalités de surveillance et d’alerte permettant d’afficher et de configurer des actions pour des événements liés au service Sauvegarde Azure. [En savoir plus ici](security-overview.md#monitoring-and-alerts-of-suspicious-activity).

### <a name="security-features-to-help-protect-hybrid-backups"></a>Fonctionnalités de sécurité pour la protection de sauvegardes hybrides

Le service Sauvegarde Azure utilise l’agent Microsoft Azure Recovery Services (MARS) pour sauvegarder et restaurer des fichiers, des dossiers ainsi que le volume ou l’état du système d’un ordinateur local sur Azure. MARS fournit désormais des fonctionnalités de sécurité : une phrase secrète à chiffrer avant le téléchargement et à déchiffrer après le téléchargement à partir de la sauvegarde Azure, les données de sauvegarde supprimées sont conservées pendant 14 jours supplémentaires à compter de la date de suppression, et une opération critique (par exemple, la modification d’une phrase secrète) ne peut être effectuée que par les utilisateurs qui disposent d’informations d’identification Azure valides. [En savoir plus ici](backup-azure-security-feature.md).

## <a name="network-considerations"></a>Considérations relatives au réseau

La sauvegarde de machines virtuelles Azure nécessite un déplacement de données du disque de votre machine virtuelle vers le coffre Recovery Services. La sauvegarde Azure offre plusieurs fonctionnalités permettant de protéger les données de sauvegarde d’être exposées par inadvertance (par exemple, une attaque de l’intercepteur sur le réseau). Tenez compte des recommandations suivantes :

### <a name="internet-connectivity"></a>Connectivité Internet

* *Sauvegarde machine virtuelle Azure* - toutes les opérations de communication et de transfert de données entre le stockage et le service Sauvegarde Azure se produisent uniquement sur le réseau principal Azure sans nécessité d’accéder à votre réseau virtuel. Par conséquent, la sauvegarde de machines virtuelles Azure placées dans des réseaux sécurisés ne vous oblige pas à autoriser l’accès à des adresses IP ou à des noms de domaine complets.

* *Les bases de données SAP HANA ou SLQ Server s’exécutant sur une machine virtuelle Azure* nécessitent une connectivité avec le service Sauvegarde Azure, Stockage Azure et Azure Active Directory. Pour ce faire, vous pouvez utiliser des points de terminaison privés ou autoriser l’accès aux IP publiques ou aux noms de domaine complets (FQDN) requis. Le fait de ne pas permettre une connectivité appropriée aux services Azure requis peut entraîner l’échec d’opérations telles que la détection de base de données, la configuration de la sauvegarde, l’exécution de sauvegardes et la restauration de données. Pour obtenir une aide complète sur le réseau lors de l’utilisation des étiquettes de NSG, du pare-feu Azure et du proxy HTTP, reportez-vous aux articles [SQL](backup-sql-server-database-azure-vms.md#establish-network-connectivity) et [SAP HANA](./backup-azure-sap-hana-database.md#establish-network-connectivity).

* *Hybrides* - l’agent MARS (Microsoft Azure Recovery Services) requiert un accès réseau pour toutes les opérations critiques (installation, configuration, sauvegarde et restauration). L’agent MARS peut se connecter au service de sauvegarde Azure sur [Azure ExpressRoute](install-mars-agent.md#use-azure-expressroute) à l’aide de l’appairage public (disponible pour les anciens circuits) et de l’homologation Microsoft, à l’aide de [points de terminaison privés](install-mars-agent.md#private-endpoints) ou via le [proxy/pare-feu avec les contrôles d’accès appropriés](install-mars-agent.md#verify-internet-access).

### <a name="private-endpoints-for-azure-backup"></a>Points de terminaison privés pour le service Sauvegarde Azure

Azure [Private Endpoint](../private-link/private-endpoint-overview.md) est une interface réseau qui vous permet de vous connecter de façon privée et sécurisée à un service basé sur Azure Private Link. Le service Sauvegarde Azure vous permet de sauvegarder vos données dans vos coffres Recovery Services pour les restaurer ultérieurement en toute sécurité en utilisant des points de terminaison privés.

* Même si des points de terminaison privés sont activés dans un coffre, ils sont uniquement utilisés pour la sauvegarde et la restauration des charges de travail SQL et SAP HANA en cas de sauvegarde de machines virtuelles Azure et de sauvegarde à l’aide de l’agent MARS.  Vous pouvez également utiliser un coffre pour la sauvegarde d’autres charges de travail (cependant, cela ne nécessiterait pas de point de terminaison privé). En plus de la sauvegarde des charges de travail SQL et SAP HANA et de la sauvegarde à l’aide de l’agent MARS, les points de terminaison privés sont également utilisés pour récupérer des fichiers en cas de sauvegarde de machines virtuelles Azure. [En savoir plus ici](private-endpoints.md#recommended-and-supported-scenarios).

* Azure Active Directory ne prend pas en charge les points de terminaison privés pour le moment. Par conséquent, les adresses IP et les noms de domaine complets requis pour le bon fonctionnement du service Azure Active Directory dans une région doivent bénéficier d’une autorisation d’accès sortant sur le réseau sécurisé lors de la sauvegarde de bases de données dans des machines virtuelles Azure et de la sauvegarde à l’aide de l’agent MARS. Vous pouvez aussi utiliser des balises de groupe de sécurité réseau (NSG) et des balises du service Pare-feu Azure pour autoriser l’accès à Azure AD, le cas échéant. En savoir plus sur les [prérequis ici](./private-endpoints.md#before-you-start).

## <a name="governance-considerations"></a>Considérations sur la gouvernance

La gouvernance dans Azure est principalement implémentée avec [Azure Policy](../governance/policy/overview.md) et [Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md). [Azure Policy](../governance/policy/overview.md) vous permet de créer, d’attribuer et de gérer des définitions de stratégie afin d’appliquer des règles pour vos ressources. Cette fonctionnalité maintient ces ressources conformes aux standards de votre entreprise. [Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md) vous permet de suivre l’utilisation du Cloud et les dépenses liées à vos ressources Azure et à d’autres fournisseurs de Cloud. En outre, les outils suivants, tels que [Calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/) et [Azure Advisor](../advisor/advisor-overview.md) jouent un rôle important dans le processus de gestion des coûts.

### <a name="azure-backup-support-two-key-scenarios-via-built-in-azure-policy"></a>Azure Backup prend en charge deux scénarios clés via des Azure Policy intégrés

* Vérifiez que les machines critiques nouvellement créées soient automatiquement sauvegardées avec les paramètres de rétention appropriés. Sauvegarde Azure fournit une stratégie intégrée (via Azure Policy) qui peut être attribuée à toutes les machines virtuelles Azure dans un emplacement spécifié d’un abonnement ou d’un groupe de ressources. Quand cette stratégie est affectée à une étendue donnée, toutes les machines virtuelles créées dans cette étendue sont automatiquement configurées pour la sauvegarde dans un coffre existant au même emplacement et dans le même abonnement. L’utilisateur peut spécifier le coffre et la stratégie de rétention auxquels les machines virtuelles sauvegardées doivent être associées. [En savoir plus ici](backup-azure-auto-enable-backup.md).

* Vérifiez que les diagnostics sont activés pour les coffres nouvellement créés pour prendre en charge les rapports. Il est souvent fastidieux d’ajouter manuellement un paramètre de diagnostic par coffre. Par ailleurs, il faut activer les paramètres de diagnostic de chaque nouveau coffre créé pour pouvoir afficher ses rapports. Pour simplifier la création de paramètres de diagnostic à grande échelle (avec Log Analytics comme destination), la Sauvegarde Azure fournit une stratégie Azure Policy intégrée. Cette stratégie ajoute un paramètre de diagnostic Log Analytics à tous les coffres d’un abonnement ou d’un groupe de ressources donné. Les sections suivantes donnent des instructions relatives à l’utilisation de cette stratégie. [En savoir plus ici](azure-policy-configure-diagnostics.md).

### <a name="azure-backup-cost-considerations"></a>Remarques sur les coûts de sauvegarde Azure

Les fonctionnalités du service sauvegarde Azure offrent la flexibilité nécessaire pour gérer efficacement vos coûts, tout en répondant aux besoins de votre entreprise en matière de continuité d’activité et de reprise d’activité (BCDR). Tenez compte des recommandations suivantes :

* Utilisez la calculatrice de prix pour évaluer et optimiser les coûts en ajustant les différents leviers. [En savoir plus ici](azure-backup-pricing.md)

* Explorateur de sauvegarde : Utilisez l’Explorateur de sauvegarde ou les Rapports de sauvegarde pour comprendre et optimiser la croissance du stockage de sauvegarde, en arrêtant les sauvegardes pour les charges de travail non critiques ou les machines virtuelles supprimées. Vous pouvez obtenir une vue agrégée de l’ensemble de votre patrimoine, du point de vue de la sauvegarde. Cela comprend non seulement des informations sur vos éléments de sauvegarde, mais également des ressources qui ne sont pas configurées pour la sauvegarde. Cela garantit que vous ne perdez jamais la protection des données critiques dans votre patrimoine et que vos sauvegardes sont optimisées pour les charges de travail non critiques ou les charges de travail supprimées.

* Optimiser la stratégie de sauvegarde
  * Optimiser les paramètres de planification et de rétention en fonction des archétypes de charge de travail (par exemple, stratégiques, non critiques)
  * Optimiser les paramètres de rétention pour la restauration instantanée
  * Choisissez le type de sauvegarde approprié pour répondre aux exigences, tout en prenant en charge les types de sauvegarde pris en charge (complète, incrémentielle, journal, différentielle) par la charge de travail dans la Sauvegarde Azure.

* Sauvegarde sélective des disques : L'exclusion de disque (fonction de prévisualisation) offre un choix efficace et rentable pour sauvegarder sélectivement des données critiques. Vous avez des données critiques à sauvegarder sur un seul disque et vous ne voulez pas sauvegarder les autres disques attachés à une machine virtuelle. Cela est également utile lorsque vous avez plusieurs solutions de sauvegarde. Par exemple, lorsque vous sauvegardez vos bases de données ou vos données avec une solution de sauvegarde de charge de travail (base de données SQL Server dans la sauvegarde de machine virtuelle Azure) et que vous souhaitez utiliser la sauvegarde au niveau de la machine virtuelle Azure pour les disques sélectionnés.

* La solution Sauvegarde Azure prend des instantanés des machines virtuelles Azure et les stocke avec les disques afin de stimuler la création de points de récupération et d'accélérer les opérations de restauration. On parle de restauration instantanée. Par défaut, les instantanés de restauration instantanée sont conservés pendant deux jours. Cette fonctionnalité autorise les opérations de restauration à partir de ces instantanés en réduisant les durées de restauration. Elle réduit le temps requis pour transformer et copier des données à partir du coffre. Vous voyez donc des coûts de stockage correspondant aux instantanés pris pendant cette période. [En savoir plus ici](backup-instant-restore-capability.md#configure-snapshot-retention).

* Le type de réplication de stockage pour les coffres Sauvegarde Azure par défaut est défini sur Géoredondant (GRS). Cette option ne peut pas être modifiée après la protection des éléments. Le stockage géo-redondant (GRS) fournit un niveau de durabilité des données plus élevé que le stockage localement redondant (LRS), permet à un abonnement d’utiliser la restauration inter-régions. Il entraîne des coûts supplémentaires. Passez en revue les compromis entre les coûts réduits et la durabilité des données, puis décidez ce qui convient le mieux pour votre scénario. [En savoir plus ici](backup-create-rs-vault.md#set-storage-redundancy)

* Si vous protégez à la fois la charge de travail qui s’exécute à l’intérieur d’une machine virtuelle et la machine virtuelle elle-même, vérifiez si cette protection double est nécessaire.

## <a name="monitoring-and-alerting-considerations"></a>Remarques sur la surveillance et l'alerte

En tant qu’administrateur ou utilisateur de sauvegarde, vous devez être en mesure de surveiller toutes les solutions de sauvegarde et de recevoir des notifications sur les scénarios importants. Cet article décrit en détail les fonctionnalités de surveillance et de notification fournies par le service Sauvegarde Azure.

### <a name="monitoring"></a>Surveillance

* Azure Backup fournit une **surveillance intégrée des tâches** pour les opérations telles que la configuration de la sauvegarde, la sauvegarde, la restauration, la suppression de la sauvegarde, etc. Cela est étendue au coffre et idéal pour la surveillance d’un coffre unique. [En savoir plus ici](backup-azure-monitoring-built-in-monitor.md#backup-jobs-in-recovery-services-vault).

* Si vous avez besoin de surveiller les activités opérationnelles à l’échelle, **l’Explorateur de sauvegarde** fournit une vue agrégée de l’ensemble de votre espace de sauvegarde, ce qui permet une analyse détaillée et une résolution des problèmes. Il s'agit d'un classeur intégré à Azure Monitor qui offre un emplacement unique et central pour vous aider à surveiller les activités opérationnelles dans l'ensemble du parc de sauvegarde sur Azure, couvrant les locataires, les emplacements, les abonnements, les groupes de ressources et les coffres. [En savoir plus ici](monitor-azure-backup-with-backup-explorer.md).
  * Utilisez-le pour identifier les ressources qui ne sont pas configurées pour la sauvegarde, et assurez-vous que vous ne manquez jamais la protection des données critiques dans votre patrimoine.
  * Le tableau de bord fournit des activités opérationnelles au cours des sept derniers jours (maximum). Si vous devez conserver ces données, vous pouvez les exporter en tant que fichier Excel et les conserver.
  * Si vous êtes un utilisateur d’Azure Lighthouse, vous pouvez afficher des informations sur plusieurs locataires, ce qui permet une surveillance sans limite.

* Si vous avez besoin de conserver et d’afficher les activités opérationnelles à long terme, utilisez **Rapports** . Les administrateurs de sauvegarde ont souvent besoin d’insights sur les sauvegardes, en fonction de données couvrant une longue période. Voici quelques-uns des cas d’usage d’une telle solution :
  * allocation et prévision du stockage cloud utilisé ;
  * audit des sauvegardes et restaurations ;
  * identification des tendances clés à différents niveaux de granularité.

* De plus :
  * Vous pouvez envoyer des données (par exemple, des travaux, des stratégies, etc.) à l’espace de travail **Log Analytics** . Cela permet d’activer les fonctionnalités des journaux d’Azure Monitor pour permettre la corrélation des données avec d’autres données d’analyse collectées par Azure Monitor, consolider les entrées de journal à partir de plusieurs abonnements et clients Azure en un seul emplacement pour l’analyse ensemble, utiliser des requêtes de journal pour effectuer une analyse complexe et obtenir des informations détaillées sur les entrées de journal. [En savoir plus ici](../azure-monitor/platform/activity-log.md#send-to-log-analytics-workspace).
  * Vous pouvez envoyer des données au hub d’événements pour envoyer des entrées en dehors d’Azure, par exemple à une solution SIEM (Security Information and Event Management) tierce ou autre solution log analytics. [En savoir plus ici](../azure-monitor/platform/activity-log.md#send-to-azure-event-hubs).
  * Envoyez le journal d’activité à un compte de stockage Azure si vous souhaitez conserver vos données de journal plus de 90 jours à des fins d’audit, d’analyse statique ou de sauvegarde. Si vous devez conserver vos événements pendant 90 jours ou moins, il est inutile de configurer l’archivage sur un compte de stockage, puisque les événements du journal d’activité sont conservés dans la plateforme Azure pendant 90 jours. [Plus d’informations](../azure-monitor/platform/activity-log.md#send-to--azure-storage)

### <a name="alerting"></a>Génération d’alertes

* Les alertes permettent principalement d’être notifié pour prendre les mesures appropriées. La section Alertes de sauvegarde affiche les alertes générées par le service Sauvegarde Azure.

* Sauvegarde Azure fournit un mécanisme de notification **d’alerte intégré** par courrier électronique pour les défaillances, les avertissements et les opérations critiques. Vous pouvez spécifier des adresses e-mail individuelles ou des listes de distribution pour être informé de la génération d’une alerte. Vous pouvez également choisir d’être informé à chaque alerte ou de les grouper dans une synthèse horaire avant d’être informé.
  * Ces alertes sont définies par le service et assurent la prise en charge des scénarios limités (échecs de sauvegarde/restauration, arrêt de la protection avec conservation des données/arrêt de la protection avec suppression des données, etc.). [En savoir plus ici](backup-azure-monitoring-built-in-monitor.md#alert-scenarios).
  * Si une opération destructive comme l’arrêt de la protection avec suppression des données est effectuée, une alerte est déclenchée et un e-mail est envoyé aux administrateurs, coadministrateurs et propriétaires de l’abonnement, même si les notifications ne sont **pas** configurées pour le coffre Recovery Services.
  * Certaines charges de travail peuvent générer une fréquence élevée d’échecs (par exemple, SQL Server toutes les 15 minutes). Pour éviter de saturer les alertes déclenchées pour chaque occurrence d’échec, les alertes sont consolidées. [En savoir plus ici](backup-azure-monitoring-built-in-monitor.md#consolidated-alerts).
  * Les alertes intégrées ne peuvent pas être personnalisées et sont limitées aux courriers électroniques définis dans le Portail Azure.

* Si vous devez **créer des alertes personnalisées** (par exemple, des alertes de travaux réussis), utilisez Log Analytics. Dans Azure Monitor, vous pouvez créer vos propres alertes dans un espace de travail Log Analytics. Les charges de travail hybrides (DPM/MABS) peuvent également envoyer des données à LA et utiliser LA pour fournir des alertes courantes entre les charges de travail prises en charge par sauvegarde Azure.

* Vous pouvez également recevoir des notifications par le biais des **journaux d’activité** intégrés au coffre Recovery Services. Toutefois, ils prennent en charge des scénarios limités et ne conviennent pas pour des opérations telles que la sauvegarde planifiée, qui s’aligne mieux avec les journaux de ressources qu’avec les journaux d’activité. Pour en apprendre davantage sur ces limitations et savoir comment utiliser un espace de travail Log Analytics pour la supervision et la génération d’alertes à grande échelle pour toutes vos charges de travail protégées par Sauvegarde Azure, consultez cet [article](backup-azure-monitoring-use-azuremonitor.md#using-log-analytics-to-monitor-at-scale).

## <a name="next-steps"></a>Étapes suivantes

Nous vous recommandons de lire les articles suivants comme points de départ pour l’utilisation de la sauvegarde Azure :

* [Vue d’ensemble du service Sauvegarde Azure](backup-overview.md)
* [Questions fréquentes (FAQ)](backup-azure-backup-faq.md)

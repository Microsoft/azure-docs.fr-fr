---
title: Chiffrement transparent des données (TDE, Transparent Data Encryption) managé par le client
description: Bring Your Own Key (BYOK) prend en charge le chiffrement transparent des données (TDE, Transparent Data Encryption) avec Azure Key Vault pour SQL Database et Azure Synapse Analytics. Vue d'ensemble de TDE avec BYOK, avantages, fonctionnement, considérations et recommandations.
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: seo-lt-2019, azure-synapse
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 03/18/2020
ms.openlocfilehash: 2a7d77579eaebd3ee951d0184e25937783420806
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325194"
---
# <a name="azure-sql-transparent-data-encryption-with-customer-managed-key"></a>Transparent Data Encryption Azure SQL avec une clé managée par le client
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

[Transparent Data Encryption (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) Azure SQL avec une clé managée par le client active le scénario Bring Your Own Key scénario (BYOK) pour la protection des données au repos et permet aux organisations d’implémenter la séparation des tâches dans la gestion des clés et des données. Avec le chiffrement transparent des données managé par le client, le client est responsable et dans un contrôle total de la gestion du cycle de vie des clés (création, chargement, rotation, suppression des clés), des autorisations d’utilisation de la clé et de l’audit des opérations sur les clés.

Dans ce scénario, la clé utilisée pour le chiffrement de la Clé de chiffrement (DEK) de la base de données, appelée protecteur TDE, est une clé asymétrique managée par le client, stockée dans un système de gestion des clés externes informatique [Azure Key Vault (AKV)](../../key-vault/general/secure-your-key-vault.md) appartenant au client et managé par le client. Key Vault fournit un stockage sécurisé hautement disponible et évolutif pour les clés de chiffrement RSA, éventuellement sauvegardé par les modules de sécurité matériels validés FIPS 140-2 niveau 2 (HSM). Il n’autorise pas l’accès direct à une clé stockée, mais fournit des services de chiffrement/déchiffrement à l’aide de la clé aux entités autorisées. La clé peut être générée par le coffre de clés, importée ou [transférée vers le coffre de clés à partir d’un dispositif HSM local](../../key-vault/keys/hsm-protected-keys.md).

Pour Azure SQL Database et Azure Synapse Analytics, le protecteur TDE est défini au niveau du serveur et est hérité par toutes les bases de données chiffrées associées à ce serveur. Pour Azure SQL Managed Instance, le protecteur TDE est défini au niveau de l’instance et il est hérité par toutes les bases de données chiffrées sur cette instance. Le terme *serveur* fait référence à la fois au serveur dans SQL Database et Azure Synapse et à une instance gérée dans SQL Managed Instance tout au long de ce document, sauf indication contraire.

> [!IMPORTANT]
> Pour ceux qui utilisent le TDE géré par le service et qui souhaitent commencer à utiliser le TDE géré par le client, les données restent chiffrées pendant le processus de basculement et il n’y a pas de temps d’arrêt ni de rechiffrement des fichiers de base de données. Le basculement d’une clé managée par le service à une clé managée par le client nécessite uniquement de rechiffrement de la clé de chiffrement (DEK), une opération rapide et en ligne.

> [!NOTE]
> <a id="doubleencryption"></a>Pour fournir aux clients Azure SQL deux couches de chiffrement des données au repos, le chiffrement de l’infrastructure (à l’aide de l’algorithme de chiffrement AES-256) avec des clés gérées par la plateforme est déployé. Cela fournit une couche supplémentaire de chiffrement au repos, ainsi que le TDE avec des clés gérées par le client, ce qui est déjà disponible. Pour Azure SQL Database et Managed Instance, toutes les bases de données, y compris la base de données master et les autres bases de données système, sont chiffrées lorsque le chiffrement de l’infrastructure est activé. À ce stade, les clients doivent demander l’accès à cette fonctionnalité. Si cette fonctionnalité vous intéresse, contactez AzureSQLDoubleEncryptionAtRest@service.microsoft.com.

## <a name="benefits-of-the-customer-managed-tde"></a>Avantages de TDE managé par le client

Le TDE managé par le client offre les avantages suivants au client :

- contrôle complet et granulaire de l’utilisation et de la gestion du protecteur TDE ;

- transparence de l’utilisation du protecteur TDE ;

- possibilité d’implémenter la séparation des tâches dans la gestion des clés et des données au sein de l’organisation ;

- l’administrateur Key Vault peut révoquer les autorisations d’accès à la clé pour rendre la base de données chiffrée inaccessible ;

- gestion centralisée des modèles dans Azure Key Vault ;

- confiance accrue de vos clients finaux, car AKV est conçu de telle sorte que Microsoft ne peut pas voir ni extraire les clés de chiffrement ;

## <a name="how-customer-managed-tde-works"></a>Fonctionnement de TDE managé par le client

![Configuration et fonctionnement de TDE managé par le client](./media/transparent-data-encryption-byok-overview/customer-managed-tde-with-roles.PNG)

Pour que le serveur puisse utiliser le protecteur TDE stocké dans AKV pour le chiffrement de la clé de chiffrement, l’administrateur du coffre de clés doit accorder les droits d’accès suivants au serveur à l’aide de son identité Azure Active Directory (Azure AD) unique :

- **obtenir** : pour récupérer la partie publique et les propriétés de la clé dans Key Vault

- **wrapKey** : pour pouvoir protéger (chiffrer) la clé de chiffrement

- **unwrapKey** : pour pouvoir ôter la protection (déchiffrer) la clé de chiffrement

L’administrateur du coffre de clés peut également [activer la journalisation des événements d’audit de coffre de clés](../../azure-monitor/insights/key-vault-insights-overview.md), afin qu’ils puissent être audités ultérieurement.

Quand le serveur est configuré pour utiliser un protecteur TDE d’AKV, le serveur envoie la clé de chiffrement de chaque base de données prenant en charge TDE au coffre de clés pour chiffrement. Le coffre de clés retourne la clé de chiffrement chiffrée, qui est alors stockée dans la base de données utilisateur.

Le cas échéant, le serveur envoie des clés de chiffrement protégées vers le coffre de clés pour le déchiffrement.

Les auditeurs peuvent utiliser Azure Monitor pour évaluer les journaux AuditEvent du coffre de clés, si la journalisation est activée.

[!INCLUDE [sql-database-akv-permission-delay](../includes/sql-database-akv-permission-delay.md)]

## <a name="requirements-for-configuring-customer-managed-tde"></a>Exigences pour la configuration de TDE managé par le client

### <a name="requirements-for-configuring-akv"></a>Exigences pour la configuration d’AKV

- Le coffre de clés et SQL Database/instance gérée doivent appartenir au même abonné Azure Active Directory. Les interactions entre un serveur et un coffre de clés inter-abonnés ne sont pas prises en charge. Pour déplacer des ressources par la suite, vous devez reconfigurer TDE avec AKV. En savoir plus sur le [déplacement des ressources](../../azure-resource-manager/management/move-resource-group-and-subscription.md).

- La fonctionnalité [suppression réversible](../../key-vault/general/soft-delete-overview.md) doit être activée sur le coffre de clés pour protéger contre la suppression accidentelle d’une clé de perte de données (ou d’un coffre de clés). Les ressources supprimées de manière réversible sont conservées pendant 90 jours, sauf si elles sont récupérées ou purgées par le client entre-temps. Les actions de *récupération* et de *vidage* ont leurs propres autorisations associées dans une stratégie d’accès au coffre de clés. La fonctionnalité de suppression réversible désactivée par défaut peut être activée via [PowerShell](../../key-vault/general/key-vault-recovery.md?tabs=azure-powershell) ou [la CLI](../../key-vault/general/key-vault-recovery.md?tabs=azure-cli). Elle ne peut pas être activée via le portail Azure.  

- Accordez au serveur ou à l’instance gérée l’accès au coffre de clés (get, wrapKey, unwrapKey) à l’aide de son identité Azure Active Directory. Lors de l’utilisation du portail Azure, l’identité Azure AD est créée automatiquement. Lors de l’utilisation de PowerShell ou de la CLI, l’identité Azure AD doit être explicitement créée et sa saisie vérifiée. Consultez [Configurer TDE avec BYOK](transparent-data-encryption-byok-configure.md) et [Configure TDE with BYOK for SQL Managed Instance](../managed-instance/scripts/transparent-data-encryption-byok-powershell.md) (Configurer TDE avec BYOK pour SQL Managed Instance) pour obtenir des instructions détaillées lors de l’utilisation de PowerShell.

- Lorsque vous utilisez le pare-feu avec AKV, vous devez activer l’option *Autoriser les services Microsoft approuvés pour contourner le pare-feu*.

### <a name="requirements-for-configuring-tde-protector"></a>Exigences pour la configuration du protecteur TDE

- Le protecteur TDE peut uniquement être une clé asymétrique, RSA ou RSA HSM. Les longueurs de clé prises en charge sont 2 048 et 3 072 octets.

- La date d’activation de la clé (si définie) doit être une date et une heure passées. La date d’expiration (si définie) doit être une date et une heure ultérieures.

- La clé doit être dans l’état *activé*.

- Si vous importez une clé existante dans le coffre de clés, veillez à la fournir dans les formats de fichiers pris en charge (.pfx, .byok ou .backup).

> [!NOTE]
> Azure SQL prend désormais en charge l’utilisation d’une clé RSA stockée dans un HSM géré en tant que protecteur TDE. Cette fonctionnalité est disponible en **préversion publique**. Azure Key Vault HSM géré est un service cloud entièrement géré, à haut niveau de disponibilité et à un seul locataire, qui vous permet de protéger les clés de chiffrement de vos applications cloud à l’aide de modules de sécurité matériels certifiés FIPS 140-2 de niveau 3. En savoir plus sur les [HSM managés](../../key-vault/managed-hsm/index.yml).


## <a name="recommendations-when-configuring-customer-managed-tde"></a>Suggestions lors de la configuration de TDE managé par le client

### <a name="recommendations-when-configuring-akv"></a>Suggestions lors de la configuration d’AKV

- Associez au maximum 500 bases de données d’usage général ou 200 bases de données critiques pour l’entreprise au total avec un coffre de clés dans un abonnement unique pour garantir une haute disponibilité lorsque le serveur accède au protecteur TDE dans le coffre de clés. Ces chiffres sont basés sur l’expérience et documentés dans les [limites du service de coffre de clés](../../key-vault/general/service-limits.md). L’objectif est d’éviter les problèmes après le basculement du serveur, car il déclenchera autant d’opérations clés sur le coffre qu’il y a de bases de données dans ce serveur.

- Définissez un verrou de ressource sur le coffre de clés pour contrôler les utilisateurs pouvant supprimer cette ressource critique et pour empêcher toute suppression accidentelle ou non autorisée. En savoir plus sur les [verrous de ressource](../../azure-resource-manager/management/lock-resources.md).

- Activer l’audit et la création de rapports sur toutes les clés de chiffrement : Le coffre de clés fournit des journaux d’activité faciles à injecter dans d’autres outils de gestion d’événements et d’informations de sécurité. Operations Management Suite [Log Analytics](../../azure-monitor/insights/key-vault-insights-overview.md) est un exemple de service déjà intégré.

- Reliez chaque serveur à deux coffres de clés résidant dans des régions différentes et détenant le même matériau clé pour garantir la haute disponibilité des bases de données chiffrées. Marquez uniquement la clé du coffre de clés dans la même région qu’un protecteur TDE. Le système bascule automatiquement vers le coffre de clés dans la région distante en cas de panne affectant le coffre situé dans la même région.

### <a name="recommendations-when-configuring-tde-protector"></a>Suggestions lors de la configuration du protecteur TDE

- Conservez une copie du protecteur TDE à un emplacement sécurisé ou déposez-le au service de dépôt.

- Si la clé est générée dans le coffre de clés, créez une sauvegarde de clé avant d’utiliser la clé dans AKV pour la première fois. La sauvegarde peut être restaurée sur un Azure Key Vault uniquement. En savoir plus sur la commande [Backup-AzKeyVaultKey](/powershell/module/az.keyvault/backup-azkeyvaultkey).

- Créez une nouvelle sauvegarde à chaque modification de la clé (par exemple, attributs de clé, balises, listes de contrôle d’accès).

- **Conservez les versions précédentes** de la clé dans le coffre de clés lors de la rotation des clés, de sorte que les anciennes sauvegardes de base de données puissent être restaurées. Lorsque le protecteur TDE est modifié pour une base de données, les anciennes sauvegardes de la base de données **ne sont pas mises à jour** pour utiliser le dernier protecteur TDE. Au moment de la restauration, le protecteur TDE doit être chiffré avec chaque sauvegarde lorsque celle-ci a été créée. Les rotations de clés peuvent être effectuées en suivant les instructions de l’article [Rotation du protecteur Transparent Data Encryption à l’aide de PowerShell](transparent-data-encryption-byok-key-rotation.md).

- Conservez toutes les clés précédemment utilisées dans AKV même après avoir basculé vers les clés managées par le service. Cela garantit la restauration des sauvegardes de base de données avec les protecteurs TDE stockés dans Azure Key Vault.  Les protecteurs TDE créés avec Azure Key Vault doivent être conservés jusqu'à ce que toutes les sauvegardes stockées restantes aient été créées avec des clés managées par le service. Effectuez des copies de sauvegarde récupérables de ces clés à l’aide de [Backup-AzKeyVaultKey](/powershell/module/az.keyvault/backup-azkeyvaultkey).

- Pour supprimer une clé potentiellement compromise pendant un incident de sécurité sans risque de perte de données, suivez les étapes indiquées dans [Supprimer une clé potentiellement compromise](transparent-data-encryption-byok-remove-tde-protector.md).

## <a name="inaccessible-tde-protector"></a>Protecteur TDE inaccessible

Lorsque le chiffrement transparent des données est configuré pour utiliser une clé managée par le client, un accès continu au protecteur TDE est requis pour que la base de données reste en ligne. Si le serveur perd l’accès au protecteur TDE managé par le client dans AKV, dans un délai de 10 minutes, une base de données commence à refuser toutes les connexions avec le message d’erreur correspondant et passe à l’état *inaccessible*. La seule action autorisée sur une base de données dans un état inaccessible est de la supprimer.

> [!NOTE]
> Si la base de données est inaccessible en raison d’une interruption réseau intermittente, aucune action n’est requise et les bases de données sont automatiquement remises en ligne.

Après la restauration de l’accès à la clé, la sauvegarde de la base de données en ligne nécessite du temps et des étapes supplémentaires, qui peuvent varier en fonction du temps écoulé sans avoir accès à la clé ni à la taille des données dans la base de données :

- Si l’accès à la clé est restauré dans un délai de 8 heures, la base de données sera automatiquement réparée dans l’heure suivante.

- Si l’accès à la clé est restauré dans un délai supérieur à huit heures, la réparation automatique n’est pas possible et la restauration de la base de données nécessite des étapes supplémentaires dans le portail et peut prendre beaucoup de temps, en fonction de la taille de la base de données. Une fois la base de données de nouveau en ligne, les paramètres précédemment configurés au niveau du serveur, tels que la configuration du [groupe de basculement](auto-failover-group-overview.md), l’historique de la limite de restauration dans le temps ainsi que les balises **seront perdus**. Par conséquent, il est recommandé d’implémenter un système de notification qui vous permet d’identifier et de résoudre les problèmes d’accès aux clés sous-jacentes dans un délai de 8 heures.

Vous trouverez ci-dessous une vue des étapes supplémentaires requises sur le portail pour remettre en ligne une base de données inaccessible.

![Base de données inaccessible TDE BYOK](./media/transparent-data-encryption-byok-overview/customer-managed-tde-inaccessible-database.jpg)


### <a name="accidental-tde-protector-access-revocation"></a>Révocation accidentelle de l’accès au protecteur TDE

Il peut arriver qu’une personne disposant de droits d’accès suffisants au coffre de clés désactive accidentellement l’accès du serveur à la clé en :

- révoquant les autorisations *get*, *wrapKey*, *unwrapKey* du coffre de clés à partir du serveur

- supprimant la clé

- supprimant le coffre de clés

- modifiant les règles de pare-feu du coffre de clés

- en supprimant l’identité managée du serveur dans Azure Active Directory

En savoir plus sur [les causes courantes pour que la base de données devienne inaccessible](/sql/relational-databases/security/encryption/troubleshoot-tde?view=azuresqldb-current&preserve-view=true#common-errors-causing-databases-to-become-inaccessible).

## <a name="monitoring-of-the-customer-managed-tde"></a>Surveillance de TDE managé par le client

Pour surveiller l’état de la base de données et activer l’alerte pour la perte d’accès au protecteur TDE, configurez les fonctionnalités Azure suivantes :

- [Azure Resource Health](../../service-health/resource-health-overview.md). Une base de données inaccessible qui a perdu l’accès au protecteur TDE apparaît comme « Non disponible » après le refus de la première connexion à la base de données.
- [Journal d’activité](../../service-health/alerts-activity-log-service-notifications-portal.md) : lorsque l’accès au protecteur TDE dans le coffre de clés géré par le client échoue, des entrées sont ajoutées au Journal d’activité.  La création d’alertes pour ces événements vous permet de rétablir l’accès dès que possible.
- [Les groupes d’actions](../../azure-monitor/platform/action-groups.md) peuvent être définis de manière à vous envoyer des notifications et des alertes en fonction de vos préférences, par exemple par e-mail/SMS/envoi (push)/notification vocale, application logique, webhook, ITSM ou Runbook Automation.

## <a name="database-backup-and-restore-with-customer-managed-tde"></a>Sauvegarde et restauration de la base de données avec TDE managé par le client

Une fois qu’une base de données est chiffrée avec TDE à l’aide d’une clé du coffre de clés, toutes les sauvegardes nouvellement générées sont également chiffrées avec le même protecteur TDE. Lorsque le protecteur TDE est modifié, les anciennes sauvegardes de la base de données **ne sont pas mises à jour** pour l’utiliser le dernier protecteur TDE.

Pour restaurer une sauvegarde chiffrée avec un protecteur TDE de Key Vault, assurez-vous que le matériel de clé est toujours est disponible pour le serveur cible. Par conséquent, nous vous recommandons de conserver toutes les anciennes versions du protecteur TDE dans le coffre de clés, afin que toutes les sauvegardes de base de données puissent être restaurées.

> [!IMPORTANT]
> À tout moment, il ne peut pas y avoir plus d’un protecteur TDE défini pour un serveur. C’est la clé marquée avec « Faire de la clé le protecteur TDE par défaut » dans le panneau Portail Azure. Toutefois, plusieurs clés supplémentaires peuvent être liées à un serveur sans les marquer comme protecteur TDE. Ces clés ne sont pas utilisées pour la protection de la clé de chiffrement, mais elles peuvent être utilisées lors de la restauration à partir d’une sauvegarde, si le fichier de sauvegarde est chiffré avec la clé avec l’empreinte numérique correspondante.

Si la clé nécessaire à la restauration d’une sauvegarde n’est plus disponible sur le serveur cible, le message d’erreur suivant est renvoyé lors de la tentative de restauration : « Le serveur cible `<Servername>` n’a pas accès à tous les URI AKV créés entre \<Timestamp #1> et \<Timestamp #2>. Veuillez réessayer l’opération après avoir restauré tous les URI AKV. »

Pour l’atténuer, exécutez la cmdlet [Get-AzSqlServerKeyVaultKey](/powershell/module/az.sql/get-azsqlserverkeyvaultkey) pour le serveur cible ou [Get-AzSqlInstanceKeyVaultKey](/powershell/module/az.sql/get-azsqlinstancekeyvaultkey) pour l’instance gérée cible afin de retourner la liste des clés disponibles et d’identifier celles qui sont manquantes. Pour garantir la restauration possible de toutes les sauvegardes, vérifiez que le serveur cible pour la sauvegarde dispose d’un accès à toutes les clés nécessaires. Ces clés n’ont pas besoin d’être marquées comme protecteur TDE.

Afin d’en savoir plus sur la récupération d’une sauvegarde pour SQL Database, consultez [Récupérer une base de données dans SQL Database](recovery-using-backups.md). Pour en savoir plus sur la récupération de sauvegarde pour un pool SQL dédié dans Azure Synapse Analytics, consultez [Récupérer un pool SQL dédié](../../synapse-analytics/sql-data-warehouse/backup-and-restore.md). Pour la sauvegarde/restauration native de SQL Server avec SQL Managed Instance, consultez [Démarrage rapide : Restaurer une base de données sur SQL Managed Instance](../managed-instance/restore-sample-database-quickstart.md)

Attention particulière pour les fichiers journaux : Les fichiers journaux sauvegardés restent chiffrés avec le protecteur TDE d’origine, même s’il a subi une rotation et si la base de données utilise maintenant un nouveau protecteur TDE.  Lors d’une restauration, les deux clés seront nécessaires pour restaurer la base de données.  Si le fichier journal utilise un protecteur TDE stocké dans Azure Key Vault, cette clé sera nécessaire lors de la restauration, même si entretemps la base de données est passée à un TDE managé par le service.

## <a name="high-availability-with-customer-managed-tde"></a>Haute disponibilité avec TDE managé par le client

Même dans les cas où aucune géoredondance n’est configurée pour le serveur, il est fortement recommandé de configurer le serveur pour utiliser deux coffres de clés différents dans deux régions différentes, avec le même matériel de clé. Cela est possible en créant un protecteur TDE utilisant le coffre de clés primaires situé dans la même région que le serveur et en clonant la clé dans un coffre de clés situé dans une autre région Azure, afin que le serveur puisse accéder à un deuxième coffre de clés si le coffre de clés primaires venait à tomber en panne pendant que la base de données fonctionne correctement.

Utilisez la cmdlet Backup-AzKeyVaultKey pour récupérer la clé au format chiffré depuis le coffre de clés primaires, puis utilisez la cmdlet Restore-AzKeyVaultKey et spécifiez un coffre de clés dans la deuxième région pour cloner la clé. Vous pouvez également utiliser le portail Azure pour sauvegarder et restaurer la clé. La clé du coffre de clés secondaire dans l’autre région ne doit pas être marquée comme protecteur TDE et n’est même pas autorisée.

En cas de panne affectant le coffre de clés principal, et uniquement dans ce cas, le système bascule automatiquement vers l’autre clé liée avec la même empreinte numérique dans le coffre de clés secondaire, le cas échéant. Notez cependant que ce commutateur ne se produira pas si le protecteur TDE est inaccessible en raison de droits d’accès révoqués ou parce que la clé ou le coffre de clés est supprimé, car il peut indiquer que le client voulait intentionnellement empêcher le serveur d’accéder à la clé.

![Haute disponibilité de serveur unique](./media/transparent-data-encryption-byok-overview/customer-managed-tde-with-ha.png)

## <a name="geo-dr-and-customer-managed-tde"></a>Fonctionnement de TDE managé par le client et à récupération d'urgence de géoréplication

Dans les scénarios de [géo-réplication active](active-geo-replication-overview.md) et de [groupes de basculement](auto-failover-group-overview.md), chaque serveur impliqué requiert un coffre de clés distinct, qui doit être colocalisé avec le serveur dans la même région Azure. Le client est responsable du maintien de la cohérence du matériel de clé dans les coffres de clés, afin que la géo-secondaire soit synchronisée et puisse prendre le relais à l’aide de la même clé de son coffre de clés local si la base de données primaire devient inaccessible en raison d’une panne dans la région et qu’un basculement est déclenché. Jusqu’à quatre bases de données secondaires peuvent être configurées et le chaînage (bases de données secondaires de secondaires) n’est pas pris en charge.

Pour éviter tout problème lors de l’établissement ou de la géoréplication en raison d’un matériel de clé incomplet, il est important de suivre les règles suivantes lors de la configuration des TDE managés par le client :

- Tous les coffres de clés impliqués doivent avoir les mêmes propriétés, ainsi que les mêmes droits d’accès pour les serveurs respectifs.

- Tous les coffres de clés impliqués doivent contenir un matériau de clé identique. Il s’applique non seulement au protecteur TDE actuel, mais aussi à tous les protecteurs TDE précédents qui peuvent être utilisés dans les fichiers de sauvegarde.

- La configuration initiale et la rotation du protecteur TDE doivent d’abord être effectuées sur la base de données secondaire, puis sur la primaire.

![Groupes de basculement et géo-reprise](./media/transparent-data-encryption-byok-overview/customer-managed-tde-with-bcdr.png)

Pour tester un basculement, suivez les étapes décrites dans [Aperçu de la géo-réplication active](active-geo-replication-overview.md). Le test de basculement doit être effectué régulièrement pour confirmer que SQL Database a conservé l’autorisation d’accès aux deux coffres de clés.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également vérifier les exemples de scripts PowerShell suivants pour les opérations courantes avec TDE managé par le client :

- [Faites pivoter le protecteur Transparent Data Encryption pour SQL Database à l’aide de PowerShell](transparent-data-encryption-byok-key-rotation.md)

- [Supprimer un protecteur Transparent Data Encryption pour SQL Database à l’aide de PowerShell](transparent-data-encryption-byok-remove-tde-protector.md)

- [Gérer Transparent Data Encryption dans SQL Managed Instance avec votre propre clé à l’aide de PowerShell](../managed-instance/scripts/transparent-data-encryption-byok-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)
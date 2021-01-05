---
title: Chiffrement des données à l’aide d’une clé gérée par le client - Azure Database pour MySQL
description: Le chiffrement des données d'Azure Database pour MySQL à l'aide d'une clé gérée par le client permet le scénario Bring Your Own Key (BYOK) pour la protection des données au repos. Il permet également aux organisations d'implémenter la séparation des tâches dans la gestion des clés et des données.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 01/13/2020
ms.openlocfilehash: f9b9681b08f5864dc34bbf1c35dc6919129c24cb
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96518802"
---
# <a name="azure-database-for-mysql-data-encryption-with-a-customer-managed-key"></a>Chiffrement des données d'Azure Database pour MySQL à l'aide d'une clé gérée par le client

Le chiffrement des données d’Azure Database pour MySQL à l’aide d’une clé gérée par le client vous permet de mettre en place votre propre scénario Bring Your Own Key (BYOK) pour la protection des données au repos. Il permet également aux organisations d'implémenter la séparation des tâches dans la gestion des clés et des données. Avec le chiffrement géré par le client, vous êtes responsable du cycle de vie des clés, des autorisations d'utilisation des clés et de l'audit des opérations sur les clés, et contrôlez totalement le processus.

Le chiffrement des données d'Azure Database pour MySQL à l'aide d'une clé gérée par le client est défini au niveau du serveur. Pour un serveur donné, une clé gérée par le client, appelée clé de chiffrement de clé (KEK), sert à chiffrer la clé de chiffrement de données (DEK) utilisée par le service. La KEK est une clé asymétrique stockée dans une instance d'[Azure Key Vault](../key-vault/general/secure-your-key-vault.md) détenue et gérée par le client. La clé de chiffrement de clé (KEK) et la clé de chiffrement de données (DEK) sont décrites plus en détail plus loin dans cet article.

Key Vault est un système de gestion de clés externe basé sur le cloud. Il fournit un stockage sécurisé hautement disponible et évolutif pour les clés de chiffrement RSA, éventuellement sauvegardé par les modules de sécurité matériels (HSM) validés FIPS 140-2 niveau 2. Il n'autorise pas l'accès direct à une clé stockée, mais fournit des services de chiffrement et de déchiffrement aux entités autorisées. Key Vault peut générer la clé, l'importer ou [la faire transférer à partir d'un appareil HSM local](../key-vault/keys/hsm-protected-keys.md).

> [!NOTE]
> Cette fonctionnalité est disponible dans toutes les régions Azure où Azure Database pour MySQL prend en charge les niveaux tarifaires « Usage général » et « Mémoire optimisée ». Pour connaître les autres limitations, consultez la section [Limitation](concepts-data-encryption-mysql.md#limitations).

## <a name="benefits"></a>Avantages

Le chiffrement des données d’Azure Database pour MySQL à l’aide de clés gérées par le client offre les avantages suivants :

* L'accès aux données est entièrement contrôlé par vous puisque vous avez la possibilité de retirer la clé et de rendre la base de données inaccessible 
* Contrôle total sur le cycle de vie de la clé, y compris la rotation de la clé à aligner avec les stratégies d'entreprise
* Gestion centralisée et organisation des clés dans Azure Key Vault
* Possibilité d'implémenter une séparation des tâches entre les responsables de la sécurité, les administrateurs de base de données et les administrateurs système


## <a name="terminology-and-description"></a>Terminologie et description

**Clé de chiffrement de données (DEK)**  : clé symétrique AES256 utilisée pour chiffrer une partition ou un bloc de données. Le chiffrement de chaque bloc de données avec une clé différente rend les attaques d’analyse du chiffrement plus difficiles. L’accès aux clés de chiffrement des données est nécessaire au fournisseur de ressources ou à l’instance d’application qui chiffre et déchiffre un bloc spécifique. Lorsque vous remplacez une clé de chiffrement de données par une nouvelle clé, seules les données du bloc associé doivent être rechiffrées avec la nouvelle clé.

**Clé de chiffrement de clé (KEK)**  : clé de chiffrement utilisée pour chiffrer les clés de chiffrement de données. Une KEK qui ne quitte jamais Key Vault permet de chiffrer et de contrôler les KEK elles-mêmes. L'entité qui a accès à la clé de chiffrement de clé peut être différente de l'entité qui a besoin de la clé de chiffrement de données. Comme la clé de chiffrement des clés est nécessaire pour déchiffrer les clés de chiffrement des données, la clé de chiffrement des clés est dès lors le point unique par lequel les clés de chiffrement des données peuvent être supprimées en supprimant la clé de chiffrement des clés.

Les DEK, chiffrées avec les KEK, sont stockées séparément. Seule une entité ayant accès à la KEK peut déchiffrer ces DEK. Pour plus d'informations, consultez [Sécurité du chiffrement des données au repos](../security/fundamentals/encryption-atrest.md).

## <a name="how-data-encryption-with-a-customer-managed-key-work"></a>Fonctionnement du chiffrement des données à l'aide d'une clé gérée par le client

:::image type="content" source="media/concepts-data-access-and-security-data-encryption/mysqloverview.png" alt-text="Diagramme illustrant le scénario Bring Your Own Key (BYOK)":::

Pour qu'un serveur MySQL utilise les clés gérées par le client stockées dans Key Vault pour le chiffrement de la DEK, un administrateur Key Vault accorde les droits d'accès suivants au serveur :

* **get** : pour récupérer la partie publique et les propriétés de la clé dans Key Vault.
* **wrapKey** : pour pouvoir chiffrer la clé de chiffrement de données. La clé DEK chiffrée est stockée dans Azure Database pour MySQL.
* **unwrapKey** : pour pouvoir déchiffrer la clé de chiffrement de données. Azure Database pour MySQL a besoin de la clé DEK déchiffrée pour chiffrer/déchiffrer les données.

L'administrateur Key Vault peut également [activer la journalisation des événements d'audit Key Vault](../azure-monitor/insights/key-vault-insights-overview.md) afin qu'ils puissent être audités ultérieurement.

Lorsque le serveur est configuré pour utiliser la clé gérée par le client stockée dans le coffre de clés, le serveur envoie la clé de chiffrement de données au coffre de clés pour les chiffrements. Key Vault retourne la clé de chiffrement chiffrée, qui est stockée dans la base de données utilisateur. De même, le cas échéant, le serveur envoie les clés de chiffrement de données protégées au coffre de clés pour le déchiffrement. Les auditeurs peuvent utiliser Azure Monitor pour consulter les journaux d'événements d'audit Key Vault, si la journalisation est activée.

## <a name="requirements-for-configuring-data-encryption-for-azure-database-for-mysql"></a>Exigences pour la configuration du chiffrement des données pour Azure Database pour MySQL

Les exigences suivantes s'appliquent à la configuration de Key Vault :

* Key Vault et Azure Database pour MySQL doivent appartenir au même locataire Azure Active Directory (Azure AD). Les interactions entre un serveur et une instance inter-locataires de Key Vault ne sont pas prises en charge. Pour déplacer des ressources Key Vault par la suite, vous devez reconfigurer le chiffrement des données.
* Activez la fonctionnalité [soft-delete](../key-vault/general/soft-delete-overview.md) sur le coffre de clés avec une période de rétention définie à **90 jours**, afin de protéger contre la perte de données en cas de suppression d’une clé accidentelle (ou Key Vault). Les ressources supprimées de manière réversible sont conservées par défaut pendant 90 jours, sauf si la période de rétention est explicitement définie sur <=90 jours. Les actions de récupération et de vidage ont leurs propres autorisations associées dans une stratégie d’accès Key Vault. Par défaut, la fonctionnalité de suppression réversible est désactivée, mais vous pouvez l'activer via PowerShell ou Azure CLI (notez que vous ne pouvez pas l'activer via le portail Azure).
* Activez la fonctionnalité de [protection contre le vidage](../key-vault/general/soft-delete-overview.md#purge-protection) sur le coffre de clés avec la période de rétention définie sur **90 jours**. La protection contre le vidage ne peut être activée qu’une fois que la suppression réversible est activée. Elle peut être activée via Azure CLI ou PowerShell. Lorsque la protection contre le vidage est activée, il n’est pas possible de purger un coffre ou un objet à l’état supprimé avant la fin de la période de rétention de 90 jours. Après une suppression réversible, les coffres et objets restent récupérables pour garantir le respect de la stratégie de rétention. 
* Accordez l'accès au coffre de clés à Azure Database pour MySQL avec les autorisations get, wrapKey, unwrapKey en utilisant son identité managée unique. Sur le portail Azure, l’identité « Service » unique est automatiquement créée lorsque le chiffrement des données est activé sur MySQL. Consultez [Configurer le chiffrement des données pour MySQL](howto-data-encryption-portal.md) pour obtenir des instructions pas à pas détaillées lorsque vous utilisez le portail Azure.

Les exigences suivantes s'appliquent à la configuration de la clé gérée par le client :

* La clé gérée par le client à utiliser pour chiffrer la clé de chiffrement de données ne peut être qu’asymétrique, RSA 2048.
* La date d’activation de la clé (si définie) doit être une date et une heure passées. La date d’expiration n’est pas définie.
* La clé doit être dans l’état *activé*.
* La clé doit avoir une [suppression réversible](../key-vault/general/soft-delete-overview.md) avec la période de rétention définie sur **90 jours**. Cela définit implicitement l’attribut de clé requis recoveryLevel : « Recoverable ». Si la rétention est définie sur < 90 jours, l’attribut recoveryLevel: « CustomizedRecoverable », n’est pas obligatoire et vous devez définir la période de rétention sur **90 jours**.
* La clé doit avec la [protection contre le vidage activée](../key-vault/general/soft-delete-overview.md#purge-protection).
* Si vous [importez une clé existante](/rest/api/keyvault/ImportKey/ImportKey) dans le coffre de clés, veillez à ce qu’elle respecte les formats de fichiers pris en charge (`.pfx`, `.byok`, `.backup`).

## <a name="recommendations"></a>Recommandations

Si vous utilisez le chiffrement de données à l'aide d'une clé gérée par le client, suivez les recommandations ci-dessous pour configurer Key Vault :

* Définissez un verrou de ressource sur le coffre de clés pour déterminer qui peut supprimer cette ressource critique et pour empêcher toute suppression accidentelle ou non autorisée.
* Activez l'audit et la création de rapports sur toutes les clés de chiffrement. Key Vault fournit des journaux d’activité faciles à injecter dans d’autres outils de gestion d’événements et d’informations de sécurité. Azure Monitor Log Analytics est un exemple de service déjà intégré.
* Assurez-vous que Key Vault et Azure Database pour MySQL résident dans la même région pour garantir un accès plus rapide aux opérations wrap et unwrap des clés de chiffrement de données.
* Verrouillez l’Azure Key Vault uniquement pour **le point de terminaison privé et les réseaux sélectionnés**, et autorisez uniquement des services *Microsoft approuvés* à sécuriser les ressources.

    :::image type="content" source="media/concepts-data-access-and-security-data-encryption/keyvault-trusted-service.png" alt-text="trusted-service-with-AKV":::

Suivez les recommandations ci-dessous pour configurer une clé gérée par le client :

* Conservez une copie de la clé gérée par le client à un emplacement sécurisé ou déposez-la au service de dépôt.

* Si Key Vault génère la clé, créez une sauvegarde de celle-ci avant sa première utilisation. La sauvegarde ne peut être restaurée que dans Key Vault. Pour plus d'informations sur la commande de sauvegarde, consultez [Backup-AzKeyVaultKey](/powershell/module/az.keyVault/backup-azkeyVaultkey).

## <a name="inaccessible-customer-managed-key-condition"></a>Condition de clé managée par le client inaccessible

Lorsque vous configurez le chiffrement des données avec une clé gérée par le client dans Key Vault, un accès continu à cette clé est requis pour que le serveur reste en ligne. Si le serveur perd l'accès à la clé gérée par le client dans Key Vault, il commence à refuser toutes les connexions dans un délai de 10 minutes. Le serveur émet un message d'erreur et affiche l'état *Inaccessible*. Voici des raisons pour lesquelles le serveur peut atteindre cet état :

* Si nous créons un serveur de limite de restauration dans le temps pour votre base de données Azure pour MySQL pour laquelle le chiffrement des données est activé, le serveur créé se trouve dans l’état *Inaccessible*. Vous pouvez résoudre ce problème à l’aide du [portail Azure](howto-data-encryption-portal.md#using-data-encryption-for-restore-or-replica-servers) ou de l’[interface CLI](howto-data-encryption-cli.md#using-data-encryption-for-restore-or-replica-servers).
* Si nous créons un réplica en lecture pour votre base de données Azure pour MySQL pour lequel le chiffrement des données est activé, le serveur de réplication se trouve dans l’état *Inaccessible*. Vous pouvez résoudre ce problème à l’aide du [portail Azure](howto-data-encryption-portal.md#using-data-encryption-for-restore-or-replica-servers) ou de l’[interface CLI](howto-data-encryption-cli.md#using-data-encryption-for-restore-or-replica-servers).
* Si vous supprimez le coffre de clés, la base de données Azure pour MySQL ne peut plus accéder à la clé et passe à l’état *Inaccessible*. Récupérez le [Key Vault](../key-vault/general/key-vault-recovery.md) et revalidez le chiffrement des données pour rendre le serveur *Disponible*.
* Si nous supprimons la clé du coffre de clés, la base de données Azure pour MySQL ne peut plus accéder à la clé et passe à l’état *Inaccessible*. Récupérez la [clé](../key-vault/general/key-vault-recovery.md) et revalidez le chiffrement des données pour rendre le serveur *Disponible*.
* Si la clé stockée dans le coffre de clés Azure expire, elle devient non valide et la base de données Azure pour MySQL passe à l’état *Inaccessible*. Étendez la date d’expiration de la clé à l’aide de [CLI](/cli/azure/keyvault/key#az-keyvault-key-set-attributes), puis revalidez le chiffrement de données pour rendre le serveur *Disponible*.

### <a name="accidental-key-access-revocation-from-key-vault"></a>Révocation accidentelle de l'accès aux clés de Key Vault

Il peut arriver qu'une personne disposant de droits d'accès suffisants à Key Vault désactive accidentellement l'accès du serveur à la clé en :

* révoquant les autorisations get, wrapKey, unwrapKey du coffre de clés à partir du serveur ;
* supprimant la clé ;
* supprimant le coffre de clés ;
* modifiant les règles de pare-feu du coffre de clés ;
* supprimant l'identité managée du serveur dans Azure AD.

## <a name="monitor-the-customer-managed-key-in-key-vault"></a>Surveiller la clé gérée par le client dans Key Vault

Pour surveiller l'état de la base de données et activer les alertes liées à la perte d'accès au protecteur TDE (Transparent Data Encryption), configurez les fonctionnalités Azure suivantes :

* [Azure Resource Health](../service-health/resource-health-overview.md) : une base de données inaccessible qui a perdu l'accès à la clé client apparaît comme « Inaccessible » après le refus de la première connexion à la base de données.
* [Journal d’activité](../service-health/alerts-activity-log-service-notifications-portal.md) : lorsque l'accès à la clé client dans le coffre de clés géré par le client échoue, des entrées sont ajoutées au journal d'activité. La création d'alertes pour ces événements vous permettra de rétablir l'accès dès que possible.

* [Groupes d'actions](../azure-monitor/platform/action-groups.md) : définissez ces groupes pour qu'ils vous envoient des notifications et des alertes basées sur vos préférences.

## <a name="restore-and-replicate-with-a-customers-managed-key-in-key-vault"></a>Restaurer et répliquer à l'aide d'une clé gérée par un client dans Key Vault

Une fois Azure Database pour MySQL chiffré à l'aide d'une clé gérée par le client stockée dans Key Vault, toute copie nouvellement créée du serveur est également chiffrée. Vous pouvez effectuer cette nouvelle copie par l'intermédiaire d'une opération locale ou de restauration géographique, ou par le biais de réplicas en lecture. La copie peut cependant être modifiée afin de refléter la clé gérée du nouveau client pour le chiffrement. Lorsque la clé gérée par le client est modifiée, les anciennes sauvegardes du serveur utilisent la clé la plus récente.

Pour éviter les problèmes lors de la configuration du chiffrement des données géré par le client, pendant la restauration ou la création d’un réplica de lecture, il est important de suivre les étapes ci-dessous sur le serveur source et les serveurs de restauration/réplication :

* Initiez le processus de création de réplicas en lecture ou de restauration à partir du serveur source Azure Database pour MySQL.
* Veillez à ce que le serveur nouvellement créé (restauré/réplica) reste inaccessible car son identité unique n'a pas encore reçu les autorisations d'accès à Key Vault.
* Sur le serveur restauré/réplica, revalidez la clé managée par le client dans les paramètres de chiffrement des données pour vous assurer que le serveur créé reçoit les autorisations wrap et unwrap sur la clé stockée dans Key Vault.

## <a name="limitations"></a>Limites

Pour Azure Database pour MySQL, la prise en charge du chiffrement des données au repos à l’aide de Customer Managed Key (CMK) présente peu de restrictions.

* La prise en charge de cette fonctionnalité est limitée aux niveaux tarifaires **Usage général** et **À mémoire optimisée**.
* Cette fonctionnalité est uniquement prise en charge dans les régions et les serveurs qui prennent en charge jusqu’à 16 To de stockage. Pour obtenir la liste des régions Azure qui prennent en charge le stockage jusqu’à 16 To, reportez-vous à la section consacrée au stockage dans la documentation [ici](concepts-pricing-tiers.md#storage)

    > [!NOTE]
    > - Pour tous les nouveaux serveurs MySQL créés dans les régions mentionnées ci-dessus, la prise en charge du chiffrement avec les clés CMK est **disponible**. En théorie, le serveur de restauration jusqu’à une date et une heure (PITR) ou le réplica de lecture ne sont pas signalés comme des ressources « nouvelles ».
    > - Pour vérifier si votre serveur approvisionné prend en charge jusqu’à 16 To, vous pouvez accéder au tableau de bord de niveau tarifaire dans le portail et voir la taille de stockage maximale prise en charge par votre serveur approvisionné. Si vous pouvez déplacer le curseur jusqu’à 4 To, il se peut que votre serveur ne prenne pas en charge le chiffrement avec les clés managées par le client. Toutefois, les données sont chiffrées à l’aide de clés managées par le service à tout moment. Posez vos questions en contactant AskAzureDBforMySQL@service.microsoft.com.

* Le chiffrement est pris en charge uniquement avec la clé de chiffrement RSA 2048.

## <a name="next-steps"></a>Étapes suivantes

Apprenez à configurer le chiffrement des données à l'aide d'une clé gérée par le client pour votre instance d'Azure Database pour MySQL à l'aide du [Portail Azure](howto-data-encryption-portal.md) et d’[Azure CLI](howto-data-encryption-cli.md).

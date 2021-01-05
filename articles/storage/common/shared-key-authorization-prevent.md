---
title: Empêcher l’autorisation avec clé partagée (préversion)
titleSuffix: Azure Storage
description: Pour exiger des clients qu’ils utilisent Azure AD pour autoriser les demandes, vous pouvez désactiver les demandes adressées au compte de stockage qui sont autorisées avec une clé partagée (préversion).
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/07/2020
ms.author: tamram
ms.reviewer: fryu
ms.openlocfilehash: 6a24713a6027c38d2b9817928f3a82161bd37314
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96936724"
---
# <a name="prevent-shared-key-authorization-for-an-azure-storage-account-preview"></a>Empêcher l’autorisation avec clé partagée pour un compte de stockage Azure (préversion)

Chaque demande sécurisée adressée à un compte Stockage Azure doit être autorisée. Par défaut, les demandes peuvent être autorisées soit avec des informations d’identification Azure Active Directory (Azure AD), soit à l’aide de la clé d’accès au compte pour l’autorisation avec clé partagée. Entre ces deux types d’autorisation, Azure AD offre une sécurité et une facilité d’utilisation supérieures par rapport à une clé partagée, et est recommandé par Microsoft. Pour exiger des clients qu’ils utilisent Azure AD pour autoriser les demandes, vous pouvez désactiver les demandes adressées au compte de stockage qui sont autorisées avec une clé partagée (préversion).

Lorsque vous désactivez l’autorisation avec clé partagée pour un compte de stockage, le service Stockage Azure rejette toutes les demandes ultérieures adressées à ce compte, qui sont autorisées avec les clés d’accès au compte. Seules les demandes sécurisées autorisées avec Azure AD aboutissent. Pour plus d’informations sur l’utilisation d’Azure AD, consultez [Autoriser l’accès aux objets blob et aux files d’attente avec Azure Active Directory](storage-auth-aad.md).

> [!WARNING]
> Le service Stockage Azure prend en charge l’autorisation Azure AD uniquement pour les demandes adressées au stockage de blob et de file d’attente. Si vous désactivez l’autorisation avec clé partagée pour un compte de stockage, les demandes adressées aux services Azure Files ou Stockage Table qui utilisent une autorisation avec clé partagée échouent. Étant donné que le portail Azure utilise toujours l’autorisation par clé partagée pour accéder aux données de fichier et de table, si vous interdisez l’autorisation avec une clé partagée pour le compte de stockage, vous ne pourrez pas accéder aux données de fichier ni de table dans le portail Azure.
>
> Microsoft recommande soit de migrer les données des services Azure Files ou Stockage Table vers un compte de stockage séparé avant de désactiver l’accès au compte avec une clé partagée, soit de ne pas appliquer ce paramètre aux comptes de stockage qui prennent en charge les charges de travail des services Azure Files ou Stockage Table.
>
> La désactivation de l’accès avec clé partagée pour un compte de stockage n’affecte pas les connexions SMB à Azure Files.

Cet article explique comment détecter les demandes envoyées avec une autorisation avec clé partagée, et comment corriger l’autorisation avec clé partagée pour votre compte de stockage. Pour savoir comment vous inscrire à la préversion, consultez [À propos de la préversion](#about-the-preview).

## <a name="detect-the-type-of-authorization-used-by-client-applications"></a>Détecter le type d’autorisation utilisé par les applications clientes

Lorsque vous désactivez l’autorisation avec clé partagée pour un compte de stockage, les demandes des clients qui utilisent les clés d’accès au compte pour l’autorisation avec clé partagée échouent. Pour comprendre comment la désactivation de l’autorisation avec clé partagée peut affecter les applications clientes avant d’apporter cette modification, activez la journalisation et les métriques du compte de stockage. Vous pouvez ensuite analyser les modèles des demandes adressées à votre compte sur une période donnée pour déterminer la manière dont les demandes sont autorisées.

Utilisez des métriques pour déterminer le nombre de demandes que le compte de stockage reçoit et qui sont autorisées avec clé partagée ou signature d’accès partagé (SAP). Utilisez des journaux pour déterminer les clients qui envoient ces demandes.

Pour plus d’informations sur l’interprétation des demandes effectuées avec une signature d’accès partagé au cours de la période de préversion, consultez [À propos de la préversion](#about-the-preview).

### <a name="monitor-how-many-requests-are-authorized-with-shared-key"></a>Surveiller le nombre de demandes autorisées avec clé partagée

Pour suivre la manière dont les demandes adressées à un compte de stockage sont autorisées, utilisez Azure Metrics Explorer dans le portail Azure. Pour plus d’informations sur Metrics Explorer, consultez [Prise en main d’Azure Metrics Explorer](../../azure-monitor/platform/metrics-getting-started.md).

Pour créer une métrique qui suit des demandes effectuées avec clé partagée ou signature d’accès partagé, procédez comme suit :

1. Accédez à votre compte de stockage dans le portail Azure. Dans la section **Supervision**, sélectionnez **Métriques**.
1. Sélectionnez **Ajouter une métrique**. Dans la boîte de dialogue **Métrique**, spécifiez les valeurs suivantes :
    1. Laissez le champ **Étendue** défini sur le nom du compte de stockage.
    1. Définissez l’**Espace de noms de métrique** sur *Compte*. Cette métrique signale toutes les demandes effectuées sur le compte de stockage.
    1. Définissez le champ **Métrique** sur *Transactions*.
    1. Définissez le champ **Agrégation** sur *Somme*.

    La nouvelle métrique affiche le nombre total de transactions effectuées sur le compte de stockage pendant un intervalle de temps donné. La métrique qui en résulte s’affiche comme indiqué dans l’image suivante :

    :::image type="content" source="media/shared-key-authorization-prevent/configure-metric-account-transactions.png" alt-text="Capture d’écran montrant comment configurer la métrique pour additionner les transactions effectuées avec clé partagée ou signature d’accès partagé":::

1. Ensuite, sélectionnez le bouton **Ajouter un filtre** afin de créer un filtre sur la métrique pour le type d’autorisation.
1. Dans la boîte de dialogue **Filtre**, spécifiez les valeurs suivantes :
    1. Définissez la valeur **Propriété** sur *Authentification*.
    1. Affectez au champ **Opérateur** le signe égal (=).
    1. Dans le champ **Valeurs**, sélectionnez *Clé de compte*, puis *SAP*.
1. Dans l’angle supérieur droit, sélectionnez l’intervalle de temps pour lequel afficher la métrique. Vous pouvez également indiquer le degré de précision de l’agrégation des demandes, en spécifiant des intervalles compris entre 1 minute et 1 mois. Par exemple, définissez l’**Intervalle de temps** sur 30 jours et la **Granularité temporelle** sur 1 jour pour afficher les demandes agrégées par jour au cours des 30 derniers jours.

Une fois que vous avez configuré la métrique, les demandes adressées à votre compte de stockage commencent à s’afficher sur le graphique. L’illustration suivante montre les demandes autorisées avec une clé partagée ou effectuées avec un jeton SAP. Les demandes sont agrégées par jour au cours des 30 derniers jours.

:::image type="content" source="media/shared-key-authorization-prevent/metric-shared-key-requests.png" alt-text="Capture d’écran montrant les demandes agrégées autorisées avec une clé partagée":::

Vous pouvez également configurer une règle d’alerte pour vous avertir quand un certain nombre de demandes autorisées avec une clé partagée sont effectuées sur votre compte de stockage. Pour plus d'informations, consultez [Créer, afficher et gérer des alertes de métrique à l'aide d'Azure Monitor](../../azure-monitor/platform/alerts-metric.md).

### <a name="analyze-logs-to-identify-clients-that-are-authorizing-requests-with-shared-key-or-sas"></a>Analyser les journaux pour identifier les clients qui autorisent les demandes avec clé partagée ou signature d’accès partagé

Les journaux du stockage Azure capturent des détails sur les demandes effectuées sur le compte de stockage, y compris la manière dont une demande a été autorisée. Vous pouvez analyser les journaux pour identifier les clients qui autorisent les demandes avec clé partagée ou jeton SAP.

Pour journaliser les demandes adressées à votre compte Stockage Azure afin d’évaluer la manière dont elles sont autorisées, vous pouvez utiliser la journalisation du Stockage Azure dans Azure Monitor (préversion). Pour plus d’informations, consultez [Superviser le stockage Azure](../blobs/monitor-blob-storage.md).

La journalisation du stockage Azure dans Azure Monitor prend en charge l’utilisation de requêtes de journal pour analyser les données des journaux. Pour interroger les journaux, vous pouvez utiliser un espace de travail Azure Log Analytics. Pour en savoir plus sur les requêtes de journal, consultez [Tutoriel : Bien démarrer avec les requêtes Log Analytics](../../azure-monitor/log-query/log-analytics-tutorial.md).

#### <a name="create-a-diagnostic-setting-in-the-azure-portal"></a>Créer un paramètre de diagnostic dans le portail Azure

Pour journaliser des données de stockage Azure avec Azure Monitor et les analyser avec Azure Log Analytics, vous devez d’abord créer un paramètre de diagnostic qui indique les types de demandes et les services de stockage pour lesquels vous souhaitez journaliser les données. Pour créer un paramètre de diagnostic dans le portail Azure, suivez ces étapes :

1. Inscrivez-vous à la [journalisation de stockage Azure dans Azure Monitor (préversion)](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxW65f1VQyNCuBHMIMBV8qlUM0E0MFdPRFpOVTRYVklDSE1WUTcyTVAwOC4u).
1. Créez un espace de travail Log Analytics dans l’abonnement contenant votre compte Stockage Azure, ou utilisez un espace de travail Log Analytics existant. Une fois que vous avez configuré la journalisation pour votre compte de stockage, les journaux sont disponibles dans l’espace de travail Log Analytics. Pour plus d’informations, consultez [Créer un espace de travail Log Analytics dans le portail Azure](../../azure-monitor/learn/quick-create-workspace.md).
1. Accédez à votre compte de stockage dans le portail Azure.
1. Dans la section Supervision, Sélectionnez **Paramètres de diagnostic (préversion)** .
1. Sélectionnez le service de stockage Azure pour lequel vous souhaitez journaliser les demandes. Par exemple, choisissez **Blob** pour journaliser les demandes dans le stockage Blob.
1. Sélectionnez **Ajouter le paramètre de diagnostic**.
1. Fournissez un nom pour le paramètre de diagnostic.
1. Sous **Détails de la catégorie**, dans la section du **journal**, choisissez **StorageRead**, **StorageWrite** et **StorageDelete** pour journaliser toutes les demandes de données adressées au service sélectionné.
1. Sous **Détails de la destination**, sélectionnez **Envoyer à Log Analytics**. Sélectionnez votre abonnement et l’espace de travail Log Analytics que vous avez créé, comme illustré dans l’image suivante.

    :::image type="content" source="media/shared-key-authorization-prevent/create-diagnostic-setting-logs.png" alt-text="Capture d’écran montrant comment créer un paramètre de diagnostic pour la journalisation des demandes":::

Vous pouvez créer un paramètre de diagnostic pour chaque type de ressource Stockage Azure dans votre compte de stockage.

Une fois le paramètre de diagnostic créé, les demandes adressées au compte de stockage sont journalisées conformément à ce paramètre. Pour plus d’informations, consultez [Créer un paramètre de diagnostic pour collecter les journaux et les métriques des ressources dans Azure](../../azure-monitor/platform/diagnostic-settings.md).

Pour obtenir des informations de référence sur les champs disponibles dans les journaux de stockage Azure dans Azure Monitor, consultez [Journaux de ressource (préversion)](../blobs/monitor-blob-storage-reference.md#resource-logs-preview).

#### <a name="query-logs-for-requests-made-with-shared-key-or-sas"></a>Interroger les journaux concernant les demandes effectuées avec clé partagée ou signature d’accès partagé

Les journaux de stockage Azure dans Azure Monitor incluent le type d’autorisation qui a été utilisé pour effectuer une demande à destination d’un compte de stockage. Pour récupérer les journaux des demandes effectuées au cours des sept derniers jours qui ont été autorisées avec clé partagée ou signature d’accès partagé, ouvrez votre espace de travail Log Analytics. Collez ensuite la requête suivante dans une nouvelle requête de journal et exécutez-la. Cette requête affiche les dix adresses IP qui ont envoyé le plus fréquemment des demandes autorisées avec clé partagée ou signature d’accès partagé :

```kusto
StorageBlobLogs
| where AuthenticationType in ("AccountKey", "SAS") and TimeGenerated > ago(7d)
| summarize count() by CallerIpAddress, UserAgentHeader, AccountName
| top 10 by count_ desc
```

Vous pouvez également configurer une règle d’alerte basée sur cette requête pour être informé des demandes autorisées avec clé partagée ou signature d’accès partagé. Pour plus d’informations, consultez [Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../../azure-monitor/platform/alerts-log.md).

## <a name="remediate-authorization-via-shared-key"></a>Corriger une autorisation via une clé partagée

Une fois que vous avez analysé la manière dont les demandes adressées à votre compte de stockage sont autorisées, vous pouvez agir pour empêcher l’accès via une clé partagée. Mais tout d’abord, vous devez mettre à jour les applications qui utilisent l’autorisation avec clé partagée pour utiliser Azure AD à la place. Pour suivre la transition, vous pouvez surveiller les journaux et les métriques, comme décrit dans [Détecter le type d’autorisation utilisé par les applications clientes](#detect-the-type-of-authorization-used-by-client-applications). Pour plus d’informations sur l’utilisation d’Azure AD avec des données de blob et de file d’attente, consultez [Autoriser l’accès aux objets blob et aux files d’attente avec Azure Active Directory](storage-auth-aad.md).

Lorsque vous êtes certain de pouvoir rejeter en toute sécurité des demandes autorisées avec clé partagée, vous pouvez définir la propriété **AllowSharedKeyAccess** pour le compte de stockage sur la valeur **false**.

La propriété **AllowBlobPublicAccess** n’est pas définie par défaut et ne retourne pas de valeur tant que vous ne la définissez pas explicitement. Le compte de stockage accepte les demandes autorisées avec clé partagée lorsque la valeur de la propriété est **null** ou **true**.

> [!WARNING]
> Si des clients accèdent actuellement aux données de votre compte de stockage avec une clé partagée, Microsoft recommande de migrer ces clients vers Azure AD avant de désactiver l’accès avec clé partagée au compte de stockage.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour désactiver l’autorisation avec clé partagée pour un compte de stockage dans le portail Azure, procédez comme suit :

1. Accédez à votre compte de stockage dans le portail Azure.
1. Localisez le paramètre **Configuration** sous **Paramètres**.
1. Définissez **Autoriser l’accès avec clé partagée** sur **Désactivé**.

    :::image type="content" source="media/shared-key-authorization-prevent/shared-key-access-portal.png" alt-text="Capture d’écran montrant comment désactiver l’accès avec clé partagée pour un compte":::

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour désactiver l’autorisation avec clé partagée pour un compte de stockage à l’aide d’Azure CLI, installez Azure CLI version 2.9.1 ou ultérieure. Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](/cli/azure/install-azure-cli). Ensuite, configurez la propriété **allowBlobPublicAccess** pour un compte de stockage nouveau ou existant.

L’exemple suivant montre comment définir la propriété **allowSharedKeyAccess** avec Azure CLI. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
$storage_account_id=$(az resource show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --resource-type Microsoft.Storage/storageAccounts \
    --query id \
    --output tsv)

az resource update \
    --ids $storage_account_id \
    --set properties.allowSharedKeyAccess=false

az resource show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --resource-type Microsoft.Storage/storageAccounts \
    --query properties.allowSharedKeyAccess \
    --output tsv
```

---

Une fois que vous avez désactivé l’autorisation avec clé partagée, toute demande adressée au compte de stockage avec une autorisation avec clé partagée échoue avec le code d’erreur 403 (Interdit). Le service Stockage Azure renvoie une erreur indiquant que l’autorisation basée sur une clé n’est pas acceptée sur le compte de stockage.

### <a name="verify-that-shared-key-access-is-not-allowed"></a>Vérifier que l’accès avec clé partagée n’est pas autorisé

Pour vérifier que l’autorisation avec clé partagée n’est plus acceptée, vous pouvez essayer d’appeler une opération de données avec la clé d’accès au compte. L’exemple suivant tente de créer un conteneur à l’aide de la clé d’accès. Cet appel échoue quand l’autorisation avec clé partagée est désactivée pour le compte de stockage. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az storage container create \
    --account-name <storage-account> \
    --name sample-container \
    --account-key <key>
    --auth-mode key
```

> [!NOTE]
> Les demandes anonymes ne sont pas autorisées et continuent si vous avez configuré le compte de stockage et le conteneur pour l’accès en lecture public anonyme. Pour en savoir plus, consultez la section [Configure anonymous public read access for containers and blobs](../blobs/anonymous-read-access-configure.md) (Configurer l’accès en lecture publique anonyme pour les conteneurs et les objets blob).

### <a name="check-the-shared-key-access-setting-for-multiple-accounts"></a>Vérifier le paramètre d’accès avec clé partagée pour plusieurs comptes

Pour vérifier le paramètre d’accès avec clé partagée d’un ensemble de comptes de stockage avec des performances optimales, vous pouvez utiliser l’Explorateur Azure Resource Graph dans le portail Azure. Pour en savoir plus sur l’utilisation de l’Explorateur Resource Graph, consultez [Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de l’Explorateur Azure Resource Graph](../../governance/resource-graph/first-query-portal.md).

L’exécution de la requête suivante dans l’Explorateur Resource Graph retourne une liste de comptes de stockage, et affiche le paramètre d’accès avec clé partagée pour chaque compte :

```kusto
resources
| where type =~ 'Microsoft.Storage/storageAccounts'
| extend allowSharedKeyAccess = parse_json(properties).allowSharedKeyAccess
| project subscriptionId, resourceGroup, name, allowSharedKeyAccess
```

## <a name="permissions-for-allowing-or-disallowing-shared-key-access"></a>Autorisations pour autoriser ou interdire l’accès avec clé partagée

Afin de définir la propriété **AllowSharedKeyAccess** pour le compte de stockage, un utilisateur doit disposer des autorisations nécessaires pour créer et gérer des comptes de stockage. Les rôles de contrôle d’accès en fonction du rôle Azure (Azure RBAC) qui fournissent ces autorisations incluent l’action **Microsoft.Storage/storageAccounts/write** ou **Microsoft.Storage/storageAccounts/\** _. Les rôles intégrés à cette action comprennent :

- Le rôle [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) d’Azure Resource Manager
- Le rôle [Contributeur](../../role-based-access-control/built-in-roles.md#contributor) d’Azure Resource Manager
- Le rôle [Contributeur de compte de stockage](../../role-based-access-control/built-in-roles.md#storage-account-contributor)

Ces rôles ne fournissent pas d’accès aux données d’un compte de stockage par le biais d’Azure Active Directory (Azure AD). Toutefois, ils incluent l’action _*Microsoft.Storage/storageAccounts/listkeys/action**, qui accorde l’accès aux clés d’accès du compte. Avec cette autorisation, un utilisateur peut utiliser les clés d’accès du compte pour accéder à toutes les données d’un compte de stockage.

Les attributions de rôles doivent être définies au niveau du compte de stockage ou à un niveau supérieur pour permettre à un utilisateur d’autoriser ou d’interdire l’accès avec clé partagée au compte de stockage. Pour plus d’informations sur l’étendue des rôles, consultez [Comprendre l’étendue pour Azure RBAC](../../role-based-access-control/scope-overview.md).

Veillez à limiter l’attribution de ces rôles aux seules personnes qui ont besoin de créer un compte de stockage ou de mettre à jour ses propriétés. Utilisez le principe du moindre privilège pour vous assurer que les utilisateurs disposent des autorisations minimales nécessaires pour accomplir leurs tâches. Pour plus d’informations sur la gestion de l’accès avec Azure RBAC, consultez [Bonnes pratiques pour Azure RBAC](../../role-based-access-control/best-practices.md).

> [!NOTE]
> Les rôles d’administrateur d’abonnement classique Administrateur de service et Co-administrateur incluent l’équivalent du rôle [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) d’Azure Resource Manager. Le rôle **Propriétaire** comprend toutes les actions, de sorte qu’un utilisateur disposant de l’un de ces rôles d’administration peut également créer et gérer des comptes de stockage. Pour plus d’informations, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles d’administrateur Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles).

## <a name="understand-how-disallowing-shared-key-affects-sas-tokens"></a>Comprendre comment la désactivation de la clé partagée affecte les jetons SAP

Quand l’accès avec clé partagée est désactivé pour le compte de stockage, le service Stockage Azure gère les jetons SAS en fonction du type de signature d’accès partagé et du service ciblé par la demande. Le tableau suivant montre comment chaque type de signature d’accès partagé est autorisé et comment le service Storage Azure gère cette SAP quand la propriété **AllowSharedKeyAccess** pour le compte de stockage est définie sur **false**.

| Type de SAP | Type d’autorisation | Comportement lorsque la propriété AllowSharedKeyAccess est définie sur false |
|-|-|-|
| SAP de délégation d’utilisateur (Stockage Blob uniquement) | Azure AD | La demande est autorisée. Microsoft recommande d’utiliser une SAP de délégation d’utilisateur dans la mesure du possible pour une plus grande sécurité. |
| SAP de service | Clé partagée | La demande est refusée pour tous les services de stockage Azure. |
| SAP de compte | Clé partagée | La demande est refusée pour tous les services de stockage Azure. |

Pour plus d’informations sur les signatures d’accès partagé, consultez [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAP)](storage-sas-overview.md).

## <a name="consider-compatibility-with-other-azure-tools-and-services"></a>Prendre en compte la compatibilité avec d’autres outils et services Azure

Plusieurs services Azure utilisent une autorisation avec clé partagée pour communiquer avec le service Stockage Azure. Si vous désactivez l’autorisation avec clé partagée pour un compte de stockage, ces services ne sont pas en mesure d’accéder aux données de ce compte, et cela risque d’affecter vos applications.

Certains outils Azure offrent la possibilité d’utiliser une autorisation Azure AD pour accéder au service Stockage Azure. Le tableau suivant répertorie certains outils Azure populaires, et indique s’ils peuvent utiliser Azure AD pour autoriser les demandes adressées au service Stockage Azure.

| Outil Azure | Autorisation Azure AD pour l’accès au service Stockage Azure |
|-|-|
| Portail Azure | Pris en charge. Pour plus d’informations sur l’autorisation avec votre compte Azure AD à partir du portail Azure, consultez [Choisir comment autoriser l’accès à des données de blob dans le portail Azure](../blobs/authorize-data-operations-portal.md). |
| AzCopy | Opération prise en charge pour le stockage blob. Pour plus d’informations sur l’autorisation des opérations AzCopy, consultez [Choisir comment vous allez fournir des informations d’identification d’autorisation](storage-use-azcopy-v10.md#choose-how-youll-provide-authorization-credentials) dans la documentation AzCopy. |
| Explorateur de stockage Azure | Pris en charge uniquement pour le stockage Slob et Azure Data Lake Storage Gen2. L’accès Azure AD au stockage de file d’attente n’est pas pris en charge. Veillez à sélectionner le locataires Azure AD approprié. Pour plus d’informations, consultez [Prise en main de l’Explorateur Stockage](../../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows#sign-in-to-azure). |
| Azure PowerShell | Pris en charge. Pour plus d’informations sur l’autorisation des commandes PowerShell pour les opérations d’objet blob ou de file d’attente avec Azure AD, consultez [Exécuter des commandes PowerShell avec des informations d’identification Azure AD pour accéder aux données d’objet blob](../blobs/authorize-data-operations-powershell.md) ou [Exécuter des commandes PowerShell avec des informations d’identification Azure AD pour accéder aux données de la file d’attente](../queues/authorize-data-operations-powershell.md). |
| Azure CLI | Pris en charge. Pour plus d’informations sur la manière d’autoriser des commandes Azure CLI avec Azure AD pour l’accès aux données de blob et de file d’attente, consultez [Exécuter des commandes Azure CLI avec des informations d’identification Azure AD pour accéder aux données d’objet blob ou de file d’attente](../blobs/authorize-data-operations-cli.md). |
| Azure IoT Hub | Pris en charge. Pour plus d’informations, consultez [Prise en charge d’IoT Hub pour les réseaux virtuels](../../iot-hub/virtual-network-support.md). |
| Azure Cloud Shell | Azure Cloud Shell est un interpréteur de commandes intégré dans le portail Azure. Azure Cloud Shell héberge des fichiers à des fins de persistance dans un partage de fichiers Azure dans un compte de stockage. Ces fichiers deviennent inaccessibles si l’autorisation avec clé partagée est désactivée pour ce compte de stockage. Pour plus d’informations, consultez [Connecter votre stockage Microsoft Azure Files](../../cloud-shell/overview.md#connect-your-microsoft-azure-files-storage). <br /><br /> Pour exécuter des commandes dans Azure Cloud Shell afin de gérer les comptes de stockage pour lesquels l’accès avec clé partagée est désactivé, commencez par vérifier que vous disposez des autorisations nécessaires pour ces comptes via Azure RBAC. Pour plus d’informations, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ?](../../role-based-access-control/overview.md). |

## <a name="about-the-preview"></a>À propos de la préversion

La préversion pour désactiver l’autorisation avec clé partagée est disponible dans le cloud public Azure. Elle est prise en charge pour les comptes de stockage qui utilisent uniquement le modèle de déploiement Azure Resource Manager. Pour plus d’informations sur les comptes de stockage qui utilisent le modèle de déploiement Azure Resource Manager, consultez [Types de compte de stockage](storage-account-overview.md#types-of-storage-accounts).

Pour vous inscrire à la préversion, consultez [Préversion publique limitée de l’accès avec clé partagée au service Stockage Azure](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxW65f1VQyNCuBHMIMBV8qlUN1o4TUtUUzZBV0JYVlhKQ1FITDlVUUU0Ui4u).

> [!IMPORTANT]
> Cette préversion est destinée uniquement à une utilisation hors production.

La préversion inclut les limitations décrites dans les sections suivantes.

### <a name="metrics-and-logging-report-all-requests-made-with-a-sas-regardless-of-how-they-are-authorized"></a>Les métriques et la journalisation signalent toutes les demandes effectuées avec une signature d’accès partagé, quelle que soit leur mode d’autorisation

Les métriques et la journalisation Azure dans Azure Monitor ne font pas la distinction entre les différents types de signatures d’accès partagé dans la préversion. Le filtre **SAP** dans Azure Metrics Explorer et le champ **SAP** dans la journalisation d’Azure Storage dans Azure Monitor signalent tous deux les demandes autorisées avec n’importe quel type de SAP. Toutefois, les différents types de signatures d’accès partagé sont autorisés de manière différente et se comportent différemment lorsque l’accès avec clé partagée est désactivé :

- Un jeton SAP de service ou un jeton SAP de compte est autorisé avec une clé partagée, et n’est pas autorisé sur une demande adressée au Stockage Blob quand la propriété **AllowSharedKeyAccess** est définie sur **false**.
- Une SAP de délégation d’utilisateur est autorisée avec Azure AD, et est autorisée sur une demande adressée au Stockage Blob quand la propriété **AllowSharedKeyAccess** est définie sur **false**.

Lorsque vous évaluez le trafic vers votre compte de stockage, gardez à l’esprit que les métriques et les journaux décrits dans [Détecter le type d’autorisation utilisé par les applications clientes](#detect-the-type-of-authorization-used-by-client-applications) peuvent inclure des demandes effectuées avec une SAP de délégation d’utilisateur. Pour plus d’informations sur la façon dont le service Stockage Azure répond à une signature d’accès partagé quand la propriété **AllowSharedKeyAccess** est définie sur **false**, consultez [Comprendre comment la désactivation de la clé partagée affecte les jetons SAP](#understand-how-disallowing-shared-key-affects-sas-tokens).

## <a name="next-steps"></a>Étapes suivantes

- [Autorisation de l’accès aux données dans Stockage Azure](storage-auth.md)
- [Autorisation de l’accès aux blobs et aux files d’attente à l’aide d’Azure Active Directory](storage-auth-aad.md)
- [Autoriser avec une clé partagée](/rest/api/storageservices/authorize-with-shared-key)
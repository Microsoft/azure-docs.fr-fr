---
title: Empêcher l’accès en lecture public anonyme aux conteneurs et aux blobs
titleSuffix: Azure Storage
description: Découvrez comment analyser les requêtes anonymes sur un compte de stockage et comment empêcher l’accès anonyme pour l’ensemble du compte de stockage ou pour un conteneur spécifique.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/09/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: blobs
ms.openlocfilehash: 179e60a41a9cd6a2277959b3cd31159c796d845d
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96937285"
---
# <a name="prevent-anonymous-public-read-access-to-containers-and-blobs"></a>Empêcher l’accès en lecture public anonyme aux conteneurs et aux blobs

L’accès en lecture public anonyme aux conteneurs et aux blobs dans le stockage Azure est un moyen pratique de partager des données, mais peut également présenter un risque pour la sécurité. Il est important de gérer judicieusement l’accès anonyme et de comprendre la manière d’évaluer l’accès anonyme à vos données. La complexité opérationnelle, une erreur humaine ou une attaque malveillante contre les données accessibles publiquement peuvent entraîner des violations de données coûteuses. Microsoft vous recommande d’activer l’accès anonyme uniquement quand cela est nécessaire pour votre scénario d’application.

Par défaut, l’accès public à vos données de blob est toujours interdit. Toutefois, la configuration par défaut d’un compte de stockage permet à un utilisateur disposant des autorisations appropriées de configurer un accès public aux conteneurs et aux blobs d’un compte de stockage. Pour renforcer la sécurité, vous pouvez interdire tout accès public au compte de stockage, quel que soit le paramètre d’accès public pour un conteneur individuel. Interdire l’accès public au compte de stockage empêche un utilisateur d’autoriser l’accès public pour un conteneur du compte. Microsoft vous recommande d’interdire l’accès public à un compte de stockage, sauf si votre scénario l’exige. L’interdiction de l’accès public permet d’éviter les violations de données causées par un accès anonyme indésirable.

Lorsque vous interdisez l’accès public aux blobs pour le compte de stockage, Stockage Azure rejette toutes les requêtes anonymes envoyées à ce compte. Une fois que l’accès public est interdit pour un compte, les conteneurs de ce compte ne peuvent plus être configurés pour l’accès public. Les conteneurs qui ont déjà été configurés pour un accès public n’acceptent plus les requêtes anonymes. Pour en savoir plus, consultez la section [Configure anonymous public read access for containers and blobs](anonymous-read-access-configure.md) (Configurer l’accès en lecture publique anonyme pour les conteneurs et les objets blob).

Cet article explique comment utiliser une infrastructure DRAG (détection, correction, audit, gouvernance) pour gérer en continu l’accès public pour vos comptes de stockage.

## <a name="detect-anonymous-requests-from-client-applications"></a>Détecter les demandes anonymes des applications clientes

Quand vous interdisez l’accès en lecture public pour un compte de stockage, vous risquez de rejeter les requêtes à destination des conteneurs et des blobs qui sont configurés pour l’accès public. L’interdiction de l’accès public pour un compte de stockage prévaut sur les paramètres d’accès public pour les conteneurs individuels de ce compte de stockage. Quand l’accès public est interdit pour le compte de stockage, toute requête anonyme ultérieure adressée à ce compte échoue.

Pour que vous compreniez dans quelle mesure l’interdiction de l’accès public peut influer sur les applications clientes, Microsoft vous recommande d’activer la journalisation et les métriques pour ce compte et d’analyser les modèles de requêtes anonymes sur un intervalle de temps. Utilisez des métriques pour déterminer le nombre de demandes anonymes adressées au compte de stockage et utilisez les journaux pour déterminer les conteneurs qui sont accessibles de façon anonyme.

### <a name="monitor-anonymous-requests-with-metrics-explorer"></a>Superviser les demandes anonymes avec Metrics Explorer

Pour effectuer le suivi des demandes anonymes adressées à un compte de stockage, utilisez Azure Metrics Explorer dans le portail Azure. Pour plus d’informations sur Metrics Explorer, consultez [Prise en main d’Azure Metrics Explorer](../../azure-monitor/platform/metrics-getting-started.md).

Suivez ces étapes pour créer une métrique qui effectue le suivi des requêtes anonymes :

1. Accédez à votre compte de stockage dans le portail Azure. Dans la section **Supervision**, sélectionnez **Métriques**.
1. Sélectionnez **Ajouter une métrique**. Dans la boîte de dialogue **Métrique**, spécifiez les valeurs suivantes :
    1. Laissez le champ Étendue défini sur le nom du compte de stockage.
    1. Définissez l’**Espace de noms de métrique** sur *Blob*. Cette métrique ne porte que sur les demandes à destination du stockage Blob.
    1. Définissez le champ **Métrique** sur *Transactions*.
    1. Définissez le champ **Agrégation** sur *Somme*.

    La nouvelle métrique affiche la somme du nombre de transactions sur le stockage Blob sur un intervalle de temps donné. La métrique qui en résulte s’affiche comme indiqué dans l’image suivante :

    :::image type="content" source="media/anonymous-read-access-prevent/configure-metric-blob-transactions.png" alt-text="Capture d’écran montrant comment configurer une métrique pour additionner les transactions blob":::

1. Ensuite, sélectionnez le bouton **Ajouter un filtre** afin de créer un filtre sur la métrique pour les demandes anonymes.
1. Dans la boîte de dialogue **Filtre**, spécifiez les valeurs suivantes :
    1. Définissez la valeur **Propriété** sur *Authentification*.
    1. Affectez au champ **Opérateur** le signe égal (=).
    1. Définissez le champ **Valeurs** sur *Anonyme*.
1. Dans l’angle supérieur droit, sélectionnez l’intervalle de temps sur lequel doit porter la métrique. Vous pouvez également indiquer le degré de précision de l’agrégation des demandes, en spécifiant des intervalles compris entre 1 minute et 1 mois.

Une fois que vous avez configuré la métrique, les demandes anonymes commencent à s’afficher sur le graphique. L’illustration suivante montre les demandes anonymes agrégées sur les trente dernières minutes.

:::image type="content" source="media/anonymous-read-access-prevent/metric-anonymous-blob-requests.png" alt-text="Capture d’écran montrant les demandes anonymes agrégées sur le stockage Blob":::

Vous pouvez également configurer une règle d’alerte pour vous avertir quand un certain nombre de demandes anonymes sont effectuées sur votre compte de stockage. Pour plus d'informations, consultez [Créer, afficher et gérer des alertes de métrique à l'aide d'Azure Monitor](../../azure-monitor/platform/alerts-metric.md).

### <a name="analyze-logs-to-identify-containers-receiving-anonymous-requests"></a>Analyser les journaux pour identifier les conteneurs recevant des demandes anonymes

Les journaux du stockage Azure capturent des détails sur les demandes effectuées sur le compte de stockage, y compris la manière dont une demande a été autorisée. Vous pouvez analyser les journaux pour identifier les conteneurs qui reçoivent des demandes anonymes.

Pour journaliser les demandes dans votre compte de stockage Azure afin d’évaluer les demandes anonymes, vous pouvez utiliser la journalisation du stockage Azure dans Azure Monitor (préversion). Pour plus d’informations, consultez [Superviser le stockage Azure](./monitor-blob-storage.md).

La journalisation du stockage Azure dans Azure Monitor prend en charge l’utilisation de requêtes de journal pour analyser les données des journaux. Pour interroger les journaux, vous pouvez utiliser un espace de travail Azure Log Analytics. Pour en savoir plus sur les requêtes de journal, consultez [Tutoriel : Bien démarrer avec les requêtes Log Analytics](../../azure-monitor/log-query/log-analytics-tutorial.md).

> [!NOTE]
> La préversion de la journalisation Stockage Azure dans Azure Monitor est prise en charge uniquement dans le cloud public Azure. Les clouds du secteur public ne prennent pas en charge la journalisation pour Stockage Azure avec Azure Monitor.

#### <a name="create-a-diagnostic-setting-in-the-azure-portal"></a>Créer un paramètre de diagnostic dans le portail Azure

Pour journaliser des données de stockage Azure avec Azure Monitor et les analyser avec Azure Log Analytics, vous devez d’abord créer un paramètre de diagnostic qui indique les types de demandes et les services de stockage pour lesquels vous souhaitez journaliser les données. Pour créer un paramètre de diagnostic dans le portail Azure, suivez ces étapes :

1. Inscrivez-vous à la [journalisation de stockage Azure dans Azure Monitor (préversion)](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxW65f1VQyNCuBHMIMBV8qlUM0E0MFdPRFpOVTRYVklDSE1WUTcyTVAwOC4u).
1. Créez un espace de travail Log Analytics dans l’abonnement qui contient votre compte de stockage Azure. Une fois que vous avez configuré la journalisation pour votre compte de stockage, les journaux sont disponibles dans l’espace de travail Log Analytics. Pour plus d’informations, consultez [Créer un espace de travail Log Analytics dans le portail Azure](../../azure-monitor/learn/quick-create-workspace.md).
1. Accédez à votre compte de stockage dans le portail Azure.
1. Dans la section Supervision, Sélectionnez **Paramètres de diagnostic (préversion)** .
1. Sélectionnez **Blob** pour journaliser les demandes effectuées sur le stockage Blob.
1. Sélectionnez **Ajouter le paramètre de diagnostic**.
1. Fournissez un nom pour le paramètre de diagnostic.
1. Sous **Détails de la catégorie**, dans la section **Journal**, choisissez les types de demandes à journaliser. Toutes les demandes anonymes étant des demandes de lecture, sélectionnez **StorageRead** pour capturer les demandes anonymes.
1. Sous **Détails de la destination**, sélectionnez **Envoyer à Log Analytics**. Sélectionnez votre abonnement et l’espace de travail Log Analytics que vous avez créé, comme illustré dans l’image suivante.

    :::image type="content" source="media/anonymous-read-access-prevent/create-diagnostic-setting-logs.png" alt-text="Capture d’écran montrant comment créer un paramètre de diagnostic pour la journalisation des demandes":::

Une fois le paramètre de diagnostic créé, les demandes adressées au compte de stockage sont journalisées conformément à ce paramètre. Pour plus d’informations, consultez [Créer un paramètre de diagnostic pour collecter les journaux et les métriques des ressources dans Azure](../../azure-monitor/platform/diagnostic-settings.md).

Pour obtenir des informations de référence sur les champs disponibles dans les journaux de stockage Azure dans Azure Monitor, consultez [Journaux de ressource (préversion)](./monitor-blob-storage-reference.md#resource-logs-preview).

#### <a name="query-logs-for-anonymous-requests"></a>Interroger les journaux pour rechercher les demandes anonymes

Les journaux de stockage Azure dans Azure Monitor incluent le type d’autorisation qui a été utilisé pour effectuer une demande à destination d’un compte de stockage. Dans votre requête de journal, filtrez sur la propriété **AuthenticationType** pour afficher les demandes anonymes.

Pour récupérer les journaux des 7 derniers jours pour les demandes anonymes sur le stockage Blob, ouvrez votre espace de travail Log Analytics. Collez ensuite la requête suivante dans une nouvelle requête de journal et exécutez-la :

```kusto
StorageBlobLogs
| where TimeGenerated > ago(7d) and AuthenticationType == "Anonymous"
| project TimeGenerated, AccountName, AuthenticationType, Uri
```

Vous pouvez également configurer une règle d’alerte basée sur cette requête pour vous avertir des demandes anonymes. Pour plus d’informations, consultez [Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../../azure-monitor/platform/alerts-log.md).

## <a name="remediate-anonymous-public-access"></a>Corriger l’accès public anonyme

Une fois que vous avez évalué les demandes anonymes destinées aux conteneurs et aux blobs dans votre compte de stockage, vous pouvez prendre des mesures pour limiter ou empêcher l’accès public. Si certains conteneurs de votre compte de stockage doivent être disponibles pour un accès public, vous pouvez configurer le paramètre d’accès public pour chaque conteneur dans votre compte de stockage. Cette option fournit le contrôle le plus précis de l’accès public. Pour plus d’informations, consultez [Définir le niveau d’accès public pour un conteneur](anonymous-read-access-configure.md#set-the-public-access-level-for-a-container).

Pour renforcer la sécurité, vous pouvez interdire l’accès public pour l’ensemble du compte de stockage. Le paramètre d’accès public pour un compte de stockage remplace les paramètres individuels des conteneurs dans ce compte. Quand vous interdisez l’accès public pour un compte de stockage, tous les conteneurs qui sont configurés pour autoriser l’accès public ne sont plus accessibles de façon anonyme. Pour plus d’informations, consultez [Autoriser ou interdire l’accès en lecture public pour un compte de stockage](anonymous-read-access-configure.md#allow-or-disallow-public-read-access-for-a-storage-account).

Si votre scénario nécessite que certains conteneurs soient disponibles pour un accès public, il peut être préférable de déplacer ces conteneurs et leurs blobs dans des comptes de stockage réservés à un accès public. Vous pouvez ensuite interdire l’accès public pour tous les autres comptes de stockage.

### <a name="verify-that-public-access-to-a-blob-is-not-permitted"></a>Vérifier que l’accès public à un blob n’est pas autorisé

Pour vérifier que l’accès public à un blob spécifique est interdit, vous pouvez essayer de télécharger le blob via son URL. Si le téléchargement s’effectue correctement, le blob est toujours accessible publiquement. Si le blob n’est pas accessible publiquement parce que l’accès public a été interdit pour le compte de stockage, un message d’erreur s’affiche, indiquant que l’accès public n’est pas autorisé sur ce compte.

L’exemple suivant montre comment utiliser PowerShell pour essayer de télécharger un blob via son URL. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
$url = "<absolute-url-to-blob>"
$downloadTo = "<file-path-for-download>"
Invoke-WebRequest -Uri $url -OutFile $downloadTo -ErrorAction Stop
```

### <a name="verify-that-modifying-the-containers-public-access-setting-is-not-permitted"></a>Vérifier que la modification du paramètre d’accès public du conteneur n’est pas autorisée

Pour vérifier que le paramètre d’accès public d’un conteneur ne peut pas être modifié après l’interdiction de l’accès public pour le compte de stockage, vous pouvez essayer de modifier le paramètre. La modification du paramètre d’accès public du conteneur échoue si l’accès public est interdit pour le compte de stockage.

L’exemple suivant montre comment utiliser PowerShell pour tenter de changer le paramètre d’accès public d’un conteneur. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$containerName = "<container-name>"

$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

Set-AzStorageContainerAcl -Context $ctx -Container $containerName -Permission Blob
```

### <a name="verify-that-creating-a-container-with-public-access-enabled-is-not-permitted"></a>Vérifier que la création d’un conteneur avec accès public activé n’est pas autorisée

Si l’accès public est interdit pour le compte de stockage, vous ne pouvez pas créer de conteneur avec un accès public autorisé. Pour vérifier, vous pouvez essayer de créer un conteneur avec accès public activé.

L’exemple suivant montre comment utiliser PowerShell pour essayer de créer un conteneur avec accès public activé. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
$rgName = "<resource-group>"
$accountName = "<storage-account>"
$containerName = "<container-name>"

$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

New-AzStorageContainer -Name $containerName -Permission Blob -Context $ctx
```

### <a name="check-the-public-access-setting-for-multiple-accounts"></a>Vérifier le paramètre d’accès public pour plusieurs comptes

Pour vérifier le paramètre d’accès public d’un ensemble de comptes de stockage avec des performances optimales, vous pouvez utiliser l’Explorateur Azure Resource Graph dans le portail Azure. Pour en savoir plus sur l’utilisation de l’Explorateur Resource Graph, consultez [Démarrage rapide : exécuter votre première requête Resource Graph à l’aide de l’Explorateur Azure Resource Graph](../../governance/resource-graph/first-query-portal.md).

La propriété **AllowBlobPublicAccess** n’est pas définie par défaut pour un compte de stockage et ne retourne pas de valeur tant que vous ne la définissez pas explicitement. Le compte de stockage autorise l’accès public quand la valeur de la propriété est **Null** ou **True**.

L’exécution de la requête suivante dans l’Explorateur Resource Graph retourne une liste de comptes de stockage et affiche le paramètre d’accès public pour chaque compte :

```kusto
resources
| where type =~ 'Microsoft.Storage/storageAccounts'
| extend allowBlobPublicAccess = parse_json(properties).allowBlobPublicAccess
| project subscriptionId, resourceGroup, name, allowBlobPublicAccess
```

L’image suivante montre les résultats d’une requête dans un abonnement. Notez que pour les comptes de stockage où la propriété **AllowBlobPublicAccess** a été définie explicitement, celle-ci apparaît dans les résultats comme **true** ou **false**. Si la propriété **AllowBlobPublicAccess** n’a pas été définie pour un compte de stockage, elle apparaît comme vide (ou null) dans les résultats de la requête.

:::image type="content" source="media/anonymous-read-access-prevent/check-public-access-setting-accounts.png" alt-text="Capture d’écran montrant les résultats de la requête pour un paramètre d’accès public sur les comptes de stockage":::

## <a name="use-azure-policy-to-audit-for-compliance"></a>Utiliser Azure Policy pour auditer la conformité

Si vous avez un grand nombre de comptes de stockage, il se peut que vous deviez effectuer un audit pour vous assurer que ces comptes sont configurés de manière à empêcher l’accès public. Pour auditer la conformité d’un ensemble de comptes de stockage, utilisez Azure Policy. Azure Policy est un service que vous pouvez utiliser pour créer, attribuer et gérer des stratégies qui appliquent des règles à des ressources Azure. Lorsque vous utilisez Azure Policy, les ressources restent conformes à vos normes d’entreprise et contrats de niveau de service. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Policy](../../governance/policy/overview.md).

### <a name="create-a-policy-with-an-audit-effect"></a>Créer une stratégie avec un effet d’audit

Azure Policy prend en charge les effets qui déterminent ce qui se produit quand une règle de stratégie est évaluée par rapport à une ressource. L’effet d’audit crée un avertissement quand une ressource n’est pas conforme, mais n’arrête pas la demande. Pour plus d’informations, consultez [Comprendre les effets d’Azure Policy](../../governance/policy/concepts/effects.md).

Pour créer une stratégie avec un effet d’audit pour le paramètre d’accès public d’un compte de stockage avec le portail Azure, procédez comme suit :

1. Dans le portail Azure, accédez au service Azure Policy.
1. Dans la section **Création**, sélectionnez **Définitions**.
1. Sélectionnez **Ajouter une définition de stratégie** pour créer une nouvelle définition de stratégie.
1. Pour le champ **emplacement de la définition**, sélectionnez le bouton **Autres** pour spécifier l’emplacement de la ressource de stratégie d’audit.
1. Spécifiez un nom pour la stratégie. Vous pouvez éventuellement spécifier une description et une catégorie.
1. Sous **Règle de stratégie**, ajoutez la définition de stratégie suivante à la section **policyrule**.

    ```json
    {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
          },
          {
            "not": {
              "field":"Microsoft.Storage/storageAccounts/allowBlobPublicAccess",
              "equals": "false"
            }
          }
        ]
      },
      "then": {
        "effect": "audit"
      }
    }
    ```

1. Enregistrez la stratégie.

### <a name="assign-the-policy"></a>Affecter la stratégie

Ensuite, attribuez la stratégie à une ressource. L’étendue de la stratégie correspond à cette ressource et à toutes les ressources qu’elle contient. Pour plus d’informations sur l’attribution de stratégie, consultez [Structure d’affectation d’Azure Policy](../../governance/policy/concepts/assignment-structure.md).

Pour attribuer la stratégie avec le portail Azure, procédez comme suit :

1. Dans le portail Azure, accédez au service Azure Policy.
1. Dans la section **Création**, sélectionnez **Attributions**.
1. Sélectionnez **Attribuer une stratégie** pour créer une attribution de stratégie.
1. Pour le champ **Étendue**, sélectionnez l’étendue de l’attribution de stratégie.
1. Pour le champ **Définition de stratégie**, sélectionnez le bouton **Autres**, puis la stratégie que vous avez définie dans la section précédente dans la liste.
1. Entrez un nom pour l’attribution de stratégie. La description est facultative.
1. Laissez l’option **Application de stratégie** définie sur *Activée*. Ce paramètre n’a aucun effet sur la stratégie d’audit.
1. Sélectionnez **Vérifier + créer** pour créer l’attribution.

### <a name="view-compliance-report"></a>Afficher le rapport de conformité

Une fois que vous avez attribué la stratégie, vous pouvez afficher le rapport de conformité. Le rapport de conformité d’une stratégie d’audit fournit des informations sur les comptes de stockage qui ne sont pas conformes à la stratégie. Pour plus d’informations, consultez [Obtenir les données de conformité de la stratégie](../../governance/policy/how-to/get-compliance-data.md).

La disponibilité du rapport de conformité peut prendre plusieurs minutes après la création de l’attribution de stratégie.

Pour afficher le rapport de conformité dans le Portail Azure, procédez comme suit :

1. Dans le portail Azure, accédez au service Azure Policy.
1. Sélectionnez **Conformité**.
1. Filtrez les résultats pour le nom de l’attribution de stratégie que vous avez créée à l’étape précédente. Le rapport indique le nombre de ressources qui ne sont pas conformes à la stratégie.
1. Vous pouvez explorer le rapport pour obtenir des détails supplémentaires, notamment une liste des comptes de stockage qui ne sont pas conformes.

    :::image type="content" source="media/anonymous-read-access-prevent/compliance-report-policy-portal.png" alt-text="Capture d’écran montrant le rapport de conformité de la stratégie d’audit pour l’accès public aux blobs":::

## <a name="use-azure-policy-to-enforce-authorized-access"></a>Utiliser Azure Policy pour appliquer l’accès autorisé

Azure Policy prend en charge la gouvernance cloud en s’assurant que les ressources Azure respectent les exigences et les normes. Pour vous assurer que les comptes de stockage de votre organisation autorisent uniquement les requêtes autorisées, vous pouvez créer une stratégie qui empêche la création d’un nouveau compte de stockage dont le paramètre d’accès public autorise les requêtes anonymes. Cette stratégie empêchera également toutes les modifications de configuration apportées à un compte existant si le paramètre d’accès public pour ce compte n’est pas conforme à la stratégie.

La stratégie d’application utilise l’effet de refus pour empêcher une requête de créer ou de modifier un compte de stockage pour autoriser l’accès public. Pour plus d’informations, consultez [Comprendre les effets d’Azure Policy](../../governance/policy/concepts/effects.md).

Pour créer une stratégie avec un effet de refus pour un paramètre d’accès public qui autorise les requêtes anonymes, suivez les mêmes étapes décrites dans [Utiliser Azure Policy pour auditer la conformité](#use-azure-policy-to-audit-for-compliance), mais fournissez le fichier JSON suivant dans la section **policyRule** de la définition de stratégie :

```json
{
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.Storage/storageAccounts"
      },
      {
        "not": {
          "field":"Microsoft.Storage/storageAccounts/allowBlobPublicAccess",
          "equals": "false"
        }
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

Une fois que vous avez créé la stratégie avec l’effet de refus et l’avez attribuée à une étendue, un utilisateur ne peut plus créer de compte de stockage qui autorise l’accès public. Un utilisateur ne peut pas non plus apporter des changements de configuration à un compte de stockage existant qui autorise actuellement l’accès public. Toute tentative en ce sens entraîne une erreur. Le paramètre d’accès public pour le compte de stockage doit être défini sur **false** pour poursuivre la création ou la configuration du compte.

L’image suivante montre l’erreur qui se produit si vous tentez de créer un compte de stockage qui autorise l’accès public (valeur par défaut pour un nouveau compte) lorsqu’une stratégie avec effet de refus exige que l’accès public soit interdit.

:::image type="content" source="media/anonymous-read-access-prevent/deny-policy-error.png" alt-text="Capture d’écran montrant l’erreur qui se produit lors de la création d’un compte de stockage en violation de la stratégie":::

## <a name="permissions-for-allowing-or-disallowing-public-access"></a>Autorisations pour autoriser ou interdire l’accès public

Afin de définir la propriété **AllowBlobPublicAccess** pour le compte de stockage, un utilisateur doit disposer des autorisations nécessaires pour créer et gérer des comptes de stockage. Les rôles de contrôle d’accès en fonction du rôle Azure (Azure RBAC) qui fournissent ces autorisations incluent l’action **Microsoft.Storage/storageAccounts/write** ou **Microsoft.Storage/storageAccounts/\** _. Les rôles intégrés à cette action comprennent :

- Le rôle [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) d’Azure Resource Manager
- Le rôle [Contributeur](../../role-based-access-control/built-in-roles.md#contributor) d’Azure Resource Manager
- Le rôle [Contributeur de compte de stockage](../../role-based-access-control/built-in-roles.md#storage-account-contributor)

Ces rôles ne fournissent pas d’accès aux données d’un compte de stockage par le biais d’Azure Active Directory (Azure AD). Toutefois, ils incluent l’action _*Microsoft.Storage/storageAccounts/listkeys/action**, qui accorde l’accès aux clés d’accès du compte. Avec cette autorisation, un utilisateur peut utiliser les clés d’accès du compte pour accéder à toutes les données d’un compte de stockage.

Les attributions de rôles doivent être définies au niveau du compte de stockage ou à un niveau supérieur pour permettre à un utilisateur d’autoriser ou d’interdire l’accès public au compte de stockage. Pour plus d’informations sur l’étendue des rôles, consultez [Comprendre l’étendue pour Azure RBAC](../../role-based-access-control/scope-overview.md).

Veillez à limiter l’attribution de ces rôles aux seules personnes qui ont besoin de créer un compte de stockage ou de mettre à jour ses propriétés. Utilisez le principe du moindre privilège pour vous assurer que les utilisateurs disposent des autorisations minimales nécessaires pour accomplir leurs tâches. Pour plus d’informations sur la gestion de l’accès avec Azure RBAC, consultez [Meilleures pratiques pour Azure RBAC](../../role-based-access-control/best-practices.md).

> [!NOTE]
> Les rôles d’administrateur d’abonnement classique Administrateur de service et Co-administrateur incluent l’équivalent du rôle [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) d’Azure Resource Manager. Le rôle **Propriétaire** comprend toutes les actions, de sorte qu’un utilisateur disposant de l’un de ces rôles d’administration peut également créer et gérer des comptes de stockage. Pour plus d’informations, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles d’administrateur Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles).

## <a name="next-steps"></a>Étapes suivantes

- [Configurer l’accès en lecture public anonyme pour les conteneurs et les objets blob](anonymous-read-access-configure.md)
- [Accéder anonymement aux conteneurs et objets blob publics avec .NET](anonymous-read-access-client.md)
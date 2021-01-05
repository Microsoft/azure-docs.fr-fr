---
title: Accès en fonction du rôle granulaire - Configurations de cluster Azure HDInsight
description: Découvrez les modifications nécessaires dans le cadre de la migration vers un accès granulaire en fonction du rôle pour des configurations de cluster HDInsight.
author: tylerfox
ms.author: tyfox
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/20/2020
ms.openlocfilehash: d2e9c1fe89866511f8eae0b900563471cd6e52e9
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92533306"
---
# <a name="migrate-to-granular-role-based-access-for-cluster-configurations"></a>Migrer vers un accès en fonction du rôle granulaire pour les configurations de cluster

Nous ajoutons d’importantes modifications pour pouvoir utiliser des accès basés sur les rôles encore plus détaillés pour obtenir des informations sensibles. Dans le cadre de ces modifications, des actions peuvent être nécessaires **d’ici le 3 septembre 2019** si vous utilisez [une des entités ou un des scénarios affectés](#am-i-affected-by-these-changes).

## <a name="what-is-changing"></a>Qu’est-ce qui change ?

Auparavant, les secrets pouvaient être obtenus par le biais de l’API HDInsight par des utilisateurs du cluster avec les [rôles Azure](../role-based-access-control/rbac-and-directory-admin-roles.md) Propriétaire, Contributeur ou Lecteur, car ils étaient accessibles à tout utilisateur ayant l’autorisation `*/read`. Les secrets sont définis comme des valeurs pouvant être utilisées pour obtenir un accès supérieur à celui d’un rôle d’utilisateur. Ils comprennent des valeurs telles que les informations d’identification HTTP de la passerelle du cluster, les clés de compte de stockage et les informations d'identification de la base de données.

À partir du 3 septembre 2019, l’accès à ces secrets nécessitera l’autorisation `Microsoft.HDInsight/clusters/configurations/action`, ce qui veut dire qu’ils ne seront plus accessibles aux utilisateurs disposant du rôle Lecteur. Les rôles qui ont cette autorisation sont Contributeur, Propriétaire et le nouveau rôle Opérateur de cluster HDInsight (plus de détails dans ce qui suit).

Nous introduisons également un nouveau rôle [Opérateur de cluster HDInsight](../role-based-access-control/built-in-roles.md#hdinsight-cluster-operator) capable de récupérer des secrets sans disposer des autorisations administratives d’un Contributeur ou d’un Propriétaire. Pour résumer :

| Role                                  | Précédemment                                                                                       | À l’avenir       |
|---------------------------------------|--------------------------------------------------------------------------------------------------|-----------|
| Lecteur                                | - Un accès en lecture, notamment aux secrets.                                                                   | - Un accès en lecture, **sans** les secrets |           |   |   |
| Opérateur de cluster HDInsight<br>(Nouveau rôle) | N/A                                                                                              | - Accès lecture/écriture, dont aux secrets         |   |   |
| Contributeur                           | - Un accès en lecture/écriture, notamment aux secrets.<br>- Création et gestion de tous les types de ressources Azure.<br>- Exécution des actions de script.     | Aucun changement |
| Propriétaire                                 | - Un accès en lecture/écriture, notamment aux secrets.<br>- Accès total à toutes les ressources<br>- Délégation de l’accès à d’autres personnes.<br>- Exécution des actions de script. | Aucun changement |

Pour plus d’informations sur l’attribution du rôle Opérateur de cluster HDInsight à un utilisateur afin de lui autoriser l’accès lecture/écriture aux secrets du cluster, consultez la section ci-dessous, [Ajouter l’attribution de rôle Opérateur de cluster HDInsight à un utilisateur](#add-the-hdinsight-cluster-operator-role-assignment-to-a-user).

## <a name="am-i-affected-by-these-changes"></a>Suis-je concerné par ces modifications ?

Les entités et scénarios suivants sont concernés :

- [API](#api) : Utilisateurs ayant recours aux points de terminaison `/configurations` et `/configurations/{configurationName}`.
- [Informations de connexion Azure HDInsight Tools for Visual Studio Code](#azure-hdinsight-tools-for-visual-studio-code) version 1.1.1 ou inférieure.
- [Azure Toolkit for IntelliJ](#azure-toolkit-for-intellij) version 3.20.0 ou inférieure.
- [Azure Data Lake et Stream Analytics Tools pour Visual Studio](#azure-data-lake-and-stream-analytics-tools-for-visual-studio) version 2.3.9000.1 ou inférieure.
- [Azure Toolkit pour Eclipse](#azure-toolkit-for-eclipse) version 3.15.0 ou inférieure.
- [Kit SDK pour .NET](#sdk-for-net)
    - [Versions 1.x ou 2.x](#versions-1x-and-2x) : utilisateurs qui ont recours aux méthodes `GetClusterConfigurations`; `GetConnectivitySettings`, `ConfigureHttpSettings`, `EnableHttp` ou `DisableHttp` de la classe ConfigurationsOperationsExtensions.
    - [Versions 3.x et versions ultérieures](#versions-3x-and-up) : Utilisateurs ayant recours aux méthodes `Get`, `Update`, `EnableHttp`, ou `DisableHttp` depuis la classe `ConfigurationsOperationsExtensions`.
- [Kit SDK pour Python](#sdk-for-python) : Utilisateurs ayant recours aux méthodes `get` ou `update` depuis la classe `ConfigurationsOperations`.
- [Kit SDK pour Java](#sdk-for-java) : Utilisateurs ayant recours aux méthodes `update` ou `get` depuis la classe `ConfigurationsInner`.
- [Kit SDK pour Go](#sdk-for-go) : Utilisateurs ayant recours aux méthodes `Get` ou `Update` depuis la structure `ConfigurationsClient`.
- [PowerShell Az.HDInsight](#azhdinsight-powershell) version inférieure à 2.0.0.
Consultez les sections qui suivent (ou utilisez les liens mentionnés plus haut) pour connaître les étapes de migration de votre scénario.

### <a name="api"></a>API

Les API suivantes seront modifiées ou déconseillées :

- [**GET /configurations/{configurationName}**](/rest/api/hdinsight/hdinsight-cluster#get-configuration) (informations sensibles supprimées)
    - Précédemment utilisé pour obtenir les types de configuration individuels (y compris les secrets).
    - À partir du 3 septembre 2019, cet appel d’API retournera les types de configuration individuels avec les secrets omis. Pour obtenir toutes les configurations, dont les secrets, utilisez le nouvel appel POST/configurations. Pour obtenir uniquement les paramètres de passerelle, utilisez le nouvel appel POST/getGatewaySettings.
- [**GET /configurations**](/rest/api/hdinsight/hdinsight-cluster#get-configuration) (déconseillé)
    - Précédemment utilisé pour obtenir toutes les configurations (y compris les secrets)
    - À partir du 3 septembre 2019, cet appel d’API sera déprécié et ne sera plus pris en charge. Pour obtenir toutes les configurations, à l’avenir, utilisez le nouvel appel POST/configurations. Pour obtenir des configurations avec des paramètres sensibles omis, utilisez l’appel GET /configurations/{configurationName}.
- [**POST /configurations/{configurationName}**](/rest/api/hdinsight/hdinsight-cluster#update-gateway-settings) (déconseillé)
    - Précédemment utilisé pour mettre à jour les informations d’identification de la passerelle.
    - À partir du 3 septembre 2019, cet appel d’API sera déprécié et ne sera plus pris en charge. Utilisez le nouvel appel POST /updateGatewaySettings à la place.

Les API de remplacement suivantes ont été ajoutées :</span>

- [**POST /configurations**](/rest/api/hdinsight/hdinsight-cluster#list-configurations)
    - Utilisez cette API pour obtenir toutes les configurations, y compris les secrets.
- [**POST /getGatewaySettings**](/rest/api/hdinsight/hdinsight-cluster#get-gateway-settings)
    - Utilisez cette API pour obtenir les paramètres de la passerelle.
- [**POST /updateGatewaySettings**](/rest/api/hdinsight/hdinsight-cluster#update-gateway-settings)
    - Utilisez cette API pour mettre à jour les paramètres de la passerelle (nom d’utilisateur et/ou mot de passe).

### <a name="azure-hdinsight-tools-for-visual-studio-code"></a>Azure HDInsight Tools pour Visual Studio Code

Si vous utilisez la version 1.1.1 ou une version inférieure, mettez à jour vers la [dernière version d’Azure HDInsight Tools pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=mshdinsight.azure-hdinsight&ssr=false) afin d’éviter toute interruption.

### <a name="azure-toolkit-for-intellij"></a>Kit de ressources Azure pour IntelliJ

Si vous utilisez la version 3.20.0 ou une version inférieure, mettez à jour vers la [dernière version du plugin Azure Toolkit pour IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij) afin d’éviter toute interruption.

### <a name="azure-data-lake-and-stream-analytics-tools-for-visual-studio"></a>Azure Data Lake et Stream Analytics Tools pour Visual Studio

Mettez à jour vers la version 2.3.9000.1 ou ultérieure d’[Azure Data Lake et Stream Analytics Tools pour Visual Studio](https://marketplace.visualstudio.com/items?itemName=ADLTools.AzureDataLakeandStreamAnalyticsTools&ssr=false#overview) afin d’éviter toute interruption.  Pour faciliter la mise à jour, consultez notre documentation, [Update Data Lake Tools for Visual Studio (Mettre à jour Data Lake Tools pour Visual Studio)](./hadoop/apache-hadoop-visual-studio-tools-get-started.md#update-data-lake-tools-for-visual-studio).

### <a name="azure-toolkit-for-eclipse"></a>Kit de ressources Azure pour Eclipse

Si vous utilisez la version 3.15.0 ou une version inférieure, mettez à jour vers la [dernière version d’Azure Toolkit pour Eclipse](https://marketplace.eclipse.org/content/azure-toolkit-eclipse) afin d’éviter toute interruption.

### <a name="sdk-for-net"></a>Kit SDK pour .NET

#### <a name="versions-1x-and-2x"></a>Versions 1.x et 2.x

Mettez à jour vers la [version 2.1.0](https://www.nuget.org/packages/Microsoft.Azure.Management.HDInsight/2.1.0) du SDK HDInsight pour .NET. Des modifications minimes du code peuvent être nécessaires si vous utilisez une méthode affectée par ces changements :

- `ClusterOperationsExtensions.GetClusterConfigurations`**ne retournera plus de paramètres sensibles** comme les clés de stockage (core-site) ou les informations d'identification HTTP (passerelle).
    - Pour récupérer toutes les configurations, y compris les paramètres sensibles, utilisez `ClusterOperationsExtensions.ListConfigurations` à l’avenir.  Notez que les utilisateurs disposant du rôle « Lecteur » ne seront pas en mesure d’utiliser cette méthode. Cela permet un contrôle granulaire sur les utilisateurs pouvant accéder aux informations sensibles d’un cluster.
    - Pour récupérer des données d’identification de passerelle HTTP, utilisez `ClusterOperationsExtensions.GetGatewaySettings`.

- `ClusterOperationsExtensions.GetConnectivitySettings` est maintenant déconseillé et a été remplacé par `ClusterOperationsExtensions.GetGatewaySettings`.

- `ClusterOperationsExtensions.ConfigureHttpSettings` est maintenant déconseillé et a été remplacé par `ClusterOperationsExtensions.UpdateGatewaySettings`.

- `ConfigurationsOperationsExtensions.EnableHttp` et `DisableHttp` sont désormais déconseillés. HTTP est désormais toujours activé. Ces méthodes ne sont donc plus nécessaires.

#### <a name="versions-3x-and-up"></a>Versions 3.x et versions ultérieures

Mettez à jour vers la [version 5.0.0](https://www.nuget.org/packages/Microsoft.Azure.Management.HDInsight/5.0.0) ou versions ultérieures du SDK HDInsight pour .NET. Des modifications minimes du code peuvent être nécessaires si vous utilisez une méthode affectée par ces changements :

- [`ConfigurationOperationsExtensions.Get`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.get?view=azure-dotnet&preserve-view=true)**ne retournera plus de paramètres sensibles** comme les clés de stockage (core-site) ou les informations d'identification HTTP (passerelle).
    - Pour récupérer toutes les configurations, y compris les paramètres sensibles, utilisez [`ConfigurationOperationsExtensions.List`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.list?view=azure-dotnet&preserve-view=true) à l’avenir.  Notez que les utilisateurs disposant du rôle « Lecteur » ne seront pas en mesure d’utiliser cette méthode. Cela permet un contrôle granulaire sur les utilisateurs pouvant accéder aux informations sensibles d’un cluster. 
    - Pour récupérer des données d’identification de passerelle HTTP, utilisez [`ClusterOperationsExtensions.GetGatewaySettings`](/dotnet/api/microsoft.azure.management.hdinsight.clustersoperationsextensions.getgatewaysettings?view=azure-dotnet&preserve-view=true). 
- [`ConfigurationsOperationsExtensions.Update`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.update?view=azure-dotnet&preserve-view=true) est maintenant déconseillé et a été remplacé par [`ClusterOperationsExtensions.UpdateGatewaySettings`](/dotnet/api/microsoft.azure.management.hdinsight.clustersoperationsextensions.updategatewaysettings?view=azure-dotnet&preserve-view=true). 
- [`ConfigurationsOperationsExtensions.EnableHttp`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.enablehttp?view=azure-dotnet&preserve-view=true) et [`DisableHttp`](/dotnet/api/microsoft.azure.management.hdinsight.configurationsoperationsextensions.disablehttp?view=azure-dotnet&preserve-view=true) sont désormais déconseillés. HTTP est désormais toujours activé. Ces méthodes ne sont donc plus nécessaires.

### <a name="sdk-for-python"></a>Kit SDK pour Python

Mettez à jour vers la [version 1.0.0](https://pypi.org/project/azure-mgmt-hdinsight/1.0.0/) ou versions ultérieures du SDK HDInsight pour Python. Des modifications minimes du code peuvent être nécessaires si vous utilisez une méthode affectée par ces changements :

- [`ConfigurationsOperations.get`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.configurationsoperations#get-resource-group-name--cluster-name--configuration-name--custom-headers-none--raw-false----operation-config-)**ne retournera plus de paramètres sensibles** comme les clés de stockage (core-site) ou les informations d'identification HTTP (passerelle).
    - Pour récupérer toutes les configurations, y compris les paramètres sensibles, utilisez [`ConfigurationsOperations.list`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.configurationsoperations#list-resource-group-name--cluster-name--custom-headers-none--raw-false----operation-config-) à l’avenir.  Notez que les utilisateurs disposant du rôle « Lecteur » ne seront pas en mesure d’utiliser cette méthode. Cela permet un contrôle granulaire sur les utilisateurs pouvant accéder aux informations sensibles d’un cluster. 
    - Pour récupérer des données d’identification de passerelle HTTP, utilisez [`ClusterOperations.get_gateway_settings`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.clustersoperations#get-gateway-settings-resource-group-name--cluster-name--custom-headers-none--raw-false----operation-config-).
- [`ConfigurationsOperations.update`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.configurationsoperations#update-resource-group-name--cluster-name--configuration-name--parameters--custom-headers-none--raw-false--polling-true----operation-config-) est maintenant déconseillé et a été remplacé par [`ClusterOperations.update_gateway_settings`](/python/api/azure-mgmt-hdinsight/azure.mgmt.hdinsight.operations.clustersoperations#update-gateway-settings-resource-group-name--cluster-name--parameters--custom-headers-none--raw-false--polling-true----operation-config-).

### <a name="sdk-for-java"></a>Kit SDK pour Java

Mettez à jour vers la [version 1.0.0](https://search.maven.org/artifact/com.microsoft.azure.hdinsight.v2018_06_01_preview/azure-mgmt-hdinsight/1.0.0/jar) ou versions ultérieures du SDK HDInsight pour Java. Des modifications minimes du code peuvent être nécessaires si vous utilisez une méthode affectée par ces changements :

- `ConfigurationsInner.get`**ne retournera plus de paramètres sensibles** comme les clés de stockage (core-site) ou les informations d'identification HTTP (passerelle).
- `ConfigurationsInner.update` est désormais déconseillé.

### <a name="sdk-for-go"></a>Kit SDK pour Go

Mettez à jour vers la [version 27.1.0](https://github.com/Azure/azure-sdk-for-go/tree/master/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight) ou versions ultérieures du SDK HDInsight pour Go. Des modifications minimes du code peuvent être nécessaires si vous utilisez une méthode affectée par ces changements :

- [`ConfigurationsClient.get`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ConfigurationsClient.Get)**ne retournera plus de paramètres sensibles** comme les clés de stockage (core-site) ou les informations d'identification HTTP (passerelle).
    - Pour récupérer toutes les configurations, y compris les paramètres sensibles, utilisez [`ConfigurationsClient.list`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ConfigurationsClient.List) à l’avenir.  Notez que les utilisateurs disposant du rôle « Lecteur » ne seront pas en mesure d’utiliser cette méthode. Cela permet un contrôle granulaire sur les utilisateurs pouvant accéder aux informations sensibles d’un cluster. 
    - Pour récupérer des données d’identification de passerelle HTTP, utilisez [`ClustersClient.get_gateway_settings`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ClustersClient.GetGatewaySettings).
- [`ConfigurationsClient.update`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ConfigurationsClient.Update) est maintenant déconseillé et a été remplacé par [`ClustersClient.update_gateway_settings`](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight#ClustersClient.UpdateGatewaySettings).

### <a name="azhdinsight-powershell"></a>Az.HDInsight PowerShell
Mettez à jour vers la version [2.0.0 d’Az PowerShell](https://www.powershellgallery.com/packages/Az) ou version ultérieure afin d’éviter toute interruption.  Des modifications minimes du code peuvent être nécessaires si vous utilisez une méthode affectée par ces changements.
- `Grant-AzHDInsightHttpServicesAccess` est maintenant déconseillé et a été remplacé par la nouvelle cmdlet `Set-AzHDInsightGatewayCredential`.
- `Get-AzHDInsightJobOutput` a été mise à jour de façon à prendre en charge la précision de l’accès en fonction du rôle à la clé de stockage.
    - Les utilisateurs avec les rôles Opérateur, Contributeur et Propriétaire de cluster HDInsight ne seront pas affectés.
    - Les utilisateurs qui ont seulement le rôle Lecteur doivent spécifier explicitement le paramètre `DefaultStorageAccountKey`.
- `Revoke-AzHDInsightHttpServicesAccess` est désormais déconseillé. HTTP est désormais toujours activé. Cette cmdlet n’est donc plus nécessaire.
 Pour plus de détails, consultez le [guide de migration az.HDInsight](https://github.com/Azure/azure-powershell/blob/master/documentation/migration-guides/Az.2.0.0-migration-guide.md#azhdinsight).

## <a name="add-the-hdinsight-cluster-operator-role-assignment-to-a-user"></a>Ajouter l’attribution de rôle Opérateur de cluster HDInsight à un utilisateur

Un utilisateur disposant du rôle [Propriétaire](../role-based-access-control/built-in-roles.md#owner) peut attribuer le rôle [Opérateur de cluster HDInsight](../role-based-access-control/built-in-roles.md#hdinsight-cluster-operator) aux utilisateurs auxquels vous voulez donner l’accès en lecture/écriture à des valeurs de configuration sensibles de cluster HDInsight (telles que les informations d’identification de la passerelle du cluster et les clés de compte de stockage).

### <a name="using-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure (CLI)

La façon la plus simple pour ajouter cette attribution de rôle consiste à utiliser la commande `az role assignment create` dans Azure CLI.

> [!NOTE]
> Cette commande doit être exécutée par un utilisateur avec le rôle Propriétaire, car ils sont les seuls à pouvoir accorder ces autorisations. `--assignee` correspond au nom du principal de service ou à l’adresse e-mail de l’utilisateur à qui vous souhaitez attribuer le rôle Opérateur de cluster HDInsight. Si vous recevez une erreur indiquant des autorisations insuffisantes, consultez le Forum aux questions ci-dessous.

#### <a name="grant-role-at-the-resource-cluster-level"></a>Attribuer un rôle au niveau des ressources (cluster)

```azurecli-interactive
az role assignment create --role "HDInsight Cluster Operator" --assignee <user@domain.com> --scope /subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.HDInsight/clusters/<ClusterName>
```

#### <a name="grant-role-at-the-resource-group-level"></a>Attribuer un rôle au niveau du groupe de ressources

```azurecli-interactive
az role assignment create --role "HDInsight Cluster Operator" --assignee user@domain.com -g <ResourceGroupName>
```

#### <a name="grant-role-at-the-subscription-level"></a>Attribuer un rôle au niveau de l’abonnement

```azurecli-interactive
az role assignment create --role "HDInsight Cluster Operator" --assignee user@domain.com
```

### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

Vous pouvez également utiliser le portail Azure pour ajouter l’attribution du rôle Opérateur de cluster HDInsight à un utilisateur. Consultez la documentation [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure - Ajouter une attribution de rôle](../role-based-access-control/role-assignments-portal.md#add-a-role-assignment).

## <a name="faq"></a>Questions fréquentes (FAQ)

### <a name="why-am-i-seeing-a-403-forbidden-response-after-updating-my-api-requests-andor-tool"></a>Pourquoi est-ce que je vois une réponse 403 (Interdit) après la mise à jour de mes demandes d’API et/ou de l’outil ?

Les configurations de cluster se trouvent maintenant derrière un contrôle d’accès en fonction du rôle précis et nécessitent l’autorisation `Microsoft.HDInsight/clusters/configurations/*` pour y accéder. Pour obtenir cette autorisation, attribuez le rôle Opérateur de cluster HDInsight, Contributeur ou Propriétaire au principal de service ou à l’utilisateur qui tente d’accéder aux configurations.

### <a name="why-do-i-see-insufficient-privileges-to-complete-the-operation-when-running-the-azure-cli-command-to-assign-the-hdinsight-cluster-operator-role-to-another-user-or-service-principal"></a>Pourquoi est-ce que je vois « Privilèges insuffisants pour effectuer l’opération » lors de l’exécution de la commande Azure CLI pour attribuer le rôle Opérateur de cluster HDInsight à un autre utilisateur ou principal de service ?

En plus d’avoir le rôle Propriétaire, l’utilisateur ou le principal de service qui exécute la commande doit disposer des autorisations Azure AD suffisantes pour rechercher les ID d’objet de la personne responsable. Ce message indique des autorisations Azure AD insuffisantes. Essayez de remplacer l’argument `-–assignee` par `–assignee-object-id` et fournissez l’ID d’objet de la personne responsable en tant que paramètre au lieu du nom (ou l’ID du principal dans le cas d’une identité managée). Pour plus d’informations, consultez la section des paramètres facultatifs de la [documentation relative à az role assignment create](/cli/azure/role/assignment#az-role-assignment-create).

Si cela ne fonctionne toujours pas, contactez votre administrateur Azure AD pour obtenir les autorisations appropriées.

### <a name="what-will-happen-if-i-take-no-action"></a>Que se passe-t-il si je n’effectue aucune action ?

À partir du 3 septembre 2019, les appels à `GET /configurations` et `POST /configurations/gateway` ne retourneront plus d’informations et l’appel à `GET /configurations/{configurationName}` ne retournera plus de paramètres sensibles comme les clés de compte de stockage ou le mot de passe du cluster. Il en va de même pour les méthodes du kit SDK et les applets de commande PowerShell correspondantes.

Si vous utilisez une version antérieure de l’un des outils pour Visual Studio, VSCode, IntelliJ ou Eclipse mentionnés ci-dessus, ils ne fonctionneront plus tant que vous n’aurez pas effectué la mise à jour.

Pour obtenir des informations plus détaillées, consultez la section correspondante de ce document pour votre scénario.
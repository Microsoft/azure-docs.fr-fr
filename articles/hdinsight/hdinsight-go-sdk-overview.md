---
title: Azure HDInsight SDK for Go
description: Documentation de référence concernant l’utilisation du SDK Azure HDInsight pour les clusters Go et Apache Hadoop
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: seodec18, devx-track-azurecli
ms.date: 01/03/2020
ms.openlocfilehash: 6fdd7b9e5fda92bd75e54ea5b4aad6a3ba6ecbea
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92748759"
---
# <a name="hdinsight-sdk-for-go-preview"></a>HDInsight SDK for Go (préversion)

## <a name="overview"></a>Vue d’ensemble
HDInsight SDK for Go fournit des classes et des fonctions qui vous permettent de gérer vos clusters HDInsight. Il inclut des opérations permettant de créer, supprimer, mettre à jour, répertorier, mettre à l’échelle, exécuter des actions de script, surveiller, obtenir des propriétés des clusters HDInsight, et bien plus encore.

> [!NOTE]  
>Une documentation de référence GoDoc pour ce kit SDK est également [disponible ici](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

* [Outil `go get`](https://github.com/golang/go/wiki/GoGetTools).
* [Go](https://golang.org/dl/).

## <a name="sdk-installation"></a>Installation du kit SDK

À partir de votre emplacement GOPATH, exécutez `go get github.com/Azure/azure-sdk-for-go/tree/master/services/preview/hdinsight/mgmt/2018-06-01-preview/hdinsight`

## <a name="authentication"></a>Authentication

Le kit de développement logiciel (SDK) doit d’abord être authentifié avec votre abonnement Azure.  Suivez l’exemple ci-dessous pour créer un principal de service et l’utiliser pour s’authentifier. Une fois cette opération terminée, vous avez une instance de `ClustersClient`, qui contient de nombreuses fonctions (décrites dans les sections suivantes) pouvant être utilisées pour effectuer des opérations de gestion.

> [!NOTE]  
> Il existe d’autres façons de s’authentifier, en plus de l’exemple suivant, peut-être mieux adaptées à vos besoins. Toutes les fonctions sont décrites ici : [Fonctions d’authentification dans le kit Azure SDK pour Go](/azure/go/azure-sdk-go-authorization)

### <a name="authentication-example-using-a-service-principal"></a>Exemple d’authentification avec un principal de service

Tout d’abord, connectez-vous à [Azure Cloud Shell](https://shell.azure.com/bash). Vérifiez que vous utilisez actuellement l’abonnement dans lequel vous souhaitez que le principal de service soit créé.

```azurecli-interactive
az account show
```

Les informations relatives à votre abonnement sont affichées au format JSON.

```json
{
  "environmentName": "AzureCloud",
  "id": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "isDefault": true,
  "name": "XXXXXXX",
  "state": "Enabled",
  "tenantId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "user": {
    "cloudShellID": true,
    "name": "XXX@XXX.XXX",
    "type": "user"
  }
}
```

Si vous n’êtes pas connecté au bon abonnement, sélectionnez le bon en exécutant : 
```azurecli-interactive
az account set -s <name or ID of subscription>
```

> [!IMPORTANT]  
> Si vous n’avez pas déjà enregistré le fournisseur de ressources HDInsight avec une autre fonction (par exemple, en créant un cluster HDInsight via le Portail Microsoft Azure), vous devez le faire une fois avant de pouvoir vous authentifier. Vous pouvez le faire à partir d’[Azure Cloud Shell](https://shell.azure.com/bash) en exécutant la commande suivante :
>```azurecli-interactive
>az provider register --namespace Microsoft.HDInsight
>```

Ensuite, choisissez un nom pour votre principal de service et créez-le avec la commande suivante :

```azurecli-interactive
az ad sp create-for-rbac --name <Service Principal Name> --sdk-auth
```

Les informations relatives au principal de service sont affichées en tant que JSON.

```json
{
  "clientId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "clientSecret": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "subscriptionId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "tenantId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

Copiez l’extrait de code ci-dessous et remplissez `TENANT_ID`, `CLIENT_ID`, `CLIENT_SECRET` et `SUBSCRIPTION_ID` avec les chaînes du JSON qui a été retourné après l’exécution de la commande pour créer le principal de service.

```golang
package main

import (
    "context"
    "github.com/Azure/go-autorest/autorest/azure/auth"
    hdi "github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2018-06-01-preview/hdinsight"
    "github.com/Azure/go-autorest/autorest/to"
)

func main() {
    var err error

    // Tenant ID for your Azure Subscription
    var TENANT_ID = ""
    // Your Service Principal App Client ID
    var CLIENT_ID = ""
    // Your Service Principal Client Secret
    var CLIENT_SECRET = ""
    // Azure Subscription ID
    var SUBSCRIPTION_ID = ""

    var credentials = auth.NewClientCredentialsConfig(CLIENT_ID, CLIENT_SECRET, TENANT_ID)
    var client = hdi.NewClustersClient(SUBSCRIPTION_ID)

    client.Authorizer, err = credentials.Authorizer()
    if (err != nil) {
        fmt.Println("Error: ", err)
    }
```

## <a name="cluster-management"></a>Gestion de cluster

> [!NOTE]  
> Cette section suppose que vous avez déjà authentifié et créé une instance `ClusterClient` que vous avez conservée dans une variable appelée `client`. Les instructions relatives à l’authentification et à l’obtention d’un `ClusterClient` se trouvent dans la section Authentification ci-dessus.

### <a name="create-a-cluster"></a>Créer un cluster

Un nouveau cluster peut être créé en appelant `client.Create()`. 

#### <a name="example"></a>Exemple

Cet exemple montre comment créer un cluster [Apache Spark](https://spark.apache.org/) avec deux nœuds principaux et un nœud Worker.

> [!NOTE]  
> Vous devez d’abord créer un groupe de ressources et un compte de stockage, comme expliqué ci-dessous. Si vous les avez déjà créés, vous pouvez ignorer ces étapes.

##### <a name="creating-a-resource-group"></a>Création d’un groupe de ressources

Vous pouvez créer un groupe de ressources à l’aide d’[Azure Cloud Shell](https://shell.azure.com/bash) en exécutant

```azurecli-interactive
az group create -l <Region Name (i.e. eastus)> --n <Resource Group Name>
```

##### <a name="creating-a-storage-account"></a>Création d’un compte de stockage

Vous pouvez créer un compte de stockage à l’aide d’[Azure Cloud Shell](https://shell.azure.com/bash) en exécutant :

```azurecli-interactive
az storage account create -n <Storage Account Name> -g <Existing Resource Group Name> -l <Region Name (i.e. eastus)> --sku <SKU i.e. Standard_LRS>
```

À présent, exécutez la commande suivante afin d’obtenir la clé pour votre compte de stockage (vous en aurez besoin pour créer un cluster) :

```azurecli-interactive
az storage account keys list -n <Storage Account Name>
```

---
L’extrait de code Go ci-dessous crée un cluster Spark avec deux nœuds principaux et un nœud Worker. Remplissez les variables vides comme expliqué dans les commentaires et n’hésitez pas à modifier d’autres paramètres en fonction de vos besoins.

```golang
// The name for the cluster you are creating
var clusterName = "";
// The name of your existing Resource Group
var resourceGroupName = "";
// Choose a username
var username = "";
// Choose a password
var password = "";
// Replace <> with the name of your storage account
var storageAccount = "<>.blob.core.windows.net";
// Storage account key you obtained above
var storageAccountKey = "";
// Choose a region
var location = "";
var container = "default";

var parameters = hdi.ClusterCreateParametersExtended {
    Location: to.StringPtr(location),
    Tags: make(map[string]*string),
    Properties: &hdi.ClusterCreateProperties {
        ClusterVersion: to.StringPtr("3.6"),
        OsType: hdi.Linux,
        ClusterDefinition: &hdi.ClusterDefinition {
            Kind: to.StringPtr("spark"),
            Configurations: map[string]map[string]interface{}{
                "gateway": {
                    "restAuthCredential.isEnabled": "True",
                    "restAuthCredential.username":  username,
                    "restAuthCredential.password":  password,
                },
            },
        },
        Tier: hdi.Standard,
        ComputeProfile: &hdi.ComputeProfile {
            Roles: &[]hdi.Role {
                hdi.Role {
                    Name: to.StringPtr("headnode"),
                    TargetInstanceCount: to.Int32Ptr(2),
                    HardwareProfile: &hdi.HardwareProfile {
                        VMSize: to.StringPtr("Large"),
                    },
                    OsProfile: &hdi.OsProfile {
                        LinuxOperatingSystemProfile: &hdi.LinuxOperatingSystemProfile {
                            Username: to.StringPtr(username),
                            Password: to.StringPtr(password),
                        },
                    },
                },
                hdi.Role {
                    Name: to.StringPtr("workernode"),
                    TargetInstanceCount: to.Int32Ptr(1),
                    HardwareProfile: &hdi.HardwareProfile {
                        VMSize: to.StringPtr("Large"),
                    },
                    OsProfile: &hdi.OsProfile {
                        LinuxOperatingSystemProfile: &hdi.LinuxOperatingSystemProfile {
                            Username: to.StringPtr(username),
                            Password: to.StringPtr(password),
                        },
                    },
                },
            },
        },
        StorageProfile: &hdi.StorageProfile {
            Storageaccounts: &[]hdi.StorageAccount {
                hdi.StorageAccount {
                    Name: to.StringPtr(storageAccount),
                    Key: to.StringPtr(storageAccountKey),
                    Container: to.StringPtr(container),
                    IsDefault: to.BoolPtr(true),
                },
            },
        },
    },
}
client.Create(context.Background(), resourceGroupName, clusterName, parameters)
```

### <a name="get-cluster-details"></a>Obtenir les détails du cluster

Pour obtenir les propriétés d’un cluster donné :

```golang
client.Get(context.Background(), "<Resource Group Name>", "<Cluster Name>")
```

#### <a name="example"></a>Exemple

Vous pouvez utiliser `get` pour confirmer que vous avez créé votre cluster avec succès.

```golang
cluster, err := client.Get(context.Background(), resourceGroupName, clusterName)
if (err != nil) {
    fmt.Println("Error: ", err)
}
fmt.Println(*cluster.Name)
fmt.Println(*cluster.ID
```

La sortie doit ressembler à :

```
<Cluster Name>
/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<Resource Group Name>/providers/Microsoft.HDInsight/clusters/<Cluster Name>
```

### <a name="list-clusters"></a>Lister les clusters

#### <a name="list-clusters-under-the-subscription"></a>Lister les clusters dans l’abonnement

```golang
client.List()
```

#### <a name="list-clusters-by-resource-group"></a>Lister les clusters par groupe de ressources

```golang
client.ListByResourceGroup("<Resource Group Name>")
```

> [!NOTE]  
> À la fois `List()` et `ListByResourceGroup()` retournent un struct `ClusterListResultPage`. Pour obtenir la page suivante, vous pouvez appeler `Next()`. Vous pouvez répéter cette opération jusqu’à ce que `ClusterListResultPage.NotDone()` retourne `false`, comme illustré dans l’exemple ci-dessous.

#### <a name="example"></a>Exemple

L’exemple suivant imprime les propriétés de tous les clusters pour l’abonnement actuel :

```golang
page, err := client.List(context.Background())
if (err != nil) {
    fmt.Println("Error: ", err)
}
for (page.NotDone()) {
    for _, cluster := range page.Values() {
        fmt.Println(*cluster.Name)
    }
    err = page.Next();
    if (err != nil) {
        fmt.Println("Error: ", err)
    }
}
```

### <a name="delete-a-cluster"></a>Suppression d'un cluster

Pour supprimer un cluster :

```golang
client.Delete(context.Background(), "<Resource Group Name>", "<Cluster Name>")
```

### <a name="update-cluster-tags"></a>Mettre à jour les balises de cluster

Vous pouvez mettre à jour les balises d’un cluster donné comme suit :

```golang
client.Update(context.Background(), "<Resource Group Name>", "<Cluster Name>", hdi.ClusterPatchParameters{<map[string]*string} of Tags>)
```

#### <a name="example"></a>Exemple

```golang
client.Update(context.Background(), "SDKTestRG", "SDKTest", hdi.ClusterPatchParameters{map[string]*string{"tag1Name" : to.StringPtr("tag1Value"), "tag2Name" : to.StringPtr("tag2Value")}})
```

### <a name="resize-cluster"></a>Redimensionner le cluster

Vous pouvez mettre à l’échelle le nombre de nœuds Worker d’un cluster en spécifiant une nouvelle taille comme suit :

```golang
client.Resize(context.Background(), "<Resource Group Name>", "<Cluster Name>", hdi.ClusterResizeParameters{<Num of Worker Nodes (int)>})
```

## <a name="cluster-monitoring"></a>Monitoring du cluster

Le kit de développement logiciel (SDK) HDInsight Management peut également être utilisé pour gérer la surveillance de vos clusters via Operations Management Suite (OMS).

De la même façon que vous avez créé `ClusterClient` pour les opérations de gestion, vous devez créer un `ExtensionClient` pour superviser les opérations. Une fois que vous avez terminé la section Authentification ci-dessus, vous pouvez créer un `ExtensionClient` comme suit :

```golang
extClient := hdi.NewExtensionsClient(SUBSCRIPTION_ID)
extClient.Authorizer, _ = credentials.Authorizer()
```

> [!NOTE]  
> Les exemples de supervision ci-dessous supposent que vous avez déjà initialisé un `ExtensionClient` appelé `extClient` et défini son `Authorizer` comme indiqué ci-dessus.

### <a name="enable-oms-monitoring"></a>Activer la supervision OMS

> [!NOTE]  
> Pour activer la supervision OMS, vous devez disposer d’un espace de travail Log Analytics existant. Si vous n’en avez pas déjà créé un, vous pouvez apprendre à le faire ici : [Créer un espace de travail Log Analytics dans le portail Azure](../azure-monitor/learn/quick-create-workspace.md).

Pour activer la surveillance OMS sur votre cluster :

```golang
extClient.EnableMonitoring(context.Background(), "<Resource Group Name", "Cluster Name", hdi.ClusterMonitoringRequest {WorkspaceID: to.StringPtr("<Workspace Id>")})
```

### <a name="view-status-of-oms-monitoring"></a>Afficher l’état de la supervision OMS

Pour obtenir l’état d’OMS sur votre cluster :

```golang
extClient.GetMonitoringStatus(context.Background(), "<Resource Group Name", "Cluster Name")
```

### <a name="disable-oms-monitoring"></a>Désactiver la supervision OMS

Pour désactiver OMS sur votre cluster :

```golang
extClient.DisableMonitoring(context.Background(), "<Resource Group Name", "Cluster Name")
```

## <a name="script-actions"></a>Actions de script

HDInsight fournit une fonction de configuration intitulée actions de script, qui appelle des scripts personnalisés pour personnaliser le cluster.

> [!NOTE]  
> Vous pouvez trouver plus d’informations sur l’utilisation des actions de script ici : [Personnaliser des clusters HDInsight Linux à l’aide d’actions de script](./hdinsight-hadoop-customize-cluster-linux.md)

### <a name="execute-script-actions"></a>Exécuter des actions de script

Vous pouvez exécuter des actions de script sur un cluster donné comme suit :

```golang
var scriptAction1 = hdi.RuntimeScriptAction{Name: to.StringPtr("<Script Name>"), URI: to.StringPtr("<URL To Script>"), Roles: <&[]string of roles>} //valid roles are "headnode", "workernode", "zookeepernode", and "edgenode"
client.ExecuteScriptActions(context.Background(), "<Resource Group Name>", "<Cluster Name>", hdi.ExecuteScriptActionParameters{PersistOnSuccess: to.BoolPtr(true), ScriptActions: &[]hdi.RuntimeScriptAction{scriptAction1}}) //add more RuntimeScriptActions to the list to execute multiple scripts
```

Pour les opérations « Supprimer une action de script » et « Lister les actions de script persistantes », vous devez créer un `ScriptActionsClient`, de la même façon que vous avez créé `ClusterClient` pour les opérations de gestion. Une fois que vous avez terminé la section Authentification ci-dessus, vous pouvez créer un `ScriptActionsClient` comme suit :

```golang
scriptActionsClient := hdi.NewScriptActionsClient(SUBSCRIPTION_ID)
scriptActionsClient.Authorizer, _ = credentials.Authorizer()
```

> [!NOTE]  
> Les exemples d’actions de script ci-dessous supposent que vous avez déjà initialisé un `ScriptActionsClient` appelé `scriptActionsClient` et défini son `Authorizer` comme indiqué ci-dessus.

### <a name="delete-script-action"></a>Supprimer une action de script

Pour supprimer une action de script persistante spécifiée sur un cluster donné :

```golang
scriptActionsClient.Delete(context.Background(), "<Resource Group Name>", "<Cluster Name>", "<Script Name>")
```

### <a name="list-persisted-script-actions"></a>Lister les actions de script persistantes

> [!NOTE]  
> `ListByCluster()` retourne un struct `ScriptActionsListPage`. Pour obtenir la page suivante, vous pouvez appeler `Next()`. Vous pouvez répéter cette opération jusqu’à ce que `ClusterListResultPage.NotDone()` retourne `false`, comme illustré dans l’exemple ci-dessous.

Pour répertorier toutes les actions de script persistantes pour le cluster spécifié :
```golang
scriptActionsClient.ListByCluster(context.Background(), "<Resource Group Name>", "<Cluster Name>")
```

#### <a name="example"></a>Exemple

```golang
page, err := scriptActionsClient.ListByCluster(context.Background(), resourceGroupName, clusterName)
if (err != nil) {
    fmt.Println("Error: ", err)
}
for (page.NotDone()) {
    for _, script := range page.Values() {
        fmt.Println(*script.Name) //There are functions to get other properties of RuntimeScriptActionDetail besides Name, such as Status, Operation, StartTime, EndTime, etc. See reference documentation.
    }
    err = page.Next();
    if (err != nil) {
        fmt.Println("Error: ", err)
    }
}
```

### <a name="list-all-scripts-execution-history"></a>Lister l’historique de toutes les exécutions de scripts

Pour cette opération, vous devez créer `ScriptExecutionHistoryClient` de la même façon que vous avez créé `ClusterClient` pour les opérations de gestion. Une fois que vous avez terminé la section Authentification ci-dessus, vous pouvez créer un `ScriptActionsClient` comme suit :

```golang
scriptExecutionHistoryClient := hdi.NewScriptExecutionHistoryClient(SUBSCRIPTION_ID)
scriptExecutionHistoryClient.Authorizer, _ = credentials.Authorizer()
```

> [!NOTE]  
> Le code ci-dessous suppose que vous avez déjà initialisé un `ScriptExecutionHistoryClient` appelé `scriptExecutionHistoryClient` et défini son `Authorizer` comme indiqué ci-dessus.

Pour répertorier tout l’historique d’exécution des scripts pour le cluster spécifié :

```golang
scriptExecutionHistoryClient.ListByCluster(context.Background(), "<Resource Group Name>", "<Cluster Name>")
```

#### <a name="example"></a>Exemple

Cet exemple imprime tous les détails de toutes les exécutions de script passées.

```golang
page, err := scriptExecutionHistoryClient.ListByCluster(context.Background(), resourceGroupName, clusterName)
if (err != nil) {
    fmt.Println("Error: ", err)
}
for (page.NotDone()) {
    for _, script := range page.Values() {
        fmt.Println(*script.Name) //There are functions to get other properties of RuntimeScriptActionDetail besides Name, such as Status, Operation, StartTime, EndTime, etc. See reference documentation.
    }
    err = page.Next();
    if (err != nil) {
        fmt.Println("Error: ", err)
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

Explorez les [documents de référence GoDoc](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/preview/hdinsight/mgmt/2015-03-01-preview/hdinsight). Les GoDocs fournissent la documentation de référence de toutes les fonctions du kit SDK.

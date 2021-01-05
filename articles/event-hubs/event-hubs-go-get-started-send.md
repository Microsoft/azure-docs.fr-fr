---
title: 'Démarrage rapide : Envoyer et recevoir des événements avec Go – Azure Event Hubs'
description: 'Démarrage rapide : Cet article décrit la procédure à suivre pour créer une application Go qui envoie des événements d’Azure Event Hubs.'
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 59cd0d757108e7579ce389d216b0ee4d569e12fd
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87002452"
---
# <a name="quickstart-send-events-to-or-receive-events-from-event-hubs-using-go"></a>Démarrage rapide : Envoyer ou recevoir des événements d’Event Hubs à l’aide de Go
Azure Event Hubs est une plateforme de diffusion de données volumineuses et un service d’ingestion d’événements, capable de recevoir et de traiter des millions d’événements par seconde. Les concentrateurs d’événements peuvent traiter et stocker des événements, des données ou la télémétrie produits par des logiciels et appareils distribués. Les données envoyées à un concentrateur d’événements peuvent être transformées et stockées à l’aide d’adaptateurs de traitement par lot/stockage ou d’un fournisseur d’analyse en temps réel. Pour une présentation détaillée d’Event Hubs, consultez [Vue d’ensemble d’Event Hubs](event-hubs-about.md) et [Fonctionnalités d’Event Hubs](event-hubs-features.md).

Ce tutoriel montre comment écrire des applications Go pour recevoir des événements à partir d’un hub d’événements ou lui en envoyer. 

> [!NOTE]
> Vous pouvez télécharger ce guide de démarrage rapide sous forme d’exemple depuis [GitHub](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/eventhubs), remplacer les chaînes `EventHubConnectionString` et `EventHubName` par les valeurs de votre hub d’événements, puis l’exécuter. Ou, vous pouvez suivre les étapes de ce didacticiel pour créer le vôtre.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce didacticiel, vous avez besoin de ce qui suit :

- Go Installé localement. Suivez [ces instructions](https://golang.org/doc/install) si nécessaire.
- Un compte Azure actif. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][] avant de commencer.
- **Créez un espace de noms Event Hubs et un Event Hub**. Utilisez le [portail Azure](https://portal.azure.com) pour créer un espace de noms de type Event Hubs et obtenez les informations de gestion nécessaires à votre application pour communiquer avec le hub d’événements. Pour créer un espace de noms et un hub d’événements, suivez la procédure décrite dans [cet article](event-hubs-create.md).

## <a name="send-events"></a>Envoyer des événements
Cette section montre comment créer une application Go pour envoyer des événements à un hub d’événements. 

### <a name="install-go-package"></a>Installer le package Go

Obteniez le package Go pour Event Hubs avec `go get` ou `dep`. Par exemple :

```bash
go get -u github.com/Azure/azure-event-hubs-go
go get -u github.com/Azure/azure-amqp-common-go/...

# or

dep ensure -add github.com/Azure/azure-event-hubs-go
dep ensure -add github.com/Azure/azure-amqp-common-go
```

### <a name="import-packages-in-your-code-file"></a>Importer des packages dans votre fichier de code

Pour importer les packages Go, utilisez l’exemple de code suivant :

```go
import (
    aad "github.com/Azure/azure-amqp-common-go/aad"
    eventhubs "github.com/Azure/azure-event-hubs-go"
)
```

### <a name="create-service-principal"></a>Créer un principal du service

Créez un principal du service en suivant les instructions fournies dans [Créer un principal du service avec Azure CLI 2.0](/cli/azure/create-an-azure-service-principal-azure-cli). Enregistrer les informations d’identification fournies dans votre environnement avec les noms suivants. Le Kit de développement logiciel (SDK) Azure pour Go et le package Event Hubs sont tous deux préconfigurés pour rechercher ces noms de variables :

```bash
export AZURE_CLIENT_ID=
export AZURE_CLIENT_SECRET=
export AZURE_TENANT_ID=
export AZURE_SUBSCRIPTION_ID= 
```

À présent, créez un fournisseur d’autorisation pour votre client Event Hubs, qui utilise ces informations d’identification :

```go
tokenProvider, err := aad.NewJWTProvider(aad.JWTProviderWithEnvironmentVars())
if err != nil {
    log.Fatalf("failed to configure AAD JWT provider: %s\n", err)
}
```

### <a name="create-event-hubs-client"></a>Créer un client Event Hubs

Le code suivant crée un client Event Hubs :

```go
hub, err := eventhubs.NewHub("namespaceName", "hubName", tokenProvider)
ctx := context.WithTimeout(context.Background(), 10 * time.Second)
defer hub.Close(ctx)
if err != nil {
    log.Fatalf("failed to get hub %s\n", err)
}
```

### <a name="write-code-to-send-messages"></a>Écrire du code pour envoyer des messages

Dans l’extrait suivant, utilisez (1) pour envoyer des messages de manière interactive à partir d’un terminal, ou (2) pour envoyer des messages au sein de votre programme :

```go
// 1. send messages at the terminal
ctx = context.Background()
reader := bufio.NewReader(os.Stdin)
for {
    fmt.Printf("Input a message to send: ")
    text, _ := reader.ReadString('\n')
    hub.Send(ctx, eventhubs.NewEventFromString(text))
}

// 2. send messages within program
ctx = context.Background()
hub.Send(ctx, eventhubs.NewEventFromString("hello Azure!"))
```

### <a name="extras"></a>Extras

Obtenez les ID des partitions dans votre hub d’événements :

```go
info, err := hub.GetRuntimeInformation(ctx)
if err != nil {
    log.Fatalf("failed to get runtime info: %s\n", err)
}
log.Printf("got partition IDs: %s\n", info.PartitionIDs)
```

Exécutez l’application pour envoyer des événements à l’Event Hub. 

Félicitations ! Vous venez d’envoyer des messages à un concentrateur d’événements.

## <a name="receive-events"></a>Recevoir des événements

### <a name="create-a-storage-account-and-container"></a>Créer un compte de stockage et un conteneur

Des états tels que des baux sur des partitions et des points de contrôle dans le flux d’événements sont partagés entre les destinataires à l’aide d’un conteneur Stockage Azure. Vous pouvez créer un compte de stockage et un conteneur avec le Kit de développement logiciel (SDK) Go ou en suivant les instructions fournies dans [Comptes de stockage Azure](../storage/common/storage-account-create.md).

Des exemples de création d’artefacts de stockage avec le Kit de développement logiciel (SDK) Go sont disponibles dans le [Référentiel d’exemples GO](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/storage) et dans l’exemple correspondant à ce didacticiel.

### <a name="go-packages"></a>Packages Go

Pour recevoir les messages, obtenez les packages Go pour Event Hubs avec `go get` ou `dep` :

```bash
go get -u github.com/Azure/azure-event-hubs-go/...
go get -u github.com/Azure/azure-amqp-common-go/...
go get -u github.com/Azure/go-autorest/...

# or

dep ensure -add github.com/Azure/azure-event-hubs-go
dep ensure -add github.com/Azure/azure-amqp-common-go
dep ensure -add github.com/Azure/go-autorest
```

### <a name="import-packages-in-your-code-file"></a>Importer des packages dans votre fichier de code

Pour importer les packages Go, utilisez l’exemple de code suivant :

```go
import (
    aad "github.com/Azure/azure-amqp-common-go/aad"
    eventhubs "github.com/Azure/azure-event-hubs-go"
    eph "github.com/Azure/azure-event-hubs-go/eph"
    storageLeaser "github.com/Azure/azure-event-hubs-go/storage"
    azure "github.com/Azure/go-autorest/autorest/azure"
)
```

### <a name="create-service-principal"></a>Créer un principal du service

Créez un principal du service en suivant les instructions fournies dans [Créer un principal du service avec Azure CLI 2.0](/cli/azure/create-an-azure-service-principal-azure-cli). Enregistrer les informations d’identification fournies dans votre environnement avec les noms suivants : Le SDK Azure pour Go et le package Event Hubs sont tous deux préconfigurés pour rechercher ces noms de variables.

```bash
export AZURE_CLIENT_ID=
export AZURE_CLIENT_SECRET=
export AZURE_TENANT_ID=
export AZURE_SUBSCRIPTION_ID= 
```

Ensuite, créez un fournisseur d’autorisation pour votre client Event Hubs, qui utilise ces informations d’identification :

```go
tokenProvider, err := aad.NewJWTProvider(aad.JWTProviderWithEnvironmentVars())
if err != nil {
    log.Fatalf("failed to configure AAD JWT provider: %s\n", err)
}
```

### <a name="get-metadata-struct"></a>Obtenir la structure des métadonnées

Obtenez une structure avec des métadonnées sur votre environnement Azure en utilisant le Kit de développement logiciel (SDK) Azure. Les opérations ultérieures utilisent cette structure pour trouver des points de terminaison corrects.

```go
azureEnv, err := azure.EnvironmentFromName("AzurePublicCloud")
if err != nil {
    log.Fatalf("could not get azure.Environment struct: %s\n", err)
}
```

### <a name="create-credential-helper"></a>Créer une assistance des informations d'identification 

Créez une assistance des informations d'identification qui utilise les informations d’identification Azure Active Directory (AAD) précédentes pour créer des informations d’identification de Signature d’accès partagé (SAP) pour le stockage. Le dernier paramètre indique à ce constructeur d’utiliser les variables d’environnement utilisées précédemment :

```go
cred, err := storageLeaser.NewAADSASCredential(
    subscriptionID,
    resourceGroupName,
    storageAccountName,
    storageContainerName,
    storageLeaser.AADSASCredentialWithEnvironmentVars())
if err != nil {
    log.Fatalf("could not prepare a storage credential: %s\n", err)
}
```

### <a name="create-a-check-pointer-and-a-leaser"></a>Créer un pointeur de contrôle et un bailleur 

Créez un **bailleur**, chargé de louer une partition à un destinataire particulier, et un **pointeur de contrôle**, chargé d’écrire des points de contrôle pour le flux de messages afin que d’autres destinataires puissent commencer à lire à partir du décalage correct.

Actuellement, un seul **StorageLeaserCheckpointer** est disponible, qui utilise le même conteneur de stockage pour gérer les baux et les points de contrôle. En plus des noms de compte et conteneur de stockage, le **StorageLeaserCheckpointer** a besoin des informations d’identification et de la structure de l’environnement Azure créées à l’étape précédente pour accéder correctement au conteneur.

```go
leaserCheckpointer, err := storageLeaser.NewStorageLeaserCheckpointer(
    cred,
    storageAccountName,
    storageContainerName,
    azureEnv)
if err != nil {
    log.Fatalf("could not prepare a storage leaserCheckpointer: %s\n", err)
}
```

### <a name="construct-event-processor-host"></a>Construire un hôte de processeur d’événements

Vous disposez à présent des éléments nécessaires pour construire un EventProcessorHost comme suit. Le même **StorageLeaserCheckpointer** est utilisé à la fois en tant que bailleur et pointeur de contrôle, comme décrit précédemment :

```go
ctx := context.Background()
p, err := eph.New(
    ctx,
    nsName,
    hubName,
    tokenProvider,
    leaserCheckpointer,
    leaserCheckpointer)
if err != nil {
    log.Fatalf("failed to create EPH: %s\n", err)
}
defer p.Close(context.Background())
```

### <a name="create-handler"></a>Créer un gestionnaire 

Créez maintenant un gestionnaire et inscrivez-le auprès de l’hôte du processeur d’événements. Au démarrage de l’hôte, il s’applique cela et tous les autres gestionnaires spécifiés aux messages entrants :

```go
handler := func(ctx context.Context, event *eventhubs.Event) error {
    fmt.Printf("received: %s\n", string(event.Data))
    return nil
}

// register the handler with the EPH
_, err := p.RegisterHandler(ctx, handler)
if err != nil {
    log.Fatalf("failed to register handler: %s\n", err)
}
```

### <a name="write-code-to-receive-messages"></a>Écrire du code pour recevoir des messages

Quand tout est configuré, vous pouvez démarrer l’hôte du processeur d’événements avec `Start(context)` pour qu’il s’exécute en permanence, ou avec `StartNonBlocking(context)` pour qu’il s’exécute uniquement aussi longtemps que des messages sont disponibles.

Ce didacticiel démarre et s’exécute comme suit. Voir l’exemple GitHub pour obtenir un exemple d’utilisation de `StartNonBlocking` :

```go
ctx := context.Background()
err = p.Start()
if err != nil {
    log.Fatalf("failed to start EPH: %s\n", err)
}
```

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants :

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Fonctionnalités et terminologie dans Azure Event Hubs](event-hubs-features.md)
- [FAQ sur les hubs d’événements](event-hubs-faq.md)


<!-- Links -->
[Event Hubs overview]: event-hubs-about.md
[compte gratuit]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio

---
title: Configurer le protocole TLS (Transport Layer Security) pour une application cliente
titleSuffix: Azure Storage
description: Configurez une application cliente pour communiquer avec le stockage Azure à l’aide d’une version minimale de TLS (Transport Layer Security).
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/08/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 37b8c79df5b208feea185292fa09c323b64fa27d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89001805"
---
# <a name="configure-transport-layer-security-tls-for-a-client-application"></a>Configurer le protocole TLS (Transport Layer Security) pour une application cliente

Pour des raisons de sécurité, un compte de stockage Azure peut exiger que les clients utilisent une version minimale de TLS (Transport Layer Security) pour envoyer des demandes. Les appels au stockage Azure échouent si le client utilise une version de TLS inférieure à la version minimale requise. Par exemple, si un compte de stockage nécessite TLS 1.2, une demande envoyée par un client qui utilise TLS 1.1 échoue.

Cet article explique comment configurer une application cliente pour qu’elle utilise une version particulière de TLS. Pour plus d’informations sur la configuration d’une version minimale requise de TLS pour un compte de stockage Azure, consultez [Configurer la version minimale requise du protocole TLS (Transport Layer Security) pour un compte de stockage](transport-layer-security-configure-minimum-version.md).

## <a name="configure-the-client-tls-version"></a>Configurer la version TLS du client

Pour qu’un client envoie une demande avec une version particulière de TLS, le système d’exploitation doit prendre en charge cette version.

Les exemples suivants montrent comment définir la version TLS du client sur 1.2 à partir de PowerShell ou de .NET. Le .NET Framework utilisé par le client doit prendre en charge TLS 1.2. Pour plus d’informations, consultez [Prise en charge de TLS 1.2](/dotnet/framework/network-programming/tls#support-for-tls-12).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

L’exemple suivant montre comment activer TLS 1.2 dans un client PowerShell :

```powershell
# Set the TLS version used by the PowerShell client to TLS 1.2.
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;

# Create a new container.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context
New-AzStorageContainer -Name "sample-container" -Context $ctx
```

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

L’exemple suivant montre comment activer TLS 1.2 dans un client .NET à l’aide de la version 12 de la bibliothèque cliente de stockage Azure :

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Networking.cs" id="Snippet_ConfigureTls12":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

L’exemple suivant montre comment activer TLS 1.2 dans un client .NET à l’aide de la version 11 de la bibliothèque cliente de stockage Azure :

```csharp
static void EnableTls12()
{
    // Enable TLS 1.2 before connecting to Azure Storage
    System.Net.ServicePointManager.SecurityProtocol = System.Net.SecurityProtocolType.Tls12;

    // Add your connection string here.
    string connectionString = "";

    // Connect to Azure Storage and create a new container.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(connectionString);
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");
    container.CreateIfNotExists();
}
```

---

## <a name="verify-the-tls-version-used-by-a-client"></a>Vérifier la version de TLS utilisée par un client

Pour vérifier que la version spécifiée de TLS a été utilisée par le client pour envoyer une demande, vous pouvez utiliser [Fiddler](https://www.telerik.com/fiddler) ou un outil similaire. Ouvrez Fiddler pour commencer à capturer le trafic réseau du client, puis exécutez l’un des exemples de la section précédente. Examinez la trace Fiddler pour vérifier que la version correcte de TLS a été utilisée pour envoyer la demande, comme illustré dans l’image suivante.

:::image type="content" source="media/transport-layer-security-configure-client-version/fiddler-trace-tls-version.png" alt-text="Capture d’écran montrant la trace Fiddler qui indique la version de TLS utilisée sur la demande":::

## <a name="next-steps"></a>Étapes suivantes

- [Configurer la version minimale requise du protocole TLS (Transport Layer Security) pour un compte de stockage](transport-layer-security-configure-minimum-version.md)
- [Recommandations de sécurité pour Stockage Blob](../blobs/security-recommendations.md)

---
title: Tutoriel pour transférer des données vers un compte de stockage avec Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Découvrez comment ajouter des comptes de stockage local et Edge sur un appareil Azure Stack Edge Pro avec GPU et vous y connecter.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to add and connect to storage accounts on Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: c24970b30e522d593a491194dbbabb0421ad7f4b
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185536"
---
# <a name="tutorial-transfer-data-via-storage-accounts-with-azure-stack-edge-pro-gpu"></a>Tutoriel : Transférer des données par le biais de comptes de stockage avec Azure Stack Edge Pro avec GPU 

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Ce tutoriel explique comment ajouter des comptes de stockage et s’y connecter sur votre appareil Azure Stack Edge Pro. Une fois que vous avez ajouté les comptes de stockage, Azure Stack Edge Pro peut transférer des données vers Azure.

Cette procédure peut prendre environ 30 minutes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Ajout d’un compte de stockage
> * Se connecter au compte de stockage

 
## <a name="prerequisites"></a>Prérequis

Avant d’ajouter des comptes de stockage à Azure Stack Edge Pro, vérifiez que :

- Vous avez installé votre appareil physique, comme décrit dans [Installer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-install.md).

- Vous avez activé l’appareil physique, comme décrit dans [Activer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).


## <a name="add-an-edge-storage-account"></a>Ajouter un compte de stockage Edge

Pour créer un compte de stockage Edge, procédez comme suit :

[!INCLUDE [Add an Edge storage account](../../includes/azure-stack-edge-gateway-add-storage-account.md)]


## <a name="connect-to-the-edge-storage-account"></a>Se connecter au compte de stockage Edge

Vous pouvez maintenant vous connecter aux API REST du stockage Edge sur *http* ou *https*.

- *https* est la méthode sécurisée et recommandée.
- *Http* est utilisé lors de la connexion par le biais de réseaux approuvés.

## <a name="connect-via-http"></a>Se connecter par le biais de http

La connexion aux API REST de stockage Edge par le biais de http implique les étapes suivantes :

- Ajouter l’adresse IP virtuelle de service cohérente avec Azure et le point de terminaison du service Blob à l’hôte distant
- Vérifier la connexion 

Chacune de ces étapes est décrite dans les sections suivantes.

### <a name="add-device-ip-address-and-blob-service-endpoint-to-the-remote-client"></a>Ajouter l’adresse IP de l’appareil et le point de terminaison de service blob à le client distant

[!INCLUDE [Add device IP and blob service endpoint to host file](../../includes/azure-stack-edge-gateway-add-device-ip-address-blob-service-endpoint.md)]


### <a name="verify-connection"></a>Vérifier la connexion

Pour vérifier la connexion, vous avez généralement besoin des informations suivantes (qui peuvent varier) collectées à l’étape précédente :

- Nom du compte de stockage.
- Clé d’accès au compte de stockage.
- Point de terminaison de service blob.

Vous avez déjà le nom du compte de stockage et le point de terminaison du service Blob. Vous pouvez récupérer la clé d’accès du compte de stockage en vous connectant à l’appareil via Azure Resource Manager à l’aide d’un client Azure PowerShell.

Suivez les étapes décrites dans [Se connecter à l’appareil via Azure Resource Manager](azure-stack-edge-j-series-connect-resource-manager.md). Une fois que vous vous êtes connecté aux API de l’appareil local via Azure Resource Manager, récupérez la liste des comptes de stockage sur l’appareil. Exécutez l’applet de commande suivante :

`Get-AzureRMStorageAccount`

Dans la liste des comptes de stockage sur l’appareil, identifiez le compte de stockage pour lequel vous avez besoin de la clé d’accès. Notez le nom du compte de stockage et le groupe de ressources.

Voici un exemple de sortie obtenue :

```azurepowershell
PS C:\windows\system32> Get-AzureRmStorageAccount

StorageAccountName ResourceGroupName Location SkuName     Kind    AccessTier CreationTime          ProvisioningState EnableHttpsTrafficOnly
------------------ ----------------- -------- -------     ----    ---------- ------------          ----------------- ----------------------
myasetiered1       myasetiered1      DBELocal StandardLRS Storage            11/27/2019 7:10:12 PM Succeeded         False
```

Pour accéder à la clé d’accès, exécutez l’applet de commande suivante :

`Get-AzureRmStorageAccountAccessKey`

Voici un exemple de sortie obtenue :

```azurepowershell
PS C:\windows\system32> Get-AzureRmStorageAccountKey

cmdlet Get-AzureRmStorageAccountKey at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
ResourceGroupName: myasetiered1
Name: myasetiered1

KeyName Value    Permissions                                                                                
------- -----    -----------                                                                                
key1    Jb2brrNjRNmArFcDWvL4ufspJjlo+Nie1uh8Mp4YUOVQNbirA1uxEdHeV8Z0dXbsG7emejFWI9hxyR1T93ZncA==        Full
key2    6VANuHzHcJV04EFeyPiWRsFWnHPkgmX1+a3bt5qOQ2qIzohyskIF/2gfNMqp9rlNC/w+mBqQ2mI42QgoJSmavg==        Full
```

Copiez et enregistrez cette clé. Vous allez utiliser cette clé pour vérifier la connexion à l’aide de Explorateur Stockage Azure.

Pour vérifier que la connexion est correctement établie, utilisez l’Explorateur Stockage pour associer un compte de stockage externe. Si vous ne disposez pas de l’Explorateur Stockage, [téléchargez-le](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409).

[!INCLUDE [Verify connection using Storage Explorer](../../includes/azure-stack-edge-gateway-verify-connection-storage-explorer.md)]


## <a name="connect-via-https"></a>Se connecter par le biais de https

La connexion aux API REST de stockage Blob Azure par le biais de https implique les étapes suivantes :

- Récupérer votre certificat de point de terminaison d’objet Blob
- Importer le certificat sur le client ou l’hôte distant
- Ajouter l’adresse IP de l’appareil et le point de terminaison de service blob au client ou à l’hôte distant
- Configurer et vérifier la connexion

Chacune de ces étapes est décrite dans les sections suivantes.

### <a name="get-certificate"></a>Obtention de certificat

L’accès au stockage Blob sur HTTPS nécessite un certificat SSL pour l’appareil. Vous allez également télécharger ce certificat sur votre appareil Azure Stack Edge Pro en tant que fichier *.pfx* avec une clé privée qui lui est associée. Pour plus d’informations sur la façon de créer (à des fins de test et de développement uniquement) et de charger ces certificats sur votre appareil Azure Stack Edge Pro, consultez :

- [Créer le certificat de point de terminaison d’objet Blob](azure-stack-edge-j-series-manage-certificates.md#create-certificates-optional).
- [Charger le certificat de point de terminaison d’objet Blob](azure-stack-edge-j-series-manage-certificates.md#upload-certificates).
- [Importer des certificats sur le client qui accède à l’appareil](azure-stack-edge-j-series-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).

### <a name="import-certificate"></a>Importation d’un certificat

Si vous utilisez l’Explorateur Stockage Azure pour vous connecter aux comptes de stockage sur l’appareil, vous devez également importer le certificat dans l’Explorateur Stockage au format PEM. Dans l’environnement Windows, le *.cer* encodé en Base64 est le même que celui au format PEM.

Pour importer les certificats sur l’Explorateur Stockage Azure, procédez comme suit :

1. Assurez-vous que l’Explorateur Stockage Azure cible les API Azure Stack. Accédez à **Modifier > Cibler les API Azure Stack**. Quand cela vous est demandé, redémarrez l’explorateur de stockage pour appliquer la modification.

2. Pour importer des certificats SSL, accédez à **Modifier > Certificats SSL > Importer des certificats**.

  
   ![Importer le certificat dans l’Explorateur Stockage](./media/azure-stack-edge-j-series-deploy-add-storage-accounts/import-cert-storage-explorer-1.png) 

3. Parcourez et fournissez la chaîne de signature et les certificats Blob. La chaîne de signature et le certificat d’objet Blob doivent être au format PEM, qui est identique au format encodé en Base64 sur le système Windows. Vous serez averti que les certificats ont été correctement importés.


### <a name="add-device-ip-address-and-blob-service-endpoint"></a>Ajouter l’adresse IP de l’appareil et le point de terminaison du service blob

Suivez les mêmes étapes que celles effectuées pour [ajouter l’adresse IP de l’appareil et le point de terminaison du service blob avec une connexion sur *http*](#add-device-ip-address-and-blob-service-endpoint-to-the-remote-client).

### <a name="configure-and-verify-connection"></a>Configurer et vérifier la connexion

Suivez les mêmes étapes que celles effectuées pour [Configurer et vérifier la connexion utilisée lors de votre connexion sur *http*](#verify-connection). La seule différence est que vous devez laisser la case *Utiliser HTTP* non cochée.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi les connaissances suivantes sur Azure Stack Edge Pro :

> [!div class="checklist"]
> * Ajout d’un compte de stockage
> * Se connecter à un compte de stockage

Pour savoir comment transformer vos données en utilisant Azure Stack Edge Pro, passez au tutoriel suivant :

> [!div class="nextstepaction"]
> [Transformer des données avec Azure Stack Edge Pro](./azure-stack-edge-j-series-deploy-configure-compute.md)



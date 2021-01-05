---
title: Fichier include
description: Fichier include
services: virtual-machines-windows
author: rothja
manager: craigg
tags: azure-service-management
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/30/2018
ms.author: jroth
ms.custom: include file
ms.openlocfilehash: 3509185baa3a9d7be90c1fa4bd8000da4a8a6fe5
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95558744"
---
## <a name="prepare-for-akv-integration"></a>Préparation pour AKV Integration
Il existe plusieurs conditions préalables pour utiliser Azure Key Vault Integration pour configurer votre machine virtuelle SQL Server : 

1. [Installation d'Azure Powershell](#install)
2. [Création d'un Azure Active Directory](#register)
3. [Création d’un coffre de clés](#createkeyvault)

Les sections suivantes décrivent ces conditions préalables et les informations que vous devez collecter pour exécuter ultérieurement des applets de commande PowerShell.

[!INCLUDE [updated-for-az](./updated-for-az.md)]

### <a name="install-azure-powershell"></a><a id="install"></a> Installer Azure PowerShell
Assurez-vous que vous avez installé la dernière version du module Azure PowerShell. Pour plus d’informations, consultez [Installer et configurer Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="register-an-application-in-your-azure-active-directory"></a><a id="register"></a> Inscrire une application dans votre instance Azure Active Directory

Tout d’abord, votre abonnement doit comporter un [Azure Active Directory](https://azure.microsoft.com/trial/get-started-active-directory/) (ADD). Cela vous permet, entre autres avantages, d'accorder des droits d'accès à votre coffre de clés à certains utilisateurs et d'applications.

Inscrivez ensuite une application auprès d'ADD. Vous obtiendrez un compte Principal du service ayant un accès à votre coffre de clés dont votre machine virtuelle a besoin. Dans l’article Azure Key Vault, vous pouvez trouver ces étapes dans la section [Inscription d’une application auprès d’Azure Active Directory](../articles/key-vault/general/manage-with-cli2.md#registering-an-application-with-azure-active-directory), ou vous pouvez voir les étapes avec des captures d’écran dans la section **Obtention d’une identité pour l’application** de [ce billet de blog](/archive/blogs/kv/azure-key-vault-step-by-step). Avant d’effectuer ces étapes, vous devez collecter les informations suivantes au cours de cette inscription. Elles seront nécessaires ultérieurement lorsque vous activerez Azure Key Vault Integration sur votre machine virtuelle SQL.

* Après avoir ajouté l’application, trouvez **l’ID d’application** (également appelé AAD ClientID ou AppID) sur le panneau **Application inscrite**.
    L’ID d’application est affecté ultérieurement au paramètre **$spName** (nom de principal du service) dans le script PowerShell pour activer l’intégration Azure Key Vault.

   ![ID de l'application](./media/virtual-machines-sql-server-akv-prepare/aad-application-id.png)

* Au cours de ces étapes, lors de la création de la clé, copiez son secret comme dans la capture d'écran suivante. Ce secret de clé est affecté ultérieurement au paramètre **$spSecret** (secret du principal du service) dans le script PowerShell.

   ![Secret AAD](./media/virtual-machines-sql-server-akv-prepare/aad-sp-secret.png)

* L’ID d’application et le secret serviront également à créer des identifiants dans SQL Server.

* Vous devez autoriser ce nouvel ID d’application (ou ID client) à disposer des autorisations d’accès suivantes : **get**, **wrapKey**, **unwrapKey**. Cette opération s'effectue avec la cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy). Pour plus d'informations, consultez [Présentation d'Azure Key Vault](../articles/key-vault/general/overview.md).

### <a name="create-a-key-vault"></a><a id="createkeyvault"></a> Créer un coffre de clés
Pour utiliser Azure Key Vault pour stocker les clés que vous utiliserez pour le chiffrement dans votre machine virtuelle, vous devez accéder à un coffre de clés. Si vous n’avez pas déjà configuré votre coffre de clés, créez-en un en suivant les étapes décrites dans la rubrique [Prise en main d’Azure Key Vault](../articles/key-vault/general/overview.md). Avant d'effectuer ces étapes, vous devez collecter certaines informations au cours de la configuration. Elles seront nécessaires ultérieurement lorsque vous activerez Azure Key Vault Integration sur votre machine virtuelle SQL.

```azurepowershell
New-AzKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia'
```

Lorsque vous arrivez à l’étape de création d’un coffre de clés, notez la propriété **vaultUri** renvoyée. Il s’agit de l’URL du coffre de clés. Dans l’exemple utilisé à cette étape et illustré ci-dessous, le nom du coffre de clés est ContosoKeyVault. Par conséquent, l’URL du coffre de clés sera https://contosokeyvault.vault.azure.net/.

L’URL du coffre de clés est affectée ultérieurement au paramètre **$akvURL** dans le script PowerShell pour activer l’intégration du coffre de clés Azure.

Maintenant que le coffre de clés a été créé, nous devons y ajouter une clé, à laquelle il sera fait référence plus tard quand nous créerons une clé asymétrique dans SQL Server.
---
title: Chiffrer des informations d’identification dans Azure Data Factory
description: Découvrez comment chiffrer et stocker des informations d’identification pour vos magasins de données locaux sur une machine avec runtime d’intégration auto-hébergé.
services: data-factory
documentationcenter: ''
author: nabhishek
manager: anandsub
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/15/2018
ms.author: abnarain
ms.openlocfilehash: cd775c5a3bf367600a4537a9409a9bb8f902f588
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82628978"
---
# <a name="encrypt-credentials-for-on-premises-data-stores-in-azure-data-factory"></a>Chiffrer des informations d’identification pour vos magasins de données locaux dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Vous pouvez chiffrer et stocker des informations d’identification pour vos magasins de données locaux (services associés avec des informations sensibles) sur une machine avec runtime d’intégration auto-hébergé. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous passez un fichier de définition JSON avec des informations d’identification à l’ <br/>applet de commande [**New-AzDataFactoryV2LinkedServiceEncryptedCredential**](/powershell/module/az.datafactory/New-AzDataFactoryV2LinkedServiceEncryptedCredential) pour générer un fichier de définition JSON de sortie avec les informations d’identification chiffrées. Ensuite, utilisez la définition JSON mise à jour pour créer les services liés.

## <a name="author-sql-server-linked-service"></a>Rédiger un service lié SQL Server
Créez un fichier JSON nommé **StorageLinkedService.json** dans n’importe quel dossier avec le contenu suivant :  

Remplacez `<servername>`, `<databasename>`, `<username>`, et `<password>` par des valeurs pour votre serveur SQL Server avant d’enregistrer le fichier. Et remplacez `<integration runtime name>` par le nom de votre runtime d’intégration. 

```json
{
    "properties": {
        "type": "SqlServer",
        "typeProperties": {
            "connectionString": "Server=<servername>;Database=<databasename>;User ID=<username>;Password=<password>;Timeout=60"
        },
        "connectVia": {
            "type": "integrationRuntimeReference",
            "referenceName": "<integration runtime name>"
        },
        "name": "SqlServerLinkedService"
    }
}
```

## <a name="encrypt-credentials"></a>Chiffrer des informations d’identification
Pour chiffrer les données sensibles de la charge utile JSON sur un runtime d’intégration auto-hébergé local, exécutez **New-AzDataFactoryV2LinkedServiceEncryptedCredential** et passez la charge utile JSON. Cette applet de commande garantit le chiffrement des informations d’identification à l’aide de DPAPI et leur stockage local sur le nœud de runtime d’intégration auto-hébergé. La charge utile de sortie contenant la référence chiffrée aux informations d’identification peut être redirigée vers un autre fichier JSON (dans ce cas, le fichier encryptedLinkedService.json).

```powershell
New-AzDataFactoryV2LinkedServiceEncryptedCredential -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "SqlServerLinkedService" -DefinitionFile ".\SQLServerLinkedService.json" > encryptedSQLServerLinkedService.json
```

## <a name="use-the-json-with-encrypted-credentials"></a>Utiliser le fichier JSON avec des informations d’identification chiffrées
À présent, utilisez le fichier JSON de sortie de la commande précédente contenant les informations d’identification chiffrées pour configurer le **SqlServerLinkedService**.

```powershell
Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "EncryptedSqlServerLinkedService" -DefinitionFile ".\encryptedSqlServerLinkedService.json" 
```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les considérations de sécurité pour le déplacement des données, consultez [Data movement security considerations](data-movement-security-considerations.md) (Considérations de sécurité pour le déplacement des données).


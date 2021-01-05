---
author: rothja
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 10/26/2018
ms.author: jroth
ms.openlocfilehash: 51dc04fbef8d09878f33d7fda6f15039d3afba3e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025890"
---
### <a name="configure-a-dns-label-for-the-public-ip-address"></a>Configurer un nom DNS pour l’adresse IP publique

Pour vous connecter au moteur de base de données SQL Server à partir d’Internet, il est recommandé de créer une étiquette DNS pour votre adresse IP publique. Vous pouvez vous connecter par adresse IP, mais l’étiquette DNS crée un enregistrement A qui est plus facile à identifier et extrait l’adresse IP publique sous-jacente.

> [!NOTE]
> Les noms DNS ne sont pas nécessaires si vous prévoyez uniquement de vous connecter à l’instance SQL Server dans le même réseau virtuel ou localement.

Pour créer un nom DNS, sélectionnez tout d’abord **Machines virtuelles** dans le portail. Sélectionnez votre machine virtuelle SQL Server pour afficher ses propriétés.

1. Dans l’aperçu de la machine virtuelle, sélectionnez votre **adresse IP publique**.

    ![adresse IP publique](./media/virtual-machines-sql-server-connection-steps/rm-public-ip-address.png)

1. Dans les propriétés de votre adresse IP publique, développez **Configuration**.

1. Entrez un nom DNS. Il s’agit d’un enregistrement A qui peut être utilisé pour la connexion à votre machine virtuelle SQL Server par nom plutôt que directement par l’adresse IP.

1. Cliquez sur le bouton **Enregistrer** .

    ![nom dns](./media/virtual-machines-sql-server-connection-steps/rm-dns-label.png)

### <a name="connect-to-the-database-engine-from-another-computer"></a>Connexion au moteur de base de données à partir d'un autre ordinateur

1. Sur un ordinateur connecté à Internet, ouvrez SQL Server Management Studio (SSMS). Si vous n’avez pas SQL Server Management Studio, vous pouvez le télécharger [ici](/sql/ssms/download-sql-server-management-studio-ssms).

1. Dans la boîte de dialogue **Se connecter au serveur** ou **Se connecter au moteur de base de données**, modifiez la valeur **Nom du serveur**. Entrez l’adresse IP ou le nom DNS complet de la machine virtuelle (déterminé lors de la tâche précédente). Vous pouvez également ajouter une virgule et indiquer le port TCP du serveur SQL. Par exemple : `mysqlvmlabel.eastus.cloudapp.azure.com,1433`.

1. Dans la zone **Authentification**, sélectionnez **Authentification SQL Server**.

1. Dans la zone **Connexion** , saisissez le nom d’une connexion SQL valide.

1. Dans la zone **Mot de passe** , saisissez le mot de passe de la connexion.

1. Cliquez sur **Connecter**.

    ![connecter ssms](./media/virtual-machines-sql-server-connection-steps/rm-ssms-connect.png)
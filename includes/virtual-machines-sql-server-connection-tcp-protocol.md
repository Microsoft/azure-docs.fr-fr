---
author: rothja
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 10/26/2018
ms.author: jroth
ms.openlocfilehash: f7af6962c3343cd9d3e35e96d88650aa6a42c6b3
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95559227"
---
1. Une fois connecté à la machine virtuelle avec le Bureau à distance, recherchez **Configuration Manager** :

    ![Ouvrir le gestionnaire de configuration SQL Server](./media/virtual-machines-sql-server-connection-tcp-protocol/sql-server-configuration-manager.png)

1. Dans le Gestionnaire de configuration SQL Server, dans le volet de la console, développez **Configuration du réseau SQL Server**.

1. Dans le volet de la console, cliquez sur **Protocoles pour MSSQLSERVER** (le nom d’instance par défaut). Dans le volet Détails, faites un clic droit sur **TCP** et cliquez sur **Activer** s’il n’est pas déjà activé.

    ![Activer TCP](./media/virtual-machines-sql-server-connection-tcp-protocol/enable-tcp.png)

1. Dans le volet de la console, cliquez sur **Services SQL Server**. Dans le volet d’informations, cliquez avec le bouton droit sur ***SQL Server (* nom de l’instance**) (l’instance par défaut est **SQL Server (MSSQLSERVER)** ), puis cliquez sur **Redémarrer** pour arrêter et redémarrer l’instance de SQL Server.

    ![Redémarrer le moteur de base de données](./media/virtual-machines-sql-server-connection-tcp-protocol/restart-sql-server.png)

1. Fermez le Gestionnaire de configuration de SQL Server.

Pour plus d'informations sur l'activation des protocoles pour le moteur de base de données SQL Server, consultez la page [Activer ou désactiver un protocole réseau de serveur](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol).
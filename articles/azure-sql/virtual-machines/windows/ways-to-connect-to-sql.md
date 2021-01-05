---
title: Se connecter à une machine virtuelle SQL Server (Resource Manager) | Microsoft Docs
description: Découvrez comment vous connecter à votre machine virtuelle SQL Server sur Azure. Cette rubrique utilise le modèle de déploiement classique. Les scénarios diffèrent selon la configuration réseau et l’emplacement du client.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: aa5bf144-37a3-4781-892d-e0e300913d03
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 12/12/2017
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 28b68178b98e53b7a7d4192ad20c05a667344969
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97356715"
---
# <a name="connect-to-a-sql-server-virtual-machine-on-azure"></a>Se connecter à une machine virtuelle SQL Server sur Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

## <a name="overview"></a>Vue d’ensemble

Cette rubrique décrit comment se connecter à SQL Server sur une machine virtuelle Azure. Elle traite certains [scénarios de connectivité généraux](#connection-scenarios) et fournit des [étapes dans le portail pour modifier les paramètres de connectivité](#change). Si vous devez résoudre un problème ou configurer la connectivité en dehors du portail, consultez la [configuration manuelle](#manual) à la fin de cette rubrique. 

Si vous préférez suivre une procédure pas à pas complète d’approvisionnement et de connectivité, consultez [Approvisionnement d’une machine virtuelle SQL Server sur Azure](create-sql-vm-portal.md).

## <a name="connection-scenarios"></a>Scénarios de connexion

La méthode utilisée par un client pour se connecter à une machine virtuelle SQL Server diffère selon l’emplacement du client et la configuration du réseau.

Si vous configurez un ordinateur virtuel SQL Server dans le portail Azure, vous avez la possibilité de spécifier le type de **connectivité SQL**.

![Option de connectivité SQL publique lors de l’approvisionnement](./media/ways-to-connect-to-sql/sql-vm-portal-connectivity.png)

Les options de connectivité sont les suivantes :

| Option | Description |
|---|---|
| **Public** | Se connecter à SQL Server via Internet. |
| **Privé** | Se connecter à SQL Server dans le même réseau virtuel. |
| **Local** | Se connecter à SQL Server localement dans le même réseau virtuel. | 

Les sections suivantes expliquent les options **Public** et **Privé** plus en détail.

## <a name="connect-to-sql-server-over-the-internet"></a>Se connecter à SQL Server via Internet

Si vous souhaitez vous connecter à votre moteur de base de données SQL Server à partir d’Internet, sélectionnez **Public** pour le type de **connectivité SQL** dans le portail lors de la configuration. Le portail effectue automatiquement les étapes suivantes :

* Il active le protocole TCP/IP pour SQL Server.
* Il configure une règle de pare-feu pour ouvrir le port TCP de SQL Server (1433 par défaut).
* Il active l’authentification SQL Server, requis pour l’accès public.
* Il configure le groupe de sécurité réseau sur l’ordinateur virtuel pour tout trafic TCP sur le port SQL Server.

> [!IMPORTANT]
> Les images de machine virtuelle des éditions SQL Server Developer et Express n’activent pas automatiquement le protocole TCP/IP. Pour les éditions Developer et Express, vous devez utiliser le gestionnaire de configuration SQL Server pour [activer manuellement le protocole TCP/IP](#manualtcp) après création de la machine virtuelle.

Tout client disposant d’un accès à Internet peut se connecter à l’instance SQL Server en spécifiant l’adresse IP publique de la machine virtuelle ou le nom DNS attribué à cette adresse IP. Si le port SQL Server est le port 1433, il est inutile de le spécifier dans la chaîne de connexion. La chaîne de connexion suivante se connecte à une machine virtuelle SQL avec une étiquette DNS de `sqlvmlabel.eastus.cloudapp.azure.com` à l’aide de l’authentification SQL (vous pouvez également utiliser l’IP publique).

```
Server=sqlvmlabel.eastus.cloudapp.azure.com;Integrated Security=false;User ID=<login_name>;Password=<your_password>
```

Même si cette chaîne permet aux clients de se connecter via Internet, cela ne signifie pas que tout le monde peut se connecter à votre instance SQL Server. Les clients externes doivent utiliser les nom d’utilisateur et mot de passe corrects. Toutefois, pour renforcer la sécurité, vous pouvez éviter d’utiliser le port 1433 bien connu. Par exemple, si vous deviez configurer SQL Server pour écouter sur le port 1500 et établir des règles appropriées de pare-feu et de groupe de sécurité réseau, vous pourriez vous connecter en ajoutant le numéro de port au nom du serveur. L’exemple suivant modifie le précédent en ajoutant un numéro de port personnalisé, **1500**, au nom du serveur :

```
Server=sqlvmlabel.eastus.cloudapp.azure.com,1500;Integrated Security=false;User ID=<login_name>;Password=<your_password>"
```

> [!NOTE]
> Lorsque vous interrogez SQL Server sur une machine virtuelle via Internet, toutes les données sortantes depuis le centre de données Azure sont sujettes à la [tarification sur les transferts de données sortantes](https://azure.microsoft.com/pricing/details/data-transfers/) standard.

## <a name="connect-to-sql-server-within-a-virtual-network"></a>Se connecter à SQL Server dans un réseau virtuel

Lorsque vous choisissez l’option **Privé** comme type de **connectivité SQL** dans le portail, Azure effectue une configuration identique à **Public** pour la plupart des paramètres. La seule différence est qu’il n’existe aucune règle de groupe de sécurité réseau pour autoriser le trafic externe sur le port SQL Server (1433 par défaut).

> [!IMPORTANT]
> Les images de machine virtuelle des éditions SQL Server Developer et Express n’activent pas automatiquement le protocole TCP/IP. Pour les éditions Developer et Express, vous devez utiliser le gestionnaire de configuration SQL Server pour [activer manuellement le protocole TCP/IP](#manualtcp) après création de la machine virtuelle.

La connectivité privée est souvent utilisée conjointement avec un [réseau virtuel](../../../virtual-network/virtual-networks-overview.md), ce qui permet plusieurs scénarios. Vous pouvez connecter des machines virtuelles au sein d’un même réseau virtuel, même si elles existent dans différents groupes de ressources. De plus, un [VPN de site à site](../../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)permet de créer une architecture hybride qui connecte les machines virtuelles aux machines et réseaux locaux.

Les réseaux virtuels vous permettent également d’associer vos machines virtuelles Azure à un domaine. Il s’agit de la seule façon d’utiliser Authentification Windows pour SQL Server. Les autres scénarios de connexion requièrent l’authentification SQL avec des noms d’utilisateur et mots de passe.

En supposant que vous ayez configuré le DNS sur votre réseau virtuel, vous pouvez vous connecter à votre instance SQL Server en spécifiant le nom d’ordinateur de la machine virtuelle SQL Server dans la chaîne de connexion. L’exemple suivant part également du principe qu’Authentification Windows a été configurée et que l’utilisateur a accès à l’instance SQL Server.

```
Server=mysqlvm;Integrated Security=true
```

## <a name="change-sql-connectivity-settings"></a><a id="change"></a>Modifier les paramètres de connectivité SQL

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Vous pouvez modifier les paramètres de connectivité pour votre machine virtuelle de SQL Server dans le portail Azure.

1. Dans le portail Azure, sélectionnez **Machines virtuelles SQL**.

2. Sélectionnez votre machine virtuelle SQL Server.

3. Sous **Paramètres**, sélectionnez **Sécurité**.

4. Modifiez le **niveau de connectivité SQL** suivant le paramètre requis. Vous pouvez éventuellement utiliser cette zone pour modifier le port SQL Server ou les paramètres d’authentification SQL.

   ![Modifier la connectivité SQL](./media/ways-to-connect-to-sql/sql-vm-portal-connectivity-change.png)

5. Attendez quelques minutes pour terminer la mise à jour.

   ![Notification de mise à jour de la machine virtuelle SQL](./media/ways-to-connect-to-sql/sql-vm-updating-notification.png)

## <a name="enable-tcpip-for-developer-and-express-editions"></a><a id="manualtcp"></a> Activer le protocole TCP/IP pour les éditions Developer et Express

Lors de la configuration des paramètres de connectivité de SQL Server, Azure n’active pas automatiquement le protocole TCP/IP pour les éditions SQL Server Developer et Express. Les étapes ci-dessous expliquent comment activer manuellement le protocole TCP/IP pour vous connecter à distance via une adresse IP.

Connectez-vous d’abord à la machine virtuelle SQL Server à l’aide du Bureau à distance.

[!INCLUDE [Connect to SQL Server VM with remote desktop](../../../../includes/virtual-machines-sql-server-remote-desktop-connect.md)]

Ensuite, activez le protocole TCP/IP avec le **gestionnaire de configuration SQL Server**.

[!INCLUDE [Connect to SQL Server VM with remote desktop](../../../../includes/virtual-machines-sql-server-connection-tcp-protocol.md)]

## <a name="connect-with-ssms"></a>Se connecter à SSMS

Les étapes suivantes décrivent comment créer une étiquette DNS pour votre machine virtuelle Azure et vous connecter à SQL Server Management Studio (SSMS).

[!INCLUDE [Connect to SQL Server in a VM Resource Manager](../../../../includes/virtual-machines-sql-server-connection-steps-resource-manager.md)]

## <a name="manual-configuration-and-troubleshooting"></a><a id="manual"></a> Configuration manuelle et résolution des problèmes

Bien que le portail offre des options de configuration automatique de la connectivité, il est utile de savoir comment configurer manuellement la connectivité. Comprendre la configuration requise peut également faciliter la résolution des problèmes.

Le tableau suivant répertorie la configuration requise pour se connecter à SQL Server sur une machine virtuelle Azure.

| Condition requise | Description |
|---|---|
| [Activer le mode d’authentification SQL Server](/sql/database-engine/configure-windows/change-server-authentication-mode#use-ssms) | L’authentification SQL Server est nécessaire pour se connecter à distance à la machine virtuelle, sauf si vous avez configuré Active Directory sur un réseau virtuel. |
| [Créer une connexion SQL](/sql/relational-databases/security/authentication-access/create-a-login) | Si vous utilisez l’authentification SQL, vous avez besoin d’un identifiant SQL avec un nom d’utilisateur et un mot de passe qui dispose également d’autorisations sur votre base de données cible. |
| [Activer le protocole TCP/IP](#manualtcp) | SQL Server doit autoriser les connexions sur TCP. |
| [Activer la règle de pare-feu pour le port SQL Server](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access) | Le pare-feu sur la machine virtuelle doit autoriser le trafic entrant sur le port SQL Server (1433 par défaut). |
| [Créer une règle de groupe de sécurité pour TCP 1433](../../../virtual-network/manage-network-security-group.md#create-a-security-rule) | Vous devez autoriser la machine virtuelle à recevoir le trafic sur le port SQL Server (1433 par défaut) si vous souhaitez vous connecter par Internet. Ceci n’est pas nécessaire pour les connexions locales et réseau virtuel uniquement. Il s’agit de la seule étape requise dans le portail Azure. |

> [!TIP]
> Les étapes décrites dans le tableau précédent sont effectuées pour vous lorsque vous configurez la connectivité dans le portail. Utilisez ces étapes uniquement pour vérifier votre configuration ou pour configurer manuellement la connectivité pour SQL Server.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des instructions d’approvisionnement en plus de ces étapes de connectivité, consultez [Approvisionnement d’une machine virtuelle SQL Server sur Azure](create-sql-vm-portal.md).

Pour d’autres rubriques relatives à l’utilisation de SQL Server sur des machines virtuelles Azure, consultez [SQL Server sur les machines virtuelles Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).
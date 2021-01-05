---
title: Gérer des règles de pare-feu - Hyperscale (Citus) - Azure Database pour PostgreSQL
description: Créer et gérer des règles de pare-feu pour Azure Database pour PostgreSQL - Hyperscale (Citus) à l’aide du portail Azure
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 9/11/2020
ms.openlocfilehash: dadd04497eae0e91bdf5ea3caad38beda35f7fa3
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91275419"
---
# <a name="manage-firewall-rules-for-azure-database-for-postgresql---hyperscale-citus"></a>Gérer des règles de pare-feu dans Azure Database pour PostgreSQL - Hyperscale (Citus)
Les règles de pare-feu au niveau du serveur peuvent être utilisées pour gérer l’accès à un nœud coordinateur Hyperscale (Citus) à partir d’une adresse IP ou d’une plage d’adresses IP spécifique.

## <a name="prerequisites"></a>Prérequis
Pour parcourir ce guide pratique, vous avez besoin des éléments suivants :
- Un groupe de serveurs (consultez [Créer une base de données Azure pour PostgreSQL – groupe de serveurs Hyperscale (Citus)](quickstart-create-hyperscale-portal.md)).

## <a name="create-a-server-level-firewall-rule-in-the-azure-portal"></a>Créer une règle de pare-feu au niveau du serveur dans le portail Azure

> [!NOTE]
> Ces paramètres sont également accessibles lors de la création d’une base de données Azure pour PostgreSQL - groupe de serveurs Hyperscale (Citus). Sous l’onglet **Réseaux**, cliquez sur **Point de terminaison public**.

> :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/0-create-public-access.png" alt-text="Portail Azure - Onglet Réseaux":::

1. Dans la page du groupe de serveurs PostgreSQL, sous le titre Sécurité, cliquez sur **Réseaux** pour ouvrir les règles de pare-feu.

   :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/1-connection-security.png" alt-text="Portail Azure - Onglet Réseaux":::

2. Cliquez sur **Ajouter l’adresse IP actuelle du client** pour créer une règle de pare-feu avec l’adresse IP publique de votre ordinateur, telle que détectée par le système Azure.

   :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/2-add-my-ip.png" alt-text="Portail Azure - Onglet Réseaux":::

Vous pouvez également cliquer sur **+ Ajouter 0.0.0.0 - 255.255.255.255** (à droite de l’option B) pour autoriser l’accès au port 5432 du nœud coordinateur non seulement à votre adresse IP, mais à l’ensemble du réseau Internet. Dans ce cas, les clients doivent toujours se connecter avec le nom d’utilisateur et le mot de passe corrects pour utiliser le cluster. Néanmoins, nous vous recommandons d’autoriser l’accès mondial uniquement sur de courtes périodes et pour des bases de données hors production.

3. Vérifiez votre adresse IP avant d’enregistrer la configuration. Dans certains cas, l’adresse IP identifiée par le portail Azure diffère de l’adresse IP utilisée lors de l’accès à Internet et aux serveurs Azure. Par conséquent, vous devrez peut-être modifier l’adresse IP de début et l’adresse IP de fin pour que la règle fonctionne comme prévu.
   Utilisez un moteur de recherche ou tout autre outil en ligne pour vérifier votre adresse IP (par exemple, lancez une recherche sur « quelle est mon adresse IP »).

   :::image type="content" source="./media/howto-hyperscale-manage-firewall-using-portal/3-what-is-my-ip.png" alt-text="Portail Azure - Onglet Réseaux":::

4. Ajoutez des plages d’adresses supplémentaires. Dans les règles de pare-feu, vous pouvez spécifier une seule adresse IP ou une plage d’adresses. Si vous souhaitez limiter la règle à une seule adresse IP, saisissez la même adresse dans les champs d’adresse IP de début et d’adresse IP de fin. L’ouverture du pare-feu permet aux administrateurs, aux utilisateurs et aux applications d’accéder au nœud coordinateur sur le port 5432.

5. Cliquez sur **Enregistrer** sur la barre d’outils pour enregistrer cette règle de pare-feu de niveau serveur. Attendez la confirmation que la mise à jour des règles de pare-feu a réussi.

## <a name="connecting-from-azure"></a>Connexion à partir d’Azure

Il existe un moyen simple d’accorder l’accès à la base de données Hyperscale (Citus) aux applications hébergées dans Azure (par exemple, une application Azure Web Apps ou celles qui s’exécutent sur une machine virtuelle Azure). Il vous suffit de définir l’option **Permettre aux services et ressources Azure d’accéder à ce groupe de serveurs** sur **Oui** dans le volet **Réseaux** du portail, puis de cliquer sur **Enregistrer**.

> [!IMPORTANT]
> Cette option configure le pare-feu pour autoriser toutes les connexions à partir d’Azure, notamment les connexions issues des abonnements d’autres clients. Lorsque vous sélectionnez cette option, vérifiez que votre connexion et vos autorisations utilisateur limitent l’accès aux seuls utilisateurs autorisés.

## <a name="manage-existing-server-level-firewall-rules-through-the-azure-portal"></a>Gérer des règles de pare-feu au niveau du serveur existantes via le portail Azure
Répétez les étapes pour gérer les règles de pare-feu.
* Pour ajouter l’ordinateur actuel, cliquez sur le bouton + **Ajouter l’adresse IP actuelle du client**. Cliquez sur **Enregistrer** pour enregistrer les modifications.
* Pour ajouter des adresses IP supplémentaires, remplissez les champs Nom de la règle, Adresse IP de début et Adresse IP de fin. Cliquez sur **Enregistrer** pour enregistrer les modifications.
* Pour modifier une règle existante, cliquez sur les champs dans la règle et modifiez-les. Cliquez sur **Enregistrer** pour enregistrer les modifications.
* Pour supprimer une règle existante, cliquez sur les points de suspension [...], puis sur **Supprimer**. Cliquez sur **Enregistrer** pour enregistrer les modifications.

## <a name="next-steps"></a>Étapes suivantes
- Apprenez-en plus sur le [concept des règles de pare-feu](concepts-hyperscale-firewall-rules.md) et notamment sur la résolution des problèmes de connexion.

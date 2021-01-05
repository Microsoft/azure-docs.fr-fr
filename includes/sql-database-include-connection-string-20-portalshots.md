---
title: Obtenir la chaîne de connexion sur le portail Azure
description: Obtenir la chaîne de connexion sur le portail Azure
keywords: connexion sql,chaîne de connexion
services: sql-database
author: dalechen
manager: craigg
ms.service: sql-database
ms.custom: develop apps
ms.topic: include
ms.date: 07/13/2018
ms.author: ninarn
ms.openlocfilehash: b06004143d4a1ec5a2808e21b1f78833e410c195
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95554175"
---
### <a name="obtain-the-connection-string-from-the-azure-portal"></a>Obtenir la chaîne de connexion sur le portail Azure
Utilisez le [portail Azure](https://portal.azure.com/) pour obtenir la chaîne de connexion nécessaire pour que votre programme client interagisse avec Azure SQL Database.

1. Sélectionnez **Tous les services** > **Bases de données SQL**.

2. Entrez le nom de votre base de données dans la zone de texte de filtre en haut à gauche du panneau **Bases de données SQL**.

3. Sélectionnez la ligne qui correspond à votre base de données.

4. Une fois que le panneau de votre base de données s’affiche, pour un meilleur confort visuel, sélectionnez les boutons de **réduction** afin de réduire les panneaux que vous avez utilisés pour la navigation et le filtrage de la base de données.

5. Dans le panneau de votre base de données, sélectionnez **Afficher les chaînes de connexion de la base de données**.

6. Copiez la chaîne de connexion appropriée. Par exemple, si vous prévoyez d’utiliser la bibliothèque de connexions ADO.NET, copiez la chaîne appropriée de l’onglet **ADO.NET**.

    ![Copier la chaîne de connexion ADO pour votre base de données][20-CopyAdoConnectionString]

7. Modifiez la chaîne de connexion si nécessaire. Par exemple, insérez votre mot de passe dans la chaîne de connexion, ou supprimez « @&lt;nom_serveur&gt; » à partir du nom d’utilisateur si le nom du serveur ou de l’utilisateur est trop long.

8. Dans le format de votre choix, collez les informations de chaîne de connexion dans votre code de programme client.

Pour plus d’informations, consultez [Connection strings and configuration files](/dotnet/framework/data/adonet/connection-strings-and-configuration-files) (Chaînes de connexion et fichiers de configuration).

<!-- Image references. -->



[20-CopyAdoConnectionString]: ./media/sql-database-include-connection-string-20-portalshots/connqry-connstr-b.png


<!--
These three includes/ files are a sequenced set, but you can pick and choose:

includes/sql-database-include-connection-string-20-portalshots.md
includes/sql-database-include-connection-string-30-compare.md
includes/sql-database-include-connection-string-40-config.md
-->
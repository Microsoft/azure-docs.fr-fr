---
title: Créer une instance gérée Azure SQL à l’aide d’Azure Data Studio
description: Créer une instance gérée Azure SQL à l’aide d’Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 651811384a8e831c41c48ca8c4849e5ee3852054
ms.sourcegitcommit: 58f12c358a1358aa363ec1792f97dae4ac96cc4b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93280476"
---
# <a name="create-sql-managed-instance---azure-arc-using-azure-data-studio"></a>Créer une instance gérée SQL – Azure Arc utilisant Azure Data Studio

Ce document vous guide tout au long des étapes d’installation d’Azure SQL Managed Instance - Azure Arc en utilisant Azure Data Studio

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="log-in-to-the-azure-arc-data-controller"></a>Se connecter au contrôleur de données Azure Arc

Avant de pouvoir créer une instance, connectez-vous au contrôleur de données Azure Arc si vous ne l’avez pas encore fait.

```console
azdata login
```

Vous êtes ensuite invité à entrer l’espace de noms dans lequel le contrôleur de données est créé, le nom d’utilisateur et le mot de passe pour vous connecter au contrôleur.  

> Si vous devez valider l’espace de noms, vous pouvez exécuter ```kubectl get pods -A``` pour obtenir une liste de tous les espaces de noms sur le cluster.

```console
Username: arcadmin
Password:
Namespace: arc
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="create-azure-sql-managed-instance-on-azure-arc"></a>Créer une instance gérée Azure SQL sur Azure Arc

- Lancer Azure Data Studio
- Sous l’onglet Connexions, cliquez sur les trois points situés en haut à gauche, puis choisissez « Nouveau déploiement »
- Dans les options de déploiement, sélectionnez **Azure SQL Managed Instance - Azure Arc** 
  > [!NOTE]
  > Vous pouvez être invité à installer l’outil [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] ici s’il n’est pas encore installé.
- Acceptez la déclaration de confidentialité et les termes du contrat de licence, puis cliquez dans le bas sur **Sélectionner**



- Dans le panneau Déployer Azure SQL Managed Instance - Azure Arc, entrez les informations suivantes :
  - Entrez un nom pour l’instance SQL Server.
  - Entrez et confirmer un mot de passe pour l’instance SQL Server.
  - Sélectionnez la classe de stockage appropriée pour les données.
  - Sélectionnez la classe de stockage appropriée pour les journaux.

- Cliquez sur le bouton **Déployer**.

- Cette action doit lancer la création de l’instance gérée Azure SQL – Azure Arc sur le contrôleur de données.

- Votre création ne devrait prendre que quelques minutes.

## <a name="connect-to-azure-sql-managed-instance---azure-arc-from-azure-data-studio"></a>Se connecter à Azure SQL Managed Instance - Azure Arc depuis Azure Data Studio

- Connectez-vous au contrôleur de données Azure Arc en spécifiant l’espace de noms, le nom d’utilisateur et le mot de passe pour le contrôleur de données : 
```console
azdata login
```

- Affichez toutes les instances Azure SQL Managed Instance provisionnées en utilisant les commandes suivantes :

```console
azdata arc sql mi list
```

La sortie doit ressembler à ceci ; copiez l’élément ServerEndpoint (y compris le numéro de port) à partir d’ici.

```console

Name          Replicas    ServerEndpoint     State
------------  ----------  -----------------  -------
sqlinstance1  1/1         25.51.65.109:1433  Ready
```

- Dans Azure Data Studio, sous l’onglet **Connexions** , cliquez sur **Nouvelle connexion** dans la vue **Serveurs**.
- Dans le panneau **Connexion** , collez l’élément ServerEndpoint dans la zone de texte Serveur.
- Sélectionnez **Connexion SQL** comme type d’authentification.
- Entrez *sa* comme nom d’utilisateur.
- Entrez le mot de passe du compte `sa`.
- Si vous le souhaitez, entrez le nom de la base de données spécifique à laquelle se connecter.
- Si vous le souhaitez, sélectionnez Ajouter un nouveau groupe de serveurs, selon vos besoins.
- Sélectionnez **Se connecter** pour vous connecter à Azure SQL Managed Instance - Azure Arc.




## <a name="next-steps"></a>Étapes suivantes

Essayez à présent de [superviser votre instance SQL](monitor-grafana-kibana.md)

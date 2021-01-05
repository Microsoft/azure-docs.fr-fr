---
title: Utiliser Azure Data Studio pour gérer votre instance PostgreSQL
description: Utiliser Azure Data Studio pour gérer votre instance PostgreSQL
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 7dcc0f916a15598060e034dcf62536ee13e2672e
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92320236"
---
# <a name="use-azure-data-studio-to-manage-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Utiliser Azure Data Studio pour gérer vos groupes de serveurs PostgreSQL Hyperscale compatibles avec Azure Arc


Cet article explique comment :
- gérer vos instances PostgreSQL à l’aide d’affichages de tableaux de bord tels que Vue d’ensemble, Chaînes de connexion, Propriétés, Azure Resource Health...
- utiliser vos données et votre schéma

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Prérequis

- [Installer azdata, Azure Data Studio et l’interface de ligne de commande Azure](install-client-tools.md)
- Installer l’outil **[!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]** et les extensions **Azure Arc** et **PostgreSQL** dans Azure Data Studio
- Créer le [contrôleur de données Azure Arc](create-data-controller-using-azdata.md)
- Lancer Azure Data Studio

## <a name="connect-to-the-azure-arc-data-controller"></a>Se connecter au contrôleur de données Azure Arc

Dans Azure Data Studio, développez le nœud **Contrôleurs Azure Arc** et sélectionnez le bouton **Connexion au contrôleur**  :

Entrez les informations de connexion à votre contrôleur de données Azure :

- **URL du contrôleur :**

    URL de connexion à votre contrôleur dans Kubernetes. Entré sous la forme de `https://<IP_address_of_the_controller>:<Kubernetes_port.` par exemple :

    ```console
    https://12.345.67.890:30080
    ```
- **Nom d’utilisateur :**

    Nom du compte d’utilisateur que vous utilisez pour vous connecter au contrôleur. Utilisez le nom que vous utilisez généralement lorsque vous exécutez `azdata login`. Il ne s’agit pas du nom de l’utilisateur PostgreSQL que vous utilisez pour vous connecter au moteur de base de données PostgreSQL en général à partir de psql.
- **Mot de passe :** Mot de passe du compte d’utilisateur que vous utilisez pour vous connecter au contrôleur


Azure Data Studio affiche votre contrôleur de données Arc. Développez-le et affichez la liste des instances PostgreSQL qu’il gère.

## <a name="manage-your-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Gérer vos groupes de serveurs PostgreSQL Hyperscale compatibles avec Azure Arc

Cliquez avec le bouton droit sur l’instance PostgreSQL que vous voulez gérer et sélectionnez [Gérer].

Affichage du tableau de bord PostgreSQL :

Comporte plusieurs tableaux de bord figurant sur le côté gauche de ce volet :

- **Vue d’ensemble :** Affiche des informations résumées sur votre instance, telles que le nom, le nom d’utilisateur de l’administrateur PostgreSQL, l’ID d’abonnement Azure, la configuration, la version du moteur de base de données, les points de terminaison pour Grafana et Kibana…
- **Chaînes de connexion :** Affiche différentes chaînes de connexion que vous devrez peut-être connecter à votre instance PostgreSQL comme psql, node. js, PHP, Ruby...
- **Diagnostiquer et résoudre les problèmes :** Page d’accueil dans laquelle vous trouverez différentes ressources qui vous aideront à dépanner votre instance au fur et à mesure que nous développerons les notebooks de résolution des problèmes
- **Nouvelle demande de support :** page d’accueil à partir de laquelle vous serez en mesure de demander l’aide de nos services de support technique dès le début de la préversion publique.

## <a name="work-with-your-data-and-schema"></a>Utiliser vos données et votre schéma

Sur le côté gauche de la fenêtre Azure Data Studio, développez le nœud **Serveurs**  :

Sélectionnez [Ajouter une connexion] et renseignez les informations de connexion à votre instance PostgreSQL :
- **Type de connexion :** PostgreSQL
- **Nom du serveur :** entrez le nom de votre instance PostgreSQL. Par exemple : postgres01
- **Type d’authentification**  : Mot de passe
- **Nom d’utilisateur :** par exemple, vous pouvez utiliser le nom d’utilisateur administrateur PostgreSQL standard/par défaut. Notez que ce champ respecte la casse.
- **Mot de passe :** vous trouverez le mot de passe du nom d’utilisateur PostgreSQL dans la chaîne de connexion psql dans la sortie de la commande `azdata postgres server endpoint -n postgres01`
- **Nom de la base de données :** définissez le nom de la base de données à laquelle vous souhaitez vous connecter. Vous pouvez laisser la valeur __par défaut__
- **Groupe de serveurs :** vous pouvez lui permettre de définir la valeur __par défaut__
- **Nom (facultatif) :** vous pouvez laisser ce champ vide
- **Avancés :**
    - **Adresse IP de l’hôte :** est l’adresse IP publique du cluster Kubernetes
    - **Port :** est le port sur lequel votre instance PostgreSQL écoute le trafic. Ce port se trouve à la fin de la chaîne de connexion psql dans la sortie de la commande `azdata postgres server endpoint -n postgres01`. Il ne s’agit pas du port 30080 sur lequel Kubernetes écoute le trafic et que vous avez entré lors de la connexion au contrôleur de données Azure dans Azure Data Studio.
    - **Autres paramètres :** Ils doivent être explicites, vous pouvez les conserver avec les valeurs par défaut/vides fournies.

Sélectionnez **[OK] et [Connecter]** pour vous connecter à votre serveur.

Une fois connecté, plusieurs expériences sont disponibles :
- **Nouvelle requête**
- **Nouveau notebook**
- **Développez l’affichage de votre serveur et parcourez/travaillez sur les objets à l’intérieur de votre base de données.**
- **...**

## <a name="next-step"></a>Étape suivante
[Surveiller votre groupe de serveurs](monitor-grafana-kibana.md)

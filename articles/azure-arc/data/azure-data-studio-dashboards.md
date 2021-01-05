---
title: Tableaux de bord Azure Data Studio
description: Tableaux de bord Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: a96be6d4da3d292b2e9881652aad28f318ccee8a
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92107570"
---
# <a name="azure-data-studio-dashboards"></a>Tableaux de bord Azure Data Studio

[Azure Data Studio](/sql/azure-data-studio/what-is) fournit une expérience similaire à celle du portail Azure pour visualiser les informations sur vos ressources Azure Arc.  Ces vues sont appelées **tableaux de bord**, et si leur présentation et leurs options sont similaires à celles que vous pouvez voir sur une ressource donnée dans le portail Azure, elles vous donnent la possibilité de voir ces informations localement dans votre environnement au cas où vous n’auriez pas de connexion disponible à Azure.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="connecting-to-a-data-controller"></a>Connexion à un contrôleur de données

### <a name="prerequisites"></a>Prérequis

- Télécharger [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio)
- L’extension Azure arc est installée

### <a name="determine-the-data-controller-server-api-endpoint-url"></a>Déterminer l’URL du point de terminaison de l’API du serveur de contrôleur de données

Vous devez d’abord connecter Azure Data Studio à votre URL de point de terminaison de l’API du service de contrôleur de données.

Pour obtenir ce point de terminaison, vous pouvez exécuter la commande suivante :

```console
kubectl get svc/controller-svc-external -n <namespace name>

#Example:
kubectl get svc/controller-svc-external -n arc
```

Vous devez obtenir une sortie similaire à celle-ci :

```console
NAME                      TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                                       AGE
controller-svc-external   LoadBalancer   10.0.175.137   52.154.152.24    30080:32192/TCP                               22h
```

Si vous utilisez un type LoadBalancer, vous allez copier l’adresse IP externe et le numéro de port. Si vous utilisez NodePort, vous allez utiliser l’adresse IP de votre serveur d’API Kubernetes et le numéro de port qui se trouve sous la colonne PORT(S).

Vous voulez maintenant construire une URL vers votre point de terminaison en combinant ces informations comme suit :

```console
https://<ip address>:<port>

Example:
https://52.154.152.24:30080
```

Prenez note de votre adresse IP, car vous allez l’utiliser à l’étape suivante.

### <a name="connect"></a>Se connecter

1. Ouvrir Azure Data Studio

1. Sélectionnez l’onglet **Connexions** sur la gauche.

Dans la partie inférieure, développez le panneau nommé **Contrôleurs Azure Arc**.

Cliquez sur l’icône + pour ajouter une nouvelle connexion de contrôleur de données.

En haut de l’écran, dans la palette de commandes, entrez l’URL que vous avez construite à l’étape 1, puis appuyez sur Entrée.
Entrez le nom d’utilisateur pour le contrôleur de données.  C’était la valeur du nom d’utilisateur que vous avez passée lors du déploiement du contrôleur de données.  Cliquez sur Entrée.
Entrez le mot de passe pour le contrôleur de données.  C’était la valeur du mot de passe que vous avez passée lors du déploiement du contrôleur de données. Cliquez sur Entrée.

Maintenant que vous êtes connecté à un contrôleur de données, vous pouvez visualiser les tableaux de bord pour le contrôleur de données, et toutes les instances SQL managées ou les ressources du groupe de serveurs PostgreSQL Hyperscale dont vous disposez.

## <a name="view-the-data-controller-dashboard"></a>Visualiser le tableau de bord du contrôleur de données

Cliquez avec le bouton droit sur le contrôleur de données dans le panneau Connexions du panneau extensible **Contrôleurs Arc** et choisissez **Gérer**.

Vous pouvez voir ici des informations détaillées sur la ressource de contrôleur de données, comme le nom, la région, le mode de connexion, le groupe de ressources, l’abonnement, le point de terminaison de contrôleur et l’espace de noms.  Vous pouvez aussi voir une liste de toutes les ressources de base de données managées gérées par le contrôleur de données.

Notez que la présentation est similaire à ce que vous pouvez voir dans le portail Azure.

Vous pouvez facilement lancer la création d’une instance SQL managée ou d’un groupe de serveurs PostgreSQL Hyperscale en cliquant sur le bouton + Nouvelle instance.

Vous pouvez également ouvrir le portail Azure dans le contexte de ce contrôleur de données en cliquant sur le bouton Ouvrir dans le portail Azure.

## <a name="view-the-sql-managed-instance-dashboards"></a>Visualiser les tableaux de bord des instances SQL managées

Si vous avez créé des instances SQL managées, vous pouvez les voir listées dans le panneau Connexions du panneau extensible Contrôleurs de données Azure, sous le contrôleur de données qui les gère.

Pour visualiser le tableau de bord des instances SQL managées pour une instance spécifique, cliquez avec le bouton droit sur l’instance, puis choisissez Gérer.

Le panneau Connexion apparaît sur la droite et vous invite à indiquer la connexion/le mot de passe pour se connecter à cette instance SQL. Si vous connaissez les informations de connexion, vous pouvez les entrer, puis cliquer sur Se connecter.  Si vous ne les connaissez pas, vous pouvez cliquer sur Annuler.  Dans les deux cas, vous accédez au tableau de bord quand le panneau Connexion se ferme.

Sous l’onglet Vue d’ensemble, vous pouvez voir des détails sur l’instance SQL managée, comme le groupe de ressources, le contrôleur de données, l’ID d’abonnement, l’état, la région, etc.  Vous pouvez également voir un lien sur lequel vous pouvez cliquer pour accéder aux tableaux de bord Grafana ou Kibana dans le contexte de cette instance SQL managée.

Si vous êtes en mesure de vous connecter à l’instance SQL managée, vous pouvez voir ici d’autres informations.

Vous pouvez supprimer l’instance SQL managée d’ici ou ouvrir le portail Azure pour afficher l’instance SQL managée dans le portail Azure.

Si vous cliquez sur l’onglet Chaînes de connexion sur la gauche, vous pouvez voir une liste de chaînes de connexion préconstruites pour cette instance SQL managée, ce qui facilite le copier/coller dans d’autres applications ou dans du code.

## <a name="view-the-postgresql-hyperscale-server-group-dashboards"></a>Visualiser les tableaux de bord du groupe de serveurs PostgreSQL Hyperscale

Si vous avez créé des groupes de serveurs PostgreSQL Hyperscale, vous pouvez les voir listés dans le panneau Connexions du panneau extensible Contrôleurs de données Azure, sous le contrôleur de données qui les gère.

Pour visualiser le tableau de bord du groupe de serveurs PostgreSQL Hyperscale pour un groupe de serveurs spécifique, cliquez avec le bouton droit sur le groupe de serveurs, puis choisissez Gérer.

Sous l’onglet Vue d’ensemble, vous pouvez voir des détails sur le groupe de serveurs, comme le groupe de ressources, le contrôleur de données, l’ID d’abonnement, l’état, la région, etc.  Vous pouvez également voir un lien sur lequel vous pouvez cliquer pour accéder aux tableaux de bord Grafana ou Kibana dans le contexte de ce groupe de serveurs.

Vous pouvez supprimer le groupe de serveurs d’ici ou ouvrir le portail Azure pour afficher le groupe de serveurs dans le portail Azure.

Si vous cliquez sur l’onglet Chaînes de connexion sur la gauche, vous pouvez voir une liste de chaînes de connexion préconstruites pour ce groupe de serveurs, ce qui facilite le copier/coller dans d’autres applications ou dans du code.

Si vous cliquez sur l’onglet Propriétés à gauche, vous pouvez voir des détails supplémentaires.

Si vous cliquez sur l’onglet Intégrité des ressources à gauche, vous pouvez voir l’état général actuel de ce groupe de serveurs.

Si vous cliquez sur l’onglet Diagnostiquer et résoudre les problèmes sur la gauche, vous pouvez lancer le notebook de résolution des problèmes de PostgreSQL.

Si vous cliquez sur l’onglet Nouvelle demande de support sur la gauche, vous pouvez lancer le portail Azure dans le contexte du groupe de serveurs et créer une demande de support Azure à partir de là.
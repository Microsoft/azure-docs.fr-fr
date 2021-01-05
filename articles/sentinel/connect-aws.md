---
title: Connecter AWS CloudTrail à Azure Sentinel | Microsoft Docs
description: Utilisez le connecteur AWS pour déléguer l’accès à Azure Sentinel à vos journaux de ressources AWS, créant une relation d’approbation entre AWS CloudTrail et Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/27/2020
ms.author: yelevin
ms.openlocfilehash: a7405824d2477d2d39c45a56ae545e58a090c321
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436604"
---
# <a name="connect-azure-sentinel-to-aws-cloudtrail"></a>Connecter Azure Sentinel à AWS CloudTrail

Utilisez le connecteur AWS pour diffuser tous vos événements AWS CloudTrail vers Azure Sentinel. Ce processus de connexion délègue l’accès pour Azure Sentinel à vos journaux de ressources AWS, créant une relation d’approbation entre AWS CloudTrail et Azure Sentinel. Pour ce faire, un rôle est créé sur AWS, qui autorise Azure Sentinel à accéder à vos journaux d’activité AWS.

> [!NOTE]
> AWS CloudTrail dispose de [limitations intégrées](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/WhatIsCloudTrail-Limits.html) dans son API LookupEvents. Il n'autorise pas plus de deux transactions par seconde (TPS) par compte, et chaque requête peut renvoyer un maximum de 50 enregistrements. Par conséquent, si un seul locataire génère constamment plus de 100 enregistrements par seconde dans une région, des backlogs et des retards d'ingestion des données en résulteront.

## <a name="prerequisites"></a>Prérequis

Vous devez disposer de l’autorisation d’écriture sur l’espace de travail Azure Sentinel.

> [!NOTE]
> Azure Sentinel collecte les événements de gestion CloudTrail de toutes les régions. Il vous est conseillé de ne pas diffuser les événements d’une région à une autre.

## <a name="connect-aws"></a>Connecter AWS 


1. Dans Azure Sentinel, sélectionnez **Connecteurs de données**, puis la ligne **Amazon Web Services** dans la table et dans le volet AWS de droite, cliquez sur **Open connector page** (Ouvrir la page du connecteur).

1. Suivez les instructions sous **Configuration** en procédant comme suit.
 
1.  Dans votre console Amazon Web Services, sous **Security, Identity & Compliance** (Sécurité, identité et conformité), sélectionnez **IAM**.

    ![AWS1](./media/connect-aws/aws-1.png)

1.  Choisissez **Roles** (Rôles) et sélectionnez **Create role** (Créer un rôle).

    ![AWS2](./media/connect-aws/aws-2.png)

1.  Choisissez **Another AWS account** (Autre compte AWS). Dans le champ **Account ID** (ID de compte), entrez l’**ID de compte Microsoft** (**123412341234**) que vous trouverez dans la page du connecteur AWS dans le portail Azure Sentinel.

    ![AWS3](./media/connect-aws/aws-3.png)

1.  Assurez-vous que l’option **Require External ID** (Exiger un ID externe) est sélectionnée, puis entrez l’ID externe (ID d’espace de travail) que vous trouverez dans la page du connecteur AWS dans le portail Azure Sentinel.

    ![AWS4](./media/connect-aws/aws-4.png)

1.  Sous **Attach permissions policy** (Attacher une stratégie d’autorisation), sélectionnez **AWSCloudTrailReadOnlyAccess**.

    ![AWS5](./media/connect-aws/aws-5.png)

1.  Entrez une étiquette (facultatif).

    ![AWS6](./media/connect-aws/aws-6.png)

1.  Ensuite, entrez un **Role name** (Nom de rôle) et sélectionnez le bouton **Create role** (Créer un rôle).

    ![AWS7](./media/connect-aws/aws-7.png)

1.  Dans la liste Roles, choisissez le rôle que vous avez créé.

    ![AWS8](./media/connect-aws/aws-8.png)

1.  Copiez **Role ARN** (ARN du rôle). Sur le portail Azure Sentinel, dans l'écran du connecteur Amazon Web Services, collez-le dans le champ **Role to add** (Rôle à ajouter), puis cliquez sur **Add** (Ajouter).

    ![AWS9](./media/connect-aws/aws-9.png)

1. Pour utiliser le schéma pertinent dans Log Analytics pour les événements AWS, recherchez **AWSCloudTrail**.

    > [!IMPORTANT]
    > À compter du 1er décembre 2020, le champ **AwsRequestId** a été remplacé par le champ **AwsRequestId_** (remarquez l’ajout du trait de soulignement). Les données de l’ancien champ **AwsRequestId** seront conservées jusqu’à la fin de la période spécifiée de rétention des données du client.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter AWS CloudTrail à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.

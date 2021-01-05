---
title: Démarrage rapide de Striim
description: Devenez rapidement opérationnel avec Striim et Azure Synapse Analytics.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 10/12/2018
ms.author: martinle
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: d6b5fa1a42201479c5d426a5bbb9188d0cb8e03d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89485972"
---
# <a name="striim-azure-synapse-analytics-marketplace-offering-install-guide"></a>Guide d’installation des offres de la Place de marché Striim Azure Synapse Analytics

Ce guide de démarrage rapide part du principe que vous disposez d’une instance préexistante d’Azure Synapse Analytics.

Recherchez Striim dans la Place de marché Azure, puis sélectionnez l’option Striim for Data Integration to Azure Synapse Analytics (Staged) 

![Installer Striim][install]

Configurez la machine virtuelle Striim avec des propriétés spécifiées, en notant le nom de cluster, le mot de passe et le mot de passe administrateur Striim

![Configurer Striim][configure]

Une fois le déploiement effectué, cliquez sur \<VM Name>-masternode dans le portail Azure, cliquez sur Se connecter, puis copiez l’URL de connexion à l’aide du compte local de la machine virtuelle 

![Connecter Striim à Azure Synapse Analytics][connect]

Téléchargez le fichier sqljdbc42.jar à partir du site <https://www.microsoft.com/en-us/download/details.aspx?id=54671> sur votre ordinateur local. 

Ouvrez une fenêtre de ligne de commande et accédez au répertoire où vous avez téléchargé le fichier jar JDBC. Copiez via SCP le fichier jar sur votre machine virtuelle Striim, en obtenant l’adresse et le mot de passe à partir du portail Azure

![Copier le fichier jar sur votre machine virtuelle][copy-jar]

Ouvrez une autre fenêtre de ligne de commande, ou utilisez un utilitaire SSH pour vous connecter avec SSH au cluster Striim

![Se connecter avec SSH au cluster][ssh]

Exécutez les commandes suivantes pour déplacer le fichier jar JDBC dans le répertoire lib de Striim, et démarrer et arrêter le serveur.

   1. sudo su
   2. cd /tmp
   3. mv sqljdbc42.jar /opt/striim/lib
   4. systemctl stop striim-node
   5. systemctl stop striim-dbms
   6. systemctl start striim-dbms
   7. systemctl start striim-node

![Démarrer le cluster Striim][start-striim]

Ouvrez maintenant votre navigateur préféré et accédez à \<DNS Name>:9080

![Accéder à l’écran de connexion][navigate]

Connectez-vous avec le nom d’utilisateur et le mot de passe que vous avez configuré dans le portail Azure, puis sélectionnez votre Assistant préféré pour commencer, ou accédez à la page des applications pour commencer à utiliser l’interface utilisateur glisser-déplacer

![Se connecter avec les informations d’identification du serveur][login]



[install]: ./media/striim-quickstart/install.png
[configure]: ./media/striim-quickstart/configure.png
[connect]:./media/striim-quickstart/connect.png
[copy-jar]:./media/striim-quickstart/copy-jar.png
[ssh]:./media/striim-quickstart/ssh.png
[start-striim]:./media/striim-quickstart/start-striim.png
[navigate]:./media/striim-quickstart/navigate.png
[login]:./media/striim-quickstart/login.png

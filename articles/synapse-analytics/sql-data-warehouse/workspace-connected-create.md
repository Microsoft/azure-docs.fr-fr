---
title: Activation des fonctionnalités de l’espace de travail Synapse
description: Ce document décrit comment un utilisateur peut activer les fonctionnalités de l’espace de travail Synapse sur un pool SQL dédié existant (anciennement SQL DW).
services: synapse-analytics
author: antvgski
manager: igorstan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/25/2020
ms.author: anvang
ms.reviewer: jrasnick
ms.openlocfilehash: a1fbc6eede6c82020b765185602c672c1162fdf8
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96548186"
---
# <a name="enabling-synapse-workspace-features-for-a-dedicated-sql-pool-formerly-sql-dw"></a>Activation des fonctionnalités de l’espace de travail Synapse pour un pool SQL dédié (anciennement SQL DW)

Tous les utilisateurs de l’entrepôt de données SQL peuvent désormais accéder à une instance de pool SQL dédié (anciennement SQL DW) et l’utiliser à l’aide de Synapse Studio et de l’espace de travail. Les utilisateurs peuvent utiliser Synapse Studio et l’espace de travail sans affecter l’automatisation, les connexions ou les outils. Cet article explique comment un utilisateur Azure Synapse Analytics existant peut activer les fonctionnalités de l’espace de travail Synapse pour un pool SQL dédié existant (anciennement SQL DW). L’utilisateur peut développer sa solution Analytics existante en tirant parti des nouvelles fonctionnalités riches désormais disponibles via l’espace de travail Synapse et Studio.   

## <a name="prerequisites"></a>Prérequis
Avant d’activer les fonctionnalités de l’espace de travail Synapse sur votre entrepôt de données, vous devez vous assurer que vous disposez des éléments suivants :
- Les droits nécessaires pour créer et gérer les ressources SQL hébergées sur le serveur logique SQL.
- Les droits nécessaires pour créer des ressources Azure Synapse.
- Un administrateur Azure Active Directory identifié sur le serveur logique

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="enabling-synapse-workspace-features-for-an-existing-dedicated-sql-pool-formerly-sql-dw"></a>Activation des fonctionnalités de l’espace de travail Synapse pour un pool SQL dédié existant (anciennement SQL DW)

Les fonctionnalités de l’espace de travail Synapse peuvent être activées sur n’importe quel pool SQL dédié existant (anciennement SQL DW) dans une région prise en charge. Cette capacité est disponible uniquement via le Portail Azure.

Pour créer un espace de travail Synapse pour votre entrepôt de données existant, procédez comme suit.
1. Sélectionnez un pool SQL dédié existant (anciennement SQL DW) et ouvrez la page de présentation.
2. Sélectionnez **Nouvel espace de travail Synapse** dans la barre de menus supérieure ou le message situé immédiatement en dessous.
3. Après avoir examiné la liste des entrepôts de données disponibles par le biais du nouvel espace de travail Synapse sur la page **Créer un espace de travail Azure Synapse**. Sélectionnez **Continuer** pour poursuivre.
4. Sur la page De base, le pool SQL dédié existant (la section **Détails du projet** doit être pré-remplie avec le même **Abonnement** et **Groupe de ressources** déployé sous le serveur logique. Et sous **Détails de l’espace de travail**, le nom de l’**Espace de travail** est prérempli avec le même nom et la même région du serveur logique SQL pour garantir que la connexion entre votre nouvel espace de travail Synapse et le serveur logique puisse être créée pendant le processus d’approvisionnement. L’approvisionnement ultérieur de cette connexion doit être conservé pour garantir un accès continu aux instances du pool SQL dédié (anciennement SQL DW) par le biais de l’espace de travail et de Studio.
5. Accédez à Sélectionner Data Lake Storage Gen2.
6. Sélectionnez **Créer** ou sélectionnez un **Data Lake Storage Gen2**  existant avant de sélectionner **Suivant : Mise en réseau**.
7. Choisissez un **mot de passe d’administrateur SQL** pour votre **instance serverless**. La modification de ce mot de passe n’entraîne pas la mise à jour ou la modification des informations d’identification SQL du serveur logique. Si vous préférez un mot de passe défini par le système, laissez ces champs vides. Ce mot de passe peut être modifié à tout moment dans le nouvel espace de travail. Le nom d’administrateur doit être le même que celui utilisé dans SQL Server.
8. Sélectionnez vos **Paramètres de mise en réseau** préférés et sélectionnez **Vérifier + créer** pour démarrer l’approvisionnement de l’espace de travail.
9. Sélectionnez **Accéder à la ressource** pour ouvrir votre nouvel espace de travail.

    > [!NOTE]
    > Toutes les instances de pool SQL dédié (anciennement SQL DW) hébergées sur le serveur logique sont disponibles via le nouvel espace de travail.

## <a name="post-provisioning-steps"></a>Étapes postérieures à l’approvisionnement
Les étapes suivantes doivent être effectuées pour s’assurer que vos instances de pool SQL dédié (anciennement SQL DW) existantes sont accessibles par le biais de Synapse Studio.
1. Dans la page de présentation de l’espace de travail Synapse, sélectionnez **Serveur connecté**. Le **Serveur connecté** vous amène au serveur logique SQL connecté qui héberge vos entrepôts de données. Dans le menu essentiel, sélectionnez **Serveur connecté**.
2. Ouvrez **Pare-feux et réseaux virtuels** et assurez-vous que votre adresse IP cliente ou une plage d’adresses IP prédéterminée a accès au serveur logique.
3. Ouvrez **Administrateur Active Directory** et assurez-vous qu’un administrateur AAD a été défini sur le serveur logique.
4. Sélectionnez l’une des instances de pools SQL dédiés (anciennement SQL DW) hébergées sur le serveur logique. Dans la page de présentation, sélectionnez **Lancer Synapse Studio** ou accédez à [Se connecter à Synapse Studio](https://web.azuresynapse.net) et connectez-vous à votre espace de travail.

5. Ouvrez le **Hub Données** et développez le pool SQL dédié dans l’Explorateur d’objets pour vous assurer que vous avez accès à votre entrepôt de données et que vous pouvez l’interroger.

## <a name="next-steps"></a>Étapes suivantes
Prise en main de l’[Espace de travail Synapse et de Studio](../get-started.md).

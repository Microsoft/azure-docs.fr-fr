---
title: Configurer des clés gérées par le client pour l’API Azure pour FHIR
description: Intégrez votre propre fonctionnalité de clé prise en charge dans l’API Azure pour FHIR par le biais de Cosmos DB
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: overview
ms.date: 09/28/2020
ms.author: matjazl
ms.openlocfilehash: 05c208ba3c9005d38b8924037748764f8d112e3a
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398179"
---
# <a name="configure-customer-managed-keys"></a>Configurer les clés gérées par le client

Quand vous créez un compte d’API Azure pour FHIR, vos données sont chiffrées à l’aide de clés managées par Microsoft par défaut. À présent, vous pouvez ajouter une deuxième couche de chiffrement pour les données à l’aide de votre propre clé que vous choisissez et gérez vous-même.

Dans Azure, cette opération s’effectue généralement à l’aide d’une clé de chiffrement dans le coffre de clés Azure du client (Azure Key Vault). Azure SQL, Stockage Azure et Cosmos DB, pour ne citer qu’eux, offrent désormais cette fonctionnalité. L’API Azure pour FHIR tire parti de cette prise en charge à partir de Cosmos DB. Quand vous créez un compte, vous avez la possibilité de spécifier un URI de clé Azure Key Vault. Nous passons cette clé à Cosmos DB quand le compte de base de données est provisionné. Quand une demande FHIR est effectuée, Cosmos DB extrait votre clé et l’utilise pour chiffrer/déchiffrer les données. Pour commencer, vous pouvez consulter les liens suivants :

- [Inscrire le fournisseur de ressources Azure Cosmos DB dans l’abonnement Azure](../cosmos-db/how-to-setup-cmk.md#register-resource-provider) 
- [Configurer votre instance Azure Key Vault](../cosmos-db/how-to-setup-cmk.md#configure-your-azure-key-vault-instance)
-  [Ajouter une stratégie d’accès à votre instance Azure Key Vault](../cosmos-db/how-to-setup-cmk.md#add-an-access-policy-to-your-azure-key-vault-instance)
- [Générer une clé dans Azure Key Vault](../cosmos-db/how-to-setup-cmk.md#generate-a-key-in-azure-key-vault)

Après avoir créé votre compte d’API Azure pour FHIR sur le portail Azure, vous pouvez voir une option de configuration « Chiffrement des données » sous « Paramètres de base de données » sous l’onglet « Paramètres supplémentaires ». Par défaut, l’option Clé gérée par le service est sélectionnée. Vous pouvez spécifier votre clé Azure Key Vault ici en sélectionnant l’option « Clé gérée par le client ». Vous pouvez entrer l’URI de la clé copié ici.

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-create.png" alt-text="Créer l’API Azure pour FHIR":::

Ou vous pouvez choisir votre clé dans le sélecteur de clés :

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-keypicker.png" alt-text="Sélecteur de clés":::

Pour les comptes FHIR existants, vous pouvez afficher le choix du chiffrement à clé (clé gérée par le service ou le client) dans le panneau « Base de données » illustré ci-dessous. Vous ne pouvez pas modifier l’option de configuration une fois que vous l’avez choisie. Par contre, vous pouvez modifier et mettre à jour votre clé.

:::image type="content" source="media/bring-your-own-key/bring-your-own-key-database.png" alt-text="Sauvegarde de la base de données":::

Vous pouvez également créer une version de la clé spécifiée ; vos données seront alors chiffrées avec la nouvelle version sans aucune interruption de service. Vous pouvez également supprimer l’accès à la clé pour supprimer l’accès aux données.
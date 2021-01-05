---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/29/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 393b29245141b2970e7c1a227d6e8b1b131c445c
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061608"
---
Si vous souhaitez créer une connexion P2S à partir d’un ordinateur client différent de celui que vous avez utilisé pour générer les certificats clients, vous devez installer un certificat client. Quand vous installez un certificat client, vous avez besoin du mot de passe créé lors de l’exportation du certificat client.

1. Recherchez le fichier *.pfx* et copiez-le sur l’ordinateur client. Sur l’ordinateur client, double-cliquez sur le fichier *.pfx* à installer. Conservez la valeur **Utilisateur actuel** pour **Emplacement du magasin** , puis sélectionnez **Suivant**.
1. N’apportez aucune modification à la page **Fichier à importer** . Sélectionnez **Suivant**.
1. Sur la page **Protection de clé privée** , entrez le mot de passe du certificat, ou vérifiez que le principal de sécurité est correct, puis sélectionnez **Suivant**.
1. Sur la page **Magasin de certificats** , laissez l’emplacement par défaut, puis sélectionnez **Suivant**.
1. Sélectionnez **Terminer**. Sur la page **Avertissement de sécurité** relative à l’installation du certificat, sélectionnez **Oui**. Il n’y a aucun risque à cela, car vous avez généré le certificat.
1. Le certificat est désormais importé.

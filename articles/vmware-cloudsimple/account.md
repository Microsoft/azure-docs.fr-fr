---
title: Gestion des comptes - Azure VMware Solution by CloudSimple
description: Décrit comment gérer les comptes sur le portail Azure VMware Solution by CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 00348d36c38fdbfdf69a2e4e80a4c720268b40c8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77025365"
---
# <a name="manage-accounts-on-the-azure-vmware-solution-by-cloudsimple-portal"></a>Gérer les comptes sur le portail Azure VMware Solution by CloudSimple

Lorsque vous créez votre service CloudSimple, celui-ci crée un compte sur CloudSimple. Ce compte est associé à votre abonnement Azure où se trouve le service. Tous les utilisateurs de l’abonnement ayant le rôle propriétaire ou contributeur ont accès au portail CloudSimple. L’ID d’abonnement Azure et l’ID de locataire associés au service CloudSimple figurent sur la page Comptes.

Pour gérer les comptes dans le portail CloudSimple, [accédez au portail](access-cloudsimple-portal.md) et sélectionnez **Compte** dans le menu latéral.

Sélectionnez **Résumé** pour afficher des informations sur la configuration CloudSimple de votre entreprise. La capacité actuelle de votre configuration Cloud est indiquée, notamment avec le nombre de clouds privés, le stockage total, la configuration du cluster vSphere, le nombre de nœuds et le nombre de cœurs de calcul. Un lien est inclus pour l’achat de nœuds supplémentaires si la configuration actuelle ne répond pas à tous vos besoins.

## <a name="email-alerts"></a>Alertes par e-mail

Vous pouvez ajouter les adresses de messagerie des personnes que vous souhaitez avertir des modifications apportées à la configuration du cloud privé.

1. Dans la zone **Alertes e-mail supplémentaires**, cliquez sur **Ajouter**.
2. Saisissez l'adresse e-mail.
3. Appuyez sur Retour.  

Pour supprimer une entrée, cliquez sur **X**.

## <a name="cloudsimple-operator-access"></a>Accès des opérateurs de CloudSimple

Le paramètre d’accès de l’opérateur permet à CloudSimple de vous aider à résoudre les problèmes en permettant à un ingénieur du support technique de se connecter à votre portail CloudSimple.  Le paramètre est activé par défaut. Toutes les actions effectuées par l’ingénieur du support technique lors de la connexion à votre compte client sont enregistrées et disponibles pour examen sur la page **Activité** > **Audit**.

Cliquez sur l’option **Activer ou désactiver l’accès des opérateurs de CloudSimple** pour activer ou désactiver l’accès.

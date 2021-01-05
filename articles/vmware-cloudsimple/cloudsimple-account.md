---
title: Gestion de compte CloudSimple – Azure
description: En savoir plus sur la gestion d’un compte CloudSimple, qui est créé en même temps que votre service CloudSimple et qui est associé à votre abonnement Azure.
author: sharaths-cs
ms.author: dikamath
ms.date: 04/10/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 68ddea6c58bfff5e82643acc947ac1e7de76d699
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88141038"
---
# <a name="account-management-overview"></a>Vue d’ensemble de la gestion de compte

Lorsque vous créez votre service CloudSimple, celui-ci crée un compte sur CloudSimple.  Ce compte est associé à votre abonnement Azure où se trouve le service.  Tous les utilisateurs de l’abonnement ayant le rôle **propriétaire** ou **contributeur** ont accès au portail CloudSimple.  L’ID d’abonnement Azure et l’ID de locataire associés au service CloudSimple figurent sur la [page Comptes](account.md).

## <a name="additional-alert-emails"></a>E-mails d’alerte supplémentaires

Vous pouvez configurer des ID d’e-mail dans CloudSimple pour recevoir des alertes :

* liées à votre service ;
* pour traitement automatique.

## <a name="cloudsimple-operator-access"></a>Accès des opérateurs de CloudSimple

Vous pouvez contrôler l’accès du personnel en charge des opérations de service au portail CloudSimple.  Le personnel en charge des opérations de service se connecte au portail lorsque vous soumettez un ticket de support.  Les opérations de service résolvent les problèmes signalés et les mesures prises sont disponibles dans des journaux d’audit.

## <a name="users"></a>Utilisateurs

Tous les utilisateurs ayant le rôle **propriétaire** ou **contributeur** dans l’abonnement ont accès au portail CloudSimple.  Lorsque vous accédez au portail, l’utilisateur est créé sur le compte CloudSimple.  Vous pouvez désactiver l’accès au portail CloudSimple pour des utilisateurs spécifiques à partir de la page Comptes.

## <a name="next-steps"></a>Étapes suivantes

* [Afficher le résumé du compte](account.md)
* [Afficher la liste d’utilisateurs](users.md)

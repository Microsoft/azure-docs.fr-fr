---
title: Azure VMware Solution by CloudSimple – Élever les privilèges CloudSimple
description: Indique comment élever les autorisations CloudSimple pour effectuer des fonctions d’administration dans le vCenter de cloud privé
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 36c6969ed89d0bb9222f52aa81de0d4128b9e533
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77025331"
---
# <a name="escalate-cloudsimple-privileges-to-perform-administrative-functions-in-private-cloud-vcenter"></a>Élevez les privilèges CloudSimple pour effectuer des fonctions d’administration dans le cloud privé vCenter

L’approche des privilèges CloudSimple est conçue pour accorder aux utilisateurs vCenter les privilèges dont ils ont besoin pour effectuer des opérations normales. Dans certains cas, un utilisateur peut avoir besoin de privilèges supplémentaires pour effectuer une tâche particulière.  Vous pouvez élever les privilèges d’un utilisateur vCenter SSO pour une période limitée.

Les privilèges peuvent être élevés pour les raison suivantes :

* Configuration des sources d’identité
* User Management
* Suppression de groupe de ports distribués
* Installation de solutions vCenter (telles que les applications de sauvegarde)
* Création de comptes de service

> [!WARNING]
> Les actions effectuées lorsque les privilèges sont élevés peuvent nuire à votre système ou le rendre indisponible. Effectuez uniquement les actions nécessaires pendant la période d’escalade.

À partir du portail CloudSimple, [élevez les privilèges](escalate-private-cloud-privileges.md) de l’utilisateur local CloudOwner sur le SSO vCenter.  Vous pouvez élever les privilèges de l’utilisateur distant uniquement si un fournisseur d’identité supplémentaire est configuré sur vCenter.  L’escalade des privilèges implique l’ajout de l’utilisateur sélectionné au groupe Administrateurs intégré vSphere.  Un seul utilisateur peut avoir des privilèges élevés.  Si vous avez besoin d’élever les privilèges d’un autre utilisateur, annulez d’abord les privilèges des utilisateurs actuels.

Les utilisateurs de sources d’identité supplémentaires doivent être ajoutés en tant que membres du groupe CloudOwner.

> [!CAUTION]
> Les nouveaux utilisateurs doivent être ajoutés uniquement à *Cloud-Owner-Group*, à *Cloud-Global-Cluster-Admin-Group*, à *Cloud-Global-Storage-Admin-Group*, à *Cloud-Global-Network-Admin-Group* ou à *Cloud-Global-VM-Admin-Group*.  Les utilisateurs ajoutés au groupe *Administrateurs* seront automatiquement supprimés.  Seuls des comptes de service doivent être ajoutés au groupe *Administrateurs*, et les comptes de service en doivent pas être utilisés pour se connecter à l’interface utilisateur web de vSphere.

Pendant la période d’escalade, CloudSimple utilise la surveillance automatisée avec les notifications d’alerte associées pour détecter toute modification accidentelle de l’environnement.

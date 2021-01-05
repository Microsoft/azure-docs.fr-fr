---
title: Fichier Include
description: Fichier Include
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 12/20/2019
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: aec1faa4de1149f08fb6fbc1cc5bf3aa2ab6becd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75371772"
---
Une signature d’accès partagé (SAP) vous permet d’accorder un accès limité aux conteneurs et aux objets blob de votre compte de stockage. Lorsque vous créez une SAP, vous spécifiez ses contraintes, notamment les ressources de Stockage Azure auxquelles un client est autorisé à accéder, les autorisations dont ils disposent sur ces ressources et la durée de validité de la SAP.

Chaque SAP est signée avec une clé. Vous pouvez signer une SAP de l’une des deux méthodes suivantes :

- Avec une clé créée à l’aide des informations d’identification d’Azure Active Directory (Azure AD). Une SAP signée avec des informations d’identification d’Azure AD est une SAP de *délégation d’utilisateur*.
- Avec la clé du compte de stockage. Une *SAP de service* et une *SAP de compte* sont signées avec la clé du compte de stockage.

Une SAP de délégation d’utilisateur offre une meilleure sécurité à une SAP signée avec la clé de compte de stockage. Microsoft recommande d’utiliser une SAP de délégation d’utilisateur dans la mesure du possible. Pour plus d’informations, consultez [Accorder un accès limité aux données avec des signatures d’accès partagé (SAP)](../articles/storage/common/storage-sas-overview.md).

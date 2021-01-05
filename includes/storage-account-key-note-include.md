---
title: Fichier include
description: Fichier include
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/06/2020
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: ba715d510dc296ffa8f9c0ee58841f284416a118
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86027327"
---
## <a name="protect-your-access-keys"></a>Protégez vos clés d’accès

Vos clés d’accès de compte de stockage sont similaires au mot de passe racine pour votre compte de stockage. Veillez toujours à protéger vos clés d’accès. Utilisez Azure Key Vault pour gérer et effectuer la rotation de vos clés en toute sécurité. Évitez de distribuer des clés d’accès à d’autres utilisateurs, de les coder en dur ou de les enregistrer en texte brut dans un emplacement accessible à d’autres personnes. Effectuez une rotation de vos clés si vous pensez qu’elles ont pu être compromises.

> [!NOTE]
> Microsoft recommande d’utiliser dans la mesure du possible Azure Active Directory (Azure AD) plutôt que des clés partagées pour autoriser les demandes effectuées sur les données Blob et File d’attente. Azure AD offre une sécurité et une facilité d’utilisation supérieures sur les clés partagées. Pour plus d’informations sur l’autorisation de l’accès aux données avec Azure AD, consultez [Autoriser l’accès aux objets blob et files d’attente Azure à l’aide d’Azure Active Directory](../articles/storage/common/storage-auth-aad.md).

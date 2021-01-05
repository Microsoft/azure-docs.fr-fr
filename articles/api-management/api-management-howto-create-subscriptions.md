---
title: Créer des abonnements dans la Gestion des API Azure | Microsoft Docs
description: Apprenez à créer des abonnements dans la Gestion des API Azure. Un abonnement est nécessaire pour obtenir les clés d’abonnement permettant d’accéder aux API.
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/14/2018
ms.author: apimpm
ms.openlocfilehash: 191323a4c150c00c93245be35c9c8af381e26b42
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87904855"
---
# <a name="create-subscriptions-in-azure-api-management"></a>Créer des abonnements dans Gestion des API Azure

Lors de la publication d’API via la Gestion des API Azure, il est facile et courant de sécuriser l’accès à ces API au moyen de clés d’abonnement. Les applications clientes qui utilisent les API publiées doivent inclure une clé d’abonnement valide dans les requêtes HTTP lorsqu’elles appellent ces API. Pour obtenir une clé d’abonnement permettant d’accéder aux API, un abonnement est nécessaire. Pour plus d’informations sur les abonnements, voir [Abonnements dans la Gestion des API Azure](api-management-subscriptions.md).

Cet article décrit la procédure de création d’abonnements sur le Portail Azure.

## <a name="prerequisites"></a>Conditions préalables requises

Pour effectuer les étapes décrites dans cet article, les prérequis sont les suivants :

+ [Créer une instance de Gestion des API](get-started-create-service-instance.md).
+ Comprendre le principe des [Abonnements dans la Gestion des API](api-management-subscriptions.md).

## <a name="create-a-new-subscription"></a>Créer un abonnement

1. Sélectionnez **Abonnements** dans le menu de gauche.
2. Sélectionnez **Ajouter un abonnement**.
3. Donnez un nom à l’abonnement et sélectionnez l’étendue.
4. Si vous le souhaitez, choisissez si l’abonnement doit être associé à un utilisateur.
5. Sélectionnez **Enregistrer**.

![Abonnements flexibles](./media/api-management-subscriptions/flexible-subscription.png)

Deux clés d’API sont fournies pour accéder aux API après la création de l’abonnement. L’une est primaire, l’autre secondaire. 

## <a name="next-steps"></a>Étapes suivantes
Sachez-en plus sur la Gestion des API :

+ Découvrez les autres [concepts](api-management-terminology.md) de la Gestion des API.
+ Suivez nos [tutoriels](import-and-publish.md) sur la Gestion des API.
+ Consultez notre [page de FAQ](api-management-faq.md) pour prendre connaissance des questions courantes.
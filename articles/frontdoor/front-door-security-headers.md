---
title: 'Tutoriel : Ajouter des en-têtes de sécurité avec le moteur de règles - Azure Front Door'
description: Ce tutoriel explique comment configurer un en-tête de sécurité via le moteur de règles sur Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/14/2020
ms.author: duau
ms.openlocfilehash: 1727193faa028a1d681f2a74df950afeb9570ec9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91270047"
---
# <a name="tutorial-add-security-headers-with-rules-engine"></a>Tutoriel : Ajouter des en-têtes de sécurité avec le moteur de règles

Ce tutoriel montre comment implémenter des en-têtes de sécurité pour empêcher les vulnérabilités des navigateurs, comme HTTP Strict-Transport-Security (HSTS), X-XSS-Protection, Content-Security-Policy ou X-Frame-Options. Les attributs basés sur la sécurité peuvent également être définis avec des cookies.

L’exemple suivant vous montre comment ajouter un en-tête Content-Security-Policy à toutes les demandes entrantes qui correspondent au chemin défini dans la route à laquelle votre configuration du moteur de règles est associée. Ici, nous autorisons uniquement les scripts de notre site de confiance, **https://apiphany.portal.azure-api.net** , à s’exécuter sur notre application.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Configurer un en-tête Content-Security-Policy dans le moteur de règles.

## <a name="prerequisites"></a>Prérequis

* Avant de pouvoir effectuer les étapes de ce tutoriel, vous devez d’abord créer un Front Door. Pour plus d’informations, consultez [Démarrage rapide : Créer une porte d’entrée](quickstart-create-front-door.md)
* Si c’est la première fois que vous utilisez la fonctionnalité Moteur de règles, découvrez comment [configurer un moteur de règles](front-door-tutorial-rules-engine.md).

## <a name="add-a-content-security-policy-header-in-azure-portal"></a>Ajouter un en-tête Content-Security-Policy dans le portail Azure

1. Cliquez sur **Ajouter** pour ajouter une nouvelle règle. Spécifiez un nom pour la règle, puis cliquez sur **Ajouter une action** > **En-tête de réponse**.

1. Pour l’opérateur, choisissez **Ajouter** pour ajouter cet en-tête en tant que réponse à toutes les demandes entrantes adressées à cette route.

1. Ajoutez le nom de l’en-tête : **Content-Security-Policy** et définissez les valeurs que cet en-tête doit accepter. Dans ce scénario, nous choisissons *« script-src 'self' https://apiphany.portal.azure-api.net  »* .

1. Une fois que vous avez ajouté toutes les règles souhaitées à votre configuration, n’oubliez pas d’accéder à votre route préférée pour associer la configuration de votre moteur de règles à votre règle de routage. Cette étape est nécessaire pour permettre à la règle de fonctionner. 

![exemple de portail](./media/front-door-rules-engine/rules-engine-security-header-example.png)

> [!NOTE]
> Dans ce scénario, nous n’avons pas ajouté de [conditions de correspondance](front-door-rules-engine-match-conditions.md) à la règle. Cette règle est appliquée à toutes les demandes entrantes qui correspondent au chemin défini dans la règle de routage. Si vous voulez qu’elle s’applique seulement à un sous-ensemble de ces demandes, veillez à ajouter vos **conditions de correspondance** spécifiques à cette règle.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans les étapes précédentes, vous avez configuré des en-têtes de sécurité avec le moteur de règles. Si vous ne voulez plus de la règle, vous pouvez la supprimer en cliquant sur Supprimer la règle.

:::image type="content" source="./media/front-door-rules-engine/rules-engine-delete-rule.png" alt-text="Supprimer la règle":::

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment configurer un pare-feu d’applications web pour votre instance Front Door, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Pare-feu d’applications web et Front Door](front-door-waf.md)

---
title: Fichier Include
description: Fichier Include
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 10/15/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 8b3c1992a1cff18390f9d1332103e0650af418e2
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347718"
---
## <a name="prepare-your-web-app"></a>Préparation de votre application web

Si vous souhaitez créer des liaisons TLS/SSL personnalisées ou activer des certificats clients pour votre application App Service, votre [plan App Service](https://azure.microsoft.com/pricing/details/app-service/) doit être au niveau **De base**, **Standard**, **Premium** ou **Isolé**. Au cours de cette étape, vous allez vous assurer que votre application web se trouve dans le niveau de tarification pris en charge.

### <a name="sign-in-to-azure"></a>Connexion à Azure

Ouvrez le [portail Azure](https://portal.azure.com).

### <a name="navigate-to-your-web-app"></a>Accès à votre application web

Recherchez et sélectionnez **App Services**.

![Sélectionner App Services](./media/app-service-ssl-prepare-app/app-services.png)

Dans la page **App Services**, sélectionnez le nom de votre application web.

![Capture d’écran de la page App Services dans le portail Azure montrant une liste de toutes les applications web en cours d’exécution, avec la première application de la liste mise en évidence.](./media/app-service-ssl-prepare-app/select-app.png)

Vous accédez à la page de gestion de votre application web.  

### <a name="check-the-pricing-tier"></a>Vérification du niveau tarifaire

Dans la navigation de gauche de la page de votre application web, accédez à la section **Paramètres** et sélectionnez **Monter en puissance (plan App Service)** .

![Menu Monter en puissance](./media/app-service-ssl-prepare-app/scale-up-menu.png)

Vérifiez que votre application web ne se trouve pas dans le niveau **F1** ou **D1**. Le niveau actuel de votre application web est encadré d’un rectangle bleu foncé.

![Vérification du niveau de tarification](./media/app-service-ssl-prepare-app/check-pricing-tier.png)

Le certificat SSL personnalisé n’est pas pris en charge pour les niveaux **F1** et **D1**. Si vous avez besoin de monter en puissance, consultez la section ci-après. Sinon, fermez la page **Monter en puissance** et ignorez la section [Évolution de votre plan App Service](#scale-up-your-app-service-plan).

### <a name="scale-up-your-app-service-plan"></a>Évolution de votre plan App Service

Sélectionnez un niveau payant (**B1**, **B2**, **B3**, ou n’importe quel niveau dans la catégorie **Production**). Pour obtenir des options supplémentaires, cliquez sur **Afficher d’autres options**.

Cliquez sur **Appliquer**.

![Sélection du niveau tarifaire](./media/app-service-ssl-prepare-app/choose-pricing-tier.png)

Lorsque la notification suivante s’affiche, cela signifie que l’opération est terminée.

![Notification de montée en puissance](./media/app-service-ssl-prepare-app/scale-notification.png)


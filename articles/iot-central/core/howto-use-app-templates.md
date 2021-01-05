---
title: Exporter une application Azure IoT Central | Microsoft Docs
description: En tant que gestionnaire de solutions, je souhaite exporter un modèle d’application pour pouvoir le réutiliser.
author: dominicbetts
ms.author: dobett
ms.date: 12/09/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: philmea
ms.openlocfilehash: aec72644f708d6363a80da28c5e571d0165fcdfa
ms.sourcegitcommit: b4f303f59bb04e3bae0739761a0eb7e974745bb7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91651834"
---
# <a name="export-your-application"></a>Exporter votre application

Cet article explique comment, en tant que gestionnaire de solutions, exporter une application IoT Central pour pouvoir la réutiliser.

Deux options s'offrent à vous :

- Vous pouvez créer une copie de votre application s’il vous faut simplement un duplicata de votre application.
- Vous pouvez créer un modèle d’application à partir de votre application si vous envisagez de créer plusieurs copies.

## <a name="copy-your-application"></a>Copier une application

Vous pouvez créer une copie de n’importe quelle application, à l’exception des instances d’appareils, de l’historique des données d’un appareil et des données utilisateur. La copie utilise un plan tarifaire Standard qui vous sera facturé. Vous ne pouvez pas créer une application qui utilise le plan Gratuit en copiant une application.

Sélectionnez **Copier**. Dans la boîte de dialogue, entrez les détails de la nouvelle application. Ensuite, sélectionnez **Copier** pour confirmer que vous souhaitez continuer. Pour plus d’informations sur les champs du formulaire, consultez le guide de démarrage rapide [Créer une application](quick-deploy-iot-central.md).

![Capture d’écran montrant la page de paramètres « Copier l’application ».](media/howto-use-app-templates/appcopy2.png)

Une fois l’application correctement copiée, vous pouvez accéder à la nouvelle application à l’aide du lien.

![Page Paramètres de l’application](media/howto-use-app-templates/appcopy3a.png)

La copie d’une application copie également la définition des règles et de l’action de messagerie. Certaines actions, comme Flow et Logic Apps, sont liées à des règles spécifiques par le biais de l’ID de règle. Quand une règle est copiée dans une autre application, elle obtient son propre ID de règle. Dans ce cas, les utilisateurs doivent créer une action, puis lui associer la nouvelle règle. En général, il est judicieux de vérifier que les règles et les actions sont à jour dans la nouvelle application.

> [!WARNING]
> Si un tableau de bord inclut des vignettes qui affichent des informations d’appareils spécifiques, ces vignettes affichent alors **La ressource demandée est introuvable** dans la nouvelle application. Vous devez reconfigurer ces vignettes pour qu’elles affichent des informations sur les appareils de votre nouvelle application.

## <a name="create-an-application-template"></a>Créer un modèle d’application

Lorsque vous créez une application Azure IoT Central, vous avez un choix de plusieurs exemples de modèles intégrés. Vous pouvez également créer vos propres modèles d’application à partir des applications IoT Central existantes. Vous pouvez ensuite utiliser vos propres modèles d’application lorsque vous créez des applications.

Lorsque vous créez un modèle d’application, il comprend les éléments suivants de votre application existante :

- Le tableau de bord par défaut de l’application, dont la disposition et toutes les vignettes que vous avez définies.
- Les modèles d’appareil, dont les mesures, les paramètres, les propriétés, les commandes et le tableau de bord.
- Les règles. Toutes les définitions de règle sont incluses. Toutefois, les actions, à l’exception des actions de messagerie, ne sont pas incluses.
- Les ensembles d’appareils, dont leurs conditions et leurs tableaux de bord.

> [!WARNING]
> Si un tableau de bord inclut des vignettes qui affichent des informations d’appareils spécifiques, ces vignettes affichent alors **La ressource demandée est introuvable** dans la nouvelle application. Vous devez reconfigurer ces vignettes pour qu’elles affichent des informations sur les appareils de votre nouvelle application.

Lorsque vous créez un modèle d’application, il n’inclut pas les éléments suivants :

- Appareils
- Utilisateurs
- Définitions des exportations de données continues

Ajoutez ces éléments manuellement à toutes les applications créées depuis un modèle d’application.

Pour créer un modèle d’application à partir d’une application IoT Central existante :

1. Accédez à la section **Administration** de votre application.
1. Sélectionnez **Exportation de modèle d’application**.
1. Sur la page **Exportation de modèle d’application**, entrez un nom et une description pour votre modèle.
1. Sélectionnez le bouton **Exporter** pour créer le modèle d’application. Vous pouvez désormais copier le **Lien de partage** pour permettre à quelqu’un de créer une application depuis le modèle :

![Créer un modèle d’application](media/howto-use-app-templates/create-template.png)

### <a name="use-an-application-template"></a>Utiliser un modèle d’application

Pour utiliser un modèle d’application afin de créer une application IoT Central, vous devez d’abord avoir créé un **Lien de partage**. Collez le **Lien de partage** dans la barre d’adresse de votre navigateur. La page **Créer une application** s’affiche avec les modèles d’application personnalisés sélectionnés :

![Créer une application à partir d’un modèle](media/howto-use-app-templates/create-app.png)

Sélectionnez votre plan tarifaire et remplissez les autres champs du formulaire. Sélectionnez ensuite **Créer** pour créer une application IoT Central à partir d’un modèle d’application.

### <a name="manage-application-templates"></a>Gérer des modèles d’application

Sur la page **Exportation de modèle d’application**, vous pouvez supprimer ou mettre à jour le modèle d’application.

Si vous supprimez un modèle d’application, vous ne pouvez plus utiliser le lien de partage généré précédemment pour créer des applications.

Pour mettre à jour votre modèle d’application, modifiez le nom du modèle ou sa description sur la page **Exportation de modèle d’application**. Sélectionnez ensuite le bouton **Exporter**. Cette action génère un nouveau **Lien de partage** et rend non valide les **Liens de partage** précédents.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à utiliser des modèles d’application, l’étape suivante conseillée est d’apprendre à [surveiller l’intégrité globale des appareils connectés à une application IoT Central](howto-monitor-application-health.md).

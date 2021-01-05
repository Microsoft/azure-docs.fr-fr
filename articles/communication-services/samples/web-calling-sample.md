---
title: Azure Communication Services - Exemple d’appel web
titleSuffix: An Azure Communication Services sample
description: Découvrez l’exemple d’appel web Communication Services.
author: chriswhilar
manager: mariusu-msft
services: azure-communication-services
ms.author: mariusu
ms.date: 10/15/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 8faa6533ba4dc7fad61c045278c5504c5f78d900
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888961"
---
# <a name="get-started-with-the-web-calling-sample"></a>Bien démarrer avec l’exemple d’appel web

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

> [!IMPORTANT]
> [Cet exemple est disponible sur Github](https://github.com/Azure-Samples/communication-services-web-calling-tutorial/).

L’**exemple d’appel web** Azure Communication Services illustre comment la bibliothèque de client d’appel Communication Services peut être utilisée pour créer une expérience d’appel avec JavaScript.

Dans ce guide de démarrage rapide, vous allez découvrir comment fonctionne l’exemple avant de l’exécuter sur votre ordinateur local. Ensuite, vous le déploierez sur Azure à l’aide de vos propres ressources Azure Communication Services.

## <a name="overview"></a>Vue d'ensemble

L’exemple d’appel web est une application web qui sert de procédure pas à pas pour les différentes fonctionnalités fournies par la bibliothèque de client d’appel web de Communication Services. 

Cet exemple a été conçu pour les développeurs, et facilite la prise en main de Communication Services. Son interface utilisateur est divisée en plusieurs sections, chacune comportant un bouton « Show code » qui vous permet de copier du code directement à partir de votre navigateur afin de le coller dans votre propre application Communication Services.

Lorsque l’[exemple d’appel web](https://github.com/Azure-Samples/communication-services-web-calling-tutorial) est exécuté sur votre ordinateur, la page d’arrivée suivante s’affiche :

:::image type="content" source="./media/web-calling-tutorial-page-1.png" alt-text="Tutoriel sur l’appel web 1" lightbox="./media/web-calling-tutorial-page-1.png":::

:::image type="content" source="./media/web-calling-tutorial-page-2.png" alt-text="Tutoriel sur l’appel web 2" lightbox="./media/web-calling-tutorial-page-2.png":::


## <a name="user-provisioning-and-sdk-initialization"></a>Provisionnement d’utilisateur et initialisation du kit SDK 

Pour commencer à utiliser la démonstration, entrez la chaîne de connexion de votre [ressource Communication Services](../quickstarts/create-communication-resource.md) dans `config.json`. Elle servira à provisionner un [jeton d’accès utilisateur](../concepts/authentication.md) afin que votre kit SDK appelant puisse être initialisé.

Entrez votre propre identificateur personnel dans User Identity. Si rien n’est spécifié ici, une identité utilisateur aléatoire est générée. 

Cliquez sur « Provisioning user and initialize SDK » pour initialiser votre kit SDK à l’aide d’un jeton provisionné par le service de provisionnement de jetons back-end. Ce service back-end se trouve dans `/project/webpack.config.js`.

Cliquez sur le bouton « Show code » pour voir l’exemple de code que vous pouvez utiliser dans votre propre solution.

Une fois votre kit SDK initialisé, vous devriez voir ce qui suit :

:::image type="content" source="./media/user-provisioning.png" alt-text="Approvisionnement d’utilisateurs" lightbox="./media/user-provisioning.png":::

Vous pouvez maintenant commencer à passer des appels à l’aide de votre ressource Communication Services !

## <a name="placing-and-receiving-calls"></a>Passage et réception d’appels

Le kit SDK d’appel web Communication Services autorise les appels **1:1**, **1:N** et de **groupe**.

Pour les appels sortants 1:1 ou 1:N, vous pouvez spécifier plusieurs identités utilisateur Communication Services à appeler à l’aide de valeurs séparées par des virgules. Vous pouvez également spécifier des numéros de téléphone traditionnels (RTC) à appeler à l’aide de valeurs séparées par des virgules. 

Lors de l’appel de numéros de téléphone RTC, spécifiez votre ID d’appelant de substitution. Cliquez sur le bouton « Place call » pour passer un appel sortant :

:::image type="content" source="./media/place-a-call.png" alt-text="Passer un appel" lightbox="./media/place-a-call.png":::

Pour joindre un appel de groupe, entrez le GUID qui identifie l’appel et cliquez sur le bouton « Join group » :

:::image type="content" source="./media/join-a-group-call.png" alt-text="Rejoindre un appel de groupe" lightbox="./media/join-a-group-call.png":::

Cliquez sur le bouton « Show code » pour afficher l’exemple de code permettant de placer des appels, de recevoir des appels et de joindre des appels de groupe.

Un appel actif se présente comme suit :

:::image type="content" source="./media/group-call.png" alt-text="Appel de groupe" lightbox="./media/group-call.png":::

Cet exemple fournit également des extraits de code pour les fonctionnalités suivantes :

  - Clic sur l’icône de vidéo pour activer/désactiver votre caméra vidéo
  - Clic sur l’icône de microphone pour activer/désactiver votre microphone
  - Clic sur l’icône de lecture pour activer/désactiver la mise en attente de l’appel
  - Clic sur l’icône d’écran pour démarrer/arrêter le partage de votre écran
  - Clic sur l’icône représentant une personne pour ajouter un participant à l’appel
  - Clic sur « Supprimer un participant » dans la liste des participants pour supprimer un participant spécifique de l’appel


## <a name="next-steps"></a>Étapes suivantes

>[!div class="nextstepaction"] 
>[Télécharger l’exemple à partir de GitHub](https://github.com/Azure-Samples/communication-services-web-calling-tutorial/)

Pour plus d’informations, consultez les articles suivants :

- Familiarisez-vous avec la [bibliothèque de client d’appel](../quickstarts/voice-video-calling/calling-client-samples.md)
- Apprenez-en davantage sur le [fonctionnement des appels](../concepts/voice-video-calling/about-call-types.md)
- Lire la [documentation de référence sur les API](/javascript/api/azure-communication-services/@azure/communication-calling/?view=azure-communication-services-js)

## <a name="additional-reading"></a>Documentation supplémentaire

- [Azure Communication GitHub](https://github.com/Azure/communication) : Autres exemples et informations dans la page GitHub officielle
- [Redux](https://redux.js.org/) - Gestion de l’état côté client
- [FluentUI](https://aka.ms/fluent-ui) - Bibliothèque d’interface utilisateur, Technologie Microsoft
- [React](https://reactjs.org/) - Bibliothèque pour la création d’interfaces utilisateur
- [ASP.NET Core](/aspnet/core/introduction-to-aspnet-core?preserve-view=true&view=aspnetcore-3.1) - Framework pour la création d’applications web
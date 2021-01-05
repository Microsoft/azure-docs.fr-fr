---
title: Fichier include
description: Fichier include
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 10/06/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 472c1770e2793d8da4e8fc76fafbf3b9073b746d
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763416"
---
## <a name="deploy-and-configure-azure-media-services"></a>Déployer et configurer Azure Media Services

La solution utilise un compte Azure Media Services pour stocker les clips vidéo de détection d’objets réalisés par l’appareil de passerelle IoT Edge.

Lorsque vous créez le compte Media Services :

- Vous devez fournir un nom de compte, un abonnement Azure, un emplacement, un groupe de ressources et un compte de stockage. Créez un compte de stockage à l’aide des paramètres par défaut lors de la création du compte Media Services.

- Sélectionnez la région **USA Est** comme emplacement.

- Créez un groupe de ressources appelé *lva-rg* dans la région **USA Est** pour les comptes de stockage et Media Services. Une fois les tutoriels terminés, supprimez facilement toutes les ressources en supprimant le groupe de ressources *lva-rg* .

Dans le portail Azure, créez le [compte Media Services](https://portal.azure.com/?r=1#create/Microsoft.MediaService).

> [!TIP]
> Tous les exemples de ces tutoriels utilisent la région **USA Est**. Vous pouvez utiliser votre région la plus proche si vous préférez.

Notez le nom de votre compte **Media Services** dans le fichier *scratchpad.txt* .

Une fois le déploiement terminé, ouvrez une fenêtre Cloud Shell et exécutez la commande suivante pour récupérer l’**ID de ressource** associé à votre compte Media Services :

```azurecli
az resource list --resource-group lva-rg --resource-type microsoft.media/mediaservices --output table --query "[].{ResourceID:id}"
```

:::image type="content" source="media/iot-central-video-analytics-part2/get-resource-id.png" alt-text="Utiliser Cloud Shell pour récupérer l’ID de ressource":::

Notez l’**ID de ressource** dans le fichier *scratchpad.txt*. Vous utiliserez plus tard cette valeur lorsque vous configurerez le module IoT Edge.

Ensuite, configurez un principal de service Azure Active Directory pour votre ressource Media Services. Sélectionnez **Accès API**, puis **Authentification du principal de service**. Créez une nouvelle application Azure Active Directory portant le même nom que votre ressource Media Services, puis créez un secret avec pour description *Accès IoT Edge*.

:::image type="content" source="./media/iot-central-video-analytics-part2/media-service-authentication.png" alt-text="Configurer une application Azure AD pour Azure Media Services":::

Une fois le secret généré, faites défiler jusqu’à la section **Copier les informations d’identification pour connecter votre application de principal du service**. Sélectionnez ensuite **JSON**. Vous pouvez copier simultanément toutes les informations d’identification depuis cet emplacement. Notez ces informations dans le fichier *scratchpad.txt*. Vous les utiliserez plus tard lorsque vous configurerez l’appareil IoT Edge.

> [!WARNING]
> Il s’agit de la seule possibilité d’afficher et d’enregistrer le secret. Si vous le perdez, vous devez en générer un autre.

## <a name="create-the-azure-iot-central-application"></a>Créer l’application Azure IoT Central

Dans cette section, vous créerez une application Azure IoT Central à partir d’un modèle. Vous allez utiliser cette application tout au long de la série de tutoriels pour générer une solution complète.

Pour créer une nouvelle application Azure IoT Central :

1. Accédez au site web du [gestionnaire d’applications d’Azure IoT Central](https://aka.ms/iotcentral).

1. Connectez-vous avec les informations d’identification que vous utilisez pour accéder à votre abonnement Azure.

1. Pour commencer à créer une application Azure IoT Central, sélectionnez **Nouvelle application** sur la page **Générer**.

1. Sélectionnez **Distribution**. La page de vente au détail affiche plusieurs modèles d’application de vente au détail.

Pour créer une application d’analytique vidéo :

1. Sélectionnez le modèle d’application **Analytique vidéo - Détection d’objets et de mouvements**. Ce modèle comprend des modèles d’appareil pour les appareils utilisés dans le tutoriel. Ce modèle comprend des exemples de tableaux de bord que les opérateurs peuvent utiliser pour effectuer des tâches telles que la surveillance et la gestion des caméras.

1. Si vous le souhaitez, choisissez un **nom d’application** convivial. Cette application est basée sur un magasin fictif de vente au détail nommé Northwind Traders. Le didacticiel utilise le **nom d’application** *d’analytique vidéo Northwind Traders*.

    > [!NOTE]
    > Si vous utilisez un **nom d’application** convivial, vous devez toujours utiliser une valeur unique pour l’**URL** de l’application.

1. Si vous disposez d’un abonnement Azure, sélectionnez votre **répertoire**, votre **abonnement Azure** et **États-Unis** comme **emplacement**. Si vous n’avez pas d’abonnement, vous pouvez activer une **version d’évaluation gratuite de 7 jours** et remplir les coordonnées requises. Ce tutoriel utilise trois appareils : deux caméras et un appareil IoT Edge et dès lors, à défaut d’utiliser la version d’évaluation gratuite, vous serez facturé pour l’utilisation.

    Pour plus d’informations sur les répertoires et abonnements et emplacements, consultez le guide de démarrage rapide [Créer une application](../articles/iot-central/core/quick-deploy-iot-central.md).

1. Sélectionnez **Create** (Créer).

    :::image type="content" source="./media/iot-central-video-analytics-part2/new-application.png" alt-text="Page de création d’une application Azure IoT Central":::

### <a name="retrieve-the-configuration-data"></a>Récupérer les données de configuration

Plus loin dans ce tutoriel, lorsque vous configurerez la passerelle IoT Edge, vous aurez besoin de certaines données de configuration de l’application IoT Central. L’appareil IoT Edge utilise ces informations pour s’inscrire et se connecter à l’application.

Dans la section **Administration**, sélectionnez **votre application** et notez l’**URL d’application** et l’**ID d’application** dans le fichier *scratchpad.txt* :

:::image type="content" source="./media/iot-central-video-analytics-part2/administration.png" alt-text="Capture d’écran illustrant le volet d’administration d’une page d’analytique vidéo avec l’URL et l’ID de l’application mises en surbrillance.":::

Sélectionnez **Jetons d’API** et générez un nouveau jeton appelé **LVAEdgeToken** pour le rôle **Opérateur** :

:::image type="content" source="./media/iot-central-video-analytics-part2/token.png" alt-text="Générer un jeton":::

Notez le jeton dans le fichier *scratchpad.txt* pour plus tard. Une fois la boîte de dialogue fermée, vous ne pouvez plus afficher le jeton.

Dans la section **Administration**, sélectionnez **Connexion de l’appareil**, puis **SAS-IoT-Devices**.

Notez la **clé primaire** des appareils dans le fichier *scratchpad.txt*. Vous utiliserez ce *jeton de signature d’accès partagé au groupe principal* lorsque vous configurerez l’appareil IoT Edge.

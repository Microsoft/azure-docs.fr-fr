---
title: Définir un nouveau type d’appareil IoT dans Azure IoT Central | Microsoft Docs
description: Cet article vous explique, en tant que développeur de solutions, comment créer un modèle d’appareil Azure IoT dans votre application Azure IoT Central. Vous définissez la télémétrie, l’état, les propriétés et les commandes pour votre type d’appareil.
author: dominicbetts
ms.author: dobett
ms.date: 12/06/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom:
- contperf-fy21q1
- device-developer
ms.openlocfilehash: 9e5e96d97494f4ba9aa28e84b046cd057fe8eba7
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033406"
---
# <a name="define-a-new-iot-device-type-in-your-azure-iot-central-application"></a>Définir un nouveau type d’appareil IoT dans votre application Azure IoT Central

*Cet article s’applique aux créateurs de solutions et aux développeurs d’appareils.*

Un modèle d’appareil est un blueprint qui définit les caractéristiques et les comportements d’un type d’appareil se connectant à une [application Azure IoT Central](concepts-app-templates.md).

Par exemple, un concepteur peut créer un modèle d’appareil pour un ventilateur connecté qui présente les caractéristiques suivantes :

- Envoie des données de télémétrie de température
- Envoie la propriété de l’emplacement
- Envoie les événements d’erreur du moteur du ventilateur
- Envoie l’état de fonctionnement du ventilateur
- Fournit une propriété de vitesse du ventilateur inscriptible
- Fournit une commande pour redémarrer l’appareil
- Vous donne une vision globale de l’appareil par le biais d’un tableau de bord

À partir de ce modèle d’appareil, un opérateur peut créer et connecter des appareils qui sont des ventilateurs réels. Tous ces ventilateurs ont des mesures, des propriétés et des commandes que les opérateurs utilisent pour les superviser et les gérer. Les opérateurs utilisent les [tableaux de bord et les formulaires des appareils](#add-dashboards) pour interagir avec les ventilateurs. Un développeur d’appareils utilise le modèle pour comprendre comment l’appareil interagit avec l’application. Pour en savoir plus, consultez [Charges utiles de télémétrie, de propriétés et de commandes](concepts-telemetry-properties-commands.md).

> [!NOTE]
> Seuls les concepteurs et les administrateurs peuvent créer, modifier et supprimer des modèles d’appareils. Tous les utilisateurs peuvent créer des appareils sur la page **Appareils** à partir de modèles d’appareils existants.

Dans une application IoT Central, un modèle d’appareil utilise un modèle d’appareil pour décrire les fonctions d’un appareil. Plusieurs choix s’offrent à vous pour créer des modèles d’appareil :

- Concevez le modèle d’appareil dans IoT Central, puis [implémentez son modèle de capacité d’appareil dans le code de votre appareil](concepts-telemetry-properties-commands.md).
- Importez un modèle d’appareil à partir du [catalogue d’appareils certifiés Azure pour l’IoT](https://aka.ms/iotdevcat). Personnalisez le modèle d’appareil selon vos besoins dans IoT Central.
- Créez un modèle d’appareil à l’aide de la [version 2 du langage DTDL (Digital Twin Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). Visual Studio Code dispose d’une extension prenant en charge la création de modèles DTDL. Pour plus d’informations, consultez [Installer et utiliser les outils de création DTDL](../../iot-pnp/howto-use-dtdl-authoring-tools.md). Publiez ensuite le modèle dans le référentiel de modèles public. Pour plus d’informations, consultez [Référentiel de modèles d’appareil](../../iot-pnp/concepts-model-repository.md). Implémentez votre code d’appareil à partir du modèle et connectez votre appareil réel à votre application IoT Central. IoT Central recherche et importe pour vous le modèle d’appareil à partir du référentiel public et génère un modèle d’appareil. Vous pouvez ensuite ajouter des propriétés de cloud, des personnalisations et des tableaux de bord dont votre application IoT Central a besoin pour le modèle d’appareil.
- Créez un modèle d’appareil à l’aide du langage DTDL. Implémentez votre code d’appareil à partir du modèle. Importez manuellement le modèle de capacité d’appareil dans votre application IoT Central, puis ajoutez les propriétés cloud, les personnalisations et les tableaux de bord nécessaires à votre application IoT Central.

Vous pouvez également ajouter des modèles d’appareil à une application IoT Central à l’aide de l’[API REST](/learn/modules/manage-iot-central-apps-with-rest-api/) ou de l’interface [CLI](howto-manage-iot-central-from-cli.md).

Certains [modèles d’application](concepts-app-templates.md) incluent déjà des modèles d’appareil qui sont utiles dans le scénario pris en charge par le modèle d’application. Par exemple, consultez [Architecture de l’analytique en magasin](../retail/store-analytics-architecture.md).

## <a name="create-a-device-template-from-the-device-catalog"></a>Créer un modèle d’appareil à partir du catalogue d’appareils

Vous pouvez rapidement commencer à créer votre solution à l’aide d’un appareil certifié. Consultez la liste dans le [catalogue d’appareils Azure IOT](https://catalog.azureiotsolutions.com/alldevices). IoT Central s’intègre au catalogue d’appareils ; ainsi, vous pouvez importer un modèle d’appareil parmi les appareils certifiés. Pour créer un modèle d’appareil à partir de l’un de ces appareils dans IoT Central :

1. Accédez à la page **Modèles d’appareil** dans votre application IoT Central.
1. Sélectionnez **+ Nouveau**, puis sélectionnez les appareils certifiés Play dans le catalogue. IoT Central crée un modèle d’appareil basé sur ce modèle d’appareil.
1. Ajoutez des propriétés de Cloud, des personnalisations ou des vues à votre modèle d’appareil.
1. Sélectionnez **Publier** pour rendre le modèle disponible afin que les opérateurs affichent et connectent des appareils.

## <a name="create-a-device-template-from-scratch"></a>Créer un modèle d’appareil en partant de zéro

Un modèle d’appareil contient :

- Un _modèle d’appareil_ qui spécifie les données de télémétrie, les propriétés et les commandes implémentées par l’appareil. Ces capacités de l'appareil sont organisées en une ou plusieurs composants.
- Des _propriétés de cloud_ qui définissent les informations sur vos appareils stockées par votre application IOT Central. Par exemple, une propriété de cloud peut enregistrer la date de la dernière maintenance d’un appareil. Ces informations ne sont jamais partagées avec l’appareil.
- Les _personnalisations_ permettent au concepteur de remplacer certaines des définitions du modèle d’appareil. Par exemple, le concepteur peut remplacer le nom d’une propriété de l’appareil. Les noms de propriétés s’affichent dans les tableaux de bord et les formulaires IoT Central.
- _Les tableaux de bord et les formulaires_ permettent au concepteur de créer une interface utilisateur qui permet aux opérateurs de surveiller et de gérer les appareils connectés à votre application.

Pour créer un modèle d’appareil dans IoT Central :

1. Accédez à la page **Modèles d’appareil** dans votre application IoT Central.
1. Sélectionnez **+ Nouveau** > **Appareil IoT**. Ensuite, sélectionnez **Next: Personnaliser**.
1. Entrez un nom pour votre modèle, par exemple **Thermostat**. Ensuite, sélectionnez **Next: Examiner**, puis sélectionnez **Créer**.
1. IoT Central crée un modèle d’appareil vide et vous offre la possibilité de créer un modèle personnalisé à partir de zéro ou d’importer un modèle DTDL.

## <a name="manage-a-device-template"></a>Gérer un modèle d’appareil

Vous pouvez renommer ou supprimer un modèle à partir de la page d’hébergement du modèle.

Après avoir ajouté un modèle d’appareil à votre modèle, vous pouvez le publier. Avant d’avoir publié le modèle, vous ne pouvez pas connecter un appareil basé sur ce modèle de telle manière que vos opérateurs le voient sur la page **Appareils**.

## <a name="create-a-capability-model"></a>Créer un modèle de capacité

Pour créer un modèle d’appareil, plusieurs possibilités s’offrent à vous :

- Utilisez IoT Central pour créer un modèle personnalisé en partant de zéro.
- Importez un modèle DTDL à partir d’un fichier JSON. Un concepteur d’appareils peut avoir utilisé Visual Studio Code pour créer un modèle d’appareil pour votre application.
- Sélectionnez l’un des appareils dans le catalogue d’appareils. Cette option importe le modèle d’appareil que le fabricant a publié pour cet appareil. Un modèle d’appareil importé de cette façon est automatiquement publié.

## <a name="manage-a-capability-model"></a>Gérer un modèle de capacité

Après avoir créé un modèle d’appareil, vous pouvez  :

- Ajouter des composants au modèle. Un modèle doit avoir au moins un composant.
- Modifier les métadonnées du modèle, telles que son ID, son espace de noms et son nom.
- Supprimer le modèle.

## <a name="create-a-component"></a>Créer un composant

Un modèle d’appareil doit avoir au moins un composant par défaut. Un composant est une collection réutilisable de fonctionnalités.

Pour créer un composant :

1. Accédez à votre modèle d’appareil, puis sélectionnez **+Ajouter un composant**.

1. Sur la page **Ajouter une interface de composant**, vous pouvez :

    - Créer un composant personnalisé à partir de zéro.
    - Importer un composant existant à partir d’un fichier DTDL. Un concepteur d’appareils peut avoir utilisé Visual Studio Code pour créer un composant pour votre appareil.

1. Après avoir créé un composant, sélectionnez **Modifier l'identité** pour modifier le nom d’affichage du composant.

1. Si vous choisissez de créer un composant personnalisé à partir de zéro, vous pouvez ajouter les fonctionnalités de votre appareil. Les fonctionnalités de l’appareil sont les données de télémétrie, les propriétés et les commandes.

### <a name="telemetry"></a>Télémétrie

Les données de télémétrie sont un flux de valeurs envoyées à partir de l’appareil, généralement par un capteur. Par exemple, un capteur peut indiquer la température ambiante.

Le tableau suivant décrit les paramètres de configuration d’une fonctionnalité de données de télémétrie :

| Champ | Description |
| ----- | ----------- |
| Nom d’affichage | Nom complet de la valeur des données de télémétrie utilisée sur les tableaux de bord et les formulaires. |
| Nom | Nom du champ dans le message de données de télémétrie. IoT Central génère une valeur pour ce champ à partir du nom d’affichage, mais vous pouvez choisir votre propre valeur si nécessaire. Ce champ doit être alphanumérique. |
| Type de fonctionnalité | Données de télémétrie. |
| Type sémantique | Type sémantique des données de télémétrie, telles que la température, l’état ou l’événement. Le choix du type sémantique détermine lequel des champs suivants est disponible. |
| schéma | Type de données de télémétrie, tel que double, chaîne ou vecteur. Les options disponibles sont déterminées par le type sémantique. Le schéma n’est pas disponible pour les types sémantiques d’événement et d’état. |
| severity | Disponible uniquement pour le type sémantique d’événement. Les gravités sont **Erreur**, **Information** ou **Avertissement**. |
| Valeurs d’état | Disponible uniquement pour le type sémantique d’état. Définissez les valeurs d’état possibles, chacune ayant un nom d’affichage, un nom, un type d’énumération et une valeur. |
| Unité | Unité pour la valeur des données de télémétrie, telles que **mph**, **%** ou **&deg;C**. |
| Unités d'affichage | Unité d’affichage à utiliser sur les tableaux de bord et les formulaires. |
| Commentaire | Commentaires sur la fonctionnalité de données de télémétrie. |
| Description | Description de la fonctionnalité de données de télémétrie. |

### <a name="properties"></a>Propriétés

Les propriétés représentent des valeurs à un moment donné. Par exemple, un appareil peut utiliser une propriété pour indiquer la température cible qu’il tente d’atteindre. Vous pouvez définir des propriétés accessibles en écriture à partir d’IoT Central.

Le tableau suivant décrit les paramètres de configuration d’une fonctionnalité de données de propriété :

| Champ | Description |
| ----- | ----------- |
| Nom d’affichage | Nom complet de la valeur de propriété utilisée sur les tableaux de bord et les formulaires. |
| Nom | Nom de la propriété. IoT Central génère une valeur pour ce champ à partir du nom d’affichage, mais vous pouvez choisir votre propre valeur si nécessaire. Ce champ doit être alphanumérique. |
| Type de fonctionnalité | Propriété. |
| Type sémantique | Type sémantique de la propriété, telles que la température, l’état ou l’événement. Le choix du type sémantique détermine lequel des champs suivants est disponible. |
| schéma | Type de données de propriété, tel que double, chaîne ou vecteur. Les options disponibles sont déterminées par le type sémantique. Le schéma n’est pas disponible pour les types sémantiques d’événement et d’état. |
| Accessible en écriture | Si la propriété n’est pas inscriptible, l’appareil peut signaler des valeurs de propriété à IoT Central. Si la propriété est inscriptible, l’appareil peut signaler des valeurs de propriétés à IoT Central, et IoT Central peut envoyer des mises à jour de propriétés à l’appareil.
| severity | Disponible uniquement pour le type sémantique d’événement. Les gravités sont **Erreur**, **Information** ou **Avertissement**. |
| Valeurs d’état | Disponible uniquement pour le type sémantique d’état. Définissez les valeurs d’état possibles, chacune ayant un nom d’affichage, un nom, un type d’énumération et une valeur. |
| Unité | Unité pour la valeur des propriétés, telles que **mph**, **%** ou **&deg;C**. |
| Unités d'affichage | Unité d’affichage à utiliser sur les tableaux de bord et les formulaires. |
| Commentaire | Commentaires sur la fonctionnalité de propriété. |
| Description | Description de la fonctionnalité de propriété. |

### <a name="commands"></a>Commandes

Vous pouvez appeler des commandes d’appareil à partir d’IoT Central. Les commandes transmettent éventuellement des paramètres à l’appareil et reçoivent une réponse de ce dernier. Par exemple, vous pouvez appeler une commande pour redémarrer un appareil en 10 secondes.

Le tableau suivant décrit les paramètres de configuration d’une fonctionnalité de commande :

| Champ | Description |
| ----- | ----------- |
| Nom d’affichage | Nom d’affichage de la commande utilisée sur les tableaux de bord et les formulaires. |
| Nom | Nom de la commande. IoT Central génère une valeur pour ce champ à partir du nom d’affichage, mais vous pouvez choisir votre propre valeur si nécessaire. Ce champ doit être alphanumérique. |
| Type de fonctionnalité | Commande. |
| Commentaire | Commentaires sur la fonctionnalité de commande. |
| Description | Description de la fonctionnalité de commande. |
| Requête | Si cette option est activée, il s’agit d’une définition du paramètre de requête, notamment le nom, le nom d’affichage, le schéma, l’unité et l’unité d’affichage. |
| response | Si cette option est activée, il s’agit d’une définition de la réponse de commande, notamment le nom, le nom d’affichage, le schéma, l’unité et l’unité d’affichage. |

Pour en savoir plus sur la façon dont les appareils implémentent les commandes, consultez [Charges utiles de télémétrie, de propriétés et de commandes > Commandes et commandes de longue durée](concepts-telemetry-properties-commands.md#commands).

#### <a name="offline-commands"></a>Commandes hors connexion

Vous pouvez choisir des commandes de mise en file d’attente si un appareil est actuellement hors connexion en activant l’option **Mettre en file d’attente si hors connexion** pour une commande dans le modèle d’appareil.

Cette option utilise des messages IoT Hub cloud-à-appareil pour envoyer les notifications aux appareils. Pour plus d’informations, consultez l’article IoT Hub [Envoyer des messages cloud-à-appareil](../../iot-hub/iot-hub-devguide-messages-c2d.md).

Messages cloud-à-appareil :

- Notifications unidirectionnelles de votre solution vers l’appareil.
- Remise des messages au moins une fois. IoT Hub conserve les messages cloud-à-appareil dans des files d’attente par appareil, garantissant ainsi la résilience face aux défaillances de connectivité et des appareils.
- Exiger que l’appareil implémente un gestionnaire de messages pour traiter le message cloud-à-appareil.

> [!NOTE]
> Cette option est disponible uniquement dans l’interface utilisateur web d’IoT Central. Ce paramètre n’est pas inclus si vous exportez un modèle ou un composant à partir du modèle d’appareil.

## <a name="manage-a-component"></a>Gérer un composant

Si vous n’avez pas publié le composant, vous pouvez modifier les fonctionnalités définies par le composant. Une fois que vous avez publié le composant, si vous souhaitez apporter des modifications, vous devez créer une version du modèle d’appareil et une [version du composant](howto-version-device-template.md). Vous pouvez apporter des modifications qui ne nécessitent pas de versioning, telles que des noms d’affichage ou des unités, dans la section **Personnaliser**.

Vous pouvez également exporter le composant en tant que fichier JSON si vous souhaitez la réutiliser dans un autre modèle de capacité.

## <a name="add-cloud-properties"></a>Ajouter des propriétés du cloud

Utilisez les propriétés de cloud pour stocker des informations sur les appareils dans IoT Central. Les propriétés de cloud ne sont jamais envoyées à un appareil. Par exemple, vous pouvez utiliser les propriétés de cloud pour stocker le nom du client qui a installé l’appareil ou la date de la dernière maintenance de l’appareil.

Le tableau suivant illustre les paramètres de configuration d’une propriété de cloud :

| Champ | Description |
| ----- | ----------- |
| Nom d’affichage | Nom complet de la valeur de propriété de cloud utilisée sur les tableaux de bord et les formulaires. |
| Nom | Nom de la propriété de cloud. IoT Central génère une valeur pour ce champ à partir du nom d’affichage, mais vous pouvez choisir votre propre valeur si nécessaire. |
| Type sémantique | Type sémantique de la propriété, telles que la température, l’état ou l’événement. Le choix du type sémantique détermine lequel des champs suivants est disponible. |
| schéma | Type de données de propriété de cloud, tel que double, chaîne ou vecteur. Les options disponibles sont déterminées par le type sémantique. |

## <a name="add-customizations"></a>Ajouter des personnalisations

Utilisez des personnalisations quand vous devez modifier un composant importé ou ajouter des fonctionnalités propres à IoT Central à une capacité. Vous pouvez uniquement personnaliser les champs qui n’interrompent pas la compatibilité du composant. Vous pouvez par exemple :

- Personnaliser le nom complet et les unités d’une capacité.
- Ajouter une couleur par défaut à utiliser quand la valeur apparaît sur un graphique.
- Spécifier les valeurs initiales, minimales et maximales d’une propriété.

Vous ne pouvez pas personnaliser le nom ou le type de la capacité. Si vous ne pouvez pas apporter certaines modifications dans la section **Personnaliser**, vous devez contrôler la version de votre modèle d’appareil et de votre composant pour modifier la capacité.

### <a name="generate-default-views"></a>Générer des vues par défaut

La génération de vues par défaut permet de visualiser rapidement les informations importantes de votre appareil. Vous avez jusqu’à trois vues par défaut générées pour votre modèle d’appareil :

- **Commandes** : affiche des commandes d’appareil et permet à votre opérateur de les répartir sur votre appareil.
- **Vue d’ensemble** : affiche la télémétrie de l’appareil sous la forme de graphiques et de métriques.
- **À propos de** : affiche des informations sur l’appareil et ses propriétés.

Une fois que vous avez sélectionné **Générer des vues par défaut**, elles sont ajoutées automatiquement sous la section **Vues** de votre modèle d’appareil.

## <a name="add-dashboards"></a>Ajouter des tableaux de bord

Ajoutez des tableaux de bord à un modèle d’appareil pour permettre aux opérateurs de visualiser un appareil à l’aide de graphiques et de métriques. Vous pouvez avoir plusieurs tableaux de bord pour un modèle d’appareil.

Pour ajouter un tableau de bord à un modèle d’appareil :

1. Accédez à votre modèle d’appareil et sélectionnez **Vues**.
1. Choisissez **Visualisation de l’appareil**.
1. Entrez un nom pour votre tableau de bord dans **Nom du tableau de bord**.
1. Ajoutez des vignettes à votre tableau de bord à partir de la liste des vignettes statiques, des propriétés, des propriétés de cloud, des données de télémétrie et des commandes. Faites glisser et déplacez les vignettes que vous souhaitez ajouter à votre tableau de bord.
1. Pour tracer plusieurs valeurs de données de télémétrie sur une seule vignette de graphique, sélectionnez les valeurs de données de télémétrie, puis sélectionnez **Combiner**.
1. Configurez chaque vignette que vous ajoutez pour personnaliser le mode d’affichage des données. Accédez à cette option en sélectionnant l’icône d’engrenage ou **Changer la configuration** sur la vignette de votre graphique.
1. Réorganisez et redimensionnez les vignettes sur votre tableau de bord.
1. Enregistrez les modifications.

### <a name="configure-preview-device-to-view-dashboard"></a>Configurer l’aperçu de l’appareil pour afficher le tableau de bord

Pour afficher et tester votre tableau de bord, sélectionnez **Configurer l’appareil en préversion**. Cette fonctionnalité vous permet de voir le tableau de bord tel que le voit votre opérateur une fois qu’il a été publié. Utilisez cette fonctionnalité pour vérifier que vos vues affichent les données qui conviennent. Vous pouvez choisir parmi les options suivantes :

- Aucun appareil en préversion
- Le véritable appareil de test que vous avez configuré pour votre modèle d’appareil
- Un appareil existant dans votre application, à l’aide de l’ID de l’appareil

## <a name="add-forms"></a>Ajouter des formulaires

Ajoutez des formulaires à un modèle d’appareil pour permettre aux opérateurs de gérer un appareil en affichant et en définissant des propriétés. Les opérateurs peuvent uniquement modifier des propriétés de cloud et des propriétés d’appareil inscriptibles. Vous pouvez avoir plusieurs formulaires pour un modèle d’appareil.

Pour ajouter un formulaire à un modèle d’appareil :

1. Accédez à votre modèle d’appareil et sélectionnez **Vues**.
1. Choisissez **Modification des données de l’appareil et du cloud**.
1. Entrez un nom pour votre formulaire dans **Nom du formulaire**.
1. Sélectionnez le nombre de colonnes à utiliser pour disposer votre formulaire.
1. Ajoutez des propriétés à une section existante de votre formulaire, ou sélectionnez des propriétés et choisissez **Ajouter une section**. Utilisez des sections pour regrouper les propriétés sur votre formulaire. Vous pouvez ajouter un titre à une section.
1. Configurez chaque propriété du formulaire pour personnaliser son comportement.
1. Disposez les propriétés sur votre formulaire.
1. Enregistrez les modifications.

## <a name="publish-a-device-template"></a>Publier un modèle d’appareil

Avant de pouvoir connecter un appareil qui implémente votre modèle d’appareil, vous devez publier votre modèle d’appareil.

Après avoir publié un modèle d’appareil, vous ne pouvez apporter que des modifications limitées au modèle d’appareil. Pour modifier un composant, vous devez [créer et publier une nouvelle version](./howto-version-device-template.md).

Pour publier un modèle d’appareil, accédez à votre modèle d’appareil et sélectionnez **Publier**.

Une fois que vous avez publié un modèle d’appareil, un opérateur peut accéder à la page **Appareils** et ajouter des appareils, réels ou simulés, qui utilisent votre modèle d’appareil. Vous pouvez continuer à modifier et à enregistrer votre modèle d’appareil au fur et à mesure que vous apportez des modifications. Quand vous souhaitez envoyer ces modifications à l’opérateur pour qu’elles s’affichent sous la page **Appareils**, vous devez systématiquement sélectionner **Publier**.

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes un développeur d’appareils, nous vous suggérons de vous renseigner sur le [contrôle de version des modèles d’appareil](./howto-version-device-template.md).

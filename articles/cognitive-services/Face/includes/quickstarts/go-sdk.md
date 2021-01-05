---
title: Démarrage rapide – Bibliothèque de client Go Visage
description: Utilisez la bibliothèque de client Visage pour Go afin de détecter des visages, rechercher des visages semblables (recherche faciale par image), identifier des visages (recherche avec reconnaissance faciale) et migrer vos données de visage.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 10/26/2020
ms.author: pafarley
ms.openlocfilehash: 65f7af56e7f0042b8d4c312d17641a537f5fd908
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999328"
---
Commencez à utiliser la reconnaissance faciale avec la bibliothèque de client Visage pour Go. Suivez les étapes suivantes pour installer le package et essayer l’exemple de code pour les tâches de base. Le service Visage vous donne accès à des algorithmes avancés pour la détection et la reconnaissance des visages dans des images.

Vous pouvez effectuer les tâches suivantes à l’aide de la bibliothèque de client du service Visage pour Go :

* [Détecter des visages sur une image](#detect-faces-in-an-image)
* [Rechercher des visages semblables](#find-similar-faces)
* [Créer et entraîner un groupe de personnes](#create-and-train-a-person-group)
* [Identifier un visage](#identify-a-face)

[Documentation de référence](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v1.0/face) | [Téléchargement du SDK](https://github.com/Azure/azure-sdk-for-go)

## <a name="prerequisites"></a>Prérequis

* La dernière version de [Go](https://golang.org/dl/)
* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="créez une ressource Visage"  target="_blank">créer une ressource Visage <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le Portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Visage. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.
* Une fois que vous avez obtenu une clé et un point de terminaison, [créez des variables d’environnement](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) pour ces derniers, nommées respectivement `FACE_SUBSCRIPTION_KEY` et `FACE_ENDPOINT`.

## <a name="setting-up"></a>Configuration

### <a name="create-a-go-project-directory"></a>Créer un répertoire de projet Go

Dans une fenêtre de console (cmd, PowerShell, terminal, bash), créez un espace de travail pour votre projet Go (nommé `my-app`) et accédez-y.

```
mkdir -p my-app/{src, bin, pkg}  
cd my-app
```

Votre espace de travail contiendra trois dossiers :

* **src** : ce répertoire contient le code source et les packages. Tous les packages installés à l’aide de la commande `go get` se trouvent dans ce dossier.
* **pkg** : ce répertoire contient les objets de package Go compilés. Ces fichiers ont tous une extension `.a`.
* **bin** : ce répertoire contient les fichiers exécutables binaires créés lors de l’exécution de `go install`.

> [!TIP]
> Pour plus d’informations sur la structure d’un espace de travail Go, consultez la [documentation du langage Go](https://golang.org/doc/code.html#Workspaces). Ce guide comprend des informations pour configurer `$GOPATH` et `$GOROOT`.

### <a name="install-the-client-library-for-go"></a>Installer la bibliothèque de client pour Go

Ensuite, installez la bibliothèque de client pour Go :

```bash
go get -u github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v1.0/face
```

Ou, si vous utilisez dep, au sein de votre dépôt, exécutez :

```bash
dep ensure -add https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v1.0/face
```

### <a name="create-a-go-application"></a>Créer une application Go

Ensuite, créez un fichier nommé `sample-app.go` dans le répertoire **src** :

```bash
cd src
touch sample-app.go
```

Ouvrez `sample-app.go` dans l’éditeur de texte ou l’environnement de développement intégré de votre choix. Ajoutez ensuite le nom du package et importez les bibliothèques suivantes :

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_imports)]

Ensuite, vous commencerez à ajouter du code pour effectuer différentes opérations du service Visage.

## <a name="object-model"></a>Modèle objet

Les classes et interfaces suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client Go du service Visage.

|Nom|Description|
|---|---|
|[BaseClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#BaseClient) | Cette classe représente votre autorisation d’utiliser le service Visage. Vous en avez besoin pour toutes les fonctionnalités de Visage. Vous pouvez l’instancier avec vos informations d’abonnement et l’utiliser pour produire des instances d’autres classes. |
|[Client](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client)|Cette classe gère les tâches de détection et de reconnaissance de base que vous pouvez effectuer avec les visages. |
|[DetectedFace](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#DetectedFace)|Cette classe représente toutes les données détectées à partir d’un visage unique dans une image. Vous pouvez l’utiliser pour récupérer des informations détaillées sur le visage.|
|[ListClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#ListClient)|Cette classe gère les constructions **FaceList** stockées dans le cloud, comprenant un ensemble de visages assortis. |
|[PersonGroupPersonClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupPersonClient)| Cette classe gère les constructions **Person** stockées dans le cloud, comprenant un ensemble de visages appartenant à une même personne.|
|[PersonGroupClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupClient)| Cette classe gère les constructions **PersonGroup** stockées dans le cloud, comprenant un ensemble d’objets **Person** assortis. |
|[SnapshotClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#SnapshotClient)|Cette classe gère la fonctionnalité de capture instantanée. Vous pouvez l’utiliser pour enregistrer temporairement toutes vos données de visage basées sur le cloud et migrer ces données vers un nouvel abonnement Azure. |

## <a name="code-examples"></a>Exemples de code

Ces exemples de code vous montrent comment effectuer des tâches de base à l’aide de la bibliothèque de client du service Visage pour Go :

* [Authentifier le client](#authenticate-the-client)
* [Détecter des visages sur une image](#detect-faces-in-an-image)
* [Rechercher des visages semblables](#find-similar-faces)
* [Créer et entraîner un groupe de personnes](#create-and-train-a-person-group)
* [Identifier un visage](#identify-a-face)

## <a name="authenticate-the-client"></a>Authentifier le client

> [!NOTE] 
> Ce guide de démarrage rapide suppose que vous avez [créé des variables d’environnement](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) pour votre clé et votre point de terminaison du service Visage, nommées `FACE_SUBSCRIPTION_KEY` et `FACE_ENDPOINT` respectivement.

Créez une fonction **main** et ajoutez-lui le code suivant pour instancier un client avec votre point de terminaison et votre clé. Créez un objet **[CognitiveServicesAuthorizer](https://godoc.org/github.com/Azure/go-autorest/autorest#CognitiveServicesAuthorizer)** avec votre clé et utilisez-le avec votre point de terminaison pour créer un objet **[Client](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client)** . Ce code instancie également un objet de contexte, qui est nécessaire pour la création d’objets clients. Il définit aussi un emplacement distant où se trouvent certains des exemples d’images utilisés dans ce guide de démarrage rapide.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_main_client)]


## <a name="detect-faces-in-an-image"></a>Détecter des visages dans une image

Ajoutez le code suivant à la méthode **main**. Ce code définit un exemple d’image distante et spécifie les fonctionnalités de visage à extraire de l’image. Il définit également le modèle IA à utiliser pour extraire des données du ou des visages détectés. Pour plus d’informations sur ces options, consultez [Spécifier un modèle de reconnaissance](../../Face-API-How-to-Topics/specify-recognition-model.md). Enfin, la méthode **[DetectWithURL](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client.DetectWithURL)** effectue l’opération de détection de visage sur l’image et enregistre les résultats dans la mémoire du programme.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_detect)]

> [!TIP]
> Vous pouvez également détecter les visages dans une image locale. Consultez les méthodes [Client](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client) comme **DetectWithStream**.

### <a name="display-detected-face-data"></a>Afficher les données de visage détectées

Le bloc de code suivant prend le premier élément dans le tableau d’objets **[DetectedFace](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#DetectedFace)** et affiche ses attributs sur la console. Si vous avez utilisé une image comportant plusieurs visages, vous devez faire une itération dans le tableau à la place.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_detect_display)]

## <a name="find-similar-faces"></a>Rechercher des visages semblables

Le code suivant utilise un visage unique détecté (la source) et recherche un ensemble d’autres visages (la cible) pour trouver des correspondances (recherche faciale par image). Quand il trouve une correspondance, il affiche l’ID du visage correspondant sur la console.

### <a name="detect-faces-for-comparison"></a>Détecter des visages pour les comparer

Tout d’abord, enregistrez une référence au visage qui a été détecté dans la section [Détecter des visages dans une image](#detect-faces-in-an-image). Ce visage constituera la source.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_similar_single_ref)]

Entrez maintenant le code suivant pour détecter un ensemble de visages dans une autre image. Ces visages constitueront la cible.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_similar_multiple_ref)]

### <a name="find-matches"></a>Rechercher des correspondances

Le code suivant utilise la méthode **[FindSimilar](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client.FindSimilar)** pour rechercher tous les visages cibles qui correspondent au visage source.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_similar)]

### <a name="print-matches"></a>Afficher les correspondances

Le code suivant affiche les détails relatifs aux correspondances sur la console.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_similar_print)]


## <a name="create-and-train-a-person-group"></a>Créer et entraîner un groupe de personnes

Pour suivre ce scénario, vous devez enregistrer les images suivantes dans le répertoire racine de votre projet : https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

Ce groupe d’images contient trois ensembles d’images avec un seul visage qui correspondent à trois personnes différentes. Le code définit trois objets **PersonGroup Person** et les associe aux fichiers image qui commencent par `woman`, `man` et `child`.

### <a name="create-persongroup"></a>Créer un PersonGroup

Une fois que vous avez téléchargé vos images, ajoutez le code suivant en bas de la méthode **main**. Ce code authentifie un objet **[PersonGroupClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupClient)** , puis l’utilise pour définir un nouvel objet **PersonGroup**.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_pg_setup)]

### <a name="create-persongroup-persons"></a>Créer des objets PersonGroup Person

Le bloc de code suivant authentifie un objet **[PersonGroupPersonClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupPersonClient)** et l’utilise pour définir trois nouveaux objets **PersonGroup Person**. Ces objets représentent chacun une seule personne dans l’ensemble d’images.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_pgp_setup)]

### <a name="assign-faces-to-persons"></a>Attribuer des visages aux personnes

Le code suivant trie les images en fonction de leur préfixe, détecte les visages, puis attribue les visages à leur objet **PersonGroup Person** respectif sur la base du nom du fichier image.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_pgp_assign)]

> [!TIP]
> Vous pouvez également créer un **PersonGroup** à partir d’images distantes référencées par URL. Consultez les méthodes [PersonGroupPersonClient](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupPersonClient) comme **AddFaceFromURL**.

### <a name="train-persongroup"></a>Entraîner le PersonGroup

Une fois que vous avez attribué les visages, vous entraînez le **PersonGroup** pour qu’il puisse identifier les caractéristiques visuelles associées à chacun de ses objets **Person**. Le code suivant appelle la méthode **train** asynchrone et interroge le résultat en affichant l’état sur la console.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_pg_train)]

> [!TIP]
> L’API Visage s’exécute sur un ensemble de modèles prédéfinis qui sont statiques par nature (les performances du modèle ne se dégradent pas ou ne s’améliorent pas quand le service est exécuté). Les résultats générés par le modèle risquent de changer si Microsoft met à jour le back-end du modèle sans migrer vers une version entièrement nouvelle du modèle. Pour bénéficier d’une version plus récente d’un modèle, vous pouvez réentraîner votre **PersonGroup**, en spécifiant le modèle plus récent en tant que paramètre avec les mêmes images d’inscription.

## <a name="identify-a-face"></a>Identifier un visage

L’opération d’identification prend une image d’une personne (ou de plusieurs personnes) et recherche l’identité de chaque visage dans l’image (recherche avec reconnaissance faciale). Il compare chaque visage détecté à un **PersonGroup**, une base de données comprenant différents objets **Person** dont les caractéristiques du visage sont connues.

> [!IMPORTANT]
> Pour exécuter cet exemple, vous devez d’abord exécuter le code fourni à la section [Créer et entraîner un groupe de personnes](#create-and-train-a-person-group).

### <a name="get-a-test-image"></a>Obtenir une image de test

Le code suivant recherche une image _test-image-person-group.jpg_ à la racine du projet et la charge dans la mémoire du programme. Vous trouverez cette image dans le même dépôt que les images utilisées dans la section [Créer et entraîner un groupe de personnes](#create-and-train-a-person-group) : https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_id_source_get)]

### <a name="detect-source-faces-in-test-image"></a>Détecter les visages sources dans l’image de test

Le bloc de code suivant effectue une détection de visage ordinaire sur l’image de test pour récupérer toutes les données de visage et les enregistrer dans un tableau.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_id_source_detect)]

### <a name="identify-faces"></a>Identifier des visages

La méthode **[Identify](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#Client.Identify)** prend le tableau des visages détectés et les compare à l’objet **PersonGroup** (qui a été défini et entraîné dans la section précédente). Si elle trouve une correspondance entre un visage détecté et un objet **Person** dans le groupe, elle enregistre le résultat.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_id)]

Ce code affiche ensuite les résultats de correspondance détaillés sur la console.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_id_print)]


## <a name="verify-faces"></a>Vérifier les visages

L’opération Vérifier accepte un ID de visage et soit un autre ID de visage, soit un objet **Person**, et détermine s’ils appartiennent à la même personne.

Le code suivant détecte les visages dans deux images sources, puis il compare chaque visage détecté à un visage détecté dans une image cible.

### <a name="get-test-images"></a>Obtenir les images de test

Les blocs de code suivants déclarent des variables qui pointent vers les images sources et cibles pour l’opération de vérification.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_ver_images)]

### <a name="detect-faces-for-verification"></a>Détecter les visages à vérifier

Le code suivant détecte des visages dans les images source et cible, et les enregistre dans des variables.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_ver_detect_source)]

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_ver_detect_target)]

### <a name="get-verification-results"></a>Obtenir les résultats de la vérification

Le code suivant compare chacune des images sources à l’image cible et affiche un message indiquant si elles appartiennent à la même personne.

[!code-go[](~/cognitive-services-quickstart-code/go/Face/FaceQuickstart.go?name=snippet_ver)]

## <a name="run-the-application"></a>Exécution de l'application

Exécutez votre application de reconnaissance faciale à partir du répertoire de l’application avec la commande `go run <app-name>`.

```bash
go run sample-app.go
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

Si vous avez créé un objet **PersonGroup** dans le cadre de ce guide de démarrage rapide et que vous voulez le supprimer, appelez la méthode **[Delete](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face#PersonGroupClient.Delete)** .

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de ce guide de démarrage rapide, vous avez appris à utiliser la bibliothèque de client Visage pour Go afin d’effectuer des tâches de reconnaissance faciale basiques. Pour plus d’informations sur la bibliothèque, reportez-vous à la documentation de référence.

> [!div class="nextstepaction"]
> [Informations de référence sur l’API Visage (Go)](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v1.0/face)

* [Qu’est ce que le service Visage ?](../../overview.md)
* Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/go/Face/FaceQuickstart.go).
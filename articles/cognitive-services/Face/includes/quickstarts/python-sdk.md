---
title: 'Démarrage rapide : Bibliothèque de client Python Visage'
description: Utilisez la bibliothèque de client Visage pour Python afin de détecter des visages, rechercher des visages semblables (recherche faciale par image) et identifier des visages (recherche avec reconnaissance faciale).
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 11/10/2020
ms.author: pafarley
ms.openlocfilehash: 7141ebe4e7894c975ba2ee6fb39d5bfd4483ed41
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999330"
---
Commencez à utiliser la reconnaissance faciale avec la bibliothèque de client Visage pour Python. Suivez les étapes suivantes pour installer le package et essayer l’exemple de code pour les tâches de base. Le service Visage vous donne accès à des algorithmes avancés pour la détection et la reconnaissance des visages dans des images.

Utilisez la bibliothèque de client Visage pour Python pour :

* [Détecter des visages sur une image](#detect-faces-in-an-image)
* [Rechercher des visages semblables](#find-similar-faces)
* [Créer et entraîner un groupe de personnes](#create-and-train-a-person-group)
* [Identifier un visage](#identify-a-face)
* [Vérifier les visages](#verify-faces)

[Documentation de référence](/python/api/azure-cognitiveservices-vision-face/?view=azure-python) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-face) | [Package (PiPy)](https://pypi.org/project/azure-cognitiveservices-vision-face/) | [Exemples C#](/samples/browse/?products=azure&term=face)

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* [Python 3.x](https://www.python.org/)
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="créez une ressource Visage"  target="_blank">créer une ressource Visage <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le Portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Visage. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="setting-up"></a>Configuration
 
### <a name="install-the-client-library"></a>Installer la bibliothèque de client

Après avoir installé Python, vous pouvez installer la bibliothèque de client avec :

```console
pip install --upgrade azure-cognitiveservices-vision-face
```

### <a name="create-a-new-python-application"></a>Créer une application Python

Créez un script Python&mdash;*quickstart-file.py*, par exemple. Puis, ouvrez-le dans votre éditeur ou IDE habituel et importez les bibliothèques suivantes.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_imports)]

> [!TIP]
> Vous voulez voir l’intégralité du fichier de code de démarrage rapide à la fois ? Vous le trouverez sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/Face/FaceQuickstart.py), qui contient les exemples de code utilisés dans ce guide de démarrage rapide.

Ensuite, créez des variables pour le point de terminaison et la clé Azure de votre ressource.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_subvars)]

> [!IMPORTANT]
> Accédez au portail Azure. Si la ressource [nom du produit] que vous avez créée dans la section **Prérequis** a été déployée, cliquez sur le bouton **Accéder à la ressource** sous **Étapes suivantes**. La clé et le point de terminaison se trouvent dans la page **Clé et point de terminaison** de la ressource, sous **Gestion des ressources**. 
>
> N’oubliez pas de supprimer la clé de votre code une fois que vous avez terminé, et ne la postez jamais publiquement. Pour la production, envisagez d’utiliser une méthode de stockage et d’accès sécurisée pour vos informations d’identification. Par exemple, [Azure Key Vault](../../../../key-vault/general/overview.md).

## <a name="object-model"></a>Modèle objet

Les classes et interfaces suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client Python Visage.

|Nom|Description|
|---|---|
|[FaceClient](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.faceclient?view=azure-python) | Cette classe représente votre autorisation d’utiliser le service Visage. Vous en avez besoin pour toutes les fonctionnalités de Visage. Vous pouvez l’instancier avec vos informations d’abonnement et l’utiliser pour produire des instances d’autres classes. |
|[FaceOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations?view=azure-python)|Cette classe gère les tâches de détection et de reconnaissance de base que vous pouvez effectuer avec les visages. |
|[DetectedFace](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.models.detectedface?view=azure-python)|Cette classe représente toutes les données détectées à partir d’un visage unique dans une image. Vous pouvez l’utiliser pour récupérer des informations détaillées sur le visage.|
|[FaceListOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.facelistoperations?view=azure-python)|Cette classe gère les constructions **FaceList** stockées dans le cloud, comprenant un ensemble de visages assortis. |
|[PersonGroupPersonOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongrouppersonoperations?view=azure-python)| Cette classe gère les constructions **Person** stockées dans le cloud, comprenant un ensemble de visages appartenant à une même personne.|
|[PersonGroupOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongroupoperations?view=azure-python)| Cette classe gère les constructions **PersonGroup** stockées dans le cloud, comprenant un ensemble d’objets **Person** assortis. |
|[ShapshotOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.snapshotoperations?view=azure-python)|Cette classe gère la fonctionnalité de capture instantanée. Vous pouvez l’utiliser pour enregistrer temporairement toutes vos données de visage informatiques et migrer ces données vers un nouvel abonnement Azure. |

## <a name="code-examples"></a>Exemples de code

Ces extraits de code montrent comment effectuer les tâches suivantes avec la bibliothèque de client Visage pour Python :

* [Authentifier le client](#authenticate-the-client)
* [Détecter des visages sur une image](#detect-faces-in-an-image)
* [Rechercher des visages semblables](#find-similar-faces)
* [Créer et entraîner un groupe de personnes](#create-and-train-a-person-group)
* [Identifier un visage](#identify-a-face)
* [Vérifier les visages](#verify-faces)

## <a name="authenticate-the-client"></a>Authentifier le client

Instanciez un client avec votre point de terminaison et la clé. Créez un objet [CognitiveServicesCredentials](/python/api/msrest/msrest.authentication.cognitiveservicescredentials?view=azure-python) avec votre clé et utilisez-le avec votre point de terminaison pour créer un objet [FaceClient](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.faceclient?view=azure-python).

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_auth)]

## <a name="detect-faces-in-an-image"></a>Détecter des visages dans une image

Le code suivant détecte un visage dans une image distante. Il affiche l’ID du visage détecté sur la console et le stocke dans la mémoire du programme. Ensuite, il détecte les visages dans une image où figurent plusieurs personnes et affiche également leurs ID sur la console. En modifiant les paramètres de la méthode [detect_with_url](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations?view=azure-python#detect-with-url-url--return-face-id-true--return-face-landmarks-false--return-face-attributes-none--recognition-model--recognition-01---return-recognition-model-false--detection-model--detection-01---custom-headers-none--raw-false----operation-config-), vous pouvez retourner différentes informations avec chaque objet [DetectedFace](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.models.detectedface?view=azure-python).

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_detect)]

> [!TIP]
> Vous pouvez également détecter les visages dans une image locale. Consultez les méthodes [FaceOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations?view=azure-python) comme **detect_with_stream**.

### <a name="display-and-frame-faces"></a>Afficher et encadrer des visages

Le code suivant génère l’image donnée à l’écran et trace des rectangles autour des visages, à l’aide de la propriété DetectedFace.faceRectangle.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_frame)]

![Jeune femme dont le visage est indiqué par un rectangle rouge](../../images/face-rectangle-result.png)

## <a name="find-similar-faces"></a>Rechercher des visages semblables

Le code suivant utilise un visage unique détecté (la source) et recherche un ensemble d’autres visages (la cible) pour trouver des correspondances (recherche faciale par image). Quand il trouve une correspondance, il affiche l’ID du visage correspondant sur la console.

### <a name="find-matches"></a>Rechercher des correspondances

Tout d’abord, exécutez le code de la section ci-dessus ([Détecter des visages dans une image](#detect-faces-in-an-image)) pour enregistrer une référence à un visage unique. Exécutez ensuite le code suivant pour obtenir des références à plusieurs visages dans une image de groupe.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_detectgroup)]

Ajoutez ensuite le bloc de code suivant pour rechercher les instances du premier visage du groupe. Pour savoir comment modifier ce comportement, consultez la section sur la méthode [find_similar](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations?view=azure-python#find-similar-face-id--face-list-id-none--large-face-list-id-none--face-ids-none--max-num-of-candidates-returned-20--mode--matchperson---custom-headers-none--raw-false----operation-config-).

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_findsimilar)]

### <a name="print-matches"></a>Afficher les correspondances

Utilisez le code suivant pour afficher les détails sur les correspondances sur la console.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_findsimilar_print)]

## <a name="create-and-train-a-person-group"></a>Créer et entraîner un groupe de personnes

Le code suivant crée un **PersonGroup** avec trois objets **Person** différents. Il associe chaque objet **Person** à un ensemble d’exemples d’images, puis s’entraîne pour pouvoir reconnaître chaque personne. 

### <a name="create-persongroup"></a>Créer un PersonGroup

Pour suivre ce scénario, vous devez enregistrer les images suivantes dans le répertoire racine de votre projet : https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

Ce groupe d’images contient trois ensembles d’images de visage correspondant à trois personnes différentes. Le code définit trois objets **Person** et les associe aux fichiers image qui commencent par `woman`, `man`et `child`.

Une fois que vous avez configuré vos images, définissez une étiquette en haut de votre script pour l’objet **PersonGroup** que vous allez créer.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroupvars)]

Ajoutez ensuite le code suivant au bas de votre script. Ce code crée un objet **PersonGroup** et trois objets **Person**.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroup_create)]

### <a name="assign-faces-to-persons"></a>Attribuer des visages aux personnes

Le code suivant trie vos images en fonction de leur préfixe, détecte les visages et attribue les visages à chaque objet **Person**.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroup_assign)]

> [!TIP]
> Vous pouvez également créer un **PersonGroup** à partir d’images distantes référencées par URL. Consultez les méthodes [PersonGroupPersonOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongrouppersonoperations?view=azure-python) comme **add_face_from_url**.

### <a name="train-persongroup"></a>Entraîner le PersonGroup

Une fois que vous avez attribué des visages, vous devez entraîner le **PersonGroup** pour qu’il puisse identifier les caractéristiques visuelles associées à chacun de ses objets **Person**. Le code suivant appelle la méthode **train** asynchrone et interroge le résultat en affichant l’état sur la console.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroup_train)]

> [!TIP]
> L’API Visage s’exécute sur un ensemble de modèles prédéfinis qui sont statiques par nature (les performances du modèle ne se dégradent pas ou ne s’améliorent pas quand le service est exécuté). Les résultats générés par le modèle risquent de changer si Microsoft met à jour le back-end du modèle sans migrer vers une version entièrement nouvelle du modèle. Pour bénéficier d’une version plus récente d’un modèle, vous pouvez réentraîner votre **PersonGroup**, en spécifiant le modèle plus récent en tant que paramètre avec les mêmes images d’inscription.

## <a name="identify-a-face"></a>Identifier un visage

L’opération d’identification prend une image d’une personne (ou de plusieurs personnes) et recherche l’identité de chaque visage dans l’image (recherche avec reconnaissance faciale). Il compare chaque visage détecté à un **PersonGroup**, une base de données comprenant différents objets **Person** dont les caractéristiques du visage sont connues.

> [!IMPORTANT]
> Pour exécuter cet exemple, vous devez d’abord exécuter le code fourni à la section [Créer et entraîner un groupe de personnes](#create-and-train-a-person-group).

### <a name="get-a-test-image"></a>Obtenir une image de test

Le code suivant recherche une image _test-image-person-group.jpg_ à la racine du projet et détecte les visages dans l’image. Vous pouvez trouver cette image avec les images utilisées pour la gestion de **PersonGroup** : https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_identify_testimage)]

### <a name="identify-faces"></a>Identifier des visages

La méthode **identify** compare un groupe de visages détectés à un **PersonGroup**. Si un visage détecté correspond à un objet **Person**, elle enregistre le résultat. Ce code affiche les résultats de correspondance détaillés sur la console.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_identify)]

## <a name="verify-faces"></a>Vérifier les visages

L’opération Vérifier accepte un ID de visage et soit un autre ID de visage, soit un objet **Person**, et détermine s’ils appartiennent à la même personne.

Le code suivant détecte des visages dans deux images sources, puis les compare à un visage détecté à partir d’une image cible.

### <a name="get-test-images"></a>Obtenir les images de test

Les blocs de code suivants déclarent des variables qui pointent vers les images source et cible pour l’opération de vérification.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify_baseurl)]

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify_photos)]

### <a name="detect-faces-for-verification"></a>Détecter les visages à vérifier

Le code suivant détecte des visages dans les images source et cible, et les enregistre dans des variables.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify_detect)]

### <a name="get-verification-results"></a>Obtenir les résultats de la vérification

Le code suivant compare chacune des images sources à l’image cible et affiche un message indiquant si elles appartiennent à la même personne.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify)]

## <a name="run-the-application"></a>Exécution de l'application

Exécutez votre application de reconnaissance faciale à partir du répertoire de l’application avec la commande `python`.

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

Si vous avez créé un **PersonGroup** dans le cadre de ce guide de démarrage rapide et que vous souhaitez le supprimer, exécutez le code suivant dans votre script :

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_deletegroup)]

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de ce guide de démarrage rapide, vous avez appris à utiliser la bibliothèque de client Visage pour Python afin d’effectuer des tâches de reconnaissance faciale basiques. Pour plus d’informations sur la bibliothèque, reportez-vous à la documentation de référence.

> [!div class="nextstepaction"]
> [Informations de référence sur l’API Visage (Python)](/python/api/azure-cognitiveservices-vision-face/?view=azure-python)

* [Qu’est ce que le service Visage ?](../../overview.md)
* Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/Face/FaceQuickstart.py).

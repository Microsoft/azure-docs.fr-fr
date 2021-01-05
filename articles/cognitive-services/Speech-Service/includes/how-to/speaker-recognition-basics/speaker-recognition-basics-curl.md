---
author: v-jaswel
ms.service: cognitive-services
ms.topic: include
ms.date: 09/28/2020
ms.author: v-jawe
ms.custom: references_regions
ms.openlocfilehash: 3a32946e10e8a81394b54fc44e4c92e8625e7ad6
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95015478"
---
Dans ce guide de démarrage rapide, vous allez découvrir les modèles de conception de base pour la reconnaissance de l’orateur à l’aide du SDK de reconnaissance vocale, notamment :

* Vérification dépendante ou indépendante du texte
* Identification de l’orateur pour identifier un échantillon vocal parmi un groupe de voix
* Suppression de profils vocaux

Pour obtenir une vue d’ensemble des concepts de reconnaissance vocale, consultez l’article de [présentation](../../../speaker-recognition-overview.md).

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

> [!IMPORTANT]
> La fonctionnalité Reconnaissance de l’orateur est actuellement *uniquement* prise en charge dans les ressources Azure Speech créées dans la région `westus`.

## <a name="text-dependent-verification"></a>Vérification dépendante du texte

La vérification de l’orateur est l’acte qui consiste à confirmer qu’un orateur correspond à une voix **inscrite** (connue). La première étape consiste à **inscrire** un profil vocal qui permet ensuite au service de comparer les futurs exemples de voix. Dans cet exemple, vous inscrivez le profil à l’aide d’une stratégie **dépendante du texte**, qui nécessite une phrase secrète spécifique à utiliser à la fois pour l’inscription et la vérification. Pour obtenir la liste des phrases secrètes prises en charge, consultez la [documentation de référence](/rest/api/speakerrecognition/).

Commencez par [créer un profil vocal](/rest/api/speakerrecognition/verification/textdependent/createprofile). Vous devrez insérer la clé et la région de votre abonnement au service Speech dans chacune des commandes curl de cet article.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tdv_create_profile":::

Notez qu’il existe trois types de profils vocaux :

- Vérification dépendante du texte
- Vérification indépendante du texte
- Identification indépendante du texte

En l’occurrence, vous créez un profil vocal de vérification dépendante du texte. Vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tdv_create_profile_response":::

Ensuite, vous [inscrivez le profil vocal](/rest/api/speakerrecognition/verification/textdependent/createenrollment). Pour la valeur du paramètre `--data-binary`, spécifiez un fichier audio sur votre ordinateur qui contient l’une des phrases secrètes prises en charge, par exemple « ma voix est mon passeport, vérifiez-moi ». Vous pouvez enregistrer un fichier audio de ce type avec une application telle que l’[Enregistreur vocal Windows](https://www.microsoft.com/p/windows-voice-recorder/9wzdncrfhwkn?activetab=pivot:overviewtab), ou vous pouvez le générer en utilisant la [synthèse vocale](../../../index-text-to-speech.yml).

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tdv_enroll":::

Vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tdv_enroll_response_1":::

Cette réponse vous indique que vous devez inscrire deux échantillons audio supplémentaires.

Une fois que vous avez inscrit un total de trois échantillons audio, vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tdv_enroll_response_2":::

Vous êtes maintenant prêt à [vérifier un échantillon audio par rapport au profil vocal](/rest/api/speakerrecognition/verification/textdependent/verifyprofile). Cet échantillon audio doit contenir la même phrase secrète que les échantillons que vous avez utilisés pour inscrire le profil vocal.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tdv_verify":::

Vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tdv_verify_response":::

La valeur `Accept` signifie que la phrase secrète a correspondu et que la vérification a réussi. La réponse contient également un score de similarité compris entre 0,0 et 1,0.

Pour terminer, [supprimez le profil vocal](/rest/api/speakerrecognition/verification/textdependent/deleteprofile).

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tdv_delete_profile":::

Il n’y a pas de réponse.

## <a name="text-independent-verification"></a>Vérification indépendante du texte

Contrairement à la vérification **dépendante du texte**, la vérification **indépendante du texte** :

* Ne nécessite pas la lecture d’une phrase secrète. Le texte à prononcer est choisi par l’utilisateur.
* Ne nécessite pas trois échantillons audio, mais *nécessite* 20 secondes d’audio en tout.

Commencez par [créer un profil de vérification indépendante du texte](/rest/api/speakerrecognition/verification/textindependent/createprofile).

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tiv_create_profile":::

Vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tiv_create_profile_response":::

Ensuite, [inscrivez le profil vocal](/rest/api/speakerrecognition/verification/textindependent/createenrollment). Là encore, au lieu de soumettre trois échantillons audio, vous devez soumettre des échantillons audio qui contiennent un total de 20 secondes d’audio.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tiv_enroll":::

Une fois que vous avez soumis suffisamment d’échantillons audio, vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tiv_enroll_response":::

Vous êtes maintenant prêt à [vérifier un échantillon audio par rapport au profil vocal](/rest/api/speakerrecognition/verification/textindependent/verifyprofile). Là encore, cet échantillon audio n’a pas besoin de contenir de phrase secrète. Il peut contenir n’importe quelles données vocales, à condition qu’elles représentent au moins quatre secondes d’audio.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tiv_verify":::

Vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tiv_verify_response":::

La valeur `Accept` signifie que la vérification a réussi. La réponse contient également un score de similarité compris entre 0,0 et 1,0.

Pour terminer, [supprimez le profil vocal](/rest/api/speakerrecognition/verification/textindependent/deleteprofile).

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tiv_delete_profile":::

Il n’y a pas de réponse.

## <a name="speaker-identification"></a>Identification de l’orateur

L’identification de l’orateur est utilisée pour déterminer **qui** parle dans un groupe donné de voix inscrites. Le processus est similaire à la **vérification indépendante du texte**. La principale différence est la possibilité de vérifier plusieurs profils vocaux à la fois, au lieu d’un seul profil.

Commencez par [créer un profil d’identification indépendante du texte](/rest/api/speakerrecognition/identification/textindependent/createprofile).

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tii_create_profile":::

Vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tii_create_profile_response":::

Ensuite, vous [inscrivez le profil vocal](/rest/api/speakerrecognition/identification/textindependent/createenrollment). Là encore, vous devez soumettre des échantillons audio qui contiennent un total de 20 secondes d’audio. Ces échantillons n’ont pas besoin de contenir de phrase secrète.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tii_enroll":::

Une fois que vous avez soumis suffisamment d’échantillons audio, vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tii_enroll_response_2":::

Vous êtes maintenant prêt à [identifier un échantillon audio à l’aide du profil vocal](/rest/api/speakerrecognition/identification/textindependent/identifysinglespeaker). La commande d’identification accepte une liste d’ID de profils vocaux possibles séparés par des virgules. En l’occurrence, vous transmettez simplement l’ID du profil vocal que vous avez créé. Toutefois, si vous le souhaitez, vous pouvez transmettre plusieurs ID de profils vocaux, chaque profil étant inscrit avec des échantillons audio à partir d’une voix différente.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tii_identify":::

Vous devez recevoir la réponse suivante.

:::code language="json" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tii_identify_response":::

La réponse contient l’ID du profil vocal qui correspond le plus à l’échantillon audio que vous avez soumis. Elle contient également une liste des profils vocaux candidats, classés par ordre de similarité.

Pour terminer, [supprimez le profil vocal](/rest/api/speakerrecognition/identification/textindependent/deleteprofile).

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speaker-recognition.sh" id="tii_delete_profile":::

Il n’y a pas de réponse.
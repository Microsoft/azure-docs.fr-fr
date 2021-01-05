---
title: Quotas et limites des services de reconnaissance vocale
titleSuffix: Azure Cognitive Services
description: Référence rapide, description détaillée et meilleures pratiques concernant les quotas et limites de la reconnaissance vocale de la solution Azure Cognitive Services
services: cognitive-services
author: alexeyo26
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/07/2020
ms.author: alexeyo
ms.openlocfilehash: 9b923ba208dd2a5111a59f67401b1e4f080b7187
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754162"
---
# <a name="speech-services-quotas-and-limits"></a>Quotas et limites des services de reconnaissance vocale

Cet article contient une référence rapide et la **description détaillée** des quotas et des limites de la reconnaissance vocale d’Azure Cognitive Services pour tous les [niveaux de prix](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/). Il présente également les meilleures pratiques pour éviter la limitation des demandes. 

## <a name="quotas-and-limits-quick-reference"></a>Référence rapide sur les quotas et limites
Passer à [Quotas et limites de la synthèse vocale](#text-to-speech-quotas-and-limits-per-speech-resource)
### <a name="speech-to-text-quotas-and-limits-per-speech-resource"></a>Quotas et limites de reconnaissance vocale par ressource vocale
Dans les tableaux ci-dessous, les paramètres sans ligne « Réglable » ne sont **pas** réglables, quel que soit le niveau de prix.

#### <a name="online-transcription"></a>Transcription en ligne

| Quota | Gratuit (F0)<sup>1</sup> | Standard (S0) |
|--|--|--|
| **Limite des requêtes simultanées (modèles de base et personnalisés)** | 1 | 20 (valeur par défaut) |
| Réglable | Non <sup>2</sup> | Oui<sup>2</sup> |

#### <a name="batch-transcription"></a>Transcription par lot
| Quota | Gratuit (F0)<sup>1</sup> | Standard (S0) |
|--|--|--|
| Limite d’API REST | La transcription Batch n’est pas disponible pour F0 | 300 requêtes par minute |
| Taille maximale de fichier d’entrée audio | N/A | 1 Go |
| Taille maximale de blob d’entrée (peut contenir plusieurs fichiers, par exemple, dans une archive zip ; veillez à noter la limite de taille de fichier indiquée ci-dessus). | N/A | 2,5 Go |
| Taille maximale de conteneur blob | N/A | 5 Go |
| Nombre maximal de blobs par conteneur | N/A | 10000 |
| Requête Nombre maximal de fichiers par transcription (en cas d’utilisation de plusieurs URL de contenu en entrée) | N/A | 1 000  |
| Nombre maximal de travaux exécutés simultanément | N/A | 2000  |

#### <a name="model-customization"></a>Personnalisation des modèles
| Quota | Gratuit (F0)<sup>1</sup> | Standard (S0) |
|--|--|--|
| Limite d’API REST | 300 requêtes par minute | 300 requêtes par minute |
| Nombre maximal de jeux de données vocaux | 2 | 500 |
| Taille maximale de fichier de jeu de données acoustiques pour l’importation de données | 2 Go | 2 Go |
| Taille maximale de fichier de jeu de données linguistiques pour l’importation de données | 200 Mo | 1,5 Go |
| Taille maximale de fichier de jeu de données de prononciation pour l’importation de données | 1 Ko | 1 Mo |
| Taille maximale de texte en cas d’utilisation du paramètre `text` dans une requête d’API de [création de modèle](https://westcentralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/CreateModel/) | 200 Ko | 500 Ko |

<sup>1</sup> Pour le niveau tarifaire **gratuit (F0)** , consultez également les allocations mensuelles sur la [page de tarification](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).<br/>
<sup>2</sup> Consultez les [explications supplémentaires](#detailed-description-quota-adjustment-and-best-practices), les [meilleures pratiques](#general-best-practices-to-mitigate-throttling-during-autoscaling) et les [instructions d’ajustement](#speech-to-text-increasing-online-transcription-concurrent-request-limit).<br/> 

### <a name="text-to-speech-quotas-and-limits-per-speech-resource"></a>Quotas et limites de la synthèse vocale par ressource vocale
Dans le tableau ci-dessous, les paramètres sans ligne « Réglable » ne sont **pas** réglables, quel que soit le niveau de prix.

| Quota | Gratuit (F0)<sup>3</sup> | Standard (S0) |
|--|--|--|
| **Nombre maximal de transactions par seconde (TPS) pour les voix Standard et Neurale** | 200<sup>4</sup> | 200<sup>4</sup> |  |
| **Limite de demandes simultanées pour Custom Voice** |  |  |
| Valeur par défaut | 10 | 10 |
| Réglable | Non<sup>5</sup> | Oui<sup>5</sup> |
| **Quotas spécifiques du protocole HTTP** |  |
| Longueur maximale de l’audio produit par demande | 10 min | 10 min |
| Nombre maximal de balises `<voice>` distinctes en SSML | 50 | 50 |
| **Quotas spécifiques de WebSocket** |  |  |
|Longueur maximale de l’audio produit par tour | 10 min | 10 min |
|Taille maximale des messages SSML par tour |64 Ko |64 Ko |
| **Limite d’API REST** | 20 demandes par minute | 25 demandes par 5 secondes |


<sup>3</sup> Pour le niveau tarifaire **gratuit (F0)** , consultez également les allocations mensuelles sur la [page de tarification](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).<br/>
<sup>4</sup> Consultez les [explications supplémentaires](#detailed-description-quota-adjustment-and-best-practices) et les [meilleures pratiques](#general-best-practices-to-mitigate-throttling-during-autoscaling).<br/>
<sup>5</sup> Consultez les [explications supplémentaires](#detailed-description-quota-adjustment-and-best-practices), les [meilleures pratiques](#general-best-practices-to-mitigate-throttling-during-autoscaling) et les [instructions d’ajustement](#text-to-speech-increasing-transcription-concurrent-request-limit-for-custom-voice).<br/> 

## <a name="detailed-description-quota-adjustment-and-best-practices"></a>Description détaillée, ajustement de quota et meilleures pratiques
Avant de demander une augmentation de quota (le cas échéant), assurez-vous qu’elle est nécessaire. Le service vocal utilise des technologies de mise à l’échelle automatique pour mettre les ressources de calcul requises en mode « à la demande », ainsi que limiter le coût pour les clients en ne conservant pas une quantité excessive de capacité matérielle. Chaque fois que votre application reçoit un code de réponse 429 (« Trop de demandes ») alors que votre charge de travail s’inscrit dans les limites définies (voir [Référence rapide sur les quotas et limites](#quotas-and-limits-quick-reference)), l’explication la plus probable est que le service est en train de mettre à l’échelle votre demande mais n’a pas encore atteint l’échelle requise et que, par conséquent, il ne dispose pas dans l’immédiat de ressources suffisantes pour servir la demande. Cet état est généralement temporaire et ne doit pas durer longtemps.

### <a name="general-best-practices-to-mitigate-throttling-during-autoscaling"></a>Meilleures pratiques générales pour atténuer la limitation lors de la mise à l’échelle automatique
Pour réduire les problèmes liés à la limitation (code de réponse 429), nous vous recommandons d’utiliser les techniques suivantes :
- Implémentez la logique de nouvelle tentative dans votre application.
- Évitez les variations nettes de la charge de travail. Augmentez la charge de travail graduellement. <br/>
*Exemple.* Votre application utilise la synthèse vocale et votre charge de travail actuelle est de 5 TPS (transactions par seconde). À la seconde suivante, vous augmentez la charge à 20 TPS (soit quatre fois plus). Le service commence immédiatement la mise à l’échelle pour répondre à la nouvelle charge. Toutefois, comme il est probable qu’il ne puisse pas le faire en une seconde, certaines demandes reçoivent le code de réponse 429.   
- Testez différents modèles d’augmentation de la charge.
  - Voir [Exemple de reconnaissance vocale](#speech-to-text-example-of-a-workload-pattern-best-practice).
- Créez des ressources vocales supplémentaires dans la même région ou dans des régions différentes, puis distribuez la charge de travail entre celles-ci à l’aide de la technique de « Tourniquet ». C’est particulièrement important pour le paramètre **TPS (transactions par seconde) de synthèse vocale** dont la valeur est de 200 par ressource vocale et ne peut pas être ajusté.  

Les sections suivantes décrivent des cas spécifiques d’ajustement des quotas.<br/>
Accédez à [Synthèse vocale. Augmentation de la limite de demandes simultanées de transcription pour Custom Voice](#text-to-speech-increasing-transcription-concurrent-request-limit-for-custom-voice).

### <a name="speech-to-text-increasing-online-transcription-concurrent-request-limit"></a>Reconnaissance vocale : augmentation de la limite du nombre de demandes simultanées de transcription en ligne
Par défaut, le nombre de demandes simultanées est limité à 20 par ressource vocale (modèle de base) ou point de terminaison personnalisé (modèle personnalisé). Pour le niveau tarifaire standard, vous pouvez augmenter ce montant. Avant de soumettre la demande, assurez-vous d’avoir bien compris le contenu de [cette section](#detailed-description-quota-adjustment-and-best-practices), ainsi que les [meilleures pratiques](#general-best-practices-to-mitigate-throttling-during-autoscaling).

L’amélioration de la limite de demandes simultanées n’affecte **pas** directement vos coûts. Les services de reconnaissance vocale utilisent le modèle « Payez uniquement pour ce que vous utilisez ». La limite définit la hauteur à laquelle le service peut mettre à l’échelle avant de commencer à limiter vos demandes.

Les limites de demandes simultanées pour les modèles **De base** et **Personnalisés** doivent être ajustées **séparément**.

La valeur existante du paramètre Limite de demandes simultanées n’est **pas visible** via le portail Azure, des outils en ligne de commande ou des demandes d’API. Pour vérifier la valeur existante, créez une demande de support Azure.

>[!NOTE]
>Les [conteneurs Speech](speech-container-howto.md) n’exigent pas d’augmentation de la limite de demandes simultanées, car ils ne sont limités que par les processeurs du matériel sur lequel ils sont hébergés. Toutefois, les conteneurs Speech ont leurs propres limitations de capacité qui doivent être prises en compte. Consultez la question *Pouvez-vous aider à planifier la capacité et à estimer les coûts des conteneurs de reconnaissance vocale locaux ?* qui figure dans la [FAQ sur les conteneurs Speech](speech-container-faq.md).

#### <a name="have-the-required-information-ready"></a>Munissez-vous des informations requises :
- Pour le **Modèle de base** :
  - ID de ressource vocale
  - Région
- Pour le **Modèle personnalisé** : 
  - Région
  - ID de point de terminaison personnalisé

- **Comment obtenir des informations (modèle de base)**  :  
  - Accédez au [portail Azure](https://portal.azure.com/).
  - Sélectionnez la ressource vocale dont vous souhaitez augmenter la limite de demandes simultanées.
  - Sélectionnez *Propriétés* (groupe *Gestion des ressources*). 
  - Copiez et enregistrez les valeurs des champs suivants :
    - **ID de ressource**
    - **Emplacement** (région de votre point de terminaison)

- **Comment obtenir des informations (modèle personnalisé)**  :
  - Accéder au portail [Speech Studio](https://speech.microsoft.com/).
  - Connectez-vous si nécessaire.
  - Accédez à Custom Speech.
  - Sélectionnez votre projet.
  - Accédez à *Déploiement*.
  - Sélectionnez le point de terminaison requis.
  - Copiez et enregistrez les valeurs des champs suivants :
    - **Région de service** (région de votre point de terminaison)
    - **ID du point de terminaison**
  
#### <a name="create-and-submit-support-request"></a>Créer et soumettre une demande de support
Initiez l’augmentation de la limite de demandes simultanées pour votre ressource ou, si nécessaire, vérifiez la limite du jour en soumettant la demande de support :

- Vérifiez que vous disposez des [informations requises](#have-the-required-information-ready).
- Accédez au [portail Azure](https://portal.azure.com/).
- Sélectionnez la ressource vocale dont vous souhaitez augmenter (ou vérifier) la limite de demandes simultanées.
- Sélectionnez *Nouvelle demande de support* (groupe *Support et dépannage*) 
- Une nouvelle fenêtre s’affiche, qui contient des informations renseignées automatiquement sur votre abonnement Azure et la ressource Azure.
- Entrez un *Résumé* (par exemple « Augmenter la limite de demandes simultanées STT »).
- Dans *Type de problème*, sélectionnez « Problèmes de quota ou d’abonnement ».
- Dans *Sous-type de problème*, sélectionnez :
  - « Augmentation du quota ou des demandes simultanées » pour une demande d’augmentation, ou
  - « Validation du quota ou de l’utilisation » pour vérifier la limite existante.
- Cliquez sur *Suivant : Solutions*
- Poursuivre avec la création de la demande.
- Sous l’onglet *Détails*, dans le champ *Description*, entrez :
  - une note indiquant que la demande concerne un quota de **reconnaissance vocale** ;
  - modèle **De base** ou **Personnalisé**.
  - les informations sur la ressource Azure que vous [collectées avant](#have-the-required-information-ready) ; 
  - complétez les informations requises, puis cliquez sur le bouton *Créer* sous l’onglet *Vérifier + créer* ;
  - notez le numéro de demande de support dans les notifications du portail Azure. Vous serez bientôt contacté pour un traitement supplémentaire.

### <a name="speech-to-text-example-of-a-workload-pattern-best-practice"></a>Reconnaissance vocale : exemple de meilleure pratique pour un modèle de charge de travail
Cet exemple présente l’approche que nous recommandons de suivre pour atténuer la limitation possible des demandes en raison d’une [mise à l’échelle automatique en cours](#detailed-description-quota-adjustment-and-best-practices). Il ne s’agit pas d’une « recette exacte », mais d’un modèle que nous invitons à suivre et à ajuster en fonction des besoins.

Supposons que la limite de demandes simultanées d’une ressource vocale est définie sur 300. Démarrez la charge de travail à partir de 20 connexions simultanées et augmentez la charge de 20 connexions simultanées toutes les 1,5 à 2 minutes. Contrôlez les réponses du service et implémentez la logique qui revient en arrière (réduit la charge) si vous recevez un trop grand nombre de codes de réponse 429. Réessayez ensuite avec le modèle 1-2-4-4 minutes (c’est-à-dire, réessayez d’augmenter la charge par incrément de 1 minute, puis, si ça ne fonctionne, de 2 minutes, etc.).

En règle générale, il est fortement recommandé de tester la charge de travail et les modèles de charge de travail avant de passer à la production.

### <a name="text-to-speech-increasing-transcription-concurrent-request-limit-for-custom-voice"></a>Synthèse vocale : extension de la limite de demandes simultanées de transcription pour Custom Voice
Par défaut, le nombre de demandes simultanées pour un point de terminaison Custom Voice est limité à 10. Pour le niveau tarifaire standard, vous pouvez augmenter ce montant. Avant de soumettre la demande, assurez-vous d’avoir bien compris le contenu de [cette section](#detailed-description-quota-adjustment-and-best-practices), ainsi que les [meilleures pratiques](#general-best-practices-to-mitigate-throttling-during-autoscaling).

L’amélioration de la limite de demandes simultanées n’affecte **pas** directement vos coûts. Les services de reconnaissance vocale utilisent le modèle « Payez uniquement pour ce que vous utilisez ». La limite définit la hauteur à laquelle le service peut mettre à l’échelle avant de commencer à limiter vos demandes.

La valeur existante du paramètre Limite de demandes simultanées n’est **pas visible** via le portail Azure, des outils en ligne de commande ou des demandes d’API. Pour vérifier la valeur existante, créez une demande de support Azure.

>[!NOTE]
>Les [conteneurs Speech](speech-container-howto.md) n’exigent pas d’augmentation de la limite de demandes simultanées, car ils ne sont limités que par les processeurs du matériel sur lequel ils sont hébergés.

#### <a name="prepare-the-required-information"></a>Préparez les informations requises :
Pour créer une demande d’augmentation, vous devez fournir votre région de déploiement et l’ID du point de terminaison personnalisé. Pour l’obtenir, procédez comme suit : 

- Accéder au portail [Speech Studio](https://speech.microsoft.com/).
- Connectez-vous si nécessaire.
- Accéder à *Custom Voice*.
- Sélectionnez votre projet.
- Accédez à *Déploiement*.
- Sélectionnez le point de terminaison requis.
- Copiez et enregistrez les valeurs des champs suivants :
    - **Région de service** (région de votre point de terminaison)
    - **ID du point de terminaison**
  
#### <a name="create-and-submit-support-request"></a>Créer et soumettre une demande de support
Initiez l’augmentation de la limite de demandes simultanées pour votre ressource ou, si nécessaire, vérifiez la limite du jour en soumettant la demande de support :

- Vérifiez que vous disposez des [informations requises](#prepare-the-required-information).
- Accédez au [portail Azure](https://portal.azure.com/).
- Sélectionnez la ressource vocale dont vous souhaitez augmenter (ou vérifier) la limite de demandes simultanées.
- Sélectionnez *Nouvelle demande de support* (groupe *Support et dépannage*) 
- Une nouvelle fenêtre s’affiche, qui contient des informations renseignées automatiquement sur votre abonnement Azure et la ressource Azure.
- Entrez un *Résumé* (par exemple « Augmenter la limite de demandes simultanées de point de terminaison personnalisé STT »).
- Dans *Type de problème*, sélectionnez « Problèmes de quota ou d’abonnement ».
- Dans *Sous-type de problème*, sélectionnez :
  - « Augmentation du quota ou des demandes simultanées » pour une demande d’augmentation, ou
  - « Validation du quota ou de l’utilisation » pour vérifier la limite existante.
- Cliquez sur *Suivant : Solutions*
- Poursuivre avec la création de la demande.
- Sous l’onglet *Détails*, dans le champ *Description*, entrez :
  - une note indiquant que la demande concerne un quota de **synthèse vocale** ;
  - les informations sur la ressource Azure que vous [collectées avant](#prepare-the-required-information) ; 
  - complétez les informations requises, puis cliquez sur le bouton *Créer* sous l’onglet *Vérifier + créer* ;
  - notez le numéro de demande de support dans les notifications du portail Azure. Vous serez bientôt contacté pour un traitement supplémentaire.


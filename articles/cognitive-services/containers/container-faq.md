---
title: Questions fréquentes (FAQ) sur les conteneurs Cognitive Services
titleSuffix: Azure Cognitive Services
description: Obtenez des réponses aux questions fréquemment posées.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: aahi
ms.openlocfilehash: 188a348b497a485be0a8091b50a3f2c11b6493c0
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94960748"
---
# <a name="azure-cognitive-services-containers-frequently-asked-questions-faq"></a>Questions fréquentes (FAQ) sur les conteneurs Azure Cognitive Services

## <a name="general-questions"></a>Questions générales

**Q : Qu’est-ce qui est disponible ?**

**R :** Les conteneurs Azure Cognitive Services permettent aux développeurs d’utiliser les mêmes API intelligentes que celles disponibles dans Azure, avec les [avantages](../cognitive-services-container-support.md#features-and-benefits) de la mise en conteneur. Certains conteneurs sont disponibles sous la forme d’une préversion contrôlée, dont l’accès peut nécessiter une application. D’autres conteneurs sont disponibles publiquement en préversion non contrôlée, ou sont mises en disponibilité générale. Pour obtenir la liste complète des conteneurs et leur disponibilité, consultez l’article [Prise en charge des conteneurs dans Azure Cognitive Services](../cognitive-services-container-support.md#container-availability-in-azure-cognitive-services). Vous pouvez également afficher les conteneurs dans le [Docker Hub](https://hub.docker.com/_/microsoft-azure-cognitive-services).

**Q : Existe-t-il des différences entre le cloud Cognitive Services et les conteneurs Cognitive Services ?**

**R :** Les conteneurs Cognitive Services sont une alternative au cloud Cognitive Services. Les conteneurs offrent les mêmes fonctionnalités que les services cloud correspondants. Les clients peuvent déployer les conteneurs localement ou dans Azure. La technologie IA de base, les niveaux tarifaires, les clés API et la signature d’API sont les mêmes pour le conteneur et les services cloud correspondants. Voici [les fonctionnalités des conteneurs et leurs avantages](../cognitive-services-container-support.md#features-and-benefits) par rapport aux services cloud équivalents.

**Q : Comment accéder à et utiliser un conteneur de préversion limitée ?**

**R :** Précédemment, les conteneurs de préversion contrôlés étaient hébergés sur le référentiel `containerpreview.azurecr.io`. À compter du 22 septembre 2020, ces conteneurs sont hébergés sur Microsoft Container Registry. Il n’est par ailleurs pas nécessaire d’utiliser la commande docker login pour les télécharger. Vous pouvez exécuter un conteneur de préversion limitée si votre ressource Azure a été créée avec l’ID d’abonnement Azure approuvé. Vous ne pourrez pas exécuter le conteneur si votre abonnement Azure n’a pas été approuvé après avoir rempli le [formulaire de demande](https://aka.ms/csgate).


**Q : Des conteneurs seront-ils disponibles pour tous les services Cognitive Services et quels sont les ensembles de conteneurs à venir ?**

**R :** Nous souhaitons proposer plus de services Cognitive Services sous la forme de conteneurs. Contactez votre gestionnaire local de compte Microsoft pour recevoir des informations sur la publication de nouveaux conteneurs et d’autres annonces concernant Cognitive Services.

**Q : Quel sera le contrat de niveau de service (SLA) des conteneurs Cognitive Services ?**

**R :** Les conteneurs Cognitive Services n’ont pas de contrat SLA.

Les configurations des ressources des conteneurs Cognitive Services étant contrôlées par les clients, Microsoft n’offrira pas de contrat SLA pour la disponibilité générale. Les clients sont libres de déployer des conteneurs localement et de définir les environnements hôtes.

> [!IMPORTANT]
> Pour en savoir plus sur les contrats de niveau de service Cognitive Services, [visitez notre page consacrée aux contrats SLA](https://azure.microsoft.com/support/legal/sla/cognitive-services/v1_1/).

**Q : Ces conteneurs sont-ils disponibles dans les clouds souverains ?**

**R :** Tout le monde ne sachant pas ce qu’est le « cloud souverain », commençons par définir ce terme :

> Le « cloud Souverain » se compose des clouds [Azure Government](../../azure-government/documentation-government-welcome.md), [Azure Allemagne](../../germany/germany-welcome.md) et [Azure Chine 21Vianet](/azure/china/overview-operations).

Malheureusement, les conteneurs Cognitive Services *ne sont pas* pris en charge dans les clouds souverains en mode natif. Les conteneurs peuvent être exécutés dans ces clouds, mais ils seront extraits du cloud public et devront envoyer des données d’utilisation au point de terminaison public.

### <a name="versioning"></a>Contrôle de version

**Q : Comment les conteneurs sont-ils mis à jour avec la dernière version ?**

**R :** Les clients peuvent choisir à quel moment mettre à jour les conteneurs qu’ils ont déployés. Les conteneurs sont marqués à l’aide d’[étiquettes Docker](https://docs.docker.com/engine/reference/commandline/tag/) standard telles que `latest` pour indiquer la version la plus récente. Nous encourageons les clients à extraire la version la plus récente des conteneurs au moment de leur publication. Pour plus d’informations sur la façon de recevoir une notification quand une image est mise à jour, consultez [Webhooks Azure Container Registry](../../container-registry/container-registry-webhook.md).
 
**Q : Quelles sont les versions qui seront prises en charge ?**

**R :** La version actuelle et la dernière version principale du conteneur seront prises en charge. Toutefois, nous encourageons les clients à rester à jour pour bénéficier des dernières technologies.
 
**Q : Comment les versions des mises à jour seront-elles gérées ?**

**R :** La publication d’une nouvelle version principale indique un changement cassant avec la signature d’API. Nous pensons que cela coïncidera généralement avec la publication d’une nouvelle version principale de l’offre Cognitive Services correspondante dans le cloud. La publication d’une version mineure implique la correction de bogues, la mises à jour du modèle ou l’ajout de nouvelles fonctionnalités qui ne constituent pas un changement cassant avec la signature d’API.

## <a name="technical-questions"></a>Questions techniques

**Q : Comment dois-je exécuter les conteneurs Cognitive Services sur des appareils IoT ?**

**R :** Vous ne disposez peut-être pas d’une connexion Internet fiable ou vous souhaitez réduire le coût de la bande passante. Si vous avez besoin d’une latence faible ou si vous traitez des données sensibles qui doivent être analysées localement, consultez [Azure IoT Edge avec des conteneurs Cognitive Services](https://azure.microsoft.com/blog/running-cognitive-services-on-iot-edge/) pour bénéficier d’une solution cohérente avec le cloud.

**Q : Ces conteneurs sont-ils compatibles avec OpenShift ?** 

Nous ne testons pas les conteneurs avec OpenShift, mais en général, les conteneurs Cognitive Services doivent pouvoir s’exécuter sur n’importe quelle plateforme prenant en charge les images Docker. Si vous utilisez OpenShift, nous vous recommandons d’exécuter les conteneurs en tant que `root-user`.

**Q : Comment indiquer des commentaires sur le produit et suggérer des fonctionnalités ?**

**R :** Les clients sont encouragés à [exprimer publiquement leurs préoccupations](https://cognitive.uservoice.com/) et à voter pour d’autres personnes qui ont signalé des problèmes potentiels se chevauchant. L’outil UserVoice peut être utilisé à la fois pour fournir des commentaires sur les produits et suggérer des fonctionnalités.

**Q : Quels sont les messages d’état et les erreurs que les conteneurs Cognitive Services renvoient ?**

**R :** Consultez le tableau suivant pour obtenir la liste des messages d’état et des erreurs.

|Statut  | Description  |
|---------|---------|
| `Valid` | Votre clé d’API est valide, aucune action n’est nécessaire. |
| `Invalid` |   Votre clé d’API n’est pas valide. Vous devez fournir une clé d’API valide pour exécuter le conteneur. Recherchez votre clé d’API et votre région de service dans la section **Clés et points de terminaison** de votre ressource Azure Cognitive Services, dans le portail Azure. |
| `Mismatch` | Vous avez fourni une clé API ou un point de terminaison pour un autre type de ressource Cognitive Services. Recherchez votre clé d’API et votre région de service dans la section **Clés et points de terminaison** de votre ressource Azure Cognitive Services. |
| `CouldNotConnect` | Le conteneur n’a pas pu se connecter au point de terminaison de facturation. Vérifiez la valeur `Retry-After` et attendez que cette période se termine avant d’effectuer des requêtes supplémentaires. |
| `OutOfQuota` | La clé d’API est hors quota. Vous pouvez soit mettre à niveau votre niveau tarifaire, soit attendre qu’un quota supplémentaire soit rendu disponible. Recherchez votre niveau dans la section **niveau tarifaire** de votre ressource Azure Cognitive Services, dans le portail Azure. |
| `BillingEndpointBusy` | Le point de terminaison de facturation est actuellement occupé. Vérifiez la valeur `Retry-After` et attendez que cette période se termine avant d’effectuer des requêtes supplémentaires. |
| `ContainerUseUnauthorized` | La clé d’API fournie n’est pas autorisée à être utilisée avec ce conteneur. Vous utilisez probablement un conteneur limité. Vous devez donc vous assurer que votre ID d’abonnement Azure est approuvé en envoyant une [demande en ligne](https://aka.ms/csgate). |
| `Unknown` | Le serveur ne peut pas traiter les demandes de facturation pour le moment. |


**Q : Qui dois-je contacter pour obtenir de l’aide ?**

**R :** Les canaux de support client sont les mêmes que ceux de l’offre Cognitive Services dans le cloud. Tous les conteneurs Cognitive Services incluent des fonctionnalités de journalisation qui aideront Microsoft et la communauté à offrir une assistance aux clients. Pour obtenir une assistance supplémentaire, consultez les options suivantes.

### <a name="customer-support-plan"></a>Plan de support clientèle

Les clients doivent consulter leur [plan de support Azure](https://azure.microsoft.com/support/plans/) pour voir qui contacter afin d’obtenir de l’aide.

### <a name="azure-knowledge-center"></a>Centre de connaissances Azure

Les clients sont libres d’explorer le [centre de connaissances Azure](https://azure.microsoft.com/resources/knowledge-center/) pour trouver des réponses à leurs questions et à leurs problèmes de support.

### <a name="stack-overflow"></a>Stack Overflow

> [Stack Overflow](https://en.wikipedia.org/wiki/Stack_Overflow) est un site de questions et de réponses destiné aux programmeurs professionnels et passionnés.

Explorez les étiquettes suivantes pour trouver des questions et des réponses correspondant peut-être à vos besoins.

* [Azure Cognitive Services](https://stackoverflow.com/questions/tagged/azure-cognitive-services)
* [Microsoft Cognitive](https://stackoverflow.com/questions/tagged/microsoft-cognitive)

**Q : Comment fonctionne la facturation ?**

**R :** Comme dans le cloud Cognitive Services, les clients sont facturés en fonction de la consommation. Les conteneurs doivent être configurés pour envoyer des données de contrôle à Azure, et les transactions seront facturées en conséquence. Les ressources utilisées dans les services hébergés et locaux seront ajoutées à un quota unique avec des tarifs à plusieurs niveaux pour chaque utilisation. Pour plus d’informations, reportez-vous à la page des tarifs de l’offre correspondante.

* [Détecteur d’anomalies][ad-containers-billing]
* [Vision par ordinateur][cv-containers-billing]
* [Visage][fa-containers-billing]
* [Form Recognizer][fr-containers-billing]
* [Language Understanding (LUIS)][lu-containers-billing]
* [API Speech Service][sp-containers-billing]
* [Analyse de texte][ta-containers-billing]

> [!IMPORTANT]
> Les conteneurs Cognitives Services ne sont pas concédés sous licence pour s’exécuter sans être connectés à Azure pour le contrôle. Les clients doivent configurer les conteneurs de manière à ce qu’ils communiquent les informations de facturation au service de contrôle à tout moment. Les conteneurs Cognitive Services n’envoient pas de données client à Microsoft.
 
**Q : Quelle est la garantie actuelle en matière de prise en charge des conteneurs ?**

**R :** Les préversions ne bénéficient d’aucune garantie. La garantie standard de Microsoft pour les logiciels d’entreprise s’appliquera une fois les conteneurs formellement annoncés comme étant en disponibilité générale.
 
**Q : Qu’advient-il des conteneurs Cognitive Services quand la connectivité Internet est perdue ?**

**R :** La licence *ne prévoit pas* que les conteneurs Cognitives Services soient exécutés sans être connectés à Azure, et ce à des fins de contrôle. Les clients doivent configurer les conteneurs de manière à ce qu’ils communiquent avec le service de contrôle à tout moment.

**Q : Combien de temps le conteneur peut-il fonctionner sans être connecté à Azure ?**

**R :** La licence *ne prévoit pas* que les conteneurs Cognitives Services soient exécutés sans être connectés à Azure, et ce à des fins de contrôle. Les clients doivent configurer les conteneurs de manière à ce qu’ils communiquent avec le service de contrôle à tout moment.
 
**Q : Quel est le matériel actuellement nécessaire pour exécuter ces conteneurs ?**

**R :** Les conteneurs Cognitive Services sont des conteneurs x64 qui peuvent exécuter n’importe quel appareil de périphérie, machine virtuelle ou nœud Linux compatible prenant en charge les conteneurs Docker Linux x64. Ils nécessitent tous des processeurs. Les configurations minimales et recommandées pour chaque offre de conteneur sont disponibles ci-dessous :

* [Détecteur d’anomalies][ad-containers-recommendations]
* [Vision par ordinateur][cv-containers-recommendations]
* [Visage][fa-containers-recommendations]
* [Form Recognizer][fr-containers-recommendations]
* [Language Understanding (LUIS)][lu-containers-recommendations]
* [API Speech Service][sp-containers-recommendations]
* [Analyse de texte][ta-containers-recommendations]
 
**Q : Ces conteneurs sont-ils actuellement pris en charge sur Windows ?**

**R :** Les conteneurs Cognitive Services sont des conteneurs Linux, mais Windows offre un certain niveau de prise en charge. Pour plus d’informations sur les conteneurs Linux sur Windows, consultez la [documentation Docker](https://blog.docker.com/2017/09/preview-linux-containers-on-windows/).
 
**Q : Comment découvrir les conteneurs ?**

**R :** Les conteneurs Cognitive Services sont disponibles dans différents emplacements, notamment le Portail Azure, le hub Docker et les registres de conteneurs Azure. Pour obtenir les emplacements de conteneur les plus récents, reportez-vous aux [images et référentiels de conteneurs](../cognitive-services-container-support.md#container-repositories-and-images).

**Q : Quelles sont les différences entre les conteneurs Cognitive Services et les offres AWS et Google ?**

**R :** Microsoft est le premier fournisseur de cloud à déplacer ses modèles IA pré-entraînés dans des conteneurs avec une facturation simple par transaction, similaire à l’utilisation d’un service cloud. Microsoft est convaincue qu’un cloud hybride offre plus de choix aux clients.

**Q : De quelles certifications de conformité font l’objet les conteneurs ?**

**R :** Les conteneurs Cognitive Services ne présentent aucune certification de conformité.

**Q : Dans quelles régions les conteneurs Cognitive Services sont-ils disponibles ?**

**R :** Les conteneurs peuvent être exécutés n’importe où dans n’importe quelle région, mais ils ont besoin d’une clé et doivent appeler Azure à des fins de contrôle. Toutes les régions prises en charge pour le service cloud sont prises en charge pour l’appel de contrôle des conteneurs.

[!INCLUDE [Containers next steps](includes/containers-next-steps.md)]

[ad-containers]: ../anomaly-Detector/anomaly-detector-container-howto.md
[cv-containers]: ../computer-vision/computer-vision-how-to-install-containers.md
[fa-containers]: ../face/face-how-to-install-containers.md
[fr-containers]: ../form-recognizer/form-recognizer-container-howto.md
[lu-containers]: ../luis/luis-container-howto.md
[sp-containers]: ../speech-service/speech-container-howto.md
[ta-containers]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md

[ad-containers-billing]: ../anomaly-Detector/anomaly-detector-container-howto.md#billing
[cv-containers-billing]: ../computer-vision/computer-vision-how-to-install-containers.md#billing
[fa-containers-billing]: ../face/face-how-to-install-containers.md#billing
[fr-containers-billing]: ../form-recognizer/form-recognizer-container-howto.md#billing
[lu-containers-billing]: ../luis/luis-container-howto.md#billing
[sp-containers-billing]: ../speech-service/speech-container-howto.md#billing
[ta-containers-billing]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md#billing

[ad-containers-recommendations]: ../anomaly-Detector/anomaly-detector-container-howto.md#container-requirements-and-recommendations
[cv-containers-recommendations]: ../computer-vision/computer-vision-how-to-install-containers.md#container-requirements-and-recommendations
[fa-containers-recommendations]: ../face/face-how-to-install-containers.md#container-requirements-and-recommendations
[fr-containers-recommendations]: ../form-recognizer/form-recognizer-container-howto.md#container-requirements-and-recommendations
[lu-containers-recommendations]: ../luis/luis-container-howto.md#container-requirements-and-recommendations
[sp-containers-recommendations]: ../speech-service/speech-container-howto.md#container-requirements-and-recommendations
[ta-containers-recommendations]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md#container-requirements-and-recommendations
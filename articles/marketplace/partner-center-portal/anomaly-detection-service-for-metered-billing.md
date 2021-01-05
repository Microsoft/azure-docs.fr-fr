---
title: Service de détection d’anomalie pour la facturation à l’usage – Place de marché Microsoft Azure
description: Décrit le fonctionnement de la détection d’anomalie, le moment où les notifications sont envoyées et ce qu’il faut en faire, ainsi que les options de support.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.author: mingshen
author: mingshen-ms
ms.date: 06/10/2020
ms.openlocfilehash: 909b818c60932f5b1d6071ea1ccf0ee4c76173b4
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92280543"
---
# <a name="anomaly-detection-service-for-metered-billing"></a>Service de détection d’anomalie pour la facturation à l’usage

Le [service de mesure de la consommation du marketplace](marketplace-metering-service-apis-faq.md) vous permet de créer des offres dans le cadre du programme du marketplace commercial qui sont facturées en fonction d’unités non standard. Avec la facturation à l’usage, vous envoyez des événements d’utilisation relatifs à l’utilisation de votre client à Microsoft et nous préparons la facturation en fonction de cette utilisation.

Des données d’utilisation incorrectes peuvent avoir des causes diverses, telles que des bogues, des erreurs de configuration dans votre suivi de la consommation ou des fraudes. Des données d’utilisation incorrectes entraînent des frais d’utilisation incorrects et des litiges de facturation.

Pour atténuer le risque, notre service de détection d’anomalie applique des algorithmes d’apprentissage automatique pour déterminer le comportement normal de la facturation à l’usage, analyser l’utilisation de la facturation à l’usage et détecter les anomalies avec une intervention minimale de l’utilisateur.

Vous êtes averti si une anomalie a été détectée dans votre utilisation de facturation à l’usage. Cela vous donne la possibilité d’enquêter et de nous informer si une anomalie est confirmée comme étant un problème réel, ce qui permet de prendre des mesures pour régler le problème de facturation des clients de manière proactive.

Outre les pics, les baisses et les changements de tendance soudains de l’utilisation de la facturation à l’usage, notre modèle tient également compte des effets saisonniers. Étant donné que la facturation à l’usage est communiquée par le biais de données de dépassement, notre modèle est également en mesure de traiter sans effort les longues périodes de données manquantes.

Voici des exemples de résultats de la détection d’anomalie. La plage attendue est indiquée par une bande jaune. L’utilisation acceptable de la facturation à l’usage est indiquée par des étoiles vertes dans la bande. L’utilisation de la facturation en dehors de la bande est indiquée par un point rouge.  

Anomalies détectées en dehors d’une tendance prévisible :

![Illustre les anomalies détectées en dehors d’une tendance prévisible.](media/anomaly-1.png)

Anomalies détectées en dehors d’une tendance cyclique récurrente :

![Illustre les anomalies détectées en dehors d’une tendance cyclique récurrente.](media/anomaly-2.png)

Anomalies détectées dans une tendance à la hausse :

![Illustre les anomalies détectées dans une tendance à la hausse.](media/anomaly-3.png)

## <a name="how-anomaly-detection-service-works"></a>Fonctionnement du service de détection d’anomalie

La détection d’anomalie est activée automatiquement pour toutes les utilisations de facturation à l’usage. Lorsque vous envoyez les événements d’utilisation à Microsoft, le service de détection d’anomalie crée un modèle de valeurs attendues en fonction des données d’utilisation passées. Ce modèle s’exécute toutes les semaines.

La détection d’anomalie fonctionne par compteur et par client. Cela signifie que chaque compteur de chaque client aura un modèle formé sur la base de l’utilisation passée de ce compteur par ce client.

Le modèle fonctionne en générant des intervalles de confiance rétrospectifs. La prévision des séries chronologiques est un modèle additif généralisé constitué d’une part de prédiction des tendances et d’une part de saisonnalité. Étant donné que le modèle est formulé comme une tâche de régression, il peut traiter sans effort les longues périodes de données manquantes. Si une observation se situe en dehors des intervalles de confiance prédits, cela signifie que l’observation ne peut pas être expliquée sur la base des modèles historiques de la facturation à l’usage et peut donc être une anomalie.

## <a name="anomaly-detection-notification"></a>Notification de détection d’anomalie

Nous envoyons chaque semaine des notifications de détection d’anomalie par e-mail. Cela comprend toutes les anomalies détectées cette semaine-là pour tous les compteurs et tous les clients. Cet e-mail est envoyé aux contacts **Ingénierie** et **Support** fournis lors de la création de l’offre.

Vous êtes tenu de vérifier si les anomalies détectées sont des problèmes réels et, le cas échéant, de contacter Microsoft pour signaler l’utilisation incorrecte (voir la section support ci-dessous).

Si vous confirmez que les anomalies détectées relèvent d’une utilisation normale, aucune autre action n’est nécessaire de votre part. Toutefois, si une anomalie représente un risque financier potentiellement élevé, nous pouvons être amenés à vous contacter pour confirmer l’utilisation.  

## <a name="when-and-how-to-get-support"></a>Quand et comment obtenir une assistance

Si vous avez envoyé des chiffres d’utilisation incorrects à Microsoft et que cela a abouti ou aboutira à une sous-facturation du client, Microsoft n’enverra pas de facture au client pour une utilisation sous-déclarée et ne vous paiera pas pour cette utilisation. Vous devrez supporter la perte de chiffre d’affaires due à la sous-déclaration.

Si un des cas suivants s’applique, vous pouvez ouvrir un ticket de support pour demander un remboursement ou un ajustement de la facturation pour vos clients :

- Vous avez confirmé qu’une des anomalies que nous avons trouvées est un problème réel et que l’utilisation incorrecte aboutirait à une **surfacturation** du client.
- Vous constatez que vous nous avez envoyé des chiffres d’utilisation incorrects et que ceux-ci aboutiraient à une **surfacturation** du client.
- Vous souhaitez demander un remboursement pour les frais relatifs à l’utilisation de la facturation à l’usage de votre client.

Pour envoyer un ticket :

1. Accédez à la page de support. Dans la **zone Parlez-nous de votre problème**, entrez « Chiffres d’utilisation incorrects ».
2. Dans les rubriques de support, dans la liste déroulante des résultats de la recherche, sélectionnez l’une des options suivantes :
    - **Marketplace commercial** > **Facturation à l’usage** > **Chiffres d’utilisation incorrects envoyés pour l’offre Applications Azure** ; ou
    - **Marketplace commercial** > **Facturation à l’usage** > **Chiffres d’utilisation incorrects envoyés pour une offre SaaS**.
3. Sous **Étape suivante**, sélectionnez le bouton **Examiner les solutions** pour être redirigé vers Espace partenaires et vous y connecter afin de soumettre un ticket de support.

Pour plus d’options de support à destination des éditeurs, consultez [Support technique pour le programme du marketplace commercial dans Espace partenaires](../support.md).

## <a name="next-step"></a>Étape suivante

- En savoir plus sur l’[API du service de mesure de la consommation du marketplace](marketplace-metering-service-apis.md).

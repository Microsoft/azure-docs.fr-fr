---
title: Déployer des certificats X.509 dans Azure IoT Central
description: Comment déployer des certificats X.509 avec votre application IoT Central
author: dominicbetts
ms.author: dobett
ms.date: 07/31/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: a9e35c7d4d64279c65971dd512bcd2107dad6594
ms.sourcegitcommit: 090ea6e8811663941827d1104b4593e29774fa19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "92000069"
---
# <a name="how-to-roll-x509-device-certificates-in-iot-central-application"></a>Comment déployer des certificats d’appareils X.509 dans l’application IoT Central

Pendant le cycle de vie de votre solution IoT, vous devrez renouveler des certificats. Deux des principales raisons pour lesquelles il convient de renouveler les certificats sont les violations de la sécurité et l’expiration des certificats.

Si vous êtes victime d’une violation de la sécurité, le renouvellement des certificats est une bonne pratique en matière de sécurité qui contribue à sécuriser votre système. Dans le cadre de la [méthodologie consistant à envisager les failles](https://download.microsoft.com/download/C/1/9/C1990DBA-502F-4C2A-848D-392B93D9B9C3/Microsoft_Enterprise_Cloud_Red_Teaming.pdf), Microsoft préconise de mettre en place des processus de sécurité réactifs en même temps que des mesures préventives. Vous devez inclure le renouvellement de vos certificats d’appareil dans ces processus de sécurité. La fréquence à laquelle vous renouvelez vos certificats dépend des exigences de sécurité de votre solution. Les clients qui disposent de solutions impliquant des données extrêmement sensibles renouvellent leurs certificats quotidiennement, tandis que les autres clients ne renouvellent leurs certificats que tous les deux ans.


## <a name="obtain-new-x509-certificates"></a>Obtenir de nouveaux certificats X.509

Vous ne pouvez pas créer vos propres certificats X.509 à l’aide d’un outil tel qu’OpenSSL. Cette approche se révèle très efficace pour tester les certificats X.509, mais offre peu de garanties en matière de sécurité. Utilisez cette approche qu’à des fins de test, sauf si vous êtes votre propre fournisseur d’autorité de certification.

## <a name="enrollment-groups-and-security-breaches"></a>Groupes d’inscription et violations de la sécurité

Pour mettre à niveau une inscription de groupe en réponse à une violation de la sécurité, vous devez utiliser l’approche suivante qui met à jour le certificat actuel immédiatement :

1. Accédez à **Administration** dans le volet gauche et sélectionnez **Connexion de l’appareil**.

2. Sélectionnez **Groupes d’inscription** et le nom du groupe dans la liste.

3. Pour mettre à jour des certificats, sélectionnez **Gérer le principal** ou **Gérer le secondaire**.

4. Ajoutez et vérifiez un certificat X.509 racine dans le groupe d’inscription.

   Exécutez cette procédure pour le certificat principal et pour le certificat secondaire s’ils sont tous deux compromis.

## <a name="enrollment-groups-and-certificate-expiration"></a>Groupes d’inscription et expiration des certificats

Si vous renouvelez des certificats pour gérer les expirations de certificats, utilisez l’approche suivante pour mettre immédiatement à jour le certificat actuel :

1. Accédez à **Administration** dans le volet gauche et sélectionnez **Connexion de l’appareil**.

2. Sélectionnez **Groupes d’inscription** et le nom du groupe dans la liste.

3. Pour mettre à jour des certificats, sélectionnez **Gérer le principal**.

4. Ajoutez et vérifiez un certificat X.509 racine dans le groupe d’inscription.

5. Plus tard, lorsque le certificat secondaire a expiré, revenez en arrière et mettez à jour ce certificat secondaire.

## <a name="individual-enrollments-and-security-breaches"></a>Inscriptions individuelles et violations de la sécurité

Si vous renouvelez des certificats en réponse à une violation de la sécurité, utilisez l’approche suivante pour mettre à jour le certificat actuel immédiatement :

1. Sélectionnez **Appareils**, puis sélectionnez l’appareil.

2. Sélectionnez **Connecter**, puis sélectionnez la méthode de connexion comme **Inscription individuelle**

3. Sélectionnez **Certificats (X.509)** en tant que mécanisme.

    ![Gestion des inscriptions individuelles](./media/how-to-roll-x509-certificates/certificate-update.png)

4. Pour mettre à jour les certificats, sélectionnez l’icône de dossier afin de sélectionner le nouveau certificat à charger pour l’entrée d’inscription. Sélectionnez **Enregistrer**.

    Exécutez cette procédure pour les certificat principal et secondaire s’ils sont tous deux compromis

## <a name="individual-enrollments-and-certificate-expiration"></a>Inscriptions individuelles et expiration des certificats

Si vous renouvelez des certificats afin de gérer les expirations de certificats, vous devez utiliser la configuration de certificat secondaire comme suit pour minimiser le temps d’arrêt lié à la tentative de provisionnement des appareils.

Lorsque le certificat secondaire approche de sa date d’expiration et doit être renouvelé, vous pouvez basculer vers la configuration principale. Cette alternance entre le certificat principal et le certificat secondaire réduit le temps d’arrêt découlant de la tentative de provisionnement des appareils.

1. Sélectionnez **Appareils**, puis sélectionnez l’appareil.

2. Sélectionnez **Connecter**, puis sélectionnez la méthode de connexion comme **Inscription individuelle**

3. Sélectionnez **Certificats (X.509)** en tant que mécanisme.

    ![Gestion des inscriptions individuelles](./media/how-to-roll-x509-certificates/certificate-update.png)

4. Pour mettre à jour le certificat secondaire, sélectionnez l’icône de dossier afin de sélectionner le nouveau certificat à charger pour l’entrée d’inscription. Sélectionnez **Enregistrer**.

5. Plus tard, lorsque le certificat principal a expiré, revenez en arrière et mettez à jour ce certificat principal.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à déployer des certificats X.509 dans votre application Azure IoT Central, vous pouvez [Être connecté à Azure IoT Central](concepts-get-connected.md).



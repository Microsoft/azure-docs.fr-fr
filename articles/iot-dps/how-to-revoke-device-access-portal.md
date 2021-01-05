---
title: Révoquer l’accès d’un appareil du Service Azure IoT Hub Device Provisioning
description: Comment révoquer l’accès d’un appareil afin d’empêcher l’approvisionnement via le Service IoT Hub Device Provisioning (DPS) Azure
author: wesmc7777
ms.author: wesmc
ms.date: 04/05/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.openlocfilehash: 120446472038076e34f62b47ba79348e5de8b972
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010926"
---
# <a name="how-to-disenroll-a-device-from-azure-iot-hub-device-provisioning-service"></a>Révoquer l’accès d’un appareil du service Azure IoT Hub Device Provisioning Service

La gestion adéquate des informations d’identification des appareils est essentielle pour les systèmes de grande envergure tels que les solutions IoT. Pour ces systèmes, une bonne pratique consiste à disposer d’un plan indiquant clairement comment révoquer l’accès pour les appareils quand leurs informations d’identification, qu’il s’agisse d’un jeton de signature d’accès partagé (SAP) ou d’un certificat X.509, peuvent être compromises. 

En inscrivant un appareil dans le service Device Provisioning, vous activez son [approvisionnement](about-iot-dps.md#provisioning-process). Un appareil provisionné est une instance inscrite auprès d’IoT Hub, qui peut recevoir son état initial de [jumeau d’appareil](~/articles/iot-hub/iot-hub-devguide-device-twins.md) et commencer à générer des rapports sur les données de télémétrie. Cet article vous explique comment désinscrire un appareil de votre instance du service de provisionnement, en empêchant ainsi tout provisionnement ultérieur.

> [!NOTE] 
> Tenez compte de la stratégie de nouvelle tentative applicable aux appareils pour lesquels vous révoquer l’accès. Par exemple, un appareil qui utilise une stratégie de nouvelles tentatives infinies peut tenter de s’inscrire de manière continue auprès du service de provisionnement. Cette situation consomme des ressources de service et peut affecter les performances.

## <a name="disallow-devices-by-using-an-individual-enrollment-entry"></a>Bloquer des appareils à l’aide d’une entrée d’inscription individuelle

Les inscriptions individuelles s’appliquent à un appareil unique et peuvent utiliser des certificats X.509 ou des jetons SAP (dans un module TPM réel ou virtuel) comme mécanisme d’attestation. (Les appareils qui utilisent des jetons SAP comme mécanisme d’attestation ne peuvent être provisionnés que par le biais d’une inscription individuelle.) Pour bloquer un appareil disposant d’une inscription individuelle, vous pouvez désactiver ou supprimer son entrée d’inscription. 

Pour bloquer provisoirement l’appareil en désactivant son entrée d’inscription : 

1. Connectez-vous au portail Azure et sélectionnez **Toutes les ressources** dans le menu de gauche.
2. Dans la liste des ressources, sélectionnez le service d’approvisionnement dans lequel vous souhaitez bloquer votre appareil.
3. Dans votre service de provisionnement, sélectionnez **Gérer les inscriptions**, puis sélectionnez l’onglet **Inscriptions individuelles**.
4. Sélectionnez l’entrée d’inscription de l’appareil que vous souhaitez bloquer. 

    ![Sélectionner votre inscription individuelle](./media/how-to-revoke-device-access-portal/select-individual-enrollment.png)

5. Faites défiler la page de votre abonnement vers le bas, sélectionnez **Désactiver** pour le commutateur **Activer l’entrée**, puis sélectionnez **Enregistrer**.  

   ![Désactiver une entrée d’inscription individuelle dans le portail](./media/how-to-revoke-device-access-portal/disable-individual-enrollment.png)

Pour bloquer définitivement l’appareil en supprimant son entrée d’inscription :

1. Connectez-vous au portail Azure et sélectionnez **Toutes les ressources** dans le menu de gauche.
2. Dans la liste des ressources, sélectionnez le service d’approvisionnement dans lequel vous souhaitez bloquer votre appareil.
3. Dans votre service de provisionnement, sélectionnez **Gérer les inscriptions**, puis sélectionnez l’onglet **Inscriptions individuelles**.
4. Cochez la case en regard de l’entrée d’inscription de l’appareil à bloquer. 
5. Sélectionnez **Supprimer** en haut de la fenêtre, puis sélectionnez **Oui** pour confirmer que vous voulez supprimer l’inscription. 

   ![Supprimer une entrée d’inscription individuelle dans le portail](./media/how-to-revoke-device-access-portal/delete-individual-enrollment.png)


Une fois la procédure terminée, vous devriez constater que votre entrée a été supprimée de la liste des inscriptions individuelles.  

## <a name="disallow-an-x509-intermediate-or-root-ca-certificate-by-using-an-enrollment-group"></a>Bloquer un certificat d’autorité de certification racine ou intermédiaire X.509 à l’aide d’un groupe d’inscription

Les certificats X.509 sont généralement organisés en une chaîne d’approbation de confiance. Si un certificat à un endroit quelconque d’une chaîne est compromis, l’approbation est rompue. Le certificat doit être bloqué pour empêcher le service d’approvisionnement d’appareils d’approvisionner des appareils en aval dans la chaîne contenant ce certificat. Pour en savoir plus sur les certificats X.509 et sur leur utilisation avec le service de provisionnement, consultez [Certificats X.509](./concepts-x509-attestation.md#x509-certificates). 

Un groupe d’inscriptions est une entrée pour des appareils partageant un mécanisme commun d’attestation de certificats X.509 signés par la même autorité de certification racine ou intermédiaire. L’entrée du groupe d’inscription est configurée avec le certificat X.509 associé à l’autorité de certification racine ou intermédiaire. L’entrée est également configurée avec toutes les valeurs de configuration, telles que l’état du jumeau et la connexion du hub IoT, qui sont partagées par les appareils dont la chaîne de certificats contient ce certificat. Pour bloquer le certificat, vous pouvez désactiver ou supprimer son groupe d’inscriptions.

Pour bloquer temporairement le certificat en désactivant son groupe d’inscriptions : 

1. Connectez-vous au portail Azure et sélectionnez **Toutes les ressources** dans le menu de gauche.
2. Dans la liste des ressources, sélectionnez le service d’approvisionnement dans lequel vous souhaitez bloquer le certificat de signature.
3. Dans votre service de provisionnement, sélectionnez **Gérer les inscriptions**, puis sélectionnez l’onglet **Groupes d’inscription**.
4. Sélectionnez le groupe d’inscriptions utilisant le certificat à bloquer.
5. Sélectionnez **Désactiver** sur le commutateur **Activer l’entrée**, puis sélectionnez **Enregistrer**.  

   ![Désactiver l’entrée du groupe d’inscriptions dans le portail](./media/how-to-revoke-device-access-portal/disable-enrollment-group.png)

    
Pour bloquer définitivement le certificat en supprimant son groupe d’inscriptions :

1. Connectez-vous au portail Azure et sélectionnez **Toutes les ressources** dans le menu de gauche.
2. Dans la liste des ressources, sélectionnez le service d’approvisionnement dans lequel vous souhaitez bloquer votre appareil.
3. Dans votre service de provisionnement, sélectionnez **Gérer les inscriptions**, puis sélectionnez l’onglet **Groupes d’inscription**.
4. Cochez la case en regard du groupe d’inscriptions du certificat à bloquer. 
5. Sélectionnez **Supprimer** en haut de la fenêtre, puis sélectionnez **Oui** pour confirmer que vous voulez supprimer le groupe d’inscription. 

   ![Supprimer l’entrée du groupe d’inscriptions dans le portail](./media/how-to-revoke-device-access-portal/delete-enrollment-group.png)

Une fois la procédure terminée, vous devriez constater que votre entrée a été supprimée de la liste des groupes d’inscription.  

> [!NOTE]
> Si vous supprimez un groupe d’inscription pour un certificat, les appareils dont la chaîne de certificats contient ce certificat peuvent quand même s’inscrire si celle-ci comporte plus en amont un groupe d’inscription activé pour le certificat racine ou un autre certificat intermédiaire.

## <a name="disallow-specific-devices-in-an-enrollment-group"></a>Bloquer des appareils spécifiques dans un groupe d’inscriptions

Les appareils qui implémentent le mécanisme d’attestation X.509 utilisent leur chaîne d’approbation et leur clé privée pour s’authentifier. Quand un appareil se connecte et s’authentifie auprès du service Device Provisioning, le service recherche d’abord une inscription individuelle qui correspond aux informations d’identification de l’appareil. Le service recherche ensuite dans les groupes d’inscription pour déterminer si l’appareil peut être provisionné. Si le service trouve une inscription individuelle désactivée pour l’appareil, il empêche l’appareil de se connecter. Le service empêche la connexion même s’il existe un groupe d’inscription activé pour une autorité de certification racine ou intermédiaire dans la chaîne de certificats de l’appareil. 

Pour bloquer un appareil individuel dans un groupe d’inscriptions, procédez comme suit :

1. Connectez-vous au portail Azure et sélectionnez **Toutes les ressources** dans le menu de gauche.
2. Dans la liste des ressources, sélectionnez le service d’approvisionnement contenant le groupe d’inscriptions pour l’appareil à bloquer.
3. Dans votre service de provisionnement, sélectionnez **Gérer les inscriptions**, puis sélectionnez l’onglet **Inscriptions individuelles**.
4. Sélectionnez le bouton **Ajouter une inscription individuelle** en haut. 
5. Dans la page **Ajouter une inscription**, sélectionnez **X.509** comme **Mécanisme** d’attestation pour l’appareil.

    Chargez le certificat de l’appareil, puis entrez l’ID de l’appareil à bloquer. Pour le certificat, utilisez le certificat d’entité finale signé qui est installé sur l’appareil. L’appareil utilise ce certificat pour l’authentification.

    ![Définir les propriétés de l’appareil bloqué](./media/how-to-revoke-device-access-portal/disable-individual-enrollment-in-enrollment-group-1.png)

6. Faites défiler la page **Ajouter une inscription** vers le bas, sélectionnez **Désactiver** pour le commutateur **Activer l’entrée**, puis sélectionnez **Enregistrer**. 

    [![Utiliser l’entrée d’inscription individuelle désactivée pour désactiver l’appareil de l’inscription de groupe, dans le portail](./media/how-to-revoke-device-access-portal/disable-individual-enrollment-in-enrollment-group.png)](./media/how-to-revoke-device-access-portal/disable-individual-enrollment-in-enrollment-group.png#lightbox)

Une fois votre inscription créée, vous devez normalement voir l’inscription désactivée de votre appareil sous l’onglet **Inscriptions individuelles**. 

## <a name="next-steps"></a>Étapes suivantes

La désinscription fait également partie du processus général d’annulation du provisionnement. L’annulation de l’approvisionnement d’un appareil comprend la désinscription du service de provisionnement, et le retrait de l’instance IoT Hub. Pour en savoir plus sur le processus complet, consultez la section [Guide pratique pour déprovisionner des appareils inscrits par votre service de provisionnement](how-to-unprovision-devices.md)
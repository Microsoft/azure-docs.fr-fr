---
title: Diagnostiquer et résoudre les problèmes de déconnexion avec le DPS IoT Hub
description: Apprendre à diagnostiquer et résoudre les erreurs courantes de connectivité des appareils pour le service Azure IoT Hub Device Provisioning (DPS)
author: xujing-ms
manager: nberdy
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: xujing
ms.openlocfilehash: 2d68314aab636180ff17e330c49c9859b8851b06
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94950684"
---
# <a name="troubleshooting-with-azure-iot-hub-device-provisioning-service"></a>Résolution des problèmes avec le service Azure IoT Hub Device Provisioning

Les problèmes de connectivité des appareils IoT sont parfois difficiles à résoudre, car il existe de nombreux points de défaillance possibles : échecs d’attestations, d’inscriptions, etc. Cet article fournit des conseils sur la détection et la résolution des problèmes de connectivité des appareils avec [Azure Monitor](../azure-monitor/overview.md).

## <a name="using-azure-monitor-to-view-metrics-and-set-up-alerts"></a>Utilisation d’Azure Monitor pour visualiser les métriques et configurer des alertes

La procédure suivante explique comment visualiser et configurer une alerte sur une métrique du service IoT Hub Device Provisioning (DPS). 

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Accédez au service IoT Hub Device Provisioning.

3. Sélectionnez **Métriques**.

4. Sélectionnez la métrique de votre choix. 
   <br />Actuellement, trois métriques sont disponibles pour le DPS :

    | Nom de métrique | Description |
    |-------|------------|
    | Tentatives d’attestation | Nombre d’appareils qui ont tenté de s’authentifier auprès du service Device Provisioning|
    | Tentatives d’enregistrement | Nombre d’appareils qui ont tenté de s’inscrire à IoT Hub après une authentification réussie|
    | Appareil attribué | Nombre d’appareils correctement attribués à IoT Hub|

5. Sélectionnez la méthode d’agrégation souhaitée pour créer une représentation visuelle de la métrique. 

6. Pour configurer une alerte pour une métrique, sélectionnez **Nouvelles règles d’alerte** en haut à droite du panneau de la métrique. Vous pouvez également accéder au panneau **Alerte** et sélectionner **Nouvelles règles d’alerte**.

7. Sélectionnez **Ajouter une condition**, puis sélectionnez la métrique et le seuil souhaités dans les invites.

Pour plus d’informations, consultez l’article [Que sont les alertes classiques dans Microsoft Azure ?](../azure-monitor/platform/alerts-overview.md).

## <a name="using-log-analytic-to-view-and-resolve-errors"></a>Utilisation de Log Analytics pour visualiser et résoudre les erreurs

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Accédez à votre hub IoT.

3. Sélectionnez **Paramètres de diagnostic**.

4. Sélectionnez **Activer les diagnostics**.

5. Activez les journaux à collecter.

    | Nom du journal | Description |
    |-------|------------|
    | DeviceOperations | Journaux relatifs aux événements de connexion d’appareil |
    | ServiceOperations | Journaux d’événements relatifs à l’utilisation du SDK de service (création ou mise à jour de groupes d’inscriptions, par exemple)|

6. Activez l’option **Envoyer à Log Analytics** ([voir les tarifs](https://azure.microsoft.com/pricing/details/log-analytics/)). 

7. Accédez à l’onglet **Journaux** dans le portail Azure sous la ressource Service Device Provisioning.

8. Cliquez sur **Exécuter** pour voir les événements récents.

9. Si des résultats s’affichent, recherchez les éléments `OperationName`, `ResultType`, `ResultSignature` et `ResultDescription` (message d’erreur) pour obtenir plus de détails sur l’erreur.


## <a name="common-error-codes"></a>Common error codes (Codes d’erreur courants)
Utilisez ce tableau pour comprendre et résoudre les erreurs courantes.

| Code d'erreur| Description | Code d’état HTTP |
|-------|------------|------------|
| 400 | Le corps de la requête n’est pas valide ; par exemple, il ne peut pas être analysé ou l’objet ne peut pas être validé.| 400 Format incorrect |
| 401 | Le jeton d’autorisation ne peut pas être validé ; par exemple, il est arrivé à expiration ou ne s’applique pas à l’URI de la requête. Ce code d’erreur est également retourné aux appareils dans le cadre du flux d’attestations du module TPM. | 401 Non autorisé|
| 404 | L’instance du service Device Provisioning ou une ressource (par exemple, une inscription) n’existe pas. |404 Introuvable |
| 412 | L’ETag de la requête ne correspond pas à l’ETag de la ressource existante, conformément à la RFC7232. | 412 Échec de la précondition |
| 429 | Les opérations sont limitées par le service. Pour connaître les limites de service spécifiques, consultez [Limites du service IoT Hub Device Provisioning](../azure-resource-manager/management/azure-subscription-service-limits.md#iot-hub-device-provisioning-service-limits). | 429 Trop de requêtes |
| 500 | Une erreur interne s’est produite. | 500 Erreur interne du serveur|
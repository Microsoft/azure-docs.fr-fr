---
title: Déprovisionner des appareils inscrits auprès du service Azure IoT Hub Device Provisioning
description: Guide pratique pour déprovisionner des appareils inscrits auprès du service Azure IoT Hub Device Provisioning (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 05/11/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: 37099b551e004ebfc702dce37c2a1499aa46ff10
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94950720"
---
# <a name="how-to-deprovision-devices-that-were-previously-auto-provisioned"></a>Guide pratique pour déprovisionner des appareils auparavant approvisionnés automatiquement 

Il peut être nécessaire de déprovisionner des appareils qui ont été provisionnés automatiquement par le biais du service Device Provisioning. Par exemple, un appareil peut avoir été vendu ou déplacé vers un autre hub IoT, ou encore perdu, volé ou compromis. 

En règle générale, le déprovisionnement d’un appareil implique deux étapes :

1. Désinscrivez l’appareil de votre service d’approvisionnement, afin de prévenir tout approvisionnement automatique futur. Selon que vous souhaitiez révoquer l’accès temporairement ou de manière permanente, vous pouvez avoir intérêt à désactiver ou supprimer une entrée d’inscription. Pour les appareils qui utilisent une attestation X.509, il vous faut désactiver/supprimer une entrée dans la hiérarchie de vos groupes d’inscription existants.  
 
   - Pour découvrir comment désinscrire un appareil, consultez la section [Révoquer l’accès des appareils à votre service de provisionnement dans le portail Azure](how-to-revoke-device-access-portal.md).
   - Pour découvrir comment révoquer l’accès des appareils par programmation à l’aide de l’un des SDK de service de provisionnement, consultez la page [Manage device enrollments with service SDKs](how-to-manage-enrollments-sdks.md) (Gérer les inscriptions d’appareils avec les SDK du service).

2. Annulez l’inscription de l’appareil à votre instance IoT Hub, afin de prévenir les communications et les transferts de données à l’avenir. Là encore, vous pouvez désactiver temporairement ou supprimer définitivement l’entrée de l’appareil dans le registre des identités pour l’instance IoT Hub où il a été provisionné. Pour en savoir plus sur la désactivation, consultez la section [Disable devices](../iot-hub/iot-hub-devguide-identity-registry.md#disable-devices) (Désactiver les appareils). Consultez la section relative à la gestion des appareils IoT pour votre ressource IoT Hub, dans le [portail Azure](https://portal.azure.com).

Les étapes précises à suivre pour déprovisionner un appareil dépendent de son mécanisme d’attestation et de son entrée d’inscription au service d’approvisionnement. Les sections suivantes fournissent une vue d’ensemble du processus, en fonction du type d’inscription et d’attestation.

## <a name="individual-enrollments"></a>Inscriptions individuelles
Les appareils qui utilisent l’attestation TPM ou l’attestation X.509 avec un certificat feuille sont approvisionnés par le biais d’une entrée d’inscription individuelle. 

Pour déprovisionner un appareil inscrit individuellement : 

1. Désinscrivez l’appareil de votre service d’approvisionnement :

   - Pour un appareil qui utilise l’attestation TPM, supprimez l’entrée d’inscription individuelle pour révoquer définitivement l’accès de l’appareil au service d’approvisionnement, ou désactivez-la pour révoquer l’accès de façon temporaire. 
   - Dans le cas des appareils qui utilisent l’attestation X.509, vous pouvez soit supprimer, soit désactiver l’entrée. Sachez toutefois que, si vous supprimez l’inscription individuelle d’un appareil qui utilise l’attestation X.509 alors qu’il existe un groupe d’inscription activé pour un certificat de signature de la chaîne de certificats de l’appareil, celui-ci pourra se réinscrire. Il est peut-être plus sûr dans ce cas de désactiver l’entrée d’inscription, ce qui empêche l’appareil de se réinscrire, qu’il existe ou non un groupe d’inscription activé pour l’un de ses certificats de signature.

2. Désactivez ou supprimez l’appareil dans le registre des identités du hub IoT dans lequel il était approvisionné. 


## <a name="enrollment-groups"></a>Groupes d’inscription
Avec l’attestation X.509, les appareils peuvent également être provisionnés par le biais d’un groupe d’inscription. Les groupes d’inscription sont configurés avec un certificat de signature (certificat intermédiaire ou certificat d’autorité de certification racine) et ils contrôlent l’accès au service de provisionnement pour les appareils dont la chaîne de certificats contient ce certificat. Pour en savoir plus sur les groupes d’inscription et les certificats X.509 avec le service de provisionnement, consultez [Attestation de certificat X.509](concepts-x509-attestation.md). 

Pour voir la liste des appareils qui ont été provisionnés par le biais d’un groupe d’inscription, vous pouvez afficher les détails du groupe d’inscription. C’est une manière simple de savoir dans quel hub IoT chaque appareil a été provisionné. Pour afficher la liste des appareils 

1. Connectez-vous au portail Azure et cliquez sur **Toutes les ressources** dans le menu de gauche.
2. Cliquez sur votre service de provisionnement dans la liste des ressources.
3. Dans votre service d’approvisionnement, cliquez sur **Gérer les inscriptions** et sélectionnez l’onglet **Groupes d’inscriptions**.
4. Cliquez sur le groupe d’inscription pour l’ouvrir.

   ![Afficher l’entrée du groupe d’inscription dans le portail](./media/how-to-unprovision-devices/view-enrollment-group.png)

Avec les groupes d’inscriptions, il existe deux scénarios à prendre en compte :

- Pour déprovisionner tous les appareils qui ont été approvisionnés par le biais d’un groupe d’inscription :
  1. Désactivez le groupe d’inscription pour interdire l’utilisation de son certificat de signature. 
  2. Utilisez la liste des appareils approvisionnés pour ce groupe d’inscription afin de désactiver ou de supprimer de son hub IoT chacun des appareils du registre des identités. 
  3. Après avoir désactivé ou supprimé tous les appareils de leurs hubs IoT respectifs, vous pouvez éventuellement supprimer le groupe d’inscription. Sachez toutefois que, si vous supprimez le groupe d’inscription et qu’il existe un groupe d’inscription activé pour un certificat de signature plus haut dans la chaîne de certificats de l’un ou plusieurs des appareils, ces appareils peuvent être réinscrits. 

- Pour déprovisionner un seul appareil d’un groupe d’inscription :
  1. Créez une inscription individuelle désactivée pour son certificat feuille (appareil). Cela révoque l’accès au service de provisionnement pour cet appareil tout en autorisant l’accès pour d’autres appareils dont la chaîne contient le certificat de signature du groupe d’inscription. Ne supprimez pas l’inscription individuelle désactivée pour l’appareil, car cela permettrait à l’appareil d’être réinscrit par le biais du groupe d’inscription. 
  2. Utilisez la liste des appareils approvisionnés de ce groupe d’inscription pour trouver le hub IoT dans lequel l’appareil était approvisionné, et désactivez-le ou supprimez-le du registre des identités de ce hub.
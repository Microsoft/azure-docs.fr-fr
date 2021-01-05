---
title: Renouveler les certificats X.509 dans le service Azure IoT Hub Device Provisioning
description: Comment renouveler des certificats X.509 auprès de votre instance DPS (Device Provisioning Service)
author: wesmc7777
ms.author: wesmc
ms.date: 08/06/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: bf8b1e04e11dee4e636826430838a467fe034e3f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94951126"
---
# <a name="how-to-roll-x509-device-certificates"></a>Renouvellement des certificats d’appareil X.509

Pendant le cycle de vie de votre solution IoT, vous devrez renouveler des certificats. Deux des principales raisons pour lesquelles il convient de renouveler les certificats sont les violations de la sécurité et l’expiration des certificats. 

Le renouvellement des certificats est une bonne pratique en matière de sécurité qui contribue à sécuriser votre système en cas de violation. Dans le cadre de la [méthodologie consistant à envisager les failles](https://download.microsoft.com/download/C/1/9/C1990DBA-502F-4C2A-848D-392B93D9B9C3/Microsoft_Enterprise_Cloud_Red_Teaming.pdf), Microsoft préconise de mettre en place des processus de sécurité réactifs en même temps que des mesures préventives. Vous devez inclure le renouvellement de vos certificats d’appareil dans ces processus de sécurité. La fréquence à laquelle vous renouvelez vos certificats dépend des exigences de sécurité de votre solution. Les clients qui disposent de solutions impliquant des données extrêmement sensibles renouvellent leurs certificats quotidiennement, tandis que les autres clients ne renouvellent leurs certificats que tous les deux ans.

Le renouvellement des certificats d’appareil implique la mise à jour du certificat stocké sur l’appareil et dans IoT Hub. Par la suite, l’appareil peut se reprovisionner lui-même dans le IoT Hub à l’aide du processus normal de [provisionnement](about-iot-dps.md#provisioning-process) avec le service Device Provisioning (DPS).


## <a name="obtain-new-certificates"></a>Obtenir de nouveau certificats

Vous disposez de nombreuses méthodes pour obtenir de nouveaux certificats pour vos appareils IoT. Vous pouvez obtenir des certificats auprès du fabricant des appareils, générer vos propres certificats ou déléguer la création de certificats à un tiers. 

Les certificats sont signés les uns par les autres de manière à former une chaîne d’approbation entre un certificat d’autorité de certification racine et un [certificat feuille](concepts-x509-attestation.md#end-entity-leaf-certificate). Le certificat de signature est le certificat utilisé pour signer le certificat feuille à la fin de la chaîne d’approbation. Un certificat de signature peut être un certificat d’autorité de certification racine ou un certificat intermédiaire dans la chaîne d’approbation. Pour plus d’informations, consultez la section [Certificats X.509](concepts-x509-attestation.md#x509-certificates).
 
Vous disposez de deux méthodes pour obtenir un certificat de signature. La première méthode, qui est recommandée pour les systèmes de production, consiste à acquérir un certificat de signature auprès d’une autorité de certification racine. Cette méthode enchaîne la sécurité à une source approuvée. 

La seconde méthode consiste à créer vos propres certificats X.509 à l’aide d’un outil tel qu’OpenSSL. Cette approche se révèle très efficace pour tester les certificats X.509, mais offre peu de garanties en matière de sécurité. Nous vous recommandons de ne l’utiliser qu’à des fins de test, sauf si vous êtes votre propre fournisseur d’autorité de certification.
 

## <a name="roll-the-certificate-on-the-device"></a>Renouveler le certificat sur l’appareil

Les certificats figurant sur un appareil doivent toujours être stockés dans un endroit sûr, comme un [module de sécurité matériel (HSM)](concepts-service.md#hardware-security-module). La méthode que vous appliquez pour renouveler les certificats d’appareil dépend de la façon dont ces derniers ont été créés et installés à l’origine. 

Si vous avez obtenu vos certificats auprès d’un tiers, vous devez étudier la manière dont celui-ci renouvelle ses certificats. Ce processus peut être inclus dans l’accord que vous avez conclu avec le tiers, ou prendre la forme d’un service distinct offert par le tiers. 

Si vous gérez vos propres certificats d’appareil, vous devrez créer votre pipeline pour la mise à jour des certificats. Assurez-vous que l’ancien certificat feuille porte le même nom commun que le nouveau certificat. L’utilisation du même nom commun permet à l’appareil de se reprovisionner lui-même sans créer d’enregistrement d’inscription en double. 


## <a name="roll-the-certificate-in-the-iot-hub"></a>Renouveler le certificat dans le IoT Hub

Le certificat d’appareil peut être ajouté manuellement à un IoT Hub. Ce certificat peut également être automatisé à l’aide d’une instance du service de provisionnement des appareils. Dans cet article, nous allons supposer que le provisionnement automatique est assuré par une instance du service de provisionnement des appareils.

Lorsqu’un appareil est initialement provisionné par le biais du provisionnement automatique, il démarre et contacte le service de provisionnement. Le service de provisionnement répond en effectuant une vérification d’identité avant de créer une identité d’appareil dans un IoT Hub en utilisant le certificat feuille de l’appareil comme information d’identification. Ensuite, le service de provisionnement indique à l’appareil l’IoT Hub auquel il est assigné, et l’appareil utilise alors son certificat feuille pour s’authentifier et se connecter à l’IoT Hub. 

Une fois qu’un nouveau certificat feuille a été déployé sur l’appareil, ce dernier ne peut plus se connecter à l’IoT Hub, car il utilise un nouveau certificat pour se connecter. L’IoT Hub reconnaît uniquement l’appareil doté de l’ancien certificat. La tentative de connexion de l’appareil se traduira alors par une erreur de connexion « non autorisée ». Pour résoudre cette erreur, vous devez mettre à jour l’entrée d’inscription de l’appareil de manière à prendre en compte le nouveau certificat feuille de l’appareil. Le service de provisionnement peut alors mettre à jour les informations du registre d’appareils IoT Hub en fonction des besoins lorsque l’appareil est reprovisionné. 

Il peut exister une exception à cet échec de connexion, dans le cas où vous avez créé un [groupe d’inscription](concepts-service.md#enrollment-group) pour votre appareil dans le service de provisionnement. Dans ce cas, si vous ne renouvelez pas le certificat racine ni les certificats intermédiaires dans la chaîne d’approbation de certificats de l’appareil, ce dernier sera reconnu si le nouveau certificat fait partie de la chaîne d’approbation définie dans le groupe d’inscription. Si cette situation survient suite à une violation de la sécurité, vous devez au moins bloquer les certificats d’appareil spécifique dans le groupe, qui sont considérés comme compromis. Pour plus d’informations, consultez [Bloquer des appareils spécifiques dans un groupe d’inscriptions](./how-to-revoke-device-access-portal.md#disallow-specific-devices-in-an-enrollment-group).

La mise à jour des entrées d’inscription pour les certificats renouvelés s’effectue sur la page **Gérer les inscriptions**. Pour accéder à cette page, procédez comme suit :

1. Connectez-vous au [Portail Azure](https://portal.azure.com) et accédez à l’instance du service IoT Hub Device Provisioning comportant l’entrée d’inscription pour votre appareil.

2. Cliquez sur **Gérer les inscriptions**.

    ![Gérer les inscriptions](./media/how-to-roll-certificates/manage-enrollments-portal.png)


La manière dont vous gérez la mise à jour de l’entrée d’inscription varie selon que vous utilisez des inscriptions individuelles ou des inscriptions de groupe. Les procédures recommandées diffèrent également selon que vous renouvelez des certificats en raison d’une violation de la sécurité ou de l’expiration des certificats. Les sections ci-après expliquent comment gérer ces mises à jour.


## <a name="individual-enrollments-and-security-breaches"></a>Inscriptions individuelles et violations de la sécurité

Si vous renouvelez des certificats suite à une violation de la sécurité, vous devez exécuter la procédure ci-après qui supprime immédiatement le certificat actuel :

1. Cliquez sur **Inscriptions individuelles**, puis cliquez sur l’entrée d’ID d’inscription dans la liste. 

2. Cliquez sur le bouton **Delete current certificate** (Supprimer le certificat actuel), puis cliquez sur l’icône de dossier afin de sélectionner le nouveau certificat à charger pour l’entrée d’inscription. Lorsque vous avez terminé, cliquez sur **Enregistrer**.

    Exécutez cette procédure à la fois pour le certificat principal et pour le certificat secondaire s’ils sont tous deux compromis.

    ![Gérer les inscriptions individuelles avec une violation de la sécurité](./media/how-to-roll-certificates/manage-individual-enrollments-portal.png)

3. Une fois que le certificat compromis a été supprimé du service de provisionnement, le certificat peut néanmoins toujours être utilisé pour établir des connexions d’appareils au hub IoT tant qu’une inscription d’appareil y existe pour celui-ci. Vous pouvez résoudre ce problème de deux façons : 

    La première méthode consiste à accéder manuellement à votre hub IoT et à supprimer immédiatement l’inscription d’appareil associée au certificat compromis. Ensuite, quand l’appareil est reprovisionné avec un certificat mis à jour, une nouvelle inscription d’appareil est créée.     

    ![Suppression d’une inscription d’appareil IoT Hub](./media/how-to-roll-certificates/remove-hub-device-registration.png)

    La deuxième méthode consiste à utiliser la prise en charge du reprovisionnement pour reprovisionner l’appareil auprès du même hub IoT. Cette approche peut être utilisée pour remplacer le certificat pour l’inscription d’appareil sur le hub IoT. Pour plus d’informations, consultez [Guide pratique pour reprovisionner des appareils](how-to-reprovision.md).

## <a name="individual-enrollments-and-certificate-expiration"></a>Inscriptions individuelles et expiration des certificats

Si vous renouvelez des certificats afin de gérer les expirations de certificats, vous devez utiliser la configuration de certificat secondaire comme suit pour minimiser le temps d’arrêt lié à la tentative de provisionnement des appareils.

Par la suite, lorsque le certificat secondaire approchera également de sa date d’expiration et devra être renouvelé, vous pourrez basculer vers la configuration principale. Cette alternance entre le certificat principal et le certificat secondaire réduit le temps d’arrêt découlant de la tentative de provisionnement des appareils.


1. Cliquez sur **Inscriptions individuelles**, puis cliquez sur l’entrée d’ID d’inscription dans la liste. 

2. Cliquez sur **Certificat secondaire**, puis cliquez sur l’icône de dossier afin de sélectionner le nouveau certificat à charger pour l’entrée d’inscription. Cliquez sur **Enregistrer**.

    ![Gestion des inscriptions individuelles à l’aide de l’expiration du certificat secondaire](./media/how-to-roll-certificates/manage-individual-enrollments-secondary-portal.png)

3. Par la suite, une fois que le certificat principal a expiré, revenez et supprimez ce certificat en cliquant sur le bouton **Delete current certificate** (Supprimer le certificat actuel).

## <a name="enrollment-groups-and-security-breaches"></a>Groupes d’inscription et violations de la sécurité

Pour mettre à jour une inscription de groupe suite à une violation de la sécurité, vous devez appliquer l’une des approches ci-après afin de supprimer immédiatement l’autorité de certification racine actuelle ou le certificat intermédiaire.

#### <a name="update-compromised-root-ca-certificates"></a>Mettre à jour les certificats d’autorité de certification racine compromis

1. Cliquez sur l’onglet **Certificats** pour votre instance de service de provisionnement des appareils.

2. Cliquez sur le certificat compromis dans la liste, puis cliquez sur le bouton **Supprimer**. Confirmez la suppression en entrant le nom du certificat, puis cliquez sur **OK**. Répétez ce processus pour tous les certificats compromis.

    ![Suppression du certificat d’autorité de certification racine](./media/how-to-roll-certificates/delete-root-cert.png)

3. Exécutez la procédure [Configurer des certificats d’autorité de certification vérifiés](how-to-verify-certificates.md) pour ajouter et vérifier de nouveaux certificats d’autorité de certification racine.

4. Cliquez sur l’onglet **Gérer les inscriptions** pour votre instance de service de provisionnement des appareils, puis cliquez sur la liste **Groupes d’inscription**. Cliquez sur le nom de votre groupe d’inscription dans la liste.

5. Cliquez sur **Certificat d’autorité de certification**, puis sélectionnez votre nouveau certificat d’autorité de certification racine. Ensuite, cliquez sur **Enregistrer**. 

    ![Sélectionner le nouveau certificat d’autorité de certification racine pour un certificat compromis](./media/how-to-roll-certificates/select-new-root-cert.png)

6. Une fois que le certificat compromis a été supprimé du service de provisionnement, le certificat peut néanmoins toujours être utilisé pour établir des connexions d’appareils au hub IoT tant que des inscriptions d’appareil y existent pour celui-ci. Vous pouvez résoudre ce problème de deux façons : 

    La première méthode consiste à accéder manuellement à votre hub IoT et à supprimer immédiatement l’inscription d’appareil associée au certificat compromis. Ensuite, quand vos appareils sont reprovisionnés avec des certificats mis à jour, une nouvelle inscription d’appareil est créée pour chacun d’eux.     

    ![Suppression d’une inscription d’appareil IoT Hub](./media/how-to-roll-certificates/remove-hub-device-registration.png)

    La deuxième méthode consiste à utiliser la prise en charge du reprovisionnement pour reprovisionner vos appareils auprès du même hub IoT. Cette approche peut être utilisée pour remplacer les certificats pour les inscriptions d’appareil sur le hub IoT. Pour plus d’informations, consultez [Guide pratique pour reprovisionner des appareils](how-to-reprovision.md).



#### <a name="update-compromised-intermediate-certificates"></a>Mettre à jour les certificats intermédiaires compromis

1. Cliquez sur **Groupes d’inscription**, puis cliquez sur le nom du groupe dans la liste. 

2. Cliquez sur **Intermediate Certificate** (Certificat intermédiaire), puis sur **Delete current certificate** (Supprimer le certificat actuel). Cliquez sur l’icône de dossier afin d’accéder au nouveau certificat intermédiaire à charger pour le groupe d’inscription. Lorsque vous avez terminé, cliquez sur **Enregistrer**. Exécutez cette procédure à la fois pour le certificat principal et pour le certificat secondaire s’ils sont tous deux compromis.

    Ce nouveau certificat intermédiaire doit être signé par un certificat d’autorité de certification racine vérifié qui a déjà été ajouté au service de provisionnement. Pour plus d’informations, consultez la section [Certificats X.509](concepts-x509-attestation.md#x509-certificates).

    ![Gérer les inscriptions individuelles pour un intermédiaire compromis](./media/how-to-roll-certificates/enrollment-group-delete-intermediate-cert.png)


3. Une fois que le certificat compromis a été supprimé du service de provisionnement, le certificat peut néanmoins toujours être utilisé pour établir des connexions d’appareils au hub IoT tant que des inscriptions d’appareil y existent pour celui-ci. Vous pouvez résoudre ce problème de deux façons : 

    La première méthode consiste à accéder manuellement à votre hub IoT et à supprimer immédiatement l’inscription d’appareil associée au certificat compromis. Ensuite, quand vos appareils sont reprovisionnés avec des certificats mis à jour, une nouvelle inscription d’appareil est créée pour chacun d’eux.     

    ![Suppression d’une inscription d’appareil IoT Hub](./media/how-to-roll-certificates/remove-hub-device-registration.png)

    La deuxième méthode consiste à utiliser la prise en charge du reprovisionnement pour reprovisionner vos appareils auprès du même hub IoT. Cette approche peut être utilisée pour remplacer les certificats pour les inscriptions d’appareil sur le hub IoT. Pour plus d’informations, consultez [Guide pratique pour reprovisionner des appareils](how-to-reprovision.md).


## <a name="enrollment-groups-and-certificate-expiration"></a>Groupes d’inscription et expiration des certificats

Si vous renouvelez des certificats afin de gérer les expirations de certificats, vous devez utiliser la configuration de certificat secondaire comme suit pour éviter tout risque de temps d’arrêt lié à la tentative de provisionnement des appareils.

Par la suite, lorsque le certificat secondaire approchera également de sa date d’expiration et devra être renouvelé, vous pourrez basculer vers la configuration principale. Cette alternance entre le certificat principal et le certificat secondaire garantit l’absence de temps d’arrêt découlant de la tentative de provisionnement des appareils. 

#### <a name="update-expiring-root-ca-certificates"></a>Mettre à jour les certificats d’autorité de certification racine arrivant à expiration

1. Exécutez la procédure [Configurer des certificats d’autorité de certification vérifiés](how-to-verify-certificates.md) pour ajouter et vérifier de nouveaux certificats d’autorité de certification racine.

2. Cliquez sur l’onglet **Gérer les inscriptions** pour votre instance de service de provisionnement des appareils, puis cliquez sur la liste **Groupes d’inscription**. Cliquez sur le nom de votre groupe d’inscription dans la liste.

3. Cliquez sur **Certificat d’autorité de certification**, puis sélectionnez votre nouveau certificat d’autorité de certification racine sous la configuration **Certificat secondaire**. Ensuite, cliquez sur **Enregistrer**. 

    ![Sélectionner le nouveau certificat d’autorité de certification racine pour l’expiration](./media/how-to-roll-certificates/select-new-root-secondary-cert.png)

4. Par la suite, lorsque le certificat principal arrivera à expiration, cliquez sur l’onglet **Certificats** pour votre instance de service de provisionnement des appareils. Cliquez sur le certificat expiré dans la liste, puis cliquez sur le bouton **Supprimer**. Confirmez la suppression en entrant le nom du certificat, puis cliquez sur **OK**.

    ![Suppression du certificat d’autorité de certification racine](./media/how-to-roll-certificates/delete-root-cert.png)



#### <a name="update-expiring-intermediate-certificates"></a>Mettre à jour les certificats intermédiaires arrivant à expiration


1. Cliquez sur **Groupes d’inscription**, puis cliquez sur le nom du groupe dans la liste. 

2. Cliquez sur **Certificat secondaire**, puis cliquez sur l’icône de dossier afin de sélectionner le nouveau certificat à charger pour l’entrée d’inscription. Cliquez sur **Enregistrer**.

    Ce nouveau certificat intermédiaire doit être signé par un certificat d’autorité de certification racine vérifié qui a déjà été ajouté au service de provisionnement. Pour plus d’informations, consultez la section [Certificats X.509](concepts-x509-attestation.md#x509-certificates).

   ![Gérer les groupes d’inscription à l’aide du certificat secondaire arrivant à expiration](./media/how-to-roll-certificates/manage-enrollment-group-secondary-portal.png)

3. Par la suite, une fois que le certificat principal a expiré, revenez et supprimez ce certificat en cliquant sur le bouton **Delete current certificate** (Supprimer le certificat actuel).


## <a name="reprovision-the-device"></a>Reprovisionner l’appareil

Une fois que le certificat a été renouvelé à la fois sur l’appareil et dans le service Device Provisioning, l’appareil peut se reprovisionner lui-même en contactant le service de provisionnement des appareils. 

Une méthode simple de configuration du reprovisionnement des appareils consiste à programmer ces derniers pour qu’ils contactent le service de provisionnement en vue de cheminer par le flux de provisionnement si l’appareil reçoit une erreur de type « non autorisé » lors d’une tentative de connexion à l’IoT Hub.

Une autre méthode permet à l’ancien certificat et au nouveau certificat d’être tous deux valides sur une courte période, et consiste à utiliser l’IoT Hub pour envoyer une commande aux appareils afin que ces derniers se réinscrivent par le biais du service de provisionnement pour mettre à jour leurs informations de connexion IoT Hub. Étant donné que chaque appareil peut traiter les commandes de manière différente, vous devrez programmer votre appareil pour qu’il sache comment procéder lors de l’appel de cette commande. Vous disposez de plusieurs méthodes pour commander votre appareil par le biais de l’IoT Hub ; nous vous recommandons d’utiliser des [méthodes directes](../iot-hub/iot-hub-devguide-direct-methods.md) ou des [travaux](../iot-hub/iot-hub-devguide-jobs.md) pour amorcer le processus.

Une fois le reprovisionnement effectué, les appareils seront en mesure de se connecter à l’IoT Hub à l’aide de leurs nouveaux certificats.


## <a name="disallow-certificates"></a>Bloquer des certificats

En cas de violation de la sécurité, vous devrez peut-être bloquer un certificat d’appareil. Pour effectuer cette opération, désactivez l’entrée d’inscription du certificat ou de l’appareil cibles. Pour plus d’informations, consultez la section relative au blocage des appareils dans l’article [Gérer la désinscription](how-to-revoke-device-access-portal.md).

Une fois qu’un certificat est inclus dans une entrée d’inscription désactivée, toute tentative d’inscription auprès d’un IoT Hub à l’aide de ce certificat échouera, même si le certificat est activé dans une autre entrée d’inscription.
 



## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les certificats X.509 dans le service Device Provisioning, consultez [Attestation de certificat X.509](concepts-x509-attestation.md) 
- Pour savoir comment effectuer une preuve de possession pour les certificats d’autorité de certification X.509 avec le service Azure IoT Hub Device Provisioning, consultez la procédure [Configurer des certificats d’autorité de certification vérifiés](how-to-verify-certificates.md).
- Pour savoir comment utiliser le portail pour créer un groupe d’inscription, consultez [Gérer les inscriptions d’appareils avec le portail Azure](how-to-manage-enrollments.md).
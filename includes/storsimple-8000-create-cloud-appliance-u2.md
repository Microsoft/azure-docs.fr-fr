---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 8b827a80059cecadd7b76263f13dd4a0d141fc7a
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95562390"
---
#### <a name="to-create-a-cloud-appliance"></a>Pour créer une appliance cloud

1. Dans le portail Azure, accédez au service **StorSimple Device Manager**.
2. Accédez au panneau **Appareils**. Dans la barre de commandes du panneau Résumé de service, cliquez sur **Créer une appliance cloud**.
    ![StorSimple - Créer une appliance cloud](./media/storsimple-8000-create-cloud-appliance-u2/sca-create1.png)
3. Dans le panneau **Créer une appliance cloud**, spécifiez les détails suivants.
   
    ![StorSimple – Créer une appliance cloud 2](./media/storsimple-8000-create-cloud-appliance-u2/sca-create2m.png)
   
   1. **Nom** : nom unique de votre appliance cloud.
   2. **Modèle** : sélectionnez le modèle de l’appliance cloud. Un modèle d’appareil 8010 offre 30 To de stockage Standard, tandis qu’un modèle d’appareil 8020 offre 64 To de stockage Premium. Spécifiez 8010 pour déployer des scénarios de récupération au niveau de l’élément à partir de sauvegardes. Sélectionnez 8020 pour déployer des charges de travail aux performances élevées et à faible latence ou pour une utilisation comme appareil secondaire à des fins de récupération d’urgence.
   3. **Version** : sélectionnez la version de l’appliance cloud. La version correspond à la version de l’image de disque virtuel qui est utilisée pour créer l’application cloud. Comme la version de l’appliance cloud détermine l’appareil physique à partir duquel vous pouvez basculer ou cloner, il est important de créer une version appropriée de l’appliance cloud.
   4. **Réseau virtuel** : spécifiez un réseau virtuel que vous souhaitez utiliser avec cette appliance cloud. Si vous utilisez Stockage Premium, vous devez sélectionner un réseau virtuel qui est pris en charge avec le compte de stockage Premium. Les réseaux virtuels non pris en charge sont grisés dans la liste déroulante. Vous êtes averti si vous sélectionnez un réseau virtuel non pris en charge.
   5. **Sous-réseau** : basé sur le réseau virtuel sélectionné, la liste déroulante affiche les sous-réseaux associés. Attribuez un sous-réseau à votre appliance cloud.
   6. **Compte de stockage** : sélectionnez un compte de stockage qui contiendra l’image de l’appliance cloud lors de l’approvisionnement. Ce compte de stockage doit être dans la même région que l’appliance cloud et le réseau virtuel. Il ne doit pas être utilisé pour le stockage des données par l’appliance physique ou cloud. Par défaut, un compte de stockage est créé à cet effet. Toutefois, si vous avez déjà un compte de stockage qui convient pour cette utilisation, vous pouvez le sélectionner dans la liste. Si vous créez une appliance cloud Premium, la liste déroulante affiche uniquement les comptes de stockage Premium.
      
      > [!NOTE]
      > L’appliance cloud ne peut fonctionner qu’avec les comptes de stockage Azure.
    
   7. Cochez la case pour indiquer que vous savez que les données stockées sur l’appliance cloud sont hébergées dans un centre de données Microsoft.
       * Lorsque vous utilisez uniquement un appareil physique, votre clé de chiffrement est conservée avec celui-ci ; par conséquent, Microsoft ne peut pas le déchiffrer.

       * Lorsque vous utilisez une appliance cloud, la clé de chiffrement et la clé de déchiffrement sont stockées dans Microsoft Azure. Pour plus d’informations, consultez les [considérations de sécurité relatives à l’utilisation d’une appliance cloud](../articles/storsimple/storsimple-8000-security.md).
   8. Cliquez sur **Créer** pour approvisionner l’appliance cloud. La mise en service de l'appareil peut prendre environ 30 minutes. Un message s’affiche une fois l’appliance cloud créée. Accédez au panneau Appareils, et la liste des appareils est actualisée pour afficher l’appliance cloud. L’état de l’appliance est défini sur **Prêt pour la configuration**.
      
      ![Appliance cloud StorSimple - Prêt pour la configuration](./media/storsimple-8000-create-cloud-appliance-u2/sca-create3.png)
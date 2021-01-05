---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: c3f913aa3de1b723cd4eae70ff8e578a8abbec12
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95563766"
---
#### <a name="to-create-a-new-service"></a>Création d’un service
1. Utilisez les informations d’identification de votre compte Microsoft pour vous connecter au [portail Microsoft Azure Government](https://portal.azure.us/).
2. Cliquez sur **+** dans le portail d’administration, puis sur **Afficher tout** sur la place de marché. Recherchez _StorSimple physique_. Sélectionnez **Série d’appareils physiques StorSimple** et cliquez dessus, puis cliquez sur **Créer**. Vous pouvez également cliquer sur **+** dans le portail d’administration, puis sur **Série d’appareils physiques StorSimple** dans **Stockage**.
3. Dans le panneau **StorSimple Device Manager** qui s’ouvre, procédez comme suit :
   
   1. Fournissez un unique **Nom de la ressource** pour votre service. Il s’agit d’un nom convivial qui peut être utilisé pour identifier le service. Le nom peut comporter entre 2 et 50 caractères qui peuvent être des lettres, des chiffres et des traits d’union. Il doit commencer et se terminer par une lettre ou un chiffre.
   2. Choisissez un **abonnement** dans la liste déroulante. L’abonnement est lié à votre compte de facturation. Ce champ n’est pas présent si vous n’avez qu’un seul abonnement.
   3. Pour le **Groupe de ressources**, sélectionnez un groupe existant en cliquant sur **Utiliser existant** ou créez-en un avec l’option **Créer**. Pour plus d’informations, consultez la page [Groupes de ressources dans Azure](../articles/azure-resource-manager/management/manage-resource-groups-portal.md).
   4. Indiquez un **emplacement** pour votre service. L’emplacement fait référence à la région géographique dans laquelle vous souhaitez déployer votre appareil. Sélectionnez **USGov Iowa** ou **USGov Virginia**.
   5. Sélectionnez **Créer un nouveau compte de stockage** pour créer automatiquement un compte de stockage avec le service. Spécifiez un nom pour ce compte de stockage. Si vous avez besoin de vos données dans un autre emplacement, désactivez cette case à cocher.
   6. Cochez la case **Épingler au tableau de bord** si vous souhaitez disposer d’un lien rapide vers ce service sur votre tableau de bord.
   7. Pour créer le service StorSimple Device Manager, cliquez sur **Créer**. La création de service dure quelques minutes. Une fois le service correctement créé, une notification s’affiche et le nouveau panneau de service s’ouvre.
---
title: Déployer StorSimple Virtual Array pour le programme des fournisseurs de solutions cloud
description: Découvrez comment un partenaire CSP peut ajouter un client ou un abonnement à un client existant, puis créer un service pour déployer StorSimple Virtual Array dans CSP.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/08/2017
ms.author: alkohli
ms.openlocfilehash: dc0cf718ee51e23fb749bdf57d5344977de009d8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021858"
---
# <a name="deploy-storsimple-virtual-array-for-cloud-solution-provider-program"></a>Déployer StorSimple Virtual Array pour le programme des fournisseurs de solutions cloud

## <a name="overview"></a>Vue d’ensemble

StorSimple Virtual Array peut être déployé par les partenaires Fournisseurs de solutions cloud (CSP) pour leurs clients. Un partenaire CSP peut créer un service StorSimple Device Manager. Ce service peut ensuite être utilisé pour déployer et gérer StorSimple Virtual Array et les partages, volumes et sauvegardes associés.

Cet article décrit comment un partenaire CSP peut ajouter un client ou un nouvel abonnement à un client existant, puis créer un service pour déployer StorSimple Virtual Array dans CSP.

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer, assurez-vous de satisfaire les exigences suivantes :

- Vous êtes inscrit dans le cadre du programme CSP.
- Vous avez des informations d’identification valides pour [l’Espace Partenaires](https://partnercenter.microsoft.com/). Les informations d’identification vous permettent de vous connecter au portail Partenaire pour ajouter de nouveaux clients, rechercher des clients ou accéder à un compte client à partir du tableau de bord Partenaire. Le CSP peut fonctionner en tant qu’administrateur StorSimple au nom du client dans le portail Azure.
                             
## <a name="add-a-customer"></a>Ajouter un client

Si vous ajoutez un client, un abonnement est automatiquement créé. Pour ajouter un client (et créer automatiquement un abonnement), effectuez les opérations suivantes dans le portail Partenaire.

1. Accédez à [l’Espace Partenaires](https://partnercenter.microsoft.com/) et connectez-vous en utilisant vos informations d’identification CSP. Cliquez sur **Dashboard**.

     ![Tableau de bord dans l’Espace Partenaires](./media/storsimple-partner-csp-deploy/image1.png)
                              
2. Dans le panneau gauche, cliquez sur **Clients**. Dans le panneau droit, cliquez sur **Ajouter des clients**. Entrez les détails du client. Cliquez sur **Suivant : Abonnements** pour créer un abonnement client.

    ![Ajouter un client](./media/storsimple-partner-csp-deploy/image2.png)

3.  Sélectionnez l’offre **Microsoft Azure**. Faites défiler vers le bas de la page et cliquez sur **Révision**.

    ![Revoir les informations d’abonnement](./media/storsimple-partner-csp-deploy/image3.png)
                              
4. Révisez les informations et cliquez sur **Envoyer**.

    ![Envoyer l’abonnement](./media/storsimple-partner-csp-deploy/image4.png)

5. Enregistrez les informations de confirmation pour référence ultérieure.

    ![Enregistrer la confirmation](./media/storsimple-partner-csp-deploy/image5.png)

6. Recherchez le client que vous venez d’ajouter ou accédez à celui-ci. Cliquez sur le **nom de la société** pour explorer les détails.

    ![Rechercher le client](./media/storsimple-partner-csp-deploy/image6.png)  

7. Dans le panneau gauche, sélectionnez **Gestion des services**. Dans le panneau droit, sous **Administrer des services**, cliquez sur **Portail de gestion Microsoft Azure** pour vous connecter en tant qu’administrateur Azure pour votre client.

    ![Connexion au portail Azure](./media/storsimple-partner-csp-deploy/image9.png)

8. Pour créer une instance StorSimple Device Manager, cliquez sur **+ Nouveau** et recherchez ou accédez à **Série d’appareils virtuels StorSimple**. Pour plus d’informations, accédez à la page [Déployer un service StorSimple Device Manager](storsimple-virtual-array-manage-service.md).

    ![Créer un service StorSimple Device Manager](./media/storsimple-partner-csp-deploy/image8.png)


## <a name="add-a-subscription"></a>Ajouter un abonnement

Dans certains cas, vous avez peut-être un client existant et vous devez ajouter un abonnement. Pour ajouter un abonnement à un client existant, effectuez les opérations suivantes dans le portail Partenaire.

1. Accédez à [l’Espace Partenaires](https://partnercenter.microsoft.com/) et connectez-vous en utilisant vos informations d’identification CSP. Cliquez sur **Dashboard**.

     ![Tableau de bord dans l’Espace Partenaires](./media/storsimple-partner-csp-deploy/image1.png)
                              
2. Dans le panneau gauche, cliquez sur **Clients**. Recherchez le client auquel vous souhaitez ajouter un abonnement ou accédez à celui-ci. Cliquez sur l’icône de ![développement de l’icône cochée](./media/storsimple-partner-csp-deploy/expand_pane_icon.png) pour développer la ligne contenant le nom de la société de votre client. Dans les détails, cliquez sur **Ajouter des abonnements**.

    ![Clients](./media/storsimple-partner-csp-deploy/image10.png)

3. Sélectionnez **Microsoft Azure** sous **Offres principales** dans l’abonnement et cliquez sur **Envoyer**. Ceci crée un nouvel abonnement.

    ![Ajouter un nouvel abonnement](./media/storsimple-partner-csp-deploy/image11.png)

6. Après la création d’un abonnement, cliquez sur **<-- Clients** dans le panneau gauche pour revenir à la page **Clients**. Recherchez le client pour lequel vous venez de créer un abonnement. Cliquez sur le **nom de la société** pour explorer les détails.

    ![Rechercher le client](./media/storsimple-partner-csp-deploy/image6.png)  

7. Dans le panneau gauche, sélectionnez **Gestion des services**. Dans le panneau droit, sous **Administrer des services**, cliquez sur **Portail de gestion Microsoft Azure** pour vous connecter en tant qu’administrateur Azure pour votre client.

    ![Connexion au portail Azure](./media/storsimple-partner-csp-deploy/image9.png)

8. Pour créer une instance StorSimple Device Manager, cliquez sur **+ Nouveau** et recherchez ou accédez à **Série d’appareils virtuels StorSimple**. Pour plus d’informations, accédez à la page [Déployer un service StorSimple Device Manager](storsimple-virtual-array-manage-service.md).

    ![Créer un service StorSimple Device Manager](./media/storsimple-partner-csp-deploy/image8.png)

## <a name="next-steps"></a>Étapes suivantes

- Si vous avez des questions supplémentaires concernant StorSimple dans le programme des fournisseurs de solutions cloud, accédez à [StorSimple in CSP: Frequently asked questions](storsimple-partner-csp-faq.md) (StorSimple dans le programme des fournisseurs de solutions cloud : Forum Aux Questions).
- Si vous êtes prêt à déployer votre instance StorSimple, accédez à [Deploy StorSimple Virtual Array for Cloud Solution Provider Program](storsimple-partner-csp-deploy.md) (Déployer StorSimple Virtual Array pour le programme des fournisseurs de solutions cloud).

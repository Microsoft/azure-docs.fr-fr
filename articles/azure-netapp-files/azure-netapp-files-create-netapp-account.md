---
title: Créer un compte NetApp pour accéder à Azure NetApp Files | Microsoft Docs
description: Ce document décrit comment accéder à Azure NetApp Files et créer un compte NetApp afin de pouvoir configurer un pool de capacité et créer un volume.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/28/2018
ms.author: b-juche
ms.openlocfilehash: e5e0c43aa8c4fbf0db89065e0fab93ee174a4ede
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006591"
---
# <a name="create-a-netapp-account"></a>Créer un compte NetApp
La création d’un compte NetApp vous permet de configurer un pool de capacité et par la suite de créer un volume. Le panneau Azure NetApp Files permet de créer un nouveau compte NetApp.

## <a name="before-you-begin"></a>Avant de commencer
Vous devez avoir reçu un e-mail de l’équipe Azure NetApp Files confirmant que vous avez désormais accès au service. Consultez [soumettre une demande de liste d’attente pour l’accès au service](azure-netapp-files-register.md#waitlist).

Vous devez également avoir inscrit votre abonnement pour l’utilisation du fournisseur de ressources NetApp. Consultez [Inscrire le fournisseur de ressources NetApp](azure-netapp-files-register.md#resource-provider).

## <a name="steps"></a>Étapes 

1. Connectez-vous au portail Azure. 
2. Accédez à Azure NetApp Files à l’aide d’une des méthodes suivantes :  
   * Recherchez **Azure NetApp Files** dans la zone de recherche du portail Azure.  
   * Cliquez sur **Tous les services** dans la navigation, puis filtrez sur Azure NetApp Files.  

   Vous pouvez placer le panneau Azure NetApp Files dans vos Favoris en cliquant sur l’icône représentant une étoile en regard de celui-ci. 

3. Cliquez sur **+Ajouter** pour créer un nouveau compte NetApp.  
   La nouvelle fenêtre de compte NetApp s’affiche.  

4. Fournissez les informations suivantes pour votre compte NetApp : 
   * **Nom du compte**  
     Indiquez un nom unique pour l’abonnement.
   * **Abonnement**  
     Sélectionnez un abonnement à partir de vos abonnements existants.
   * **Groupe de ressources**   
     Utilisez un groupe de ressources existant ou créez-en un.
   * **Lieu**  
     Sélectionnez la région dans laquelle vous souhaitez que le compte et ses ressources enfants soient situés.  

     ![Nouveau compte NetApp](../media/azure-netapp-files/azure-netapp-files-new-netapp-account.png)


5. Cliquez sur **Créer**.     
   Le compte NetApp que vous avez créé apparaît désormais dans le panneau Azure NetApp Files. 

> [!NOTE] 
> Si vous n’avez pas été autorisé à accéder au service Azure NetApp Files, vous recevrez l’erreur suivante lorsque vous essaierez de créer le premier compte NetApp :  
>
> `{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"NotFound","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidResourceType\",\r\n \"message\": \"The resource type could not be found in the namespace 'Microsoft.NetApp' for api version '2017-08-15'.\"\r\n }\r\n}"}]}`

## <a name="next-steps"></a>Étapes suivantes  

[Configurer un pool de capacité](azure-netapp-files-set-up-capacity-pool.md)


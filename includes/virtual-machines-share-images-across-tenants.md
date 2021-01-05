---
title: Fichier Include
description: Fichier Include
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 04/25/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: f74d4cbc17e49345534a37e9e6612a36e19be295
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73903701"
---
Toutefois, si vous souhaitez partager à grande échelle des images hors de votre locataire Azure, vous devez créer une inscription d’application pour faciliter le partage.  À l’aide d’une inscription d’application, vous pouvez activer des scénarios de partage plus complexes, comme : 

* la gestion d’images partagées lorsqu’une entreprise en acquiert une autre, ou la répartition de l’infrastructure Azure entre plusieurs locataires distincts. 
* Les partenaires Azure gèrent l’infrastructure Azure pour le compte de leurs clients. Les images sont personnalisées au sein du locataire des partenaires, mais les déploiements d’infrastructure s’effectuent dans celui du client. 


## <a name="create-the-app-registration"></a>Créer l’inscription d’application

Créez une inscription d’application qui sera utilisée par les deux locataires pour partager les ressources des galeries d’images.
1. Accédez à la fonction [Inscriptions d’applications (préversion) dans le portail Microsoft Azure](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType//sourceType/).    
1. Sélectionnez l’option **Nouvelle inscription** dans le menu situé en haut de la page.
1. Dans **Nom**, saisissez *myGalleryApp*.
1. Dans **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
1. Dans **URI de redirection**, saisissez *https://www.microsoft.com* et sélectionnez **Inscrire**. Une fois l’inscription d’application créée, la page Vue d’ensemble s’ouvre.
1. Sur cette page, copiez **l’ID d’application (client)** et enregistrez-le pour une utilisation ultérieure.   
1. Sélectionnez **Certificats et secrets**, puis sélectionnez **Nouveau secret client**.
1. Dans **Description**, saisissez *Secret d’application proposé entre plusieurs locataires dans la galerie d’images partagées*.
1. Dans **Expires**, conservez la valeur par défaut, **Dans 1 n** et sélectionnez **Ajouter**.
1. Copiez la valeur du secret et placez-la dans un endroit sûr. Vous ne pouvez pas la récupérer une fois que vous avez quitté cette page.


Fournissez l’autorisation de l’inscription d’application pour utiliser la galerie d’images partagées.
1. Dans le portail Azure, sélectionnez la galerie Shared Image Gallery que vous souhaitez partager avec un autre locataire.
1. Sélectionnez **Contrôle d’accès (IAM)** et, sous **Ajouter une attribution de rôle**, sélectionnez *Ajouter*. 
1. Sous **Rôle**, sélectionnez **Lecteur**.
1. Sous **Attribuer l’accès à**, laissez la valeur **Utilisateur, groupe ou principal du service Azure AD**.
1. Sous **Sélectionner**, saisissez *myGalleryApp* et sélectionnez cet élément lorsqu’il s’affiche dans la liste. Lorsque vous avez terminé, sélectionnez **Enregistrer**.


## <a name="give-tenant-2-access"></a>Accorder l’accès au locataire 2

Accordez l’accès au locataire 2 à l’application, en demandant la connexion via un navigateur. Remplacez *\<Tenant2 ID>* par l’ID du locataire avec lequel que vous voulez partager votre galerie d’images. Remplacez *\<Application (client) ID>* par l’ID d’application de l’inscription de l’application que vous avez créée. Lorsque vous avez effectué ces remplacements, collez l’adresse URL dans un navigateur et suivez les invites pour vous connecter au locataire 2.

```
https://login.microsoftonline.com/<Tenant 2 ID>/oauth2/authorize?client_id=<Application (client) ID>&response_type=code&redirect_uri=https%3A%2F%2Fwww.microsoft.com%2F 
```

Dans le [portail Microsoft Azure](https://portal.azure.com), connectez-vous en tant que locataire 2 et accordez à l’inscription d’application un accès au groupe de ressources dans lequel vous souhaitez créer la machine virtuelle.

1. Sélectionnez le groupe de ressources, puis **Contrôle d’accès (IAM)** . Sous **Ajouter une attribution de rôle**, sélectionnez **Ajouter**. 
1. Sous **Rôle**, saisissez **Contributeur**.
1. Sous **Attribuer l’accès à**, laissez la valeur **Utilisateur, groupe ou principal du service Azure AD**.
1. Sous **Sélectionner**, saisissez *myGalleryApp*, puis sélectionnez cet élément lorsqu’il s’affiche dans la liste. Lorsque vous avez terminé, sélectionnez **Enregistrer**.

> [!NOTE]
> Vous devez attendre que la version d’image soit totalement intégrée et répliquée avant de pouvoir utiliser la même image managée pour créer une autre version d’image.


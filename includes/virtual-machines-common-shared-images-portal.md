---
title: Fichier include
description: Fichier include
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 11/06/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 1af80f208c72af9434d596f2c1219c08c0e3f719
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015987"
---
## <a name="create-an-image-gallery"></a>Créer une galerie d’images

Une galerie d’images est la ressource principale utilisée pour activer le partage d’image. Les caractères autorisés pour le nom de galerie sont les lettres majuscules ou minuscules, les chiffres et les points. Le nom de galerie ne peut pas contenir de tirets.  Les noms de galerie doivent être uniques dans votre abonnement. 

L’exemple suivant crée une galerie nommée *myGallery* dans le groupe de ressources *myGalleryRG*.

1. Connectez-vous au portail Azure sur https://portal.azure.com.
1. Utilisez le type **Galerie d’images partagées** dans la zone de recherche, puis sélectionnez **Galerie d’images partagées** dans les résultats.
1. Sur la page **Galerie d’images partagées**, cliquez sur **Ajouter**.
1. Dans la page **Créer une galerie Shared Image Gallery**, sélectionnez l’abonnement approprié.
1. Dans **Groupe de ressources**, sélectionnez **Créer** et saisissez *myResourceGroup* comme nom.
1. Dans **Nom**, saisissez *myGallery* comme nom de la galerie.
1. Laissez la valeur par défaut pour **Région**.
1. Vous pouvez saisir une brève description de la galerie, par exemple *Ma galerie d’images pour le test.* Cliquez ensuite sur **Vérifier + créer**.
1. Une fois la validation réussie, sélectionnez **Créer**.
1. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.


## <a name="create-an-image-definition"></a>Créer une définition d’image 

Les définitions d’image créent un regroupement logique des images. Elles sont utilisées pour gérer les informations sur les versions d’image créées au sein de celles-ci. Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](../articles/virtual-machines/windows/shared-image-galleries.md#image-definitions).

Créez la définition de l’image de galerie à l’intérieur de votre galerie. Dans cet exemple, l’image de galerie est nommée *myImageDefinition*.

1. Sur la page de votre nouvelle galerie d’images, sélectionnez **Ajouter une nouvelle définition d’image** à partir du haut de la page. 
1. Sous **Ajouter une nouvelle définition d’image à la galerie Shared Image Gallery**, pour **Région**, sélectionnez *USA Est*.
1. Pour **Nom de la définition d’image**, saisissez *myImageDefinition*.
1. Pour **Système d’exploitation**, sélectionnez l’option appropriée en fonction de votre machine virtuelle source.  
1. Pour **Génération de machine virtuelle**, sélectionnez l'option en fonction de votre machine virtuelle source. Dans la plupart des cas, ce sera *Gen 1*. Pour plus d'informations, voir [Prise en charge des machines virtuelles de 2e génération](../articles/virtual-machines/generation-2.md).
1. Pour **État du système d’exploitation**, sélectionnez l’option appropriée en fonction de votre machine virtuelle source. Pour plus d’informations, voir [Machines virtuelles généralisées et spécialisées](../articles/virtual-machines/linux/shared-image-galleries.md#generalized-and-specialized-images).
1. Pour **Éditeur**, saisissez *myPublisher*. 
1. Pour **Offre**, saisissez *myOffer*.
1. Pour **SKU**, saisissez *mySKU*.
1. Quand vous avez terminé, sélectionnez **Vérifier + créer**.
1. Une fois la définition de l’image validée, sélectionnez **Créer**.
1. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.


## <a name="create-an-image-version"></a>Créer une version d’image

Créer une version de l’image à partir d’une image managée. Dans cet exemple, la version d'image, *1.0.0*, elle est répliquée dans les deux centres de données *USA Centre-Ouest* et *USA Centre Sud*. Lors du choix des régions cibles pour la réplication, n’oubliez pas que vous devez également inclure la région *source* en tant que cible pour la réplication.

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Les étapes de création d'une version d'image sont légèrement différentes, selon que la source est une image généralisée ou une capture instantanée d'une machine virtuelle spécialisée. 

### <a name="option-generalized"></a>Option : Généralisée

1. Sur la page de votre définition d’image, sélectionnez **Ajouter une version** à partir du haut de la page.
1. Dans **Région**, sélectionnez la région où est stockée votre image managée. Les versions d’images doivent être créées dans la même région que l’image managée à partir de laquelle elles sont créées.
1. Pour **Nom**, saisissez *1.0.0*. Le nom de version d’image doit respecter le format *version majeure*.*version mineure*.*correctif* en utilisant des nombres entiers. 
1. Dans **Image source**, sélectionnez votre image managée source à partir de la liste déroulante.
1. Dans **Exclure de la plus récente**, conservez la valeur par défaut *Non*.
1. Pour **Date de fin de vie**, sélectionnez une date dans le calendrier dans quelques mois.
1. Dans **Réplication**, laissez le **Nombre de réplicas par défaut** sur 1. Vous devez répliquer vers la région source, donc laissez le premier réplica à la valeur par défaut et choisissez comme deuxième région de réplica *USA Est*.
1. Quand vous avez terminé, sélectionnez **Vérifier + créer**. Azure validera la configuration.
1. Une fois la définition d’image validée, sélectionnez **Créer**.
1. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.

La réplication de l’image à l’ensemble des régions cibles peut prendre un certain temps.

### <a name="option-specialized"></a>Option : Spécialisée

1. Sur la page de votre définition d’image, sélectionnez **Ajouter une version** à partir du haut de la page.
1. Dans **Région**, sélectionnez la région où est stockée votre capture instantanée. Les versions d’images doivent être créées dans la même région que la source à partir de laquelle elles sont créées.
1. Pour **Nom**, saisissez *1.0.0*. Le nom de version d’image doit respecter le format *version majeure*.*version mineure*.*correctif* en utilisant des nombres entiers. 
1. Dans **Capture instantanée de disque de système d’exploitation**, sélectionnez la capture instantanée de votre machine virtuelle source dans le menu déroulant. Si votre machine virtuelle source possède un disque de données que vous souhaitez inclure, sélectionnez le bon numéro **LUN** dans le menu déroulant, puis choisissez la capture instantanée du disque de données pour **Capture instantanée du disque de données**. 
1. Dans **Exclure de la plus récente**, conservez la valeur par défaut *Non*.
1. Pour **Date de fin de vie**, sélectionnez une date dans le calendrier dans quelques mois.
1. Dans **Réplication**, laissez le **Nombre de réplicas par défaut** sur 1. Vous devez répliquer vers la région source, donc laissez le premier réplica à la valeur par défaut et choisissez comme deuxième région de réplica *USA Est*.
1. Quand vous avez terminé, sélectionnez **Vérifier + créer**. Azure validera la configuration.
1. Une fois la définition d’image validée, sélectionnez **Créer**.
1. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.

## <a name="share-the-gallery"></a>Partager la galerie

Nous vous recommandons de partager l’accès au niveau de la galerie d’image. Les procédures suivantes vous guident dans les étapes de partage de la galerie que vous venez de créer.

1. Ouvrez le [portail Azure](https://portal.azure.com).
1. Dans le menu de gauche, sélectionnez **Groupes de ressources**. 
1. Dans la liste des groupes de ressources, sélectionnez **myGalleryRG**. Le panneau de votre groupe de ressources s’ouvre.
1. Dans le menu à gauche de la page **myGalleryRG**, sélectionnez **Contrôle d’accès (IAM)** . 
1. Sous **Ajouter une attribution de rôle**, sélectionnez **Ajouter**. Le volet **Ajouter une attribution de rôle** s’ouvre. 
1. Sous **Rôle**, sélectionnez **Lecteur**.
1. Sous **Attribuer l’accès à**, laissez la valeur par défaut **Utilisateur, groupe ou principal du service Azure AD**.
1. Sous **Sélectionner**, saisissez l’adresse de messagerie de la personne que vous souhaitez inviter.
1. Si l’utilisateur se trouve en dehors de votre organisation, vous verrez le message **Cet utilisateur recevra un e-mail qui lui permettra de collaborer avec Microsoft.** Sélectionnez l’utilisateur avec son adresse e-mail, puis cliquez sur **Enregistrer**.

Si l’utilisateur se trouve en dehors de votre organisation, il recevra un message d’invitation à rejoindre l’organisation. L’utilisateur doit accepter l’invitation, puis il pourra voir la galerie et toutes les définitions et versions d’image versions dans sa liste de ressources.
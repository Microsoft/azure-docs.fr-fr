---
title: 'Démarrage rapide : Exporter des données pour Microsoft Azure Data Box'
description: Découvrez comment exporter rapidement vos données Azure Data Box dans le portail Azure
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: quickstart
ms.date: 07/17/2020
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 789bd2f62673e7faf562d3a407bf2f0a4fd861bf
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92125490"
---
# <a name="quickstart-get-started-with-azure-data-box-to-export-data-from-azure"></a>Démarrage rapide : Bien démarrer avec Azure Data Box pour exporter des données depuis Azure

Ce guide de démarrage rapide explique comment exporter des données Azure vers votre emplacement à l’aide du portail Azure. Vous y apprendrez notamment comment raccorder, configurer et copier des données depuis Azure. Le démarrage rapide est effectué dans le portail Azure et sur l’interface utilisateur web locale de l’appareil.

Pour obtenir des instructions pas à pas sur le déploiement et le suivi, consultez [Tutoriel : Créer une commande d’exportation pour Azure Data Box](data-box-deploy-export-ordered.md)

## <a name="prerequisites"></a>Prérequis

Avant de commencer :

* L’abonnement que vous utilisez pour le service Data Box est un des types suivants :
  * Contrat Entreprise (EA) Microsoft. En savoir plus sur les [abonnements EA](https://azure.microsoft.com/pricing/enterprise-agreement/).
  * Fournisseur de solutions cloud (CSP). En savoir plus sur le [programme Azure CSP](/azure/cloud-solution-provider/overview/azure-csp-overview).
  * Microsoft Azure Sponsorship. En savoir plus sur le [programme Azure Sponsorship](https://azure.microsoft.com/offers/ms-azr-0036p/).

* Vous disposez d’un accès propriétaire ou contributeur à l’abonnement pour créer une commande Data Box.
* Passez en revue les [consignes de sécurité pour votre Data Box](data-box-safety.md).
* Vous disposez d’un appareil Azure Data Box pour déplacer des données vers votre emplacement. Votre ordinateur hôte doit
  * Exécuter un [système d’exploitation pris en charge](data-box-system-requirements.md)
  * Être connecté à un réseau haut débit. Nous vous recommandons vivement d’utiliser au minimum une connexion 10 GbE. Si vous ne disposez pas d’une connexion 10 GbE, vous pouvez utiliser une liaison de données 1 GbE. Cependant, cela a une incidence sur les vitesses de copie.
* Vous devez avoir accès à une surface plane sur laquelle placer la Data Box. Si vous souhaitez placer l’appareil sur un rack standard, vous avez besoin d’un emplacement 7U dans le rack de votre centre de données. Vous pouvez placer l’appareil à plat ou en position verticale dans le rack.
* Vous disposez des câbles suivants pour connecter votre Data Box à l’ordinateur hôte.
  * Deux câbles en cuivre Twinax SFP+ 10 GbE (à utiliser avec les interfaces réseau DATA 1 et DATA 2)
  * Un câble réseau RJ-45 CAT 6 (à utiliser avec l’interface réseau de gestion (MGMT))
  * Un câble réseau RJ-45 CAT 6A OU RJ-45 CAT 6 (à utiliser avec l’interface réseau DATA 3 et configurés en mode 10 Gbit/s ou 1 Gbit/s, respectivement)

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="order"></a>JSON

Cette étape prend environ 10 minutes.

1. Créez une ressource Azure Data Box dans le portail Azure.
2. Sélectionnez un abonnement existant activé pour ce service et choisissez le type de transfert **Exportation vers Azure**. Indiquez la **région Azure source** où résident les données et le **pays de destination** pour le transfert des données.
3. Sélectionnez **Data Box**. La capacité utile maximale de la solution est de 80 To, et vous pouvez créer plusieurs commandes pour des tailles de données supérieures.
4. Sélectionnez **Ajouter un compte de stockage et un type d’exportation**, puis **Sélectionner l’option d’exportation**.
5. Entrez les détails de la commande et les informations d’expédition. Si le service est disponible dans votre région, fournissez les adresses e-mail de notification, vérifiez le résumé, puis créez la commande.

Une fois la commande créée, l’appareil est préparé pour l’expédition.

## <a name="cable"></a>Câble

Cette étape prend environ 10 minutes.

Lorsque vous recevez la Data Box, procédez comme suit pour brancher les câbles, vous connecter et activer l’appareil. Cette étape prend environ 10 minutes.

1. S’il existe des preuves que l’appareil a été trafiqué ou endommagé, arrêtez-vous. Contactez le support Microsoft pour demander un appareil de remplacement.
2. Avant de brancher votre appareil, assurez-vous d’avoir les câbles suivants :

   * Un cordon d’alimentation relié à la terre (inclus) de 10 A ou plus, muni d’un connecteur IEC60320 C-13 à une extrémité pour se connecter à l’appareil.
   * Un câble réseau RJ-45 CAT 6 (à utiliser avec l’interface réseau de gestion (MGMT))
   * Deux câbles en cuivre Twinax SFP+ 10 GbE (à utiliser avec les interfaces réseau 10 Gbit/s DATA 1 et DATA 2)
   * Un câble réseau RJ-45 CAT 6A OU RJ-45 CAT 6 (à utiliser avec l’interface réseau DATA 3 et configurés en mode 10 Gbit/s ou 1 Gbit/s, respectivement)

3. Retirez et placez l’appareil sur une surface plane.

4. Branchez l’appareil comme indiqué ci-dessous.  

    ![Vue arrière d’une Data Box raccordée](media/data-box-deploy-set-up/data-box-cabled-dhcp.png)  

    1. Connectez le câble d'alimentation à l'appareil.
    2. Utilisez le câble réseau RJ-45 CAT 6 pour connecter votre ordinateur hôte au port de MGMT sur l'appareil.
    3. Utilisez le câble en cuivre Twinax SFP+ pour connecter au moins une interface réseau de 10 Gbit/s (de préférence supérieure à 1 Gbit/s), DATA 1 ou DATA 2 pour les données.
    4. Allumez l'appareil. Le bouton d’alimentation est situé sur le panneau avant de l’appareil.

## <a name="connect"></a>Se connecter

Cette étape prend environ 5 à 7 minutes.

1. Pour obtenir le mot de passe de l’appareil, accédez à **Général** > **Détails sur l’appareil** dans le [portail Azure](https://portal.azure.com).
2. Assignez une adresse IP statique 192.168.100.5 et le sous-réseau 255.255.255.0 à la carte Ethernet de l’ordinateur que vous utilisez pour vous connecter à Data Box. Accédez à l’interface utilisateur web locale de l’appareil à l’adresse `https://192.168.100.10`. Une fois l’appareil sous tension, la connexion peut prendre jusqu'à 5 minutes.
3. Connectez-vous avec le mot de passe à partir du portail Azure. Une erreur vous signale un problème avec le certificat de sécurité du site web. Suivez les instructions spécifiques au navigateur pour afficher la page web.
4. Par défaut, les paramètres réseau de l’interface de données 10 Gbit/s (ou 1 Gbit/s) sont configurés en mode DHCP. Si nécessaire, vous pouvez configurer cette interface en mode statique et fournir une adresse IP.

## <a name="copy-data"></a>Copier des données

Le temps nécessaire à cette opération dépend de la taille de vos données et la vitesse du réseau.

1. Si vous utilisez un client Windows, utilisez un outil de copie de fichiers compatible avec SMB tel que Robocopy. Pour l’hôte NFS, utilisez la commande `cp` ou `rsync` pour copier les données. Pour plus d’informations sur l’utilisation de Robocopy pour copier des données, consultez [Robocopy](/previous-versions/technet-magazine/ee851678(v=msdn.10)).
2. Connectez-vous aux partages d’appareils et commencez à copier les données sur l’ordinateur hôte.
<!-- 1. Connect to the device shares using the path:`\\<IP address of your device>\ShareName`. To get the share access credentials, go to the **Connect & copy** page in the local web UI of the Data Box. --> 

## <a name="ship-to-azure"></a>Envoyer à Azure

Cette opération prend environ 10 à 15 minutes.

1. Accédez à la page **Préparation de l'expédition** dans l'interface utilisateur web locale et commencez la préparation de l'expédition.
2. Mettez l’appareil hors tension à partir de l’interface utilisateur web locale. Débranchez les câbles de l’appareil.
3. Enroulez le cordon d’alimentation fourni avec la Data Box et fixez-le à l’arrière de l’appareil.
4. L’étiquette de retour doit être visible sur l’écran E-ink. Si l’écran E-ink n’affiche pas l’étiquette, accédez à **Vue d’ensemble** > **Télécharger une étiquette d’expédition** dans le portail Azure. Téléchargez l’étiquette d’expédition et collez-la sur l’appareil.
5. Planifiez un enlèvement avec votre transporteur régional si vous retournez l’appareil.
6. Une fois que la Data Box a été scannée et récupérée par le transporteur, l’état de la commande dans le portail affiche **Récupérée**. Un ID de suivi s’affiche également.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Cette étape dure 2 à 3 minutes.

* Vous pouvez annuler la commande Data Box dans le portail Azure avant son traitement. Une fois la commande traitée, elle ne peut plus être annulée. La commande suit une progression jusqu’à la phase finale. Pour annuler la commande, accédez à **Vue d’ensemble** et sélectionnez **Annuler** dans la barre de commandes.

* Vous pouvez supprimer la commande une fois que l’état **Terminé** ou **Annulé** s’affiche dans le portail Azure. Pour supprimer la commande, accédez à **Vue d’ensemble** et sélectionnez **Supprimer** dans la barre de commandes.

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez déployé une commande d’exportation entre Azure et une Azure Data Box. Pour en savoir plus sur la gestion Azure Data Box, passez au didacticiel suivant :

> [!div class="nextstepaction"]
> [Use the Azure portal to administer Data Box](data-box-portal-admin.md) (Utiliser le portail Azure pour administrer Data Box)
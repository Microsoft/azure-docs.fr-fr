---
title: Validation fonctionnelle d’une offre AppSource Dynamics 365 Finance and Operations dans la Place de marché Azure.
description: Comment valider de manière fonctionnelle une offre Dynamics 365 Finance and Operations dans la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: emuench
ms.author: navits
ms.date: 07/17/2020
ms.openlocfilehash: b9685081c0beacd745a83067b9d9876384933377
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93131239"
---
# <a name="appsource-dynamics-365-finance-and-operations-functional-validation"></a>Validation fonctionnelle d’AppSource Dynamics 365 Finance and Operations

Pour une première publication dans l’[Espace partenaires](https://partner.microsoft.com/dashboard/home), les offres pour Dynamics 365 Finance and Operations nécessitent deux validations fonctionnelles :

- Charger une vidéo de démonstration de l’environnement Dynamics 365 qui présente les fonctionnalités de base
- Présenter des captures d’écran montrant l’environnement LCS ([Lifecycle Services](https://lcs.dynamics.com/)) de la solution

> [!NOTE]
> Les publications de recertification ultérieures ne nécessitent pas de démonstration. Pour plus d’informations, consultez le [document sur les stratégies AppSource](/legal/marketplace/certification-policies#1440-dynamics-365-finance-ops).

## <a name="how-to-validate"></a>Processus de validation

Deux options sont disponibles pour la validation fonctionnelle :

- Participer à une conférence téléphonique de 30 minutes pendant les heures de bureau (Heure standard du Pacifique, PST), afin de nous montrer et d’enregistrer l’environnement et la solution [LCS](https://lcs.dynamics.com/), ou
- Dans l’Espace partenaires, accéder à [Place de marché commerciale](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) > **Vue d’ensemble** , puis charger l’URL d’une vidéo de démonstration et des captures d’écran LCS sous l’onglet Contenu supplémentaire de l’offre.

L’équipe de certification Microsoft examinera la vidéo et les fichiers, puis elle approuvera la solution ou vous enverra des e-mails concernant les étapes à suivre.

### <a name="option-1-30-minute-conference-call"></a>Option 1 : Conférence téléphonique de 30 minutes

Pour planifier un appel de révision finale, envoyez un e-mail à l’adresse [appsourceCRM@microsoft.com](mailto:appsourceCRM@microsoft.com) en mentionnant le nom de votre offre et vos disponibilités entre 8 h 00 et 17 h heure standard du Pacifique.

### <a name="option-2-upload-a-demo-video-and-lcs-screenshots"></a>Option n°2 : Charger une vidéo de démonstration et des captures d’écran LCS

1. Enregistrez une vidéo et chargez son adresse dans le site d’hébergement de votre choix. Voici les conditions à respecter concernant la vidéo :

    - Elle doit pouvoir être visionnée par l’équipe de certification Microsoft.
    - Sa durée doit être inférieure à 20 minutes
    - Elle doit comprendre au maximum trois des fonctionnalités principales de votre solution dans l’environnement Dynamics 365.

    > [!NOTE]
    > Vous pouvez utiliser une vidéo marketing existante si celle-ci respecte ces conditions.

2. Prenez les captures d’écran suivantes de l’environnement [LCS](https://lcs.dynamics.com/), qui correspondent à l’offre ou à la solution que vous souhaitez publier. Elles doivent être suffisamment nettes pour que l’équipe de certification puisse en lire le texte. Enregistrez les captures d’écran au format JPG. Plutôt que de fournir des captures d’écran, vous pouvez autoriser [appSourceCRM@microsoft.com](mailto:appSourceCRM@microsoft.com) à accéder à votre environnement LCS pour qu’ils puissent vérifier la configuration.

    1. Accédez à **LCS** > **Concepteur de processus d’entreprise** > **Bibliothèque de projets**. Prenez des captures d’écran de toutes les étapes du processus. Incluez les colonnes **Diagrammes** et **Révisé** , comme illustré ici :

       :::image type="content" source="media/dynamics-365-finance-operations/project-library.png" alt-text="Affiche la fenêtre de la bibliothèque de projets.":::

      2. Accédez à **LCS** > **Gestion des solutions** > **Tester le package de solution**. Prenez des captures d’écran qui incluent une vue d’ensemble du package et de son contenu, comme dans les exemples suivants :

    | Champ | Image <img src="" width="400px">|
    | --- | --- |
    | Vue d’ensemble du package | [![Screenshot that shows the "Package overview" window.](media/dynamics-365-finance-operations/package-overview-45.png)](media/dynamics-365-finance-operations/package-overview.png#lightbox) |
    | <ul><li>Approbateurs de solution</li></ul> | [![Vue d’ensemble du package](media/dynamics-365-finance-operations/solution-approvers-45.png)](media/dynamics-365-finance-operations/solution-approvers.png#lightbox) |
    | Contenu d'un package<ul><li>Modèle</li><li>Package logiciel déployable</li></ul> | [![Contenu du package - Capture 1](media/dynamics-365-finance-operations/package-contents-1-45.png)](media/dynamics-365-finance-operations/package-contents-1.png#lightbox) |
    | <ul><li>Configuration GER</li><li>Sauvegarde de base de données</li></ul><br>Les artefacts ne sont pas nécessaires dans la section **Configuration GER**. | [![Contenu du package - Capture 2](media/dynamics-365-finance-operations/package-contents-2-45.png)](media/dynamics-365-finance-operations/package-contents-2.png#lightbox) |
    | <ul><li>Modèle de rapport Power BI</li><li>Artefact BPM</li></ul><br>Les artefacts ne sont pas nécessaires dans la section **Power BI**. | [![Contenu du package - Capture 3](media/dynamics-365-finance-operations/package-contents-3-45.png)](media/dynamics-365-finance-operations/package-contents-3.png#lightbox) |
    | <ul><li>Traiter le package de données</li><li>Contrat de licence et politique de confidentialité de la solution</li></ul><br>Dans les offres Finance and Operations, les sections **Configuration GER** et **Modèle de rapport Power BI** sont facultatives. | [![Contenu du package - Capture 4](media/dynamics-365-finance-operations/package-contents-4-45.png)](media/dynamics-365-finance-operations/package-contents-4.png#lightbox) |

    Pour en savoir plus sur chaque section du portail LCS, consultez le [Guide de l’utilisateur LCS](/dynamics365/fin-ops-core/dev-itpro/lifecycle-services/lcs-user-guide).

3. Chargez le package dans l’Espace partenaires.

    1. Créez un document texte contenant l’adresse de la vidéo de démonstration et des captures d’écran, ou enregistrez chaque capture d’écran au format JPG.
    2. Regroupez le texte et tous les fichiers JPG dans un fichier .zip, puis chargez celui-ci dans la [Place de marché commerciale](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) de l’Espace partenaires de votre offre Finance and Operations, sous l’onglet **Contenu supplémentaire**.

    [![Affiche la fenêtre de la bibliothèque de projets](media/dynamics-365-finance-operations/supplemental-content.png)](media/dynamics-365-finance-operations/supplemental-content.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la création d’une offre, consultez : [Créer une offre Dynamics 365 for Operations](./partner-center-portal/create-new-operations-offer.md).
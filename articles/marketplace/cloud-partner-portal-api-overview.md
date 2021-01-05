---
title: Référence d’API pour le portail Cloud Partner – Place de marché commerciale Microsoft
description: Description, prérequis et liste des opérations de l’API marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 57d3d1c6bb14db3eb2ca499069934a628d2f7fea
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92425776"
---
# <a name="cloud-partner-portal-api-reference"></a>Référence d’API pour le portail Cloud Partner

> [!NOTE]
> Les API de Portail Cloud Partner sont intégrées à Espace partenaires et continueront d’y fonctionner. La transition introduit de légères modifications. Passez en revue les [modifications apportées aux API CPP](#changes-to-cpp-apis-after-the-migration-to-partner-center) listées dans ce document pour vous garantir que votre code continuera de fonctionner après la migration vers l’Espace partenaires. Les API de Portail Cloud Partner doivent uniquement être utilisées pour les produits existants intégrés avant la transition vers Espace partenaires. Les nouveaux produits doivent utiliser les API de soumission d’Espace partenaires.

Les API REST du portail Cloud Partner permettent de récupérer de manipuler par programmation des charges de travail, des offres et des profils d’éditeur. Les API utilisent le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour appliquer les autorisations appropriées au moment du traitement.

Cette référence contient les détails techniques des API REST du portail Cloud Partner. Les exemples de charge utile de ce document sont fournis à titre de référence uniquement et sont susceptibles de changer lors de l’ajout de nouvelles fonctionnalités.

## <a name="prerequisites-and-considerations"></a>Prérequis et considérations

Avant d’utiliser les API, consultez :

- L’article [Prérequis](./cloud-partner-portal-api-prerequisites.md) pour savoir comment ajouter un principal de service à votre compte et obtenir un jeton d’accès Azure Active Directory (Azure AD) pour l’authentification.
- Les deux stratégies de [contrôle de la concurrence](./cloud-partner-portal-api-concurrency-control.md) disponibles pour appeler ces API.
- Les autres [considérations](./cloud-partner-portal-api-considerations.md) relatives aux API, comme le contrôle de version et la gestion des erreurs.

## <a name="changes-to-cpp-apis-after-the-migration-to-partner-center"></a>Modifications apportées aux API CPP après la migration vers l’Espace partenaires

| **API** | **Description de la modification** | **Impact** |
| ------- | ---------------------- | ---------- |
| POST Publish, GoLive, Cancel | Pour les offres migrées, l’en-tête de réponse aura un format différent, mais continuera à fonctionner de la même façon, indiquant un chemin relatif pour récupérer l’état de l’opération. | Lors de l’envoi d’une des requêtes de publication correspondantes pour une offre, l’en-tête d’emplacement aura l’un des deux formats suivants en fonction de l’état de migration de l’offre :<ul><li>Offres non migrées<br>`/api/operations/{PublisherId}${offerId}$2$preview?api-version=2017-10-31`</li><li>Offres migrées<br>`/api/publishers/{PublisherId}/offers/{offereId}/operations/408a4835-0000-1000-0000-000000000000?api-version=2017-10-31`</li> |
| Opération GET | Pour les types d’offres qui prenaient auparavant en charge le champ « notification-email » dans la réponse, ce champ est déprécié et n’est plus retourné pour les offres migrées. | Pour les offres migrées, nous n’enverrons plus de notifications à la liste des e-mails spécifiés dans les requêtes. Au lieu de cela, le service d’API s’alignera sur le processus de notification par e-mail dans l’Espace partenaires pour envoyer des e-mails. Plus précisément, les notifications seront envoyées à l’adresse e-mail définie dans la section Coordonnées du vendeur de vos paramètres de compte dans l’Espace partenaires, pour vous informer de la progression de l’opération.<br><br>Veuillez vousérifiez l’adresse e-mail définie dans la section Coordonnées du vendeur de vos [Paramètres du compte](https://partner.microsoft.com/dashboard/account/management) dans l’Espace partenaires afin de vous assurer que l’adresse e-mail correcte est fournie pour les notifications.  |

## <a name="common-tasks"></a>Tâches courantes

Cette référence décrit en détail les API qui permettent d’effectuer les tâches courantes suivantes.

### <a name="offers"></a>Offres

- [Récupérer toutes les offres](./cloud-partner-portal-api-retrieve-offers.md)
- [Récupérer une offre en particulier](./cloud-partner-portal-api-retrieve-specific-offer.md)
- [Récupérer l’état de l’offre](./cloud-partner-portal-api-retrieve-offer-status.md)
- [Créer une offre](./cloud-partner-portal-api-creating-offer.md)
- [Publier une offre](./cloud-partner-portal-api-publish-offer.md)

### <a name="operations"></a>Opérations

- [Récupérer des opérations](./cloud-partner-portal-api-retrieve-operations.md)
- [Annuler des opérations](./cloud-partner-portal-api-cancel-operations.md)

### <a name="publish-an-app"></a>Publier une application

- [Démarrer](./cloud-partner-portal-api-go-live.md)

### <a name="other-tasks"></a>Autres tâches

- [Définir la tarification des offres de machine virtuelle](./cloud-partner-portal-api-setting-price.md)

### <a name="troubleshooting"></a>Dépannage

- [Résoudre les erreurs d’authentification](./cloud-partner-portal-api-troubleshooting-authentication-errors.md)

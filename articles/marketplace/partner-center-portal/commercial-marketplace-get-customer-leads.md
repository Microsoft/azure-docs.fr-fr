---
title: Gestion des prospects du marketplace commercial de Microsoft
description: En savoir plus sur la génération et la réception des prospects de vos offres Microsoft AppSource et Place de marché Azure
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: trkeya
ms.author: trkeya
ms.date: 10/01/2020
ms.openlocfilehash: 1d5f2248d94796d5e3ee76301642a95abddebfe4
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94489333"
---
# <a name="customer-leads-from-your-commercial-marketplace-offer"></a>Prospects de votre offre de marketplace commercial

Les prospects sont des clients intéressés par vos offres sur [Microsoft AppSource](https://appsource.microsoft.com) ou la [Place de marché Azure](https://azuremarketplace.microsoft.com), ou qui procèdent à leur déploiement. Vous pouvez recevoir des prospects une fois que votre offre est publiée sur le marketplace commercial. Cet article explique les concepts de gestion des prospects suivants :

* Comment votre offre publiée sur le marketplace commercial génère des prospects pour que vous ne manquiez aucune opportunité commerciale. 
* Comment connecter votre système de gestion de la relation client (CRM) à votre offre pour pouvoir gérer vos prospects dans un emplacement central.
* Les données de prospect que nous vous envoyons afin que vous puissiez effectuer le suivi des clients qui vous ont contacté.

## <a name="generate-customer-leads"></a>Générer des prospects

Voici les endroits où les prospects sont générés :

- Un client consent à partager ses informations après avoir sélectionné **Me contacter** sur la Place de marché commerciale. Il s’agit d’un prospect d’ *intérêt initial*. Nous partageons avec vous des informations sur ce client qui a manifesté de l’intérêt pour votre produit. Il constitue la partie supérieure de l’entonnoir d’acquisition.

    ![Me contacter dans Dynamics 365](./media/commercial-marketplace-get-customer-leads/dynamics-365-contact-me.png)

- Un client sélectionne **Obtenir maintenant** (ou **Créer** sur le [portail Azure](https://portal.azure.com/)) pour obtenir votre offre. Il s’agit d’un prospect *actif*. Nous partageons avec vous des informations sur ce client qui a commencé à déployer votre produit.

    ![Bouton SQL : Télécharger maintenant](./media/commercial-marketplace-get-customer-leads/sql-get-it-now.png)

    ![Bouton Windows Server : Créer](./media/commercial-marketplace-get-customer-leads/windows-server-create.png)

- Un client sélectionne **Version d’évaluation** ou **Essai gratuit** pour essayer votre offre. Les versions d’évaluation et les essais gratuits constituent pour vous une occasion de partager votre activité instantanément avec des clients potentiels, sans barrières.

    ![Bouton Dynamics 365 : Version d’évaluation](./media/commercial-marketplace-get-customer-leads/dynamics-365-test-drive.png)

    ![Bouton Dynamics 365 : Essai gratuit](./media/commercial-marketplace-get-customer-leads/dynamics-365-free-trial.png)

## <a name="connect-to-your-crm-system"></a>Connexion à votre système CRM

[!INCLUDE [Links to lead configuration for different CRM systems](./includes/connect-lead-management.md)]

## <a name="understand-lead-data"></a>Comprendre les données de prospect

Chaque prospect que vous recevez pendant le processus d’acquisition de clients dispose de données dans des champs spécifiques. Le premier champ à rechercher est le champ `LeadSource`, qui suit ce format : **Source-Action** | **Offre**.

**Sources** La valeur de ce champ est renseignée en fonction de la place de marché qui a généré le prospect. Les valeurs possibles sont `"AzureMarketplace"`, `"AzurePortal"` et `"AppSource (SPZA)"`.

**Actions**  : La valeur de ce champ est renseignée en fonction de l’action effectuée par le client sur la Place de marché, qui a généré le prospect.

Les valeurs possibles sont les suivantes :

- **« INS »**  : signifie *installation*. Cette action est proposée sur la Place de marché Azure ou dans AppSource lorsqu’un client acquiert votre produit.
- **« PLT »**  : désigne un *essai gratuit mené par un partenaire*. Cette action est proposée dans AppSource lorsqu’un client sélectionne l’option **Me contacter**.
- **« DNC »**  : signifie *Ne pas contacter*. Cette action apparaît dans AppSource lorsqu’un partenaire qui a été répertorié sur la page de votre application reçoit une demande de contact. Nous partageons une notification indiquant que ce client a été répertorié sur votre application, mais qu’il n’est pas nécessaire de le contacter.
- **« Create »**  : cette action apparaît uniquement sur le portail Azure et est générée lorsqu’un client achète votre offre sur son compte.
- **« StartTestDrive »**  : cette action est proposée uniquement pour l’option **Version d’évaluation** et elle est générée quand un client démarre sa version d’évaluation.

**Offres**  : Vous pouvez avoir plusieurs offres sur la Place de marché commerciale. La valeur de ce champ est renseignée en fonction de l’offre qui a généré le prospect. L’ID de serveur de publication et l’ID d’offre sont tous deux envoyés dans ce champ et sont des valeurs que vous avez fournies lors de la publication de l’offre sur la Place de marché.

Les exemples suivants présentent des valeurs au format `publisherid.offerid` attendu : 

- `checkpoint.check-point-r77-10sg-byol`
- `bitnami.openedxcypress`
- `docusign.3701c77e-1cfa-4c56-91e6-3ed0b622145`

## <a name="customer-information"></a>Informations client

Les informations du client sont envoyées dans plusieurs champs. L’exemple suivant présente les informations client qui sont contenues dans un prospect :

- FirstName : John
- LastName : Smith
- Adresse e-mail : jsmith\@microsoft.com
- Téléphone : 1234567890
- Pays : US
- Société : Microsoft
- Titre : CTO

>[!NOTE]
>Les données de l’exemple précédent ne sont pas toujours toutes disponibles pour chaque prospect. Comme vous recevez des prospects de plusieurs étapes, comme mentionné dans la section « Générer des prospects », la meilleure façon de les traiter consiste à dédupliquer les enregistrements et à personnaliser les suivis. De cette manière, chaque client reçoit un message approprié et vous créez une relation unique.

## <a name="best-practices-for-lead-management"></a>Meilleures pratiques de gestion des prospects

Voici quelques recommandations pour faire avancer les prospects dans le cycle de vente :

- **Processus**  : Définissez un processus de vente clair, avec des jalons, des analyses et des responsabilités claires dans l’équipe.
- **Qualification**  : Définissez les conditions préalables qui indiquent si un prospect était pleinement qualifié. Assurez-vous que les représentants commerciaux ou marketing qualifient les prospects avec soins avant de les incorporer dans le processus de vente.
- **Suivi**  : N’oubliez pas d’effectuer un suivi dans les 24 heures. Le prospect sera intégré au système CRM de votre choix juste après que le client aura déployé la version d’évaluation ; envoyez-lui un e-mail à chaud. Demandez à programmer un appel téléphonique afin de mieux comprendre si votre produit est une bonne solution pour résoudre le problème. Une transaction implique en général de nombreux appels de suivi.
- **Soins**  : Soignez vos prospects pour bénéficier d’une plus grande marge de profits. Restez en contact avec eux sans les harceler. Nous vous recommandons d’envoyer au moins quelques e-mails aux prospects avant de clore le dossier ; ne renoncez pas après la première tentative. N’oubliez pas que ces clients ont passé du temps à tester votre produit dans le cadre d’un essai gratuit : ce sont d’excellents prospects.

Une fois les paramètres techniques configurés, incorporez ces prospects à votre stratégie commerciale et marketing, ainsi qu’à vos processus opérationnels actuels. Nous cherchons à mieux comprendre votre processus de vente global. C’est pourquoi nous tenons à travailler en étroite collaboration avec vous pour vous apporter des prospects de qualité et des données en quantité suffisante pour garantir votre réussite. Nous apprécions vos commentaires sur la façon dont nous pouvons optimiser et améliorer les prospects que nous vous envoyons, accompagnés de toutes les données nécessaires pour assurer la réussite de ces clients. Faites-nous savoir si vous souhaitez nous [communiquer vos commentaires](mailto:AzureMarketOnboard@microsoft.com) et suggestions pour permettre à votre équipe commerciale de mieux exploiter les prospects de la Place de marché commerciale.

## <a name="next-steps"></a>Étapes suivantes

- [FAQ et résolution des problèmes liés à la gestion des prospects](../lead-management-faq.md)
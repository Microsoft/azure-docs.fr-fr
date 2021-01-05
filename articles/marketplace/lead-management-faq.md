---
title: Questions sur la gestion des prospects et résolution des problèmes relatifs à cette activité – Espace partenaires Microsoft
description: Découvrez les erreurs courantes et les questions les plus fréquentes concernant la configuration des prospects de marketplace commercial dans l’espace partenaires
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 10/01/2020
ms.openlocfilehash: 681498073a5f111e3babd50443623589ffeec02f
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92330918"
---
# <a name="common-questions-and-troubleshooting-for-lead-configuration"></a>Questions fréquentes et résolution des problèmes concernant la configuration des prospects

Cet article répond à certaines questions fréquentes sur la gestion des prospects dans le cadre de vos offres de marketplace commercial. Il présente également des erreurs que vous pouvez rencontrer lors de la configuration de prospects sur votre système de gestion des relations avec la clientèle dans l’espace partenaires.

## <a name="common-questions-about-lead-management"></a>Questions courantes sur la gestion des prospects

#### <a name="where-can-i-get-help-in-setting-up-my-lead-destination"></a>Où puis-je obtenir de l’aide pour configurer la destination des prospects ?

Pour consulter une présentation générale de la méthode à suivre pour connecter votre système de gestion des relations avec la clientèle à vos offres de marketplace commercial, consultez la page [Prospects de votre offre de marketplace commercial](partner-center-portal/commercial-marketplace-get-customer-leads.md) . Si vous rencontrez une erreur, passez en revue les conseils de résolution des problèmes ci-dessous. Pour bénéficier d’une assistance plus personnalisée, envoyez un ticket de support via la page [Aide et support de l’Espace partenaires](https://aka.ms/marketplacepublishersupport). Sélectionnez ensuite **Création d’offre** > **Votre type d’offre** > **Configuration de la gestion des prospects**.

#### <a name="am-i-required-to-configure-a-lead-destination-in-order-to-publish-an-offer-in-the-commercial-marketplace"></a>Dois-je obligatoirement configurer une destination de prospect pour publier une offre sur la place de marché commerciale ?

La réponse dépend du type d’offre que vous publiez. Les offres de logiciel en tant que service (SaaS) et d’engagement client Dynamics 365 utilisent **Me contacter** pour lister toutes les offres Dynamics 365 pour la finance et les opérations, toutes les offres Dynamics 365 Business Central et toutes les offres de services de conseil. Par conséquent, elles nécessitent une connexion à une destination de prospect. Si votre type d’offre ne figure pas dans cette liste, une connexion à une destination de prospect n’est pas nécessaire. Nous vous recommandons de configurer une destination de prospect pour ne pas manquer d’opportunités commerciales.

#### <a name="how-can-i-find-the-test-lead"></a>Comment trouver le prospect test ?

Recherchez `"MSFT_TEST"` dans votre destination de prospect. Vous trouverez ci-dessous un exemple de prospect test de Microsoft. Notez que le format du prospect test varie en fonction de la destination du prospect.

```
{
    "userDetails": {
      "FirstName": "MSFT_TEST_636573304831318844",
      "LastName": "MSFT_TEST_636573304831318844",
      "Email": "MSFT_TEST_636573304831318844@test.com",
      "Phone": "1234567890",
      "Country": "US",
      "Company": "MSFT_TEST_636573304831318844",
      "Title": "MSFT_TEST_636573304831318844"
    },
    "LeadSource": "AzureMarketplace",
    "ActionCode": "INS",
    "OfferTitle": "Contoso Test"
    "Description": "MSFT_TEST_636573304831318844"
}
```

#### <a name="i-have-a-live-offer-but-why-am-i-not-seeing-any-leads"></a>J’ai une offre en ligne, mais je ne vois aucun prospect. Pourquoi ?

Assurez-vous que votre connexion à la destination du prospect est valide. Nous vous enverrons un prospect test une fois que vous aurez sélectionné **Publier** dans votre offre, dans l’Espace partenaires. Si vous voyez le prospect test, la connexion est valide. Vous pouvez également tester votre connexion de prospect en essayant d’acquérir la préversion de l’offre au cours de l’étape de préversion. Sélectionnez **Obtenir maintenant**, **Me contacter** ou **Essai gratuit** dans la liste sur la Place de marché commerciale.

De plus, vérifiez de rechercher les bonnes données. Pour consulter une explication des données de prospect que nous envoyons à votre destination de prospect, consultez la section [Comprendre les données de prospect](partner-center-portal/commercial-marketplace-get-customer-leads.md).

#### <a name="i-configured-azure-blob-storage-as-my-lead-destination-but-why-dont-i-see-the-lead"></a>J’ai configuré le stockage Blob Azure comme destination de prospect. Pourquoi le prospect n’est-il pas visible ?

Le stockage Blob Azure n’est plus pris en charge comme destination de prospect et vous risquez de manquer les prospects générés par votre offre. Basculez vers l’une des autres options de [destination des prospects](partner-center-portal/commercial-marketplace-get-customer-leads.md). 

#### <a name="i-received-an-email-from-the-commercial-marketplace-but-why-cant-i-find-the-lead-in-my-crm"></a>J’ai reçu un e-mail de la Place de marché commerciale, mais pourquoi le prospect est-il introuvable dans mon système CRM ?

Il est possible que le domaine de messagerie de l’utilisateur final soit .edu. Pour des raisons de confidentialité, nous ne transmettons pas d’informations personnelles en provenance du domaine .edu. Soumettez un ticket de support via [Aide et support de l’Espace partenaires](https://aka.ms/marketplacepublishersupport).

#### <a name="i-configured-an-azure-table-as-my-lead-destination-how-can-i-view-the-leads"></a>J’ai configuré une table Azure comme destination de prospect. Comment afficher les prospects ?

Vous pouvez accéder aux données de prospect stockées dans la table Azure à partir du portail Azure. Vous pouvez également télécharger et installer gratuitement l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) pour afficher les données de table de votre compte de stockage Azure.

#### <a name="i-configured-an-azure-table-as-my-lead-destination-can-i-get-notified-whenever-a-new-commercial-marketplace-lead-is-sent"></a>J’ai configuré une table Azure comme destination de prospect. Puis-je recevoir une notification chaque fois qu’un nouveau prospect est envoyé par la Place de marché commerciale ?

Oui. Suivez les instructions figurant dans [Configurer la gestion des prospects à l’aide d’une table Azure](partner-center-portal/commercial-marketplace-lead-management-instructions-azure-table.md) pour configurer un flux Microsoft Flow qui envoie un e-mail si un prospect est ajouté à la table Azure.

#### <a name="i-configured-salesforce-as-my-lead-destination-but-why-cant-i-find-the-leads"></a>J’ai configuré Salesforce comme destination de prospect, mais pourquoi les prospects sont-ils introuvables ?

Vérifiez si le formulaire Web-to-Lead est un champ obligatoire avec liste déroulante. Si c’est le cas, faites-en un champ de texte non obligatoire.

#### <a name="there-was-an-issue-with-my-lead-destination-and-i-missed-some-leads-can-i-have-them-sent-to-me-in-an-email"></a>Un problème s’est produit au niveau de ma destination des prospects, ce qui m’a fait manquer des prospects. Puis-je me les faire envoyer par e-mail ?

En raison des stratégies relatives aux informations personnelles, nous ne pouvons pas partager les informations sur les prospects via un e-mail non sécurisé.

#### <a name="i-configured-an-azure-table-as-my-lead-destination-how-much-will-it-cost"></a>J’ai configuré une table Azure comme destination de prospect. Combien cela va-t-il coûter ?

Les données de génération de prospects sont peu volumineuses. Elles représentent moins de 1 Go pour presque tous les serveurs de publication. Le coût dépend du nombre de prospects reçus. Par exemple, si 1 000 prospects sont reçus en un mois, le coût est d’environ 50 cents. Pour plus d’informations sur la tarification du stockage, consultez [Vue d’ensemble de la tarification de Stockage Azure](https://azure.microsoft.com/pricing/details/storage/).

Si vous n’avez pas trouvé de réponse à votre question, contactez le Support Microsoft via [Aide et support de l’Espace partenaires](https://aka.ms/marketplacepublishersupport). Sélectionnez ensuite **Création d’offre** > **Votre type d’offre** > **Configuration de la gestion des prospects**.

#### <a name="im-receiving-email-notifications-when-new-customer-leads-are-received-how-can-i-configure-someone-else-to-receive-these-emails"></a>Je reçois des notifications par e-mail lors de la réception de nouveaux prospects. Comment puis-je configurer un autre destinataire pour ces e-mails ?

Accédez à votre offre dans l’Espace partenaires, puis accédez à la page **Configuration de l’offre** > **Gestion des prospects** > **Modifier**. Mettez à jour les adresses e-mail dans le champ **E-mail du contact**.

## <a name="troubleshooting-lead-configuration-errors"></a><a id="publishing-config-errors"></a> Résolution des erreurs de configuration des prospects

**Impossible d’enregistrer le prospect dans Dynamics CRM. Vérifiez les paramètres de compte Dynamics CRM. LastCRMError : Impossible de se connecter à Dynamics CRM, LastCRMException :** 

> Si l’authentification Microsoft 365 a été sélectionnée, vérifiez la validité du compte d’utilisateur et du mot de passe. Si Azure Active Directory a été sélectionné, vérifiez que l’ID de locataire, l’ID d’application et la clé secrète de l’application correspondent à ce qui a été configuré dans Azure Active Directory. Suivez les instructions [ici](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md). Si le nom d’utilisateur/mot de passe du compte est valide, vérifiez que celui-ci a accès à Dynamics 365 et qu’une licence lui est attribuée (étapes 11 à 15 si vous utilisez Azure Active Directory ou les paramètres de sécurité si vous utilisez un utilisateur Office). 

**Impossible d’enregistrer le prospect dans Dynamics CRM. L’utilisateur ne dispose pas d’autorisations de création pour l’attribut leadsourcecode dans l’entité prospect** 

> Il manque un ou plusieurs rôles de sécurité à l’application/utilisateur pour le rédacteur de prospects de la Place de marché Microsoft. Suivez les étapes 11 à 15 si vous utilisez Azure Active Directory ou Paramètres de sécurité si vous utilisez un utilisateur Office [ici](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md).

**Impossible d’enregistrer le prospect dans Dynamics CRM en utilisant AAD. Exception:: Locataire introuvable. Cette instance peut se produire s’il n’existe aucun abonnement actif pour le locataire.**  

> L’ID de répertoire fourni dans la section de gestion des prospects n’est pas un répertoire valide. Procurez-vous l’ID de répertoire en suivant les instructions fournies à l’étape 2 (sous Azure Active Directory) [ici](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md).

**Impossible d’enregistrer le prospect dans Dynamics CRM. LastCRMError : SecLib::RetrievePrivilegeForUser en échec - aucun rôle attribué à l’utilisateur.**  

> Résolution : attribuez un rôle de sécurité au rédacteur de prospects de la Place de marché Microsoft. Suivez les instructions fournies [ici](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md) sous Paramètres de sécurité.

**Impossible d’enregistrer le prospect dans Dynamics CRM en utilisant AAD. Exception:: L’application associée à l’identificateur est introuvable dans le répertoire** 

> L’ID d’application fourni dans la section de gestion des prospects n’est pas un répertoire valide. Procurez-vous l’ID de répertoire en suivant les instructions fournies à l’étape 8 (sous Azure Active Directory, à partir d’[ici](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)). 

**Impossible d’enregistrer le prospect dans Dynamics CRM en utilisant AAD. Exception:: L’identificateur de locataire demandé n’est pas valide et n’est pas un format de domaine externe valide** 

> L’ID de répertoire fourni dans la section de gestion des prospects n’est pas un répertoire valide. Procurez-vous l’ID de répertoire en suivant les instructions fournies à l’étape 2 (sous Azure Active Directory, à partir d’[ici](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)). 

**Impossible d’enregistrer le prospect dans Dynamics CRM en utilisant AAD. Exception:: Erreur de validation des informations d’identification. Le secret client fourni n’est pas valide.** 

> Résolution : Connectez-vous au portail Azure et vérifiez que la clé d’application correspond à celle de l'Espace partenaires. Générez un mot de passe en suivant les instructions fournies à l’étape 10 (sous Azure Active Directory, à partir d’[ici](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md)). 

**Impossible d’enregistrer le prospect dans Dynamics CRM. LastCRMError : Le canal de demande a dépassé le délai de réponse après 00:02:00. Augmentez la valeur de délai d’attente passée à l’appel de Request ou augmentez la valeur de SendTimeout de la liaison. Le temps alloué à cette opération fait peut-être partie d’un délai d’attente plus long.**  

> Résolution : Connectez-vous à l'Espace partenaires, puis dans Configuration de l’offre >> Prospects >> URL, vérifiez qu’il s’agit d’une instance Dynamics CRM valide.


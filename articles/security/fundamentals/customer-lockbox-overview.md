---
title: Customer Lockbox pour Microsoft Azure
description: Vue d’ensemble technique de Customer Lockbox pour Microsoft Azure, qui permet de contrôler les accès de fournisseur cloud lorsque Microsoft doit accéder aux données client.
author: TerryLanfear
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.author: terrylan
manager: rkarlin
ms.date: 09/15/2020
ms.openlocfilehash: 01232aa101e2964354acfbeb6cea341a0da33ca6
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489886"
---
# <a name="customer-lockbox-for-microsoft-azure"></a>Customer Lockbox pour Microsoft Azure

> [!NOTE]
> Pour utiliser cette fonctionnalité, votre organisation doit disposer d’un [plan de support Azure](https://azure.microsoft.com/support/plans/) avec un niveau minimal de **développeur**.

Customer Lockbox pour Microsoft Azure fournit une interface dans laquelle les clients peuvent vérifier et approuver/refuser les demandes d’accès aux données client. Il est utilisé lorsqu’un ingénieur Microsoft doit accéder aux données client dans le cadre d’une demande de support.

Cet article décrit comment les demandes Customer Lockbox sont initiées, suivies et stockées en vue d’audits et de révisions ultérieures.

Customer Lockbox est désormais disponible au plus grand nombre et actuellement activé pour l’accès Bureau à distance sur les machines virtuelles.

## <a name="supported-services-and-scenarios-in-preview"></a>Services et scénarios pris en charge en préversion

Les services suivants sont désormais en version préliminaire pour Customer Lockbox :

- Gestion des API
- Azure App Service
- Cognitive Services
- Container Registry
- Azure Database pour MySQL
- Azure Databricks
- Azure Data Box
- Explorateur de données Azure
- Azure Data Factory
- Azure Database pour PostgreSQL
- Azure Functions
- HDInsight
- Azure Kubernetes Service
- Azure Monitor
- Stockage Azure
- Azure SQL DB
- Transfert d'abonnement Azure
- Azure Synapse Analytics
- Machines virtuelles (qui couvrent désormais également l’accès aux images mémoire et aux disques managés)

Pour activer Customer Lockbox pour ces offres de préversion pour votre organisation, inscrivez-vous à [Customer Lockbox pour la préversion publique d’Azure](https://aka.ms/customerlockbox/insiderprogram).

## <a name="supported-services-and-scenarios-in-general-availability"></a>Services et scénarios pris en charge dans la disponibilité générale

Les services et les scénarios suivants sont actuellement disponibles globalement pour Customer Lockbox.

### <a name="remote-desktop-access-to-virtual-machines"></a>Accès Bureau à distance aux machines virtuelles

Customer Lockbox est actuellement activé pour les demandes d’accès Bureau à distance sur les machines virtuelles. Les charges de travail suivantes sont prises en charge :
- Platform as a service (PaaS) - Azure Cloud Services (rôle web et rôle de travail)
- Infrastructure as a Service (IaaS) - Windows et Linux (Azure Resource Manager uniquement)
- Groupe de machines virtuelles identiques - Windows et Linux

> [!NOTE]
> Les instances IaaS Classic ne sont pas prises en charge par Customer Lockbox. Si vous avez des charges de travail exécutées sur des instances IaaS Classic, nous vous recommandons de les migrer vers des modèles de déploiement Resource Manager. Pour obtenir des instructions, voir [Migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](../../virtual-machines/migration-classic-resource-manager-overview.md)

#### <a name="detailed-audit-logs"></a>Journaux d’audit détaillés

Pour les scénarios qui impliquent l’accès Bureau à distance, vous pouvez utiliser les journaux des événements Windows pour passer en revue les actions effectuées par l’ingénieur Microsoft. Envisagez d’utiliser Azure Security Center pour collecter vos journaux des événements et de copier les données dans votre espace de travail à des fins d’analyse. Pour plus d’informations, voir [Collecte de données dans Azure Security Center](../../security-center/security-center-enable-data-collection.md).

## <a name="workflow"></a>Workflow

Les étapes suivantes décrivent un workflow classique pour une demande Customer Lockbox.

1. Une personne dans votre organisation a un problème avec sa charge de travail Azure.

2. Une fois que cette personne a identifié le problème, sans pour autant le résoudre, elle ouvre un ticket de support sur le [portail Azure](https://ms.portal.azure.com/signin/index/?feature.settingsportalinstance=mpac). Le ticket est affecté à un ingénieur du support client Azure.

3. Un ingénieur du support Azure examine la demande de service et détermine les étapes suivantes pour résoudre le problème.

4. Si l’ingénieur du support ne parvient pas à résoudre le problème à l’aide des données de télémétrie et des outils standard, l’étape suivante consiste à demander des autorisations élevées à l’aide d’un service d’accès juste à temps (JAT). Cette demande peut émaner de l’ingénieur du support d’origine ou d’un autre ingénieur quand le problème est remonté à l’équipe Azure DevOps.

5. Une fois la demande d’accès soumise par l’ingénieur Azure, le service juste-à-temps évalue la demande en tenant compte des facteurs suivants :
    - Étendue de la ressource
    - Si l’auteur de la demande est une identité isolée ou utilise l’authentification multifacteur
    - Niveaux d’autorisation

    Selon la règle JIT, cette demande peut également inclure une approbation des approbateurs Microsoft internes. Par exemple, l’approbateur peut être le responsable du support client ou le responsable DevOps.

6. Lorsque la demande nécessite un accès direct aux données client, une demande Customer Lockbox est lancée. Par exemple, pour un accès Bureau à distance à la machine virtuelle d’un client.

    La demande a désormais l’état **Client averti** : elle attend l’approbation du client avant que l’accès soit accordé.

7. Dans l’organisation du client, l’utilisateur qui a le [rôle de propriétaire](../../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) pour l’abonnement Azure reçoit un e-mail de la part de Microsoft l’informant de la demande d’accès en attente. Pour les demandes Customer Lockbox, cette personne est l’approbateur désigné.

    Exemple d’e-mail :

    ![Azure Customer Lockbox - Notification par courrier électronique](./media/customer-lockbox-overview/customer-lockbox-email-notification.png)

8. La notification par courrier électronique contient un lien vers le panneau **Customer Lockbox** dans le portail Azure. Via ce lien, l’approbateur désigné se connecte au portail Azure pour afficher des demandes en attente de l’organisation pour Customer Lockbox :

    ![Azure Customer Lockbox - Page d’accueil](./media/customer-lockbox-overview/customer-lockbox-landing-page.png)

   La demande reste dans la file d’attente du client pendant quatre jours. Passé ce délai, la demande d’accès expire automatiquement et aucun accès n’est accordé aux ingénieurs Microsoft.

9. Pour obtenir les détails de la demande en attente, l’approbateur désigné peut sélectionner la demande Customer Lockbox sous les **demandes en attente** :

    ![Azure Customer Lockbox - Afficher la demande en attente](./media/customer-lockbox-overview/customer-lockbox-pending-requests.png)

10. L’approbateur désigné peut également sélectionner l’**ID DE DEMANDE DE SERVICE** pour afficher la demande de ticket de support créée par l’utilisateur d’origine. Ces informations permettent de comprendre pourquoi le support Microsoft a été sollicité et de connaître l’historique du problème signalé. Par exemple :

    ![Azure Customer Lockbox - Afficher la demande du ticket de support](./media/customer-lockbox-overview/customer-lockbox-support-ticket.png)

11. Après avoir examiné la demande, l’approbateur désigné sélectionne **Approuver** ou **Refuser** :

    ![Azure Customer Lockbox - Sélectionner Approuver ou Refuser](./media/customer-lockbox-overview/customer-lockbox-approval.png)

    Selon la sélection :
    - **Approuver** :  l’accès est accordé à l’ingénieur Microsoft. L’accès est accordé pour une durée par défaut de huit heures.
    - **Refuser** : la demande d’accès avec élévation de privilèges de l’ingénieur Microsoft est rejetée, et aucune action supplémentaire n’est nécessaire.

À des fins d’audit, les actions effectuées dans ce workflow sont enregistrées dans les [journaux de demandes Customer Lockbox](#auditing-logs).

## <a name="auditing-logs"></a>Journaux d’activité d’audit

Les journaux Customer Lockbox sont stockés dans les journaux d’activité. Dans le portail Azure, sélectionnez **Journaux d’activité** pour afficher les informations d’audit relatives aux demandes Customer Lockbox. Vous pouvez filtrer selon des actions spécifiques :
- **Refuser une demande Lockbox**
- **Créer une demande Lockbox**
- **Approuver une demande Lockbox**
- **Expiration d’une demande Lockbox**

Par exemple :

![Azure Customer Lockbox - Journaux d’activité](./media/customer-lockbox-overview/customer-lockbox-activitylogs.png)

## <a name="customer-lockbox-integration-with-azure-security-benchmark"></a>Intégration de Customer Lockbox au benchmark de sécurité Azure

Nous avons introduit un nouveau contrôle de base de référence ([3.13](../benchmarks/security-control-identity-access-control.md#313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios)) dans le benchmark de sécurité Azure qui couvre la mise en application de Customer Lockbox. Les clients peuvent désormais tirer parti du benchmark afin de passer en revue la mise en application de Customer Lockbox pour un service.

## <a name="exclusions"></a>Exclusions

Les demandes Customer Lockbox ne sont pas déclenchées dans les scénarios de support d’ingénierie suivants :

- Un ingénieur Microsoft doit réaliser une activité qui ne fait pas partie des procédures de fonctionnement standard. Par exemple, pour récupérer ou restaurer des services dans des scénarios inattendus ou imprévisibles.

- Un ingénieur Microsoft accède à la plateforme Azure dans le cadre de la résolution des problèmes et accède par inadvertance aux données client. Par exemple, lors de la résolution des problèmes, l’équipe de réseau Azure capture des paquets sur un périphérique réseau. Toutefois, si le client a chiffré les données en transit, l’ingénieur ne peut pas lire les données.

## <a name="next-steps"></a>Étapes suivantes

Customer Lockbox est automatiquement disponible pour tous les clients qui ont un [plan de support Azure](https://azure.microsoft.com/support/plans/) avec un niveau minimal de **développeur**.

Lorsque vous disposez d’un plan de support éligible, vous n’avez rien à faire pour activer Customer Lockbox. Les demandes Customer Lockbox sont initiées par un ingénieur Microsoft si cette action est nécessaire pour l’avancement d’un ticket de support créé par une personne de votre organisation.
---
title: 'Conception d’identités hybrides : stratégie de protection des données Azure | Microsoft Docs'
description: Vous définissez la stratégie de protection des données pour que votre solution d’identités hybrides réponde aux exigences de l’entreprise que vous avez définies.
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: e76fd1f4-340a-492a-84d9-e05f3b7cc396
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: e991fb0c60e8f08eb43cb7799027d4200263c9b5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89659543"
---
# <a name="define-data-protection-strategy-for-your-hybrid-identity-solution"></a>Définir la stratégie de protection des données pour votre solution d’identités hybrides
Dans cette tâche, vous allez définir la stratégie de protection des données pour que votre solution d’identités hybrides réponde aux exigences de l’entreprise définies dans :

* [Déterminer les exigences de protection des données](plan-hybrid-identity-design-considerations-dataprotection-requirements.md)
* [Déterminer les exigences de gestion de contenu](plan-hybrid-identity-design-considerations-contentmgt-requirements.md)
* [Déterminer les exigences de contrôle d’accès](plan-hybrid-identity-design-considerations-accesscontrol-requirements.md)
* [Déterminer les exigences de réponse aux incidents](plan-hybrid-identity-design-considerations-incident-response-requirements.md)

## <a name="define-data-protection-options"></a>Définir les options de protection des données
Comme expliqué dans la section [Déterminer les exigences de synchronisation de répertoire](plan-hybrid-identity-design-considerations-directory-sync-requirements.md), Microsoft Azure AD peut se synchroniser avec vos services Active Directory Domain Services (AD DS) locaux. Cette intégration permet aux organisations d’utiliser Azure AD pour vérifier les informations d’identification de l’utilisateur quand il essaie d’accéder aux ressources d’entreprise. Vous pouvez effectuer cette opération pour les deux scénarios : les données au repos en local et dans le cloud. L’accès aux données dans Azure AD requiert l’authentification des utilisateurs via un service d’émission de jeton de sécurité (STS).

Une fois authentifié, le nom d’utilisateur principal (UPN) est lu à partir du jeton d’authentification. Ensuite, le système d’autorisation détermine la partition répliquée et le conteneur correspondant au domaine de l’utilisateur. Les informations sur l’existence de l’utilisateur, l’état activé et le rôle permettent ensuite au système d’autorisation de déterminer si l’accès au locataire cible est autorisé pour l’utilisateur dans cette session. Certaines actions autorisées (en particulier, créer un utilisateur et réinitialiser un mot de passe) créent une piste d’audit qu’un administrateur client peut utiliser par pour gérer les efforts ou enquêtes de conformité.

Le déplacement des données à partir de votre centre de données local dans Azure Storage via une connexion Internet n’est pas toujours réalisable en raison du volume de données, de la disponibilité de la bande passante ou d’autres considérations. Le [service Import/Export Azure Storage](../../storage/common/storage-import-export-service.md) fournit une option basée sur le matériel pour placer/récupérer de grands volumes de données dans Blob Storage. Cela vous permet d’envoyer des lecteurs de disques durs [chiffrés BitLocker](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn306081(v=ws.11)#BKMK_BL2012R2) directement à un centre de données Azure où des opérateurs cloud chargent le contenu sur votre compte de stockage, ou ils peuvent télécharger vos données Azure pour que vos lecteurs vous les retournent. Seuls les disques chiffrés sont acceptés pour ce processus (à l’aide d’une clé BitLocker générée par le service lui-même lors de la configuration du projet). La clé BitLocker est fournie à Azure séparément, fournissant ainsi le partage de clé hors bande.

Dans la mesure où les données en transit peuvent exister dans différents scénarios, il est également utile de savoir que Microsoft Azure utilise la [mise en réseau virtuel](https://azure.microsoft.com/documentation/services/virtual-network/) pour isoler le trafic des clients les uns des autres, en utilisant des mesures telles que les pare-feu de niveau hôte et invité, le filtrage des paquets IP, le blocage des ports et les points de terminaison HTTPS. Toutefois, la plupart des communications internes d’Azure, y compris d’infrastructure à infrastructure et d’infrastructure à client (en local), sont également chiffrées. Un autre scénario important est la communication au sein des centres de données Azure ; Microsoft gère les réseaux pour garantir qu’aucune machine virtuelle ne peut emprunter l’identité ou espionner l’adresse IP d’une autre machine virtuelle. TLS/SSL est utilisé lors de l’accès d’Azure Storage ou des bases de données SQL ou lors de la connexion aux services cloud. Dans ce cas, l’administrateur du client est responsable de l’obtention d’un certificat TLS/SSL et du déploiement sur son infrastructure client. Le trafic de données entre des machines virtuelles dans le même déploiement ou entre des locataires dans un déploiement unique via Microsoft Azure Virtual Network peut être protégé à l’aide des protocoles de communication chiffrée comme HTTPS, SSL/TLS ou d’autres.

En fonction des réponses apportées aux questions de la section [Déterminer les exigences de protection des données](plan-hybrid-identity-design-considerations-dataprotection-requirements.md), vous devez être en mesure de déterminer la façon dont vous souhaitez protéger vos données et dont la solution d’identités hybrides peut vous aider pour ce processus. Le tableau suivant indique les options prises en charge par Azure qui sont disponibles pour chaque scénario de protection des données.

| Option de protection des données | Au repos dans le cloud | Au repos en local | En transit |
| --- | --- | --- | --- |
| Chiffrement de lecteur BitLocker |X |X | |
| SQL Server pour chiffrer les bases de données |X |X | |
| Chiffrement de machine virtuelle à machine virtuelle | | |X |
| SSL/TLS | | |X |
| VPN | | |X |

> [!NOTE]
> Consultez [Conformité par fonctionnalité](https://azure.microsoft.com/support/trust-center/services/) sur [Centre de confidentialité Microsoft Azure](https://azure.microsoft.com/support/trust-center/) pour en savoir plus sur les certifications avec lesquelles chaque service Azure est conforme.
> Étant donné que les options de protection des données utilisent une approche multicouche, la comparaison entre ces options n’est pas pertinente pour cette tâche. Assurez-vous que vous exploitez toutes les options disponibles pour chaque état des données.
>
>

## <a name="define-content-management-options"></a>Définir les options de gestion de contenu

L’un des avantages de l’utilisation d’Azure AD pour gérer une infrastructure d’identités hybrides est que le processus est entièrement transparent du point de vue de l’utilisateur final. L’utilisateur tente d’accéder à une ressource partagée, la ressource requiert l’authentification, l’utilisateur doit envoyer une demande d’authentification à Azure AD afin d’obtenir le jeton et d’accéder à la ressource. Ce processus se déroule intégralement en arrière-plan, sans intervention de l’utilisateur. 

Les organisations préoccupées par la confidentialité des données requièrent généralement la classification des données pour leur solution. Si leur infrastructure locale actuelle est déjà utilisée par la classification des données, il est possible d’utiliser Azure AD comme référentiel principal pour l’identité de l’utilisateur. Le [kit de classification des données](/previous-versions/tn-archive/hh204743(v=technet.10)) pour Windows Server 2012 R2 est un outil commun utilisé en local pour la classification des données. Cet outil peut aider à identifier, classer et protéger les données sur les serveurs de fichiers dans votre cloud privé. Il est également possible d’utiliser la [classification automatique des fichiers](/windows-server/identity/solution-guides/deploy-automatic-file-classification--demonstration-steps-) dans Windows Server 2012 pour accomplir cette tâche.

Si votre organisation ne dispose pas de la classification des données mais a besoin de protéger des fichiers sensibles sans ajouter de nouveaux serveurs en local, il est possible d’utiliser le [service Azure Rights Management](/azure/information-protection/what-is-azure-rms)Microsoft.  Azure RMS utilise des stratégies de chiffrement, d’identité et d’autorisation pour aider à sécuriser vos fichiers et votre courrier électronique, et ce, sur plusieurs appareils (téléphones, tablettes et PC). Azure RMS est un service cloud pour lequel il n’est pas nécessaire de configurer explicitement des approbations avec d’autres organisations avant de pouvoir partager du contenu protégé. Si les organisations disposent déjà de Microsoft 365 ou d’un annuaire Azure AD, la collaboration entre les organisations est automatiquement prise en charge. Vous pouvez également synchroniser uniquement les attributs d’annuaire dont Azure RMS a besoin pour prendre en charge une identité commune pour vos comptes Active Directory locaux, en utilisant les services de synchronisation Azure Active Directory (Azure AD Sync) ou Azure AD Connect.

Une partie essentielle de la gestion de contenu est de comprendre qui a accès à quelle ressource. Une fonctionnalité de journalisation complète est donc importante de la solution de gestion des identités. Azure AD fournit des journaux sur plus de 30 jours, incluant les fonctionnalités suivantes :

* Modifications apportées à l’appartenance à un rôle (p. ex., utilisateur ajouté au rôle Administrateur général)
* Mises à jour des informations d’identification (p. ex., modifications de mot de passe)
* Gestion de domaine (p. ex., vérification d’un domaine personnalisé, suppression d’un domaine)
* Ajout ou suppression d’applications
* Gestion des utilisateurs (p. ex., ajout, suppression, mise à jour d’un utilisateur)
* Ajout ou suppression de licences

> [!NOTE]
> Consultez [Gestion des journaux d’audit et de sécurité Microsoft Azure](https://download.microsoft.com/download/B/6/C/B6C0A98B-D34A-417C-826E-3EA28CDFC9DD/AzureSecurityandAuditLogManagement_11132014.pdf) pour en savoir plus sur les fonctionnalités de journalisation dans Azure.
> En fonction des réponses apportées aux questions de la section [Déterminer les exigences de gestion de contenu](plan-hybrid-identity-design-considerations-contentmgt-requirements.md), vous devez être en mesure de déterminer la façon dont vous souhaitez que le contenu soit géré dans votre solution d’identités hybrides. Bien que toutes les options exposées dans le tableau 6 soient capables de s’intégrer avec Azure AD, il est important de définir lesquelles sont les plus adaptées à vos besoins métier.
>
>

| Options de gestion de contenu | Avantages | Inconvénients |
| --- | --- | --- |
| Centralisation en local (Active Directory Rights Management Server) |Contrôle total sur l’infrastructure de serveurs responsable de la classification des données <br> Fonction intégrée dans Windows Server, sans licence ou abonnement supplémentaire <br> Peut être intégré à Active AD dans un scénario hybride <br> Prise en charge des fonctionnalités de gestion des droits relatifs à l’information (IRM) dans Microsoft Online Services par exemple Exchange Online, SharePoint Online et Microsoft 365 <br> Prise en charge des produits serveur Microsoft locaux, par exemple Exchange Server, SharePoint Server et des serveurs de fichiers exécutant Windows Server et l’infrastructure de classification des fichiers (ICF) |Maintenance plus importante (suivi des mises à jour, de la configuration et des mises à niveau potentielles) étant donné que le service informatique possède le serveur <br> Requiert une infrastructure de serveurs locale<br> N’exploite pas les fonctionnalités Azure en mode natif |
| Centralisation dans le cloud (Azure RMS) |Plus facile à gérer que la solution locale <br> Peut être intégré à Active DS dans un scénario hybride <br>  Entièrement intégré dans Azure AD <br> Ne requiert pas un serveur local pour le déploiement du service <br> Prise en charge des produits serveur Microsoft locaux, par exemple Exchange Server, SharePoint Server et des serveurs de fichiers exécutant Windows Server et l’infrastructure de classification des fichiers (ICF) <br> Le service informatique peut contrôler complètement les clés de ses clients grâce à la fonctionnalité BYOK. |Votre organisation doit posséder un abonnement au cloud prenant en charge RMS <br> Votre organisation doit posséder un répertoire Azure AD pour prendre en charge l’authentification utilisateur pour les services RMS |
| Hybride (Azure RMS intégré à Active Directory Rights Management Server en local) |Ce scénario cumule les avantages des options de centralisation locale et centralisation dans le cloud. |Votre organisation doit posséder un abonnement au cloud prenant en charge RMS <br> Votre organisation doit posséder un répertoire Azure AD pour prendre en charge l’authentification utilisateur pour les services RMS, <br> Requiert une connexion entre le service cloud Azure et l’infrastructure locale |

## <a name="define-access-control-options"></a>Définir les options de contrôle d’accès
En tirant parti des fonctionnalités d’authentification, d’autorisation et de contrôle d’accès disponibles dans Azure AD, vous pouvez permettre à votre entreprise d’utiliser un référentiel central d’identités, tout en autorisant les utilisateurs et partenaires à utiliser l’authentification unique (SSO), comme illustré dans la figure suivante :

![Gestion centralisée](./media/plan-hybrid-identity-design-considerations/centralized-management.png)

Gestion centralisée et intégration complète avec d’autres répertoires

Azure Active Directory fournit l’authentification unique à des milliers d’applications SaaS et à des applications web locales. Consultez l’article [Liste de compatibilité de fédération Azure Active Directory : les fournisseurs d’identité tiers qui peuvent être utilisés pour mettre en œuvre l’authentification unique](how-to-connect-fed-compatibility.md) pour plus d’informations sur l’authentification unique par des tiers testés par Microsoft. Cette fonctionnalité permet aux organisations de mettre en œuvre un éventail de scénarios B2B tout en conservant le contrôle de la gestion de l’identité et de l’accès. Toutefois, pendant le processus de conception B2B, il est important de comprendre la méthode d’authentification qui est utilisée par le partenaire et de valider si cette méthode est prise en charge par Azure. Les méthodes suivantes sont prises en charge par Azure AD :

* SAML (Security Assertion Markup Language)
* OAuth
* Kerberos
* Jetons
* Certificats

> [!NOTE]
> Consultez [Protocoles d’authentification d’Azure Active Directory](/previous-versions/azure/dn151124(v=azure.100)) pour en savoir plus sur chaque protocole et ses fonctionnalités dans Azure.
>
>

À l’aide du support d’Azure AD, les applications métier mobiles peuvent recourir à la même expérience d’authentification facile Mobile Services pour permettre aux employés de se connecter à leurs applications mobiles avec leurs informations d’identification Active Directory d’entreprise. Avec cette fonctionnalité, Azure AD est pris en charge comme fournisseur d’identité dans Mobile Services avec les autres fournisseurs d’identité déjà pris en charge (incluant les comptes Microsoft, ainsi que les ID Facebook, Google et Twitter). Si les applications locales utilisent des informations d’identification de l’utilisateur se trouvant dans les services AD DS de la société, l’accès des partenaires et des utilisateurs provenant du cloud doit être transparent. Vous pouvez gérer le contrôle d’accès conditionnel de l’utilisateur aux applications web (basées sur le cloud), à l’API web, aux services cloud de Microsoft, aux applications SaaS tierces et aux applications clientes (mobiles), et bénéficier de fonctions de sécurité, d’audit et de création de rapports au même endroit. Toutefois, il est recommandé de valider l’implémentation dans un environnement hors production ou avec un nombre limité d’utilisateurs.

> [!TIP]
> Il est important de mentionner qu’Azure AD n’a pas de stratégie de groupe comme les services AD DS. Pour appliquer la stratégie pour les appareils, vous avez besoin d’une solution de gestion des appareils mobiles telle que [Microsoft Intune](/mem/intune/).
>
>

Une fois que l’utilisateur est authentifié à l’aide d’Azure AD, il est important d’évaluer le niveau d’accès dont l’utilisateur dispose. Le niveau d’accès dont dispose l’utilisateur sur une ressource peut varier. Bien qu’Azure AD puisse ajouter une couche de sécurité supplémentaire en contrôlant l’accès à certaines ressources, gardez à l’esprit que la ressource elle-même peut également avoir sa propre liste de contrôle d’accès séparée, comme le contrôle d’accès pour les fichiers situés sur un serveur de fichiers. L’illustration suivante résume les niveaux de contrôle d’accès dont vous pouvez disposer dans un scénario hybride :

![Contrôle d’accès](./media/plan-hybrid-identity-design-considerations/accesscontrol.png)

Chaque interaction du schéma présenté dans la Figure X représente un scénario de contrôle d’accès qui peut être couvert par Azure AD. Une description de chaque scénario est disponible ci-dessous :

1. Accès conditionnel aux applications hébergées en local : Vous pouvez utiliser des appareils inscrits avec des stratégies d’accès pour les applications qui sont configurées pour utiliser les services AD FS avec Windows Server 2012 R2.

2. Contrôle d’accès au portail Azure :  Azure vous permet également de contrôler l’accès au portail à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC). Cette méthode permet aux entreprises de limiter le nombre d’opérations qu’un individu peut effectuer dans le portail Azure. En utilisant Azure RBAC pour contrôler l’accès au portail, les administrateurs informatiques peuvent déléguer l’accès à l’aide des approches suivantes en matière de gestion des accès :

   - Affectation de rôles basés sur les groupes : Vous pouvez affecter l’accès aux groupes Azure AD qui peuvent être synchronisés à partir de votre annuaire Active Directory local. Cela vous permet d’exploiter les investissements existants effectués par votre organisation dans les outils et processus pour la gestion des groupes. Vous pouvez également utiliser la fonctionnalité de gestion déléguée des groupes d’Azure AD Premium.
   - Rôles intégrés dans Azure : Vous pouvez utiliser trois rôles (Propriétaire, Contributeur et Lecteur) pour vous assurer que les utilisateurs et groupes sont uniquement autorisés à effectuer les tâches dont ils ont besoin pour faire leur travail.
   -  Accès granulaire aux ressources : Vous pouvez affecter des rôles aux utilisateurs et aux groupes pour un abonnement spécifique, un groupe de ressources ou une ressource Azure individuelle comme un site web ou une base de données. De cette façon, vous pouvez vous assurer que les utilisateurs ont accès à toutes les ressources dont ils ont besoin, et non aux ressources qu’ils n’ont pas besoin de gérer.

   > [!NOTE]
   > Si vous créez des applications et souhaitez personnaliser le contrôle d’accès pour celles-ci, il est également possible d’utiliser les rôles d’application Azure AD pour l’autorisation. Passez en revue cet [exemple WebApp-RoleClaims-DotNet](https://github.com/AzureADSamples/WebApp-RoleClaims-DotNet) sur la manière de créer votre application de sorte qu’elle utilise cette fonctionnalité.


3. Accès conditionnel pour les applications Microsoft 365 avec Microsoft Intune : Les administrateurs informatiques peuvent configurer des stratégies d’appareil d’accès conditionnel pour sécuriser les ressources d’entreprise, tout en autorisant les travailleurs de l’information sur des appareils compatibles à accéder aux services. 
  
4. Accès conditionnel pour les applications SaaS : [Cette fonctionnalité](https://cloudblogs.microsoft.com/enterprisemobility/2015/06/25/azure-ad-conditional-access-preview-update-more-apps-and-blocking-access-for-users-not-at-work/) vous permet de configurer des règles d’accès pour l’authentification multifacteur pour chaque application et de bloquer l’accès des utilisateurs qui ne sont pas connectés à un réseau approuvé. Vous pouvez appliquer les règles d’authentification multifacteur à tous les utilisateurs affectés à l’application, ou uniquement aux utilisateurs au sein de groupes de sécurité spécifiques. Les utilisateurs qui accèdent à l’application à partir d’une adresse IP qui appartient au réseau de l’organisation peuvent être exclus de l’obligation d’une authentification multifacteur.

Étant donné que les options de contrôle d’accès utilisent une approche multicouche, la comparaison entre ces options n’est pas pertinente pour cette tâche. Assurez-vous que vous exploitez toutes les options disponibles pour chaque scénario dont vous devez contrôler l’accès à vos ressources.

## <a name="define-incident-response-options"></a>Définir les options de réponse aux incidents
Azure AD peut aider le service informatique à identifier les risques de sécurité potentiels dans l’environnement en surveillant l’activité de l’utilisateur. Le service informatique peut utiliser les rapports d’accès et d’utilisation Azure AD pour obtenir une visibilité complète sur l’intégrité et la sécurité du répertoire de votre société. Grâce à ces informations, un administrateur informatique est capable de déterminer plus précisément les risques de sécurité potentiels et donc de les atténuer au maximum.  [L’abonnement Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) intègre un ensemble de rapports de sécurité qui peuvent permettre au service informatique d’obtenir ces informations. Les [rapports Azure AD](../reports-monitoring/overview-reports.md) sont classés comme suit :

* **Rapports d’anomalies** : Contiennent des événements de connexion qui se sont avérés anormaux. L’objectif est de vous faire part de ces activités et de vous permettre de déterminer si un événement est suspect.
* **Rapports d’application intégrée** : Fournissent des insights sur l’utilisation des applications cloud dans votre organisation. Azure Active Directory permet d’intégrer des milliers d'applications du cloud.
* **Rapports d’erreurs** : Indiquent les erreurs qui peuvent survenir lors de la configuration de comptes sur des applications externes.
* **Rapports spécifiques à l’utilisateur** : Affichent les données d’activité relatives aux appareils/connexions d’un utilisateur spécifique.
* **Journaux d’activité** : Contiennent un enregistrement de tous les événements audités durant les 24 dernières heures, 7 derniers jours ou 30 derniers jours, des modifications d’activité de groupes, et des activités d’enregistrement et de réinitialisation de mot de passe.

> [!TIP]
> Autre rapport pouvant également aider l’équipe de réponse aux incidents à travailler sur un dossier : le rapport [Utilisateur avec des informations d’identification volées](https://cloudblogs.microsoft.com/enterprisemobility/2015/06/15/azure-active-directory-premium-reporting-now-detects-leaked-credentials/) . Ce rapport couvre toutes les correspondances entre la liste d’informations d’identification volées et votre locataire.
>


Les autres rapports intégrés importants dans Azure AD qui peuvent être utilisés durant une investigation de réponse aux incidents sont les suivants :

* **Activité de réinitialisation de mot de passe**: fournit à l’administrateur des indications sur la fréquence de réinitialisation de mot de passe dans l’organisation.
* **Activité d’enregistrement de réinitialisation de mot de passe**: fournit des indications sur les utilisateurs qui ont enregistré leurs méthodes pour la réinitialisation de mot de passe, et sur les méthodes qu’ils ont sélectionnées.
* **Activité de groupe** : fournit un historique des modifications apportées au groupe (p. ex., les utilisateurs ajoutés ou supprimés) qui ont été lancées dans le volet d’accès.

En plus de la fonctionnalité de création de rapports de base d’Azure AD Premium que vous pouvez utiliser durant un processus d’investigation de réponse aux incidents, le service informatique peut également tirer parti d’un rapport d’audit pour obtenir les informations suivantes :

* Modifications apportées à l’appartenance à un rôle (par exemple, utilisateur ajouté au rôle Administrateur général)
* Mises à jour des informations d’identification (par exemple, modifications de mot de passe)
* Gestion de domaine (par exemple, vérification d’un domaine personnalisé, suppression d’un domaine)
* Ajout ou suppression d’applications
* Gestion des utilisateurs (par exemple, ajout, suppression, mise à jour d’un utilisateur)
* Ajout ou suppression de licences

Étant donné que les options de réponse aux incidents utilisent une approche multicouche, la comparaison entre ces options n’est pas pertinente pour cette tâche. Assurez-vous que vous exploitez toutes les options disponibles pour chaque scénario pour lequel vous devez utiliser la fonctionnalité de création de rapports Azure AD dans le cadre du processus de réponse aux incidents de votre entreprise.

## <a name="next-steps"></a>Étapes suivantes
[Déterminer les tâches de gestion des identités hybrides](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md)

## <a name="see-also"></a>Voir aussi
[Présentation des considérations relatives à la conception](plan-hybrid-identity-design-considerations-overview.md)
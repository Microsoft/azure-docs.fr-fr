---
title: Azure AD Domain Services pour les fournisseurs de solutions Cloud | Microsoft Docs
description: Découvrez comment activer et gérer des domaines managés Azure Active Directory Domain Services pour les fournisseurs de solutions Azure Cloud
services: active-directory-ds
author: justinha
ms.assetid: 56ccb219-11b2-4e43-9f07-5a76e3cd8da8
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: d8edafff9b6534e5f1ce1c4581595ee187dfd432
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619897"
---
# <a name="azure-active-directory-domain-services-deployment-and-management-for-azure-cloud-solution-providers"></a>Déploiement et gestion d'Azure Active Directory Domain Services pour les fournisseurs de solutions Azure Cloud

Azure CSP (Fournisseurs de solutions Azure Cloud) est un programme destiné aux partenaires Microsoft qui fournit un canal de licence pour divers services cloud Microsoft. Azure CSP permet aux partenaires de gérer les ventes, de prendre en charge les relations relatives à la facturation, de fournir un support technique et sur la facturation et d’être le seul point de contact du client. De plus, Azure CSP fournit un ensemble complet d’outils, notamment un portail libre-service et des API connexes. Ces outils permettent aux partenaires CSP de provisionner et gérer facilement des ressources Azure et de fournir une facturation pour les clients et leurs abonnements.

Le [portail Espace partenaires](/partner-center/azure-plan-lp) est le point d’entrée de tous les partenaires Azure CSP. Il fournit des fonctionnalités de gestion de client riches, un traitement automatisé, etc. Les partenaires Azure CSP peuvent utiliser les fonctionnalités de l’Espace partenaires en utilisant une interface utilisateur basée sur le web ou PowerShell et différents appels d’API.

Le schéma suivant illustre le fonctionnement général du modèle CSP. Ici, Contoso dispose d'un locataire Azure Active Directory (Azure AD). Il a un partenariat avec un fournisseur de solutions cloud, qui déploie et gère les ressources de son abonnement Azure CSP. Contoso peut également avoir des abonnements Azure ordinaires (directs), qui lui sont facturés directement.

![Vue d’ensemble du modèle CSP](./media/csp/csp_model_overview.png)

Le locataire du partenaire CSP présente trois groupes d’agents spéciaux : les agents d’*administration*, les agents de *support technique* et les agents de *vente*.

Le groupe des agents d’*administration* est attribué au rôle d’administrateur de locataire dans le locataire Azure AD de Contoso. Ainsi, un utilisateur appartenant au groupe d’agents d’administration du partenaire CSP a des privilèges d’administration de locataire dans le locataire Azure AD de Contoso.

Quand le partenaire CSP provisionne un abonnement Azure CSP pour Contoso, son groupe d’agents d’administration est affecté au rôle de propriétaire pour cet abonnement. Ainsi, les agents d’administration du partenaire CSP ont les privilèges nécessaires pour provisionner des ressources Azure telles que des machines virtuelles, des réseaux virtuels et Azure AD Domain Services pour le compte de Contoso.

Pour plus d’informations, consultez la [présentation d’Azure CSP](/partner-center/azure-plan-lp).

## <a name="benefits-of-using-azure-ad-ds-in-an-azure-csp-subscription"></a>Avantages d’utiliser Azure AD DS dans un abonnement Azure CSP

Azure Active Directory Domain Services (Azure AD DS) fournit des services de domaine managés, comme la jonction de domaine, la stratégie de groupe, le protocole LDAP, et l’authentification Kerberos/NTLM entièrement compatible avec Windows Server Active Directory Domain Service. Au fil des décennies, de nombreuses applications ont été générées pour fonctionner avec Active Directory à l’aide de ces fonctionnalités. De nombreux éditeurs de logiciels indépendants (ISV) ont généré et déployé des applications dans les locaux de leurs clients. La prise en charge de ces applications est difficile, car elle implique l’accès aux différents environnements dans lesquels les applications sont déployées. Avec des abonnements Azure CSP, vous avez une solution plus simple avec l’adaptabilité et la flexibilité d’Azure.

Azure AD DS prend en charge les abonnements Azure CSP. Vous pouvez déployer votre application dans un abonnement Azure CSP lié à l’annuaire Azure AD de votre locataire. Ainsi, vos employés (équipes du support) peuvent gérer, administrer et assurer la maintenance des machines virtuelles sur lesquelles votre application est déployée à l’aide des informations d’identification d’entreprise de votre organisation.

Vous pouvez également déployer un domaine managé Azure AD DS dans le locataire Azure AD de votre client. Votre application est alors connectée au domaine managé de votre client. Les fonctionnalités au sein de votre application qui s’appuient sur Kerberos / NTLM, LDAP ou [l’API System.DirectoryServices](/dotnet/api/system.directoryservices) fonctionnent de manière fluide dans le domaine de votre client. Les clients profitent de la consommation de votre application en tant que service, sans avoir à se soucier de la gestion de l’infrastructure sur laquelle l’application est déployée.

Vous acquittez toute la facturation des ressources Azure que vous utilisez dans cet abonnement, y compris Azure AD DS. Vous contrôlez en totalité la relation avec le client en ce qui concerne les ventes, la facturation, le support technique, etc. Grâce à la flexibilité de la plateforme Azure CSP, une petite équipe d’agents de support peut prendre en charge de nombreux clients pour lesquels des instances de votre application sont déployées.

## <a name="csp-deployment-models-for-azure-ad-ds"></a>Modèles de déploiement CSP pour Azure AD DS

Il existe deux façons d’utiliser Azure AD DS avec un abonnement Azure CSP. Choisissez celui qui convient, suivant l’importance qu’accordent vos clients à la sécurité et à la simplicité.

### <a name="direct-deployment-model"></a>Modèle de déploiement direct

Dans ce modèle de déploiement, Azure AD DS est activé au sein d’un réseau virtuel appartenant à l’abonnement Azure CSP. Les agents d’administration du partenaire CSP ont les privilèges suivants :

* Privilèges d’*administrateur général* dans le locataire Azure AD du client.
* Privilèges de *propriétaire d’abonnement* sur l’abonnement Azure CSP.

![Modèle de déploiement direct](./media/csp/csp_direct_deployment_model.png)

Dans ce modèle de déploiement, les agents d’administration du fournisseur CSP peuvent gérer les identités pour le client. Ces agents d’administration peuvent effectuer différentes tâches telles que l'approvisionnement de nouveaux utilisateurs et groupes ou l'ajout d'applications dans le locataire Azure AD du client.

Ce modèle de déploiement peut être adapté aux petites organisations qui n’ont pas d’administrateur d’identité dédié ou qui préfèrent que le partenaire CSP gère les identités pour leur compte.

### <a name="peered-deployment-model"></a>Modèle de déploiement appairé

Dans ce modèle de déploiement, Azure AD DS est activé au sein d’un réseau virtuel appartenant au client, un abonnement Azure direct payé par le client. Le partenaire CSP peut déployer des applications au sein d’un réseau virtuel appartenant à l’abonnement CSP du client. Les réseaux virtuels peuvent être connectés à l’aide du peering de réseau virtuel Azure.

Avec ce déploiement, les charges de travail ou applications déployées par le partenaire CSP dans l’abonnement Azure CSP peuvent se connecter au domaine managé du client provisionné dans l’abonnement Azure direct du client.

![Modèle de déploiement appairé](./media/csp/csp_peered_deployment_model.png)

Ce modèle de déploiement fournit une séparation des privilèges et permet aux agents de support technique du partenaire CSP d’administrer l’abonnement Azure et de déployer et gérer les ressources qu’il contient. Toutefois, les agents de support technique du partenaire CSP ne sont pas tenus de disposer de privilèges d’administrateur général sur l’annuaire Azure AD du client. Les administrateurs d’identité du client peuvent continuer à gérer les identités pour leur organisation.

Ce modèle de déploiement peut être adapté aux scénarios où un éditeur de logiciels indépendant fournit une version hébergée de son application locale, qui doit également se connecter à l’annuaire Azure AD du client.

## <a name="administer-azure-ad-ds-in-csp-subscriptions"></a>Administrer Azure AD DS dans les abonnements CSP

Tenez compte des considérations importantes suivantes quand vous administrez un domaine managé dans un abonnement Azure CSP :

* **Les agents d’administration CSP peuvent approvisionner un domaine managé à l’aide de leurs informations d’identification :** Azure AD DS prend en charge les abonnements Azure CSP. Les utilisateurs appartenant au groupe d’agents d’administration d’un partenaire CSP peuvent approvisionner un nouveau domaine managé.

* **Les fournisseurs de solutions cloud peuvent programmer au moyen d’un script la création de domaines managés pour leurs clients à l’aide de PowerShell :** Pour plus d'informations, consultez [Activer Azure AD DS à l’aide de PowerShell](powershell-create-instance.md).

* **Les agents d’administration CSP ne peuvent pas effectuer de tâches de gestion en continu sur le domaine managé à l’aide de leurs informations d’identification :** les utilisateurs administrateurs CSP ne peuvent pas effectuer de tâches de gestion courantes dans le domaine managé à l’aide de leurs informations d’identification. Ces utilisateurs étant externes au locataire Azure AD du client, leurs informations d’identification ne sont pas disponibles dans le locataire Azure AD du client. Azure AD DS n’a pas accès aux hachages de mot de passe Kerberos et NTLM pour ces utilisateurs. Dès lors, les utilisateurs ne peuvent pas être authentifiés sur les domaines managés.

  > [!WARNING]
  > Vous devez créer un compte d’utilisateur dans l’annuaire du client pour effectuer des tâches d’administration courantes sur le domaine managé.
  >
  > Vous ne pouvez pas vous connecter au domaine managé à l’aide des informations d’identification d’un utilisateur administrateur CSP. Pour ce faire, utilisez les informations d’identification d’un compte d’utilisateur appartenant au locataire Azure AD du client. Vous avez besoin de ces informations d’identification pour les tâches telles que la jonction de machines virtuelles au domaine managé, l’administration du système DNS ou l’administration de la stratégie de groupe.

* **Le compte d’utilisateur créé pour l’administration courante doit être ajouté au groupe *Administrateurs AAD DC* :** le groupe *Administrateurs AAD DC* dispose de privilèges pour effectuer certaines tâches d’administration déléguée dans le domaine managé. Ces tâches comprennent la configuration du système DNS, la création d’unités d’organisation et l’administration de la stratégie de groupe.
    
    Pour qu’un partenaire CSP effectue ces tâches sur un domaine managé, un compte d’utilisateur doit être créé dans le locataire Azure AD du client. Les informations d’identification pour ce compte doivent être partagées avec les agents d’administration du partenaire CSP. De plus, ce compte d’utilisateur doit être ajouté au groupe *Administrateurs AAD DC* pour qu’il puisse effectuer des tâches de configuration sur le domaine managé.

## <a name="next-steps"></a>Étapes suivantes

Pour commencez, [inscrivez-vous au programme Fournisseur de solutions Azure Cloud](/partner-center/enrolling-in-the-csp-program). Vous pourrez ensuite Activer Azure AD Domain Services à l’aide du [portail Azure](tutorial-create-instance.md) ou d'[Azure PowerShell](powershell-create-instance.md).
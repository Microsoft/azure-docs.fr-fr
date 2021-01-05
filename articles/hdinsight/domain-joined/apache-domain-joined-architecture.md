---
title: Architecture Azure HDInsight avec le pack Sécurité Entreprise
description: Découvrez comment planifier la sécurité d’Azure HDInsight avec le pack Sécurité Entreprise.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: omidm
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 03/11/2020
ms.openlocfilehash: 86d951089e4247d9b959476c812b98e170d92bd8
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547977"
---
# <a name="use-enterprise-security-package-in-hdinsight"></a>Utiliser le pack Sécurité Entreprise dans HDInsight

Le cluster Azure HDInsight standard est un cluster à un seul utilisateur. Il convient à la plupart des entreprises qui font appel à des équipes d’application plus petites pour créer leurs charges de travail de données importantes. Chaque utilisateur peut créer un cluster dédié à la demande, et le détruire quand il n’est plus nécessaire.

De nombreuses entreprises se sont orientées vers un modèle dans lequel les clusters sont gérés par les équipes informatiques, et partagés par plusieurs équipes d'application. Ces entreprises de plus grande taille ont besoin d’un accès multi-utilisateur à chaque cluster dans Azure HDInsight.

HDInsight s’appuie sur le fournisseur d’identité répandu, Active Directory, de façon managée. En intégrant HDInsight à [Azure Active Directory Domain Services (Azure AD DS)](../../active-directory-domain-services/overview.md), vous pouvez accéder aux clusters en utilisant vos informations d’identification de domaine.

Les machines virtuelles dans HDInsight sont joints à votre domaine fourni. Ainsi, tous les services exécutés sur HDInsight (Apache Ambari, serveur Apache Hive, Apache Ranger, serveur Apache Spark Thrift, etc.) fonctionnent de manière transparente pour l’utilisateur authentifié. Les administrateurs peuvent ensuite créer des stratégies d’autorisation strictes à l’aide d’Apache Ranger, de façon à fournir un contrôle de l’accès en fonction du rôle pour les ressources du cluster.

## <a name="integrate-hdinsight-with-active-directory"></a>Intégrer HDInsight avec Active Directory

Apache Hadoop open source s'appuie sur le protocole Kerberos pour l'authentification et la sécurité. Ainsi, les nœuds de cluster HDInsight avec le pack Sécurité Entreprise (ESP) sont joints à un domaine qui est géré par Azure AD DS. La sécurité Kerberos est configurée pour les composants Hadoop sur le cluster.

Les éléments suivants sont créés automatiquement :

- Un principal de service pour chaque composant Hadoop
- Un principal de machine pour chaque machine jointe au domaine
- Une unité d'organisation pour chaque cluster afin de stocker ces principaux de service et de machine

Pour résumer, vous devez configurer un environnement avec :

- Un domaine Active Directory (managé par Azure AD DS). **Le nom de domaine doit comporter 39 caractères au maximum pour fonctionner avec Azure HDInsight.**
- LDAP sécurisé (LDAPS) activé dans Azure AD DS.
- Une connectivité réseau appropriée, du réseau virtuel HDInsight au réseau virtuel Azure AD DS, si vous choisissez des réseaux virtuels distincts pour ces services. Une machine virtuelle à l’intérieur du réseau virtuel HDInsight doit avoir une visibilité directe sur Azure AD DS via le peering de réseau virtuel. Si HDInsight et Azure AD DS sont déployés au sein du même réseau virtuel, la connectivité est automatiquement fournie et aucune action supplémentaire n'est nécessaire.

## <a name="set-up-different-domain-controllers"></a>Configurer différents contrôleurs de domaine

Pour l’instant, HDInsight prend uniquement en charge Azure AD DS comme étant le contrôleur de domaine principal que le cluster utilise pour la communication Kerberos. Par contre, d’autres configurations Active Directory complexes sont possibles, tant que ce type de configuration aboutit à l’activation d’Azure AD DS pour l’accès de HDInsight.

### <a name="azure-active-directory-domain-services"></a>Azure Active Directory Domain Services

[Azure AD DS](../../active-directory-domain-services/overview.md) fournit un domaine managé qui est entièrement compatible avec Windows Server Active Directory. Microsoft effectue la gestion, les mises à jour correctives et la surveillance du domaine dans une configuration à haute disponibilité. Vous pouvez déployer votre cluster sans vous préoccuper de la maintenance des contrôleurs de domaine.

Les utilisateurs, groupes et mots de passe sont synchronisés à partir d'Azure AD. La synchronisation unidirectionnelle à partir de votre instance Azure AD dans Azure AD DS permet aux utilisateurs de se connecter au cluster en utilisant les mêmes informations d’identification d’entreprise.

Pour plus d’informations, consultez [Configurer des clusters HDInsight avec ESP en utilisant Azure AD DS](./apache-domain-joined-configure-using-azure-adds.md).

### <a name="on-premises-active-directory-or-active-directory-on-iaas-vms"></a>Service Active Directory local ou Active Directory sur des machines virtuelles IaaS

Si vous disposez d’une instance Active Directory locale ou de configurations Active Directory plus complexes pour votre domaine, vous pouvez synchroniser ces identités sur Azure AD au moyen d’Azure AD Connect. Vous pouvez ensuite activer Azure AD DS sur ce locataire Active Directory.

Kerberos s’appuyant sur les hachages de mot de passe, vous devez [activer la synchronisation du hachage de mot de passe sur Azure AD DS](../../active-directory-domain-services/tutorial-create-instance.md).

Si vous utilisez la fédération avec les services AD FS (Active Directory Federation Services), vous devez activer la synchronisation du hachage de mot de passe. (Pour connaître la configuration recommandée, consultez [cette vidéo](https://youtu.be/qQruArbu2Ew).) La synchronisation du hachage de mot de passe facilite la récupération d'urgence en cas de défaillance de votre infrastructure AD FS, et offre également une protection contre les fuites d'informations d'identification. Pour plus d’informations, consultez [Activer la synchronisation du hachage de mot de passe avec la synchronisation Azure AD Connect](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md).

L'utilisation exclusive du service Active Directory local ou d'Active Directory sur des machines virtuelles IaaS, sans Azure AD ni Azure AD DS, n'est pas une configuration prise en charge pour les clusters HDInsight avec ESP.

Si vous rencontrez des problèmes d’authentification alors que la fédération est utilisée et que les hachages de mot de passe sont correctement synchronisés, vérifiez que l’authentification par mot de passe cloud est activée pour le principal de service PowerShell. Si ce n'est pas le cas, vous devez définir une [stratégie de découverte du domaine d'accueil](../../active-directory/manage-apps/configure-authentication-for-federated-users-portal.md) pour votre locataire Azure AD. Pour vérifier et définir la stratégie de découverte du domaine d'accueil :

1. Installez le [module Azure AD PowerShell](/powershell/azure/active-directory/install-adv2) en préversion.

   ```powershell
   Install-Module AzureAD
   ```

2. Connectez-vous en utilisant les informations d'identification d'un administrateur général (administrateur de locataire).

   ```powershell
   Connect-AzureAD
   ```

3. Assurez-vous que le principal de service Microsoft Azure PowerShell a déjà été créé.

   ```powershell
   Get-AzureADServicePrincipal -SearchString "Microsoft Azure Powershell"
   ```

4. Si ce n'est pas le cas, créez le principal de service.

   ```powershell
   $powershellSPN = New-AzureADServicePrincipal -AppId 1950a258-227b-4e31-a9cf-717495945fc2
   ```

5. Créez la stratégie et joignez-la à ce principal de service.

   ```powershell
    # Determine whether policy exists
    Get-AzureADPolicy | Where {$_.DisplayName -eq "EnableDirectAuth"}

    # Create if not exists
    $policy = New-AzureADPolicy `
        -Definition @('{"HomeRealmDiscoveryPolicy":{"AllowCloudPasswordValidation":true}}') `
        -DisplayName "EnableDirectAuth" `
        -Type "HomeRealmDiscoveryPolicy"

    # Determine whether a policy for the service principal exist
    Get-AzureADServicePrincipalPolicy `
        -Id $powershellSPN.ObjectId

    # Add a service principal policy if not exist
    Add-AzureADServicePrincipalPolicy `
        -Id $powershellSPN.ObjectId `
        -refObjectID $policy.ID
   ```

## <a name="next-steps"></a>Étapes suivantes

- [Configurer des clusters HDInsight avec ESP](apache-domain-joined-configure-using-azure-adds.md)
- [Configurer des stratégies Apache Hive pour des clusters HDInsight avec ESP](apache-domain-joined-run-hive.md)
- [Gérer des clusters HDInsight avec ESP](apache-domain-joined-manage.md)
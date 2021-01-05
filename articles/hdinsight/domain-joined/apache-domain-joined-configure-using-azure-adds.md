---
title: Configurer des clusters pour l’intégration à Active Directory
titleSuffix: Azure HDInsight
description: Découvrez comment configurer un cluster HDInsight intégré à Active Directory en utilisant Azure Active Directory Domain Services et la fonctionnalité Pack Sécurité Entreprise.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: seodec18,seoapr2020, contperf-fy21q2
ms.date: 10/30/2020
ms.openlocfilehash: 248d909e633607271aec7c2c9b8a373f111f7d98
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031480"
---
# <a name="configure-hdinsight-clusters-for-active-directory-integration-with-enterprise-security-package"></a>Configurer des clusters HDInsight pour l’intégration à Active Directory avec le Pack Sécurité Entreprise

Dans cet article, vous allez découvrir comment créer et configurer un cluster HDInsight intégré à Active Directory en utilisant une fonctionnalité appelée Pack Sécurité Entreprise (ESP), Azure Active Directory Domain Services (Azure AD-DS) et votre annuaire Active Directory local existant.

Si vous recherchez un tutoriel sur l’installation et la configuration d’un domaine dans Azure et la création d’un cluster compatible ESP, consultez [Créer et configurer des clusters Pack Sécurité Entreprise dans Azure HDInsight](apache-domain-joined-create-configure-enterprise-security-cluster.md).

## <a name="background"></a>Arrière-plan

Pack Sécurité Entreprise (ESP) fournit l’intégration à Active Directory pour Azure HDInsight. Cette intégration permet aux utilisateurs du domaine d’utiliser leurs informations d’identification de domaine pour s’authentifier auprès des clusters HDInsight et d’exécuter des tâches de Big Data.

> [!NOTE]  
> La protection ESP est généralement disponible dans HDInsight 3.6 et 4.0 pour les types de clusters suivants : Apache Spark, Interactive, Hadoop et HBase. Le type de cluster ESP pour Apache Kafka est en préversion et ne bénéficie que d’une prise en charge au mieux. Les clusters ESP créés avant la date de disponibilité générale ESP (1er octobre 2018) ne sont pas pris en charge.

## <a name="prerequisites"></a>Prérequis

Quelques conditions préalables doivent être remplies pour pouvoir créer un cluster HDInsight compatible ESP :

- Activez Azure AD-DS.
- Vérifiez l’état d’intégrité d’Azure AD-DS pour vous assurer que la synchronisation est terminée.
- Créez et autorisez une identité managée.
- Effectuez la configuration de la mise en réseau pour le DNS et les problèmes connexes.

Chacun de ces éléments sera traité en détail ci-dessous.

### <a name="enable-azure-ad-ds"></a>Activer Azure AD DS

L’activation d’Azure AD DS est une condition préalable à la création d’un cluster HDInsight avec ESP. Pour plus d’informations, consultez [Activer Azure Active Directory Domain Services à l’aide du Portail Azure](../../active-directory-domain-services/tutorial-create-instance.md).

Quand Azure AD DS est activé, tous les utilisateurs et les objets commencent la synchronisation d’Azure Active Directory (Azure AD) vers Azure AD DS par défaut. La longueur de l’opération de synchronisation varie selon le nombre d’objets dans Azure AD. La synchronisation peut prendre plusieurs jours pour des centaines de milliers d’objets.

Le nom de domaine que vous utilisez avec Azure AD DS doit comporter 39 caractères au maximum pour fonctionner avec HDInsight.

Vous pouvez choisir de synchroniser uniquement les groupes qui ont besoin d’accéder aux clusters HDInsight. Cette option de synchroniser uniquement certains groupes est appelée *synchronisation délimitée*. Pour obtenir des instructions, consultez [Configurer la synchronisation étendue depuis Azure AD vers votre domaine managé](../../active-directory-domain-services/scoped-synchronization.md).

Lorsque vous activez le protocole LDAP sécurisé, indiquez le nom de domaine dans le nom de l’objet, et l’autre nom de l’objet dans le certificat. Si votre nom de domaine est *contoso100.onmicrosoft.com*, assurez-vous que ce nom exact existe dans le nom de l’objet et l’autre nom de l’objet de votre certificat. Pour plus d’informations, consultez [Configurer le protocole LDAP sécurisé pour un domaine managé Azure AD DS](../../active-directory-domain-services/tutorial-configure-ldaps.md).

L’exemple suivant crée un certificat auto-signé. Le nom de domaine *contoso100.onmicrosoft.com* figure à la fois dans `Subject` (nom de l’objet) et `DnsName` (autre nom de l’objet).

```powershell
$lifetime=Get-Date
New-SelfSignedCertificate -Subject contoso100.onmicrosoft.com `
  -NotAfter $lifetime.AddDays(365) -KeyUsage DigitalSignature, KeyEncipherment `
  -Type SSLServerAuthentication -DnsName *.contoso100.onmicrosoft.com, contoso100.onmicrosoft.com
```

> [!NOTE]  
> Seuls les administrateurs de locataires disposent des privilèges pour activer Azure AD DS. Si l’espace de stockage en cluster correspond à Azure Data Lake Storage Gen1 ou Gen2, vous ne devez désactiver Azure AD Multi-Factor Authentication que pour les utilisateurs qui accèdent au cluster à l’aide d’une authentification Kerberos de base.
>
> Vous pouvez utiliser des [adresses IP approuvées](../../active-directory/authentication/howto-mfa-mfasettings.md#trusted-ips) ou l’[accès conditionnel](../../active-directory/conditional-access/overview.md) afin de désactiver l’authentification multifacteur pour des utilisateurs spécifiques, *seulement* quand ils accèdent à la plage d’adresses IP du réseau virtuel du cluster HDInsight. Si vous utilisez l’accès conditionnel, vérifiez que le point de terminaison de service Active Directory est activé sur le réseau virtuel HDInsight.
>
> Si l’espace de stockage en cluster correspond au stockage Blob Azure, ne désactivez pas l’authentification multifacteur.

### <a name="check-azure-ad-ds-health-status"></a>Vérifier l’état d’intégrité d’Azure AD DS

Affichez l’état d’intégrité d’Azure Active Directory Domain Services en sélectionnant **Intégrité** dans la catégorie **Gérer**. Assurez-vous que l’état d’Azure AD DS est vert (en cours d’exécution) et que la synchronisation est terminée.

![Intégrité d’Azure AD DS](./media/apache-domain-joined-configure-using-azure-adds/hdinsight-aadds-health.png)

### <a name="create-and-authorize-a-managed-identity"></a>Créer et autoriser une identité managée

Utilisez une *identité managée attribuée à l’utilisateur* pour simplifier les opérations sécurisées de services de domaine. Lorsque vous affectez le rôle **Contributeur des services de domaine HDInsight** à l’identité managée, il peut lire, créer, modifier et supprimer des opérations de services de domaine.

Certaines opérations de services de domaine, telles que la création d’unités d’organisation ou de principaux de service, sont nécessaires pour le Pack Sécurité Entreprise HDInsight. Vous pouvez créer des identités managées dans n’importe quel abonnement. Pour plus d’informations sur les identités managées en général, consultez l’article [Identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md). Pour plus d’informations sur le fonctionnement des identités managées dans Azure HDInsight, consultez [Identités managées dans Azure HDInsight](../hdinsight-managed-identities.md).

Pour configurer des clusters ESP, créez une identité managée affectée à l’utilisateur, si vous n’en possédez pas déjà une. Voir [`Create, list, delete, or assign a role to a user-assigned managed identity by using the Azure portal`](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md).

Ensuite, affectez le rôle **Contributeur HDInsight Domain Services** à l’identité managée dans **Contrôle d’accès** pour Azure AD DS. Vous avez besoin de privilèges d’administrateur Azure AD DS pour effectuer cette attribution de rôle.

![Contrôle d’accès Azure Active Directory Domain Services](./media/apache-domain-joined-configure-using-azure-adds/hdinsight-configure-managed-identity.png)

L’attribution du rôle **Contributeur aux services de domaine HDInsight** permet de s’assurer que l’identité a un accès approprié (`on behalf of`) pour effectuer les opérations de services de domaine sur le domaine Azure AD DS. Ces opérations incluent la création et la suppression d’unités d’organisation.

Une fois que le rôle est attribué à l’identité managée, l’administrateur Azure AD DS gère qui l’utilise. Tout d’abord, l’administrateur sélectionne l’identité managée dans le portail. Il sélectionne ensuite **Contrôle d’accès (IAM)** sous **Vue d’ensemble**. L’administrateur attribue le rôle **Opérateur d’identité managée** aux utilisateurs ou groupes qui souhaitent créer des clusters ESP.

Par exemple, l’administrateur Azure AD DS peut attribuer ce rôle au groupe **MarketingTeam** pour l’identité managée **sjmsi**. L’image suivante en contient un exemple. Cette attribution garantit que les bonnes personnes de l’organisation peuvent utiliser l’identité managée pour créer des clusters ESP.

![Attribution du rôle Opérateur d'identité managée HDInsight](./media/apache-domain-joined-configure-using-azure-adds/hdinsight-managed-identity-operator-role-assignment.png)

### <a name="network-configuration"></a>Configuration réseau

> [!NOTE]  
> Azure AD DS doit être déployé dans un réseau virtuel basé sur Azure Resource Manager. Les réseaux virtuels classiques ne sont pas pris en charge pour Azure AD DS. Pour plus d’informations, consultez [Activer Azure Active Directory Domain Services à l’aide du Portail Azure](../../active-directory-domain-services/tutorial-create-instance-advanced.md#create-and-configure-the-virtual-network).

Activez Azure AD DS. Un serveur DNS (Domain Name System) local s’exécute alors sur les machines virtuelles Active Directory. Configurez votre réseau virtuel Azure AD DS pour utiliser ces serveurs DNS personnalisés. Pour rechercher les adresses IP qui conviennent, sélectionnez **Propriétés** dans la catégorie **Gérer** et examinez ce qui est répertorié sous **ADRESSE IP SUR LE RÉSEAU VIRTUEL**.

![Rechercher les adresses IP des serveurs DNS locaux](./media/apache-domain-joined-configure-using-azure-adds/hdinsight-aadds-dns1.png)

Modifiez la configuration des serveurs DNS dans le réseau virtuel Azure AD DS. Pour utiliser ces adresses IP personnalisées, sélectionnez **Serveurs DNS** dans la catégorie **Paramètres**. Sélectionnez ensuite l’option **Personnalisé**, entrez la première adresse IP dans la zone de texte, puis sélectionnez **Enregistrer**. Ajoutez des adresses IP supplémentaires en suivant la même procédure.

![Mise à jour de la configuration DNS du réseau virtuel](./media/apache-domain-joined-configure-using-azure-adds/hdinsight-aadds-vnet-configuration.png)

Il est plus facile de placer l’instance Azure AD DS et le cluster HDInsight dans le même réseau virtuel Azure. Si vous envisagez d’utiliser différents réseaux virtuels, vous devez les appairer de manière à ce que le contrôleur de domaine soit visible pour les machines virtuelles HDInsight. Pour en savoir plus, consultez [Peering de réseaux virtuels](../../virtual-network/virtual-network-peering-overview.md).

Une fois les réseaux virtuels appairés, configurez le réseau virtuel HDInsight pour utiliser un serveur DNS personnalisé. Entrez ensuite les adresses IP privées Azure AD DS en tant qu’adresses de serveur DNS. Lorsque les deux réseaux virtuels utilisent les mêmes serveurs DNS, votre nom de domaine personnalisé est résolu vers l’adresse IP qui convient et accessible à partir de HDInsight. Par exemple, si votre nom de domaine est `contoso.com`, après cette étape, `ping contoso.com` doit être résolu à la bonne adresse IP Azure AD DS.

![Configuration des serveurs DNS personnalisés pour un réseau virtuel appairé](./media/apache-domain-joined-configure-using-azure-adds/hdinsight-aadds-peered-vnet-configuration.png)

Si vous utilisez des règles de groupe de sécurité réseau (NSG) dans votre sous-réseau HDInsight, vous devez autoriser les [adresses IP nécessaires](../hdinsight-management-ip-addresses.md) pour le trafic entrant et le trafic sortant.

Pour tester la configuration de votre réseau, joignez une machine virtuelle Windows au réseau virtuel/sous-réseau HDInsight et effectuez un test ping sur le nom de domaine. (Il doit être résolu en une adresse IP.) Exécutez **ldp.exe** pour accéder au domaine Azure AD DS. Joignez ensuite cette machine virtuelle Windows au domaine pour confirmer que tous les appels RPC requis aboutissent entre le client et le serveur.

Utilisez **nslookup** pour confirmer l’accès réseau à votre compte de stockage, ou toute base de données externe que vous pouvez utiliser (par exemple, metastore Hive externe ou Ranger DB). Assurez-vous que les [ports requis](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) sont autorisés dans les règles de groupe de sécurité réseau du sous-réseau Azure AD DS, si un groupe de sécurité réseau sécurise Azure AD DS. Si la jonction de domaine de cette machine virtuelle Windows est réussie, vous pouvez passer à l’étape suivante et créer des clusters ESP.

## <a name="create-an-hdinsight-cluster-with-esp"></a>Créer un cluster HDInsight avec ESP

Après avoir correctement configuré les étapes précédentes, l’étape suivante consiste à créer le cluster HDInsight avec ESP activé. Lorsque vous créez un cluster HDInsight, vous pouvez activer le pack Sécurité Entreprise sous l’onglet **Sécurité + réseau**. Pour un modèle Azure Resource Manager à déployer, utilisez l’expérience du portail une seule fois. Téléchargez ensuite le modèle prérempli sur la page **Vérifier + créer** pour une réutilisation ultérieure.

Vous pouvez également activer la fonctionnalité [Broker d’ID HDInsight](identity-broker.md) lors de la création du cluster. La fonctionnalité Broker d’ID vous permet de vous connecter à Ambari à l’aide de l’authentification multifacteur et d’obtenir les tickets Kerberos requis sans avoir besoin de hachages de mot de passe dans Azure AD DS.

> [!NOTE]  
> Les six premiers caractères des noms de cluster ESP doivent être uniques dans votre environnement. Par exemple, si vous disposez de plusieurs clusters ESP situés dans différents réseaux virtuels, choisissez une convention d’affectation de noms garantissant que les six premiers caractères des noms de cluster sont uniques.

![Validation de domaine pour le pack Sécurité Entreprise Azure HDInsight](./media/apache-domain-joined-configure-using-azure-adds/azure-portal-cluster-security-networking-esp.png)

Après l’activation d’ESP, les erreurs de configuration courantes associées à Azure AD DS sont automatiquement détectées et validées. Après avoir corrigé ces erreurs, vous pouvez passer à l’étape suivante.

![Pack Sécurité Entreprise Azure HDInsight – Échec de validation de domaine](./media/apache-domain-joined-configure-using-azure-adds/azure-portal-cluster-security-networking-esp-error.png)

Quand vous créez un cluster HDInsight avec ESP, vous devez fournir les paramètres suivants :

* **Utilisateur administrateur de cluster** : Choisissez un administrateur pour votre cluster à partir de votre instance Azure AD DS synchronisée. Ce compte de domaine doit être déjà synchronisé et disponible dans Azure AD DS.

* **Groupe d’accès au cluster** : Les groupes de sécurité dont vous voulez que les utilisateurs puissent avoir accès au cluster et le synchroniser doivent être disponibles dans Azure AD DS. Le groupe HiveUsers en est un exemple. Pour plus d’informations, consultez [Créer un groupe et ajouter des membres dans Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

* **URL LDAPS** : par exemple `ldaps://contoso.com:636`.

Vous pouvez choisir l’identité managée que vous avez créée dans la liste déroulante **Identité managée affectée par l’utilisateur** lors de la création d’un nouveau cluster.

![Identité managée Azure HDInsight ESP Active Directory Domain Services](./media/apache-domain-joined-configure-using-azure-adds/azure-portal-cluster-security-networking-identity.png).

## <a name="next-steps"></a>Étapes suivantes

* Pour configurer des stratégies Hive et exécuter des requêtes Hive, consultez [Configurer des stratégies Apache Hive pour des clusters HDInsight avec ESP](apache-domain-joined-run-hive.md).
* Pour vous connecter à des clusters HDInsight avec ESP à l’aide de SSH, consultez [Utiliser SSH avec Apache Hadoop Linux sur HDInsight à partir de Linux, Unix ou OS X](../hdinsight-hadoop-linux-use-ssh-unix.md#authentication-domain-joined-hdinsight).

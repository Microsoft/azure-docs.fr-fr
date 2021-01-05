---
title: Didacticiel - Configurer HTTPS sur un domaine personnalisé Azure CDN | Microsoft Docs
description: Dans ce didacticiel, vous allez apprendre à activer et désactiver HTTPS sur votre domaine personnalisé de point de terminaison Microsoft Azure CDN.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: 10337468-7015-4598-9586-0b66591d939b
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/1/2019
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 12cf4b029424bbbdb7449e6e1d04684ed485ef97
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92779014"
---
# <a name="tutorial-configure-https-on-an-azure-cdn-custom-domain"></a>Tutoriel : Configurer HTTPS sur un domaine personnalisé Azure CDN

Ce didacticiel montre comment activer le protocole HTTPS pour un domaine personnalisé associé à un point de terminaison Azure CDN. En utilisant le protocole HTTPS sur votre domaine personnalisé (par exemple, https :\//www.contoso.com), vous vous assurez que vos données sensibles sont remises en toute sécurité via le chiffrement TLS/SSL lors de l’envoi sur Internet. Lorsque votre navigateur web est connecté à un site web via HTTPS, ce protocole valide le certificat de sécurité du site et vérifie qu’il est fourni par une autorité de certification légitime. Ce processus assure la sécurité et protège également vos applications web contre les attaques.

Azure CDN prend en charge HTTPS sur le nom d’hôte d’un point de terminaison CDN par défaut. Par exemple, si vous créez un point de terminaison CDN (tel que https:\//contoso.azureedge.net), HTTPS est automatiquement activé.  

Voici quelques-uns des attributs clés de la fonctionnalité HTTPS personnalisée :

- Aucun coût supplémentaire : l’acquisition ou le renouvellement de certificat n’entraîne aucun coût supplémentaire, tout comme le trafic HTTPS. Vous payez uniquement pour l’acheminement des Go du CDN.

- Activation simple : le provisionnement en un clic est disponible à partir du [portail Azure](https://portal.azure.com). Vous pouvez également utiliser l’API REST ou d’autres outils de développeur pour activer la fonctionnalité.

- La gestion complète de certificats est disponible : l’approvisionnement et la gestion de tous les certificats sont assurés pour vous. Les certificats sont automatiquement provisionnés et renouvelés avant expiration, ce qui élimine les risques d’interruption de service en raison d’une expiration de certificat.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Activer le protocole HTTPS sur votre domaine personnalisé.
> - utiliser un certificat géré par le CDN ; 
> - utiliser votre propre certificat ;
> - Valider le domaine
> - Désactiver le protocole HTTPS sur votre domaine personnalisé.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)] 

Avant d’effectuer les étapes de ce tutoriel, vous devez d’abord créer un profil CDN et au moins un point de terminaison CDN. Pour plus d’informations, consultez [Démarrage rapide : Créer un point de terminaison et un profil de réseau de distribution de contenu Azure](cdn-create-new-endpoint.md).

En outre, vous devez associer un domaine personnalisé Azure CDN sur votre point de terminaison CDN. Pour plus d’informations, consultez [Didacticiel : Ajouter un domaine personnalisé à votre point de terminaison Azure CDN](cdn-map-content-to-custom-domain.md).

> [!IMPORTANT]
> Les certificats gérés par CDN ne sont pas disponibles pour les domaines racine ou apex. Si votre domaine personnalisé Azure CDN est un domaine racine ou apex, vous devez utiliser la fonctionnalité Apportez votre propre certificat. 
>

---

## <a name="tlsssl-certificates"></a>Certificats TLS/SSL
Pour activer le protocole HTTPS afin de fournir du contenu de façon sécurisée sur un domaine personnalisé Azure CDN, vous devez utiliser un certificat TLS/SSL. Vous pouvez choisir d’utiliser un certificat qui est géré par Azure CDN ou votre propre certificat.


# <a name="option-1-default-enable-https-with-a-cdn-managed-certificate"></a>[Option 1 (par défaut) : activer HTTPS avec un certificat géré par CDN](#tab/option-1-default-enable-https-with-a-cdn-managed-certificate)

Lorsque vous utilisez un certificat géré par CDN, la fonctionnalité HTTPS peut être activée en quelques clics. Azure CDN gère complètement les tâches de gestion des certificats telles que l’approvisionnement et le renouvellement. Le processus démarre dès que vous activez la fonctionnalité. Si le domaine personnalisé est déjà mappé au point de terminaison CDN, aucune action supplémentaire n’est requise. Azure CDN traite les étapes et termine votre requête automatiquement. Toutefois, si votre domaine personnalisé est mappé à un autre emplacement, vous devez utiliser la messagerie pour valider votre propriété de domaine.

Pour activer HTTPS sur un domaine personnalisé, suivez ces étapes :

1. Accédez au [portail Azure](https://portal.azure.com) pour trouver un certificat géré par votre Azure CDN. Recherchez et sélectionnez **Profils CDN** . 

2. Choisissez votre profil **Azure CDN Standard fourni par Microsoft** , **Azure CDN Standard fourni par Akamai** , **Azure CDN Standard fourni par Verizon** ou **Azure CDN Premium fourni par Verizon** .

3. Dans la liste des points de terminaison CDN, sélectionnez celui contenant votre domaine personnalisé.

    ![Liste des points de terminaison](./media/cdn-custom-ssl/cdn-select-custom-domain-endpoint.png)

    La page **Point de terminaison** s’affiche.

4. Dans la liste des domaines personnalisés, sélectionnez celui pour lequel vous souhaitez activer HTTPS.

    ![Capture d’écran montrant la page Domaine personnalisé avec l’option Utiliser mon propre certificat.](./media/cdn-custom-ssl/cdn-custom-domain.png)

    La page **Domaine personnalisé** s’affiche.

5. Sous Type de gestion de certificats, sélectionnez **CDN géré** .

6. Sélectionnez **Activé** pour activer le protocole HTTPS.

    ![Statut HTTPS sur un domaine personnalisé](./media/cdn-custom-ssl/cdn-select-cdn-managed-certificate.png)

7. Faites [Valider le domaine](#validate-the-domain).


# <a name="option-2-enable-https-with-your-own-certificate"></a>[Option n°2 : activer HTTPS avec votre propre certificat](#tab/option-2-enable-https-with-your-own-certificate)

> [!IMPORTANT]
> Cette option n’est disponible qu’avec les profils **Azure CDN de Microsoft** et **Azure CDN de Verizon** . 
>
 
Vous pouvez utiliser votre propre certificat pour activer la fonctionnalité HTTPS. Ce processus s’effectue via une intégration à Azure Key Vault, ce qui vous permet de stocker vos certificats en toute sécurité. Azure CDN utilise ce mécanisme sécurisé pour obtenir le certificat, et quelques étapes supplémentaires sont nécessaires. Quand vous créez votre certificat TLS/SSL, vous devez le créer avec une autorité de certification autorisée (CA). Autrement, si vous utilisez une autorité de certification non autorisée, votre demande sera rejetée. Pour obtenir la liste des autorités de certification autorisées, consultez [Autorités de certification autorisées pour l’activation de protocole HTTPS personnalisé sur Azure CDN](cdn-troubleshoot-allowed-ca.md). Pour le profil **Azure CDN de Verizon** , toute autorité de certification valide sera acceptée. 

### <a name="prepare-your-azure-key-vault-account-and-certificate"></a>Préparer votre compte et votre certificat Azure Key Vault
 
1. Azure Key Vault : vous devez disposer d’un compte Azure Key Vault en cours d’exécution sous le même abonnement que le profil Azure CDN, et des points de terminaison CDN souhaités pour activer le protocole HTTPS personnalisé. Si ce n’est déjà fait, créez un compte Azure Key Vault.
 
2. Certificats Azure Key Vault : si vous disposez déjà d’un certificat, vous pouvez le charger directement vers votre compte Azure Key Vault. Vous pouvez également en créer un directement à l’aide d’Azure Key Vault à partir d’une des autorités de certification (CA) partenaires de ce coffre Azure Key Vault. 

### <a name="register-azure-cdn"></a>Inscrire Azure CDN

Inscrivez Azure CDN comme application dans Azure Active Directory via PowerShell.

1. Si nécessaire, installez [Azure PowerShell](/powershell/azure/install-az-ps) sur votre machine locale.

2. Dans PowerShell, exécutez la commande suivante :

     `New-AzADServicePrincipal -ApplicationId "205478c0-bd83-4e1b-a9d6-db63a3e1e1c8"`

    ![Inscrire Azure CDN dans PowerShell](./media/cdn-custom-ssl/cdn-register-powershell.png)
              

### <a name="grant-azure-cdn-access-to-your-key-vault"></a>Accorder à Azure CDN l’accès à votre coffre de clés
 
Accordez l’autorisation d’Azure CDN pour accéder aux certificats (secrets) de votre compte Azure Key Vault.

1. Dans votre compte de coffre de clés, sous PARAMÈTRES, sélectionnez **Stratégies d’accès** , puis **Ajouter nouveau** pour créer une stratégie.

    ![Créer une stratégie d’accès](./media/cdn-custom-ssl/cdn-new-access-policy.png)

2. Dans **Sélectionner principal** , recherchez **205478c0-bd83-4e1b-a9d6-db63a3e1e1c8** , puis choisissez **Microsoft.Azure.Cdn** . Cliquez sur **Sélectionner** .

    ![Paramètres de stratégie d’accès](./media/cdn-custom-ssl/cdn-access-policy-settings.png)

3. Sélectionnez **Autorisations de certificat** , puis cochez les cases **Obtenir** et **Lister** pour autoriser CDN à mettre à en œuvre ces autorisations pour obtenir et lister les certificats.

4. Sélectionnez **Autorisations du secret** , puis cochez les cases **Obtenir** et **Lister** pour autoriser CDN à mettre à en œuvre ces autorisations pour obtenir et lister les secrets.

5. Sélectionnez **OK** . 

    Azure CDN peut désormais accéder à ce coffre de clés et aux certificats (secrets) qui sont stockés dans ce coffre de clés.
 
### <a name="select-the-certificate-for-azure-cdn-to-deploy"></a>Sélectionner le certificat pour Azure CDN à déployer
 
1. Revenez au portail Azure CDN et sélectionnez le profil et le point de terminaison CDN souhaités pour activer le protocole HTTPS personnalisé. 

2. Dans la liste des domaines personnalisés, sélectionnez celui pour lequel vous souhaitez activer HTTPS.

    La page **Domaine personnalisé** s’affiche.

3. Sous Type de gestion de certificats, sélectionnez **Utiliser mon propre certificat** . 

    ![Configurer votre certificat](./media/cdn-custom-ssl/cdn-configure-your-certificate.png)

4. Sélectionnez un coffre de clés, un certificat (secret) et une version du certificat.

    Azure CDN répertorie les informations suivantes : 
    - les comptes du coffre de clés de votre ID d’abonnement ; 
    - les certificats (secrets) dans le coffre de clés sélectionné ; 
    - les versions disponibles du certificat. 
 
5. Sélectionnez **Activé** pour activer le protocole HTTPS.
  
6. Lorsque vous utilisez votre propre certificat, la validation du domaine n’est pas nécessaire. Passez à [En attente de la propagation](#wait-for-propagation).

---

## <a name="validate-the-domain"></a>Valider le domaine

Si vous avez déjà un domaine personnalisé en cours d’utilisation qui est mappé à votre point de terminaison personnalisé avec un enregistrement CNAME, ou si vous utilisez votre propre certificat, passez à  
[Le domaine personnalisé est mappé à votre point de terminaison CDN](#custom-domain-is-mapped-to-your-cdn-endpoint-by-a-cname-record). Sinon, si l’entrée d’enregistrement CNAME pour votre point de terminaison n’existe plus ou s’il contient le sous-domaine cdnverify, passez à [Le domaine personnalisé n’est pas mappé à votre point de terminaison CDN](#custom-domain-is-not-mapped-to-your-cdn-endpoint).

### <a name="custom-domain-is-mapped-to-your-cdn-endpoint-by-a-cname-record"></a>Le domaine personnalisé est mappé à votre point de terminaison CDN par un enregistrement CNAME

Lors de l’ajout d’un domaine personnalisé à votre point de terminaison, vous avez créé un enregistrement CNAME dans la table DNS de votre registre de domaines à mapper au nom d’hôte de votre point de terminaison CDN. Si cet enregistrement CNAME existe toujours et qu’il ne contient pas le sous-domaine cdnverify, l’autorité de certification (AC) DigiCert l’utilise pour valider automatiquement la propriété de votre domaine personnalisé. 

Si vous utilisez votre propre certificat, la validation du domaine n’est pas nécessaire.

Votre enregistrement CNAME doit être au format suivant, où *Nom* est le nom de votre domaine personnalisé et *Valeur* est le nom d’hôte de votre point de terminaison CDN :

| Nom            | Type  | Valeur                 |
|-----------------|-------|-----------------------|
| <www.contoso.com> | CNAME | contoso.azureedge.net |

Pour plus d’informations sur les enregistrements CNAME, consultez [Create the CNAME DNS record](./cdn-map-content-to-custom-domain.md) (Créer l’enregistrement DNS CNAME).

Si le format de votre enregistrement CNAME est correct, DigiCert vérifie automatiquement le nom de votre domaine personnalisé et crée un certificat dédié pour votre nom de domaine. DigiCert ne vous envoie pas de courrier électronique de vérification, et vous ne devez pas approuver votre requête. Le certificat est valide pendant un an et est automatiquement renouvelé avant son expiration. Passez à [En attente de la propagation](#wait-for-propagation). 

La validation automatique prend généralement quelques heures. Si votre domaine n’est pas validé dans les 24 heures, ouvrez un ticket de support.

>[!NOTE]
>Si vous avez un enregistrement CAA (Certificate Authority Authorization) auprès de votre fournisseur DNS, il doit inclure DigiCert en tant qu’autorité de certification valide. Un enregistrement CAA permet aux propriétaires de domaine de spécifier avec leurs fournisseurs DNS les autorités de certification autorisées à émettre des certificats pour leur domaine. Si une autorité de certification reçoit une commande de certificat pour un domaine qui contient un enregistrement CCA et que cette autorité de certification n’est pas répertoriée comme émetteur autorisé, elle ne peut pas émettre le certificat pour ce domaine ou sous-domaine. Pour plus d’informations sur la gestion des enregistrements CAA, consultez [Administrer les enregistrements CAA](https://support.dnsimple.com/articles/manage-caa-record/). Pour obtenir un outil d’enregistrement CAA, consultez [Assistance pour les enregistrements CAA](https://sslmate.com/caa/).

### <a name="custom-domain-is-not-mapped-to-your-cdn-endpoint"></a>Le domaine personnalisé n’est pas mappé à votre point de terminaison CDN

>[!NOTE]
>Si vous utilisez **Azure CDN d’Akamai** , le CNAME suivant doit être configuré pour activer la validation de domaine automatisée. « _acme-challenge.&lt;nom d’hôte du domaine personnalisé&gt; -> CNAME -> &lt;nom d’hôte du domaine personnalisé&gt;.ak-acme-challenge.azureedge.net »

Si l’entrée d’enregistrement CNAME contient le sous-domaine cdnverify, suivez les autres instructions de cette étape.

DigiCert envoie un e-mail de vérification aux adresses e-mail suivantes. Vérifiez que vous pouvez effectuer directement l’approbation à partir de l’une des adresses suivantes :

admin@&lt;votre-nom-de-domaine.com&gt;  
administrateur@&lt;votre-nom-de-domaine.com&gt;  
webmaster@&lt;votre-nom-de-domaine.com&gt;  
hostmaster@&lt;votre-nom-de-domaine.com&gt;  
postmaster@&lt;votre-nom-de-domaine.com&gt;  

Au bout de quelques minutes, vous devriez recevoir un e-mail similaire à celui de l’exemple suivant et vous demandant d’approuver la requête. Si vous utilisez un filtre de courrier indésirable, ajoutez verification@digicert.com à sa liste verte. Si vous ne recevez pas d’e-mail dans les 24 heures, contactez le support Microsoft.
    
![E-mail de validation du domaine](./media/cdn-custom-ssl/domain-validation-email.png)

Lorsque vous cliquez sur le lien d’approbation, vous êtes dirigé vers le formulaire d’approbation en ligne suivant : 
    
![Formulaire de validation du domaine](./media/cdn-custom-ssl/domain-validation-form.png)

Suivez les instructions du formulaire ; deux options de vérification vous sont proposées :

- Vous pouvez approuver toutes les commandes futures passées par l’intermédiaire du même compte pour le même domaine racine, par exemple contoso.com. Nous vous recommandons cette approche si vous prévoyez d’ajouter des domaines personnalisés pour le même domaine racine.

- Vous pouvez approuver uniquement le nom d’hôte spécifique utilisé dans cette requête. L’approbation supplémentaire est nécessaire pour les demandes suivantes.

Après l’approbation, DigiCert termine la création du certificat pour votre nom de domaine personnalisé. Le certificat est valide pendant un an et est automatiquement renouvelé avant son expiration.

## <a name="wait-for-propagation"></a>En attente de la propagation

Une fois le nom de domaine validé, l’activation de la fonctionnalité HTTPS sur un domaine personnalisé peut prendre jusqu’à 6 à 8 heures. Quand le processus est terminé, l’état HTTPS personnalisé est défini sur **Activé** dans le portail Azure, et les quatre étapes de l’opération sont marquées comme terminées dans la boîte de dialogue Domaine personnalisé. Votre domaine personnalisé est désormais prêt à utiliser HTTPS.

![Boîte de dialogue de l’activation de HTTPS](./media/cdn-custom-ssl/cdn-enable-custom-ssl-complete.png)

### <a name="operation-progress"></a>Progression de l’opération

Le tableau suivant présente le déroulement de l’opération qui s’exécute lorsque vous activez HTTPS. Une fois que vous avez activé HTTPS, les quatre étapes de l’opération s’affichent dans la boîte de dialogue Domaine personnalisé. À mesure que chaque étape devient active, des détails de sous-étape supplémentaires s’affichent sous l’étape qui est en cours d’exécution. Toutes ces sous-étapes ne se produiront pas. Lorsqu’une étape se termine correctement, une coche verte apparaît en regard de celle-ci. 

| Étape de l’opération | Détails de la sous-étape de l’opération | 
| --- | --- |
| 1 Soumission de la requête | Envoi de la requête |
| | Votre requête HTTPS est en cours de soumission. |
| | Votre requête HTTPS a été correctement envoyée. |
| 2 Validation du domaine | Le domaine est automatiquement validé si l’enregistrement CNAME est mappé au point de terminaison CDN. Sinon, une demande de vérification est envoyée à l’adresse de messagerie répertoriée dans l’enregistrement de l’inscription de votre domaine (inscrit WHOIS). Vérifiez le domaine dès que possible. |
| | La propriété du domaine a été correctement validée. |
| | La requête de validation de la propriété du domaine a expiré ( le client n’a probablement pas répondu dans les 6 jours). Le protocole HTTPS n’est pas activé sur votre domaine. * |
| | La requête de validation de la propriété du domaine a été rejetée par le client. Le protocole HTTPS n’est pas activé sur votre domaine. * |
| 3 Approvisionnement du certificat | L’émission, par l’autorité de certification, du certificat nécessaire à l’activation HTTPS sur votre domaine est en cours. |
| | Le certificat a été émis et son déploiement est en cours sur le réseau CDN. Cette opération peut durer 6 heures. |
| | Le certificat a été correctement déployé sur le réseau CDN. |
| 4 Fin | Le protocole HTTPS a été correctement activé sur votre domaine. |

\* Ce message ne s’affiche pas à moins qu’une erreur se soit produite. 

Si une erreur survient avant la soumission de la requête, le message d’erreur suivant s’affiche :

<code>
We encountered an unexpected error while processing your HTTPS request. Please try again and contact support if the issue persists.
</code>



## <a name="clean-up-resources---disable-https"></a>Nettoyer les ressources : désactiver HTTPS

Dans les étapes précédentes, vous avez activé le protocole HTTPS sur votre domaine personnalisé. Si vous ne souhaitez plus utiliser votre domaine personnalisé avec HTTPS, vous pouvez désactiver HTTPS en procédant comme suit :

### <a name="disable-the-https-feature"></a>Désactiver la fonctionnalité HTTPS 

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Profils CDN** . 

2. Choisissez votre profil **Azure CDN Standard fourni par Microsoft** , **Azure CDN Standard fourni par Verizon** ou **Azure CDN Premium fourni par Verizon** .

3. Dans la liste des points de terminaison, sélectionnez celui contenant votre domaine personnalisé.

4. Choisissez le domaine personnalisé pour lequel vous souhaitez désactiver HTTPS.

    ![Liste Domaines personnalisés](./media/cdn-custom-ssl/cdn-custom-domain-HTTPS-enabled.png)

5. Choisissez **Désactiver** pour désactiver HTTPS, puis sur sélectionnez **Appliquer** .

    ![Boîte de dialogue HTTPS personnalisé](./media/cdn-custom-ssl/cdn-disable-custom-ssl.png)

### <a name="wait-for-propagation"></a>En attente de la propagation

Après la désactivation de la fonctionnalité HTTPS sur un domaine personnalisé, il peut s’écouler jusqu’à 6 à 8 heures avant que cette modification soit effective. Quand le processus est terminé, l’état HTTPS personnalisé est défini sur **Désactivé** dans le portail Azure, et les trois étapes de l’opération sont marquées comme terminées dans la boîte de dialogue Domaine personnalisé. Votre domaine personnalisé ne peut plus utiliser HTTPS.

![Boîte de dialogue de la désactivation de HTTPS](./media/cdn-custom-ssl/cdn-disable-custom-ssl-complete.png)

#### <a name="operation-progress"></a>Progression de l’opération

Le tableau suivant présente le déroulement de l’opération qui s’exécute lorsque vous désactivez HTTPS. Une fois que vous avez désactivé HTTPS, les trois étapes de l’opération s’affichent dans la boîte de dialogue Domaine personnalisé. À mesure que chaque étape devient active, des détails supplémentaires s’affichent sous l’étape en cours d’exécution. Lorsqu’une étape se termine correctement, une coche verte apparaît en regard de celle-ci. 

| Progression de l’opération | Détails de l’opération | 
| --- | --- |
| 1 Soumission de la requête | Envoi de la requête |
| 2 Annulation de l’approvisionnement du certificat | Suppression du certificat |
| 3 Fin | Le certificat est supprimé |

## <a name="frequently-asked-questions"></a>Forum aux questions

1. *Qui est le fournisseur de certificats et quel est le type de certificat utilisé ?*

    Pour **Azure CDN de Verizon** et **Azure CDN de Microsoft** , un certificat dédié/unique fourni par Digicert est utilisé pour votre domaine personnalisé. 

2. *Utilisez-vous TLS/SSL SNI ou un protocole IP ?*

    **Azure CDN de Verizon** et **Azure CDN Standard de Microsoft** utilisent tous deux TLS/SSL SNI.

3. *Que se passe-t-il si je ne reçois pas l’e-mail de vérification de domaine de DigiCert ?*

    Si vous avez une entrée CNAME pour votre domaine personnalisé qui pointe directement vers le nom d’hôte de votre point de terminaison (et si vous n’utilisez pas le nom de sous-domaine cdnverify), vous ne recevrez pas de courrier électronique de vérification du domaine. La validation se fait automatiquement. Autrement, si vous n’avez pas d’entrée CNAME et si vous n’avez pas reçu d’e-mail dans les 24 heures, contactez le support Microsoft.

4. *Un certificat SAN est-il moins sécurisé qu’un certificat dédié ?*
    
    Un certificat SAN suit les mêmes normes de sécurité et de chiffrement qu’un certificat dédié. Tous les certificats TLS/SSL émis utilisent la norme SHA-256 pour une sécurité améliorée du serveur.

5. *Ai-je besoin d’un enregistrement CAA (Certificate Authority Authorization) auprès de mon fournisseur DNS ?*

    Non, un enregistrement CAA n’est pas requis. Toutefois, si vous en avez un, il doit inclure DigiCert en tant qu’autorité de certification valide.

6. *À compter du 20 juin 2018, Azure CDN de Verizon utilise un certificat dédié avec TLS/SSL SNI par défaut. Que deviennent mes domaines personnalisés qui utilisent un certificat SAN (Subject Alternative Names) et TLS/SSL basé sur IP ?*

    Vos domaines existants seront progressivement migrés vers un certificat unique dans les mois à venir si Microsoft constate que seules des demandes de clients SNI sont effectuées dans votre application. Si Microsoft détecte que certaines demandes de clients non-SNI sont effectuées dans votre application, vos domaines resteront dans le certificat SAN avec TLS/SSL basé sur IP. Dans tous les cas, il n’y aura aucune interruption de votre service ou dans la prise en charge des demandes de vos clients, que ces demandes soient SNI ou non.

7. *Comment les renouvellements de certificats fonctionnent quand j’utilise mon propre certificat ?*

    Pour vous assurer qu’un certificat plus récent est déployé dans l’infrastructure PoP, chargez simplement le nouveau certificat dans Azure KeyVault, puis dans vos paramètres TLS/SSL sur Azure CDN. Choisissez ensuite la version de certificat la plus récente, puis cliquez sur Enregistrer. Azure CDN propagera alors votre nouveau certificat mis à jour. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> - Activer le protocole HTTPS sur votre domaine personnalisé.
> - utiliser un certificat géré par le CDN ; 
> - utiliser votre propre certificat ;
> - valider le domaine ;
> - Désactiver le protocole HTTPS sur votre domaine personnalisé.

Passez au tutoriel suivant pour apprendre à configurer HTTPS sur votre point de terminaison CDN.

> [!div class="nextstepaction"]
> [Tutoriel : Définir des règles de mise en cache d’Azure CDN](cdn-caching-rules-tutorial.md)
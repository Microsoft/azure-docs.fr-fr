---
title: 'Tutoriel : Configurer HTTPS sur un domaine personnalisé Azure Front Door | Microsoft Docs'
description: Dans ce tutoriel, vous allez apprendre à activer et désactiver HTTPS dans votre configuration Azure Front Door pour un domaine personnalisé.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/21/2020
ms.author: duau
ms.openlocfilehash: 6c6d33a36c4a0b71932e8c19c8f6dd105c33817c
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368330"
---
# <a name="tutorial-configure-https-on-a-front-door-custom-domain"></a>Tutoriel : Configurer HTTPS sur un domaine personnalisé Front Door

Ce didacticiel montre comment activer le protocole HTTPS pour un domaine personnalisé associé à votre porte d’entrée sous la section d’hôtes frontend. En utilisant le protocole HTTPS sur votre domaine personnalisé (par exemple, https :\//www.contoso.com), vous vous assurez que vos données sensibles sont remises en toute sécurité via le chiffrement TLS/SSL lors de l’envoi sur Internet. Lorsque votre navigateur web est connecté à un site web par le biais de HTTPS, ce protocole valide le certificat de sécurité du site et vérifie qu’il est fourni par une autorité de certification légitime. Ce processus assure la sécurité et protège également vos applications web contre les attaques.

Par défaut, Azure Front Door prend en charge HTTPS sur un nom d’hôte Front Door par défaut. Par exemple, si vous créez une instance Front Door (comme `https://contoso.azurefd.net`), HTTPS est automatiquement activé pour les requêtes effectuées sur `https://contoso.azurefd.net`. Toutefois, une fois que vous intégrez le domaine personnalisé « www.contoso.com », vous devez également activer HTTPS pour cet hôte frontend.   

Voici quelques-uns des attributs clés de la fonctionnalité HTTPS personnalisée :

- Aucun coût supplémentaire : l’acquisition ou le renouvellement de certificat n’entraîne aucun coût supplémentaire, tout comme le trafic HTTPS. 

- Activation simple : le provisionnement en un clic est disponible à partir du [portail Azure](https://portal.azure.com). Vous pouvez également utiliser l’API REST ou d’autres outils de développeur pour activer la fonctionnalité.

- La gestion complète de certificats est disponible : l’approvisionnement et la gestion de tous les certificats sont assurés pour vous. Les certificats sont automatiquement provisionnés et renouvelés avant expiration, ce qui élimine les risques d’interruption de service en raison d’une expiration de certificat.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Activer le protocole HTTPS sur votre domaine personnalisé.
> - Utiliser un certificat géré par AFD 
> - Utiliser votre propre certificat, autrement dit un certificat TLS/SSL personnalisé
> - Valider le domaine
> - Désactiver le protocole HTTPS sur votre domaine personnalisé


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Avant d’effectuer les étapes de ce didacticiel, vous devez créer une porte d’entrée et au moins un domaine personnalisé intégré. Pour plus d’informations, consultez [Didacticiel : Ajouter un domaine personnalisé à votre Front Door](front-door-custom-domain.md).

## <a name="tlsssl-certificates"></a>Certificats TLS/SSL

Pour activer le protocole HTTPS afin de fournir du contenu de façon sécurisée sur un domaine personnalisé Front Door, vous devez utiliser un certificat TLS/SSL. Vous pouvez utiliser un certificat géré par Azure Front Door ou votre propre certificat.


### <a name="option-1-default-use-a-certificate-managed-by-front-door"></a>Option 1 (valeur par défaut) : utiliser un certificat géré par Front Door

Lorsque vous utilisez un certificat géré par Azure Front Door, la fonctionnalité HTTPS peut être activée en quelques clics. Azure Front Door gère complètement les tâches de gestion des certificats telles que l’approvisionnement et le renouvellement. Le processus démarre dès que vous activez la fonctionnalité. Si le domaine personnalisé est déjà mappé à l’hôte frontend par défaut Front Door (`{hostname}.azurefd.net`), aucune action supplémentaire n’est requise. Front Door traite les étapes et termine votre requête automatiquement. Toutefois, si votre domaine personnalisé est mappé à un autre emplacement, vous devez utiliser la messagerie pour valider votre propriété de domaine.

Pour activer HTTPS sur un domaine personnalisé, suivez ces étapes :

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre profil **Front Door** .

2. Dans la liste des hôtes frontend, sélectionnez le domaine personnalisé pour lequel vous souhaitez activer le protocole HTTPS.

3. Sous la section **HTTPS sur un domaine personnalisé** , cliquez sur **Activé** , puis sélectionnez **Porte d’entrée managée** en tant que source du certificat.

4. Cliquez sur Enregistrer.

5. Faites [Valider le domaine](#validate-the-domain).

> [!NOTE]
> Pour les certificats managés AFD, la limite de 64 caractères de DigiCert est appliquée. En cas de dépassement de cette limite, la validation échoue.


### <a name="option-2-use-your-own-certificate"></a>Option n°2 : utiliser votre propre certificat ;

Vous pouvez utiliser votre propre certificat pour activer la fonctionnalité HTTPS. Ce processus s’effectue via une intégration à Azure Key Vault, ce qui vous permet de stocker vos certificats en toute sécurité. Azure Front Door utilise ce mécanisme sécurisé pour obtenir le certificat, et quelques étapes supplémentaires sont nécessaires. Quand vous créez votre certificat TLS/SSL, vous devez le créer avec une autorité de certification autorisée. Autrement, si vous utilisez une autorité de certification non autorisée, votre demande sera rejetée. Pour obtenir la liste des autorités de certification autorisées, consultez [Autorités de certification autorisées pour l’activation du protocole HTTPS personnalisé sur Azure Front Door](front-door-troubleshoot-allowed-ca.md).

#### <a name="prepare-your-azure-key-vault-account-and-certificate"></a>Préparer votre compte et votre certificat Azure Key Vault
 
1. Azure Key Vault : vous devez avoir un compte Azure Key Vault activé sous le même abonnement que votre Front Door à utiliser pour activer le protocole HTTPS personnalisé. Si ce n’est déjà fait, créez un compte Azure Key Vault.

> [!WARNING]
> Azure Front Door ne prend actuellement en charge que les comptes Key Vault dans le même abonnement que la configuration Front Door. Le choix d’un Key Vault sous un autre abonnement que votre porte d’entrée entraîne un échec.

2. Certificats Azure Key Vault : si vous disposez déjà d’un certificat, vous pouvez le charger directement vers votre compte Azure Key Vault. Vous pouvez également en créer un directement à l’aide d’Azure Key Vault à partir d’une des autorités de certification (CA) partenaires de ce coffre Azure Key Vault. Chargez votre certificat en tant qu’objet de **certificat** et non en tant que **secret** .

> [!NOTE]
> Pour votre propre certificat TLS/SSL, Front Door ne prend pas en charge les certificats avec des algorithmes de chiffrement EC.

#### <a name="register-azure-front-door"></a>Inscrire Azure Front Door

Inscrivez le principal du service pour Azure Front Door en tant qu’application dans Azure Active Directory par le biais de PowerShell.

> [!NOTE]
> Cette action nécessite des autorisations d’administrateur général et doit être exécutée **une seule fois** par locataire.

1. Si nécessaire, installez [Azure PowerShell](/powershell/azure/install-az-ps) dans PowerShell sur votre ordinateur local.

2. Dans PowerShell, exécutez la commande suivante :

     `New-AzADServicePrincipal -ApplicationId "ad0e1c7e-6d38-4ba4-9efd-0bc77ba9f037"`              

#### <a name="grant-azure-front-door-access-to-your-key-vault"></a>Accorder à Azure Front Door l’accès à votre coffre de clés
 
Accordez à Azure Front Door l’autorisation d’accéder aux certificats de votre compte Azure Key Vault.

1. Dans votre compte de coffre de clés, sous PARAMÈTRES, sélectionnez **Stratégies d’accès** , puis **Ajouter nouveau** pour créer une stratégie.

2. Dans **Sélectionner le principal** , recherchez **ad0e1c7e-6d38-4ba4-9efd-0bc77ba9f037** , puis choisissez **Microsoft.Azure.Frontdoor** . Cliquez sur **Sélectionner** .

3. Sous **Autorisations du secret** , sélectionnez **Obtenir** pour permettre à Front Door de récupérer le certificat.

4. Sous **Autorisations de certificat** , sélectionnez **Obtenir** pour permettre à Front Door de récupérer le certificat.

5. Sélectionnez **OK** . 

    Azure Front Door peut désormais accéder à ce coffre de clés et aux certificats qui sont stockés dans ce coffre de clés.
 
#### <a name="select-the-certificate-for-azure-front-door-to-deploy"></a>Sélectionner le certificat à déployer pour Azure Front Door
 
1. Revenez à votre porte d’entrée sur le portail. 

2. Dans la liste des domaines personnalisés, sélectionnez celui pour lequel vous souhaitez activer HTTPS.

    La page **Domaine personnalisé** s’affiche.

3. Sous Type de gestion de certificats, sélectionnez **Utiliser mon propre certificat** . 

4. Azure Front Door nécessite que l’abonnement du compte Key Vault soit identique à celui de Front Door. Sélectionnez un coffre de clés, un certificat (secret) et une version du certificat.

    Azure Front Door liste les informations suivantes : 
    - les comptes du coffre de clés de votre ID d’abonnement ; 
    - les certificats (secrets) dans le coffre de clés sélectionné ; 
    - les versions disponibles du certificat. 

> [!NOTE]
> Si vous laissez la version de certificat vide, vous obtiendrez :
> - La version la plus récente du certificat qui est sélectionné.
> - La rotation automatique des certificats pour la toute dernière version en date, quand une version plus récente du certificat est disponible dans votre coffre de clés.
 
5. Lorsque vous utilisez votre propre certificat, la validation du domaine n’est pas nécessaire. Passez à [En attente de la propagation](#wait-for-propagation).

## <a name="validate-the-domain"></a>Valider le domaine

Si vous avez déjà un domaine personnalisé en cours d’utilisation qui est mappé à votre point de terminaison personnalisé avec un enregistrement CNAME, ou si vous utilisez votre propre certificat, passez à  
[Le domaine personnalisé est mappé à votre porte d’entrée](#custom-domain-is-mapped-to-your-front-door-by-a-cname-record). Autrement, si l’entrée d’enregistrement CNAME pour votre domaine n’existe plus ou si elle contient le sous-domaine afdverify, passez à [Le domaine personnalisé n’est pas mappé à votre porte d’entrée](#custom-domain-is-not-mapped-to-your-front-door).

### <a name="custom-domain-is-mapped-to-your-front-door-by-a-cname-record"></a>Le domaine personnalisé est mappé à votre porte d’entrée par un enregistrement CNAME

Lors de l’ajout d’un domaine personnalisé aux hôtes frontend de votre porte d’entrée, vous avez créé un enregistrement CNAME dans la table DNS de votre bureau d’enregistrement de domaines afin de le mapper au nom d’hôte .azurefd.net par défaut de votre porte d’entrée. Si cet enregistrement CNAME existe toujours et s’il ne contient pas le sous-domaine afdverify, l’autorité de certification DigiCert l’utilise pour valider automatiquement la propriété de votre domaine personnalisé. 

Si vous utilisez votre propre certificat, la validation du domaine n’est pas nécessaire.

Votre enregistrement CNAME doit être au format suivant, où *Nom* désigne le nom de votre domaine personnalisé et *Valeur* désigne le nom d’hôte .azurefd.net par défaut de votre porte d’entrée :

| Nom            | Type  | Valeur                 |
|-----------------|-------|-----------------------|
| <www.contoso.com> | CNAME | contoso.azurefd.net |

Pour plus d’informations sur les enregistrements CNAME, consultez [Create the CNAME DNS record](../cdn/cdn-map-content-to-custom-domain.md) (Créer l’enregistrement DNS CNAME).

Si le format de votre enregistrement CNAME est correct, DigiCert vérifie automatiquement le nom de votre domaine personnalisé et crée un certificat dédié pour votre nom de domaine. DigiCert ne vous envoie pas de courrier électronique de vérification, et vous ne devez pas approuver votre requête. Le certificat est valide pendant un an et est automatiquement renouvelé avant son expiration. Passez à [En attente de la propagation](#wait-for-propagation). 

La validation automatique prend généralement quelques minutes. Si votre domaine n’est pas validé dans l’heure, ouvrez un ticket de support.

>[!NOTE]
>Si vous avez un enregistrement CAA (Certificate Authority Authorization) auprès de votre fournisseur DNS, il doit inclure DigiCert en tant qu’autorité de certification valide. Un enregistrement CAA permet aux propriétaires de domaine de spécifier avec leurs fournisseurs DNS les autorités de certification autorisées à émettre des certificats pour leur domaine. Si une autorité de certification reçoit une commande de certificat pour un domaine qui contient un enregistrement CCA et que cette autorité de certification n’est pas répertoriée comme émetteur autorisé, elle ne peut pas émettre le certificat pour ce domaine ou sous-domaine. Pour plus d’informations sur la gestion des enregistrements CAA, consultez [Administrer les enregistrements CAA](https://support.dnsimple.com/articles/manage-caa-record/). Pour obtenir un outil d’enregistrement CAA, consultez [Assistance pour les enregistrements CAA](https://sslmate.com/caa/).

### <a name="custom-domain-is-not-mapped-to-your-front-door"></a>Le domaine personnalisé n’est pas mappé à votre porte d’entrée

Si l’entrée d’enregistrement CNAME de votre point de terminaison n’existe plus ou si elle contient le sous-domaine afdverify, suivez les autres instructions de cette étape.

Dès que vous avez activé HTTPS sur votre domaine personnalisé, l’autorité de certification DigiCert valide la propriété de votre domaine en contactant son inscrit, en fonction de ses informations [WHOIS](http://whois.domaintools.com/) du domaine. Une prise de contact est établie par le biais de l’adresse de messagerie (par défaut) ou le numéro de téléphone répertorié dans l’inscription WHOIS. Vous devez effectuer la validation de domaine avant que le protocole HTTPS soit activé sur votre domaine personnalisé. Vous disposez de six jours ouvrables pour approuver le domaine. Les requêtes qui ne sont pas approuvées dans ce délai de six jours ouvrables sont automatiquement annulées. 

![Enregistrement WHOIS](./media/front-door-custom-domain-https/whois-record.png)

DigiCert envoie également un e-mail de vérification à des adresses e-mail supplémentaires. Si les informations sur l’inscrit WHOIS sont privées, vérifiez que vous pouvez effectuer directement l’approbation à partir de l’une des adresses suivantes :

admin@&lt;votre-nom-de-domaine.com&gt;  
administrateur@&lt;votre-nom-de-domaine.com&gt;  
webmaster@&lt;votre-nom-de-domaine.com&gt;  
hostmaster@&lt;votre-nom-de-domaine.com&gt;  
postmaster@&lt;votre-nom-de-domaine.com&gt;  

Au bout de quelques minutes, vous devriez recevoir un e-mail similaire à celui de l’exemple suivant et vous demandant d’approuver la requête. Si vous utilisez un filtre de courrier indésirable, ajoutez admin@digicert.com à sa liste verte. Si vous ne recevez pas d’e-mail dans les 24 heures, contactez le support Microsoft.

Lorsque vous cliquez sur le lien d’approbation, vous êtes dirigé vers un formulaire d’approbation en ligne. Suivez les instructions du formulaire ; deux options de vérification vous sont proposées :

- Vous pouvez approuver toutes les commandes futures passées par l’intermédiaire du même compte pour le même domaine racine, par exemple contoso.com. Nous vous recommandons cette approche si vous prévoyez d’ajouter des domaines personnalisés pour le même domaine racine.

- Vous pouvez approuver uniquement le nom d’hôte spécifique utilisé dans cette requête. L’approbation supplémentaire est nécessaire pour les demandes suivantes.

Après l’approbation, DigiCert termine la création du certificat pour votre nom de domaine personnalisé. Le certificat est valide pendant un an et est automatiquement renouvelé avant son expiration.

## <a name="wait-for-propagation"></a>En attente de la propagation

Une fois le nom de domaine validé, l’activation de la fonctionnalité HTTPS sur un domaine personnalisé peut prendre jusqu’à 6 à 8 heures. Quand le processus est terminé, l’état HTTPS personnalisé est défini sur **Activé** dans le portail Azure, et les quatre étapes de l’opération sont marquées comme terminées dans la boîte de dialogue Domaine personnalisé. Votre domaine personnalisé est désormais prêt à utiliser HTTPS.

### <a name="operation-progress"></a>Progression de l’opération

Le tableau suivant présente le déroulement de l’opération qui s’exécute lorsque vous activez HTTPS. Une fois que vous avez activé HTTPS, les quatre étapes de l’opération s’affichent dans la boîte de dialogue Domaine personnalisé. À mesure que chaque étape devient active, des détails de sous-étape supplémentaires s’affichent sous l’étape qui est en cours d’exécution. Toutes ces sous-étapes ne se produiront pas. Lorsqu’une étape se termine correctement, une coche verte apparaît en regard de celle-ci. 

| Étape de l’opération | Détails de la sous-étape de l’opération | 
| --- | --- |
| 1 Soumission de la requête | Envoi de la requête |
| | Votre requête HTTPS est en cours de soumission. |
| | Votre requête HTTPS a été correctement envoyée. |
| 2 Validation du domaine | Le domaine est automatiquement validé si l’enregistrement CNAME est mappé à l’hôte frontend .azurefd.net par défaut de votre porte d’entrée. Sinon, une demande de vérification est envoyée à l’adresse de messagerie répertoriée dans l’enregistrement de l’inscription de votre domaine (inscrit WHOIS). Vérifiez le domaine dès que possible. |
| | La propriété du domaine a été correctement validée. |
| | La requête de validation de la propriété du domaine a expiré ( le client n’a probablement pas répondu dans les 6 jours). Le protocole HTTPS n’est pas activé sur votre domaine. * |
| | La requête de validation de la propriété du domaine a été rejetée par le client. Le protocole HTTPS n’est pas activé sur votre domaine. * |
| 3 Approvisionnement du certificat | L’émission, par l’autorité de certification, du certificat nécessaire à l’activation HTTPS sur votre domaine est en cours. |
| | Le certificat a été émis et son déploiement est en cours pour votre porte d’entrée. Ce processus peut prendre jusqu’à 1 heure. |
| | Le certificat a été correctement déployé pour votre porte d’entrée. |
| 4 Fin | Le protocole HTTPS a été correctement activé sur votre domaine. |

\* Ce message ne s’affiche pas à moins qu’une erreur se soit produite. 

Si une erreur survient avant la soumission de la requête, le message d’erreur suivant s’affiche :

<code>
We encountered an unexpected error while processing your HTTPS request. Please try again and contact support if the issue persists.
</code>

## <a name="frequently-asked-questions"></a>Forum aux questions

1. *Qui est le fournisseur de certificats et quel est le type de certificat utilisé ?*

    Un certificat dédié/unique, fourni par Digicert, est utilisé pour votre domaine personnalisé. 

2. *Utilisez-vous TLS/SSL SNI ou un protocole IP ?*

    Azure Front Door utilise TLS/SSL SNI.

3. *Que se passe-t-il si je ne reçois pas l’e-mail de vérification de domaine de DigiCert ?*

    Si une entrée CNAME pour votre domaine personnalisé pointe directement vers le nom d’hôte de votre point de terminaison (et si vous n’utilisez pas le nom de sous-domaine afdverify), vous ne recevrez pas de courrier électronique de vérification du domaine. La validation se fait automatiquement. Autrement, si vous n’avez pas d’entrée CNAME et si vous n’avez pas reçu d’e-mail dans les 24 heures, contactez le support Microsoft.

4. *Un certificat SAN est-il moins sécurisé qu’un certificat dédié ?*
    
    Un certificat SAN suit les mêmes normes de sécurité et de chiffrement qu’un certificat dédié. Tous les certificats TLS/SSL émis utilisent la norme SHA-256 pour une sécurité améliorée du serveur.

5. *Ai-je besoin d’un enregistrement CAA (Certificate Authority Authorization) auprès de mon fournisseur DNS ?*

    Non, un enregistrement CAA n’est pas requis. Toutefois, si vous en avez un, il doit inclure DigiCert en tant qu’autorité de certification valide.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans les étapes précédentes, vous avez activé le protocole HTTPS sur votre domaine personnalisé. Si vous ne souhaitez plus utiliser votre domaine personnalisé avec HTTPS, vous pouvez désactiver HTTPS en procédant comme suit :

### <a name="disable-the-https-feature"></a>Désactiver la fonctionnalité HTTPS 

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre configuration **Azure Front Door** .

2. Dans la liste d’hôtes frontend, cliquez sur le domaine personnalisé pour lequel vous souhaitez désactiver HTTPS.

3. Cliquez sur **Désactiver** pour désactiver HTTPS, puis cliquez sur **Enregistrer** .

### <a name="wait-for-propagation"></a>En attente de la propagation

Après la désactivation de la fonctionnalité HTTPS sur un domaine personnalisé, il peut s’écouler jusqu’à 6 à 8 heures avant que cette modification soit effective. Quand le processus est terminé, l’état HTTPS personnalisé est défini sur **Désactivé** dans le portail Azure, et les trois étapes de l’opération sont marquées comme terminées dans la boîte de dialogue Domaine personnalisé. Votre domaine personnalisé ne peut plus utiliser HTTPS.

#### <a name="operation-progress"></a>Progression de l’opération

Le tableau suivant présente le déroulement de l’opération qui s’exécute lorsque vous désactivez HTTPS. Une fois que vous avez désactivé HTTPS, les trois étapes de l’opération s’affichent dans la boîte de dialogue Domaine personnalisé. À mesure que chaque étape devient active, des détails supplémentaires s’affichent sous l’étape en cours d’exécution. Lorsqu’une étape se termine correctement, une coche verte apparaît en regard de celle-ci. 

| Progression de l’opération | Détails de l’opération | 
| --- | --- |
| 1 Soumission de la requête | Envoi de la requête |
| 2 Annulation de l’approvisionnement du certificat | Suppression du certificat |
| 3 Fin | Le certificat est supprimé |

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

* Charger un certificat sur Key Vault.
* Valider un domaine.
* Activer HTTPS pour votre domaine personnalisé.

Pour savoir comment configurer une stratégie de géofiltrage destinée à votre Front Door, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Configurer une stratégie de géofiltrage](front-door-geo-filtering.md)
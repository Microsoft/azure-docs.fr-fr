---
title: Rotation des certificats pour Azure SQL Database et SQL Managed Instance
description: Découvrez les modifications à venir dues aux changements de certificat racine qui affectent Azure SQL Database et Azure SQL Managed Instance.
author: srdan-bozovic-msft
ms.author: srbozovi
ms.service: sql-db-mi
ms.subservice: service
ms.topic: conceptual
ms.date: 09/13/2020
ms.openlocfilehash: b0a10744d2b48fa620b48b731144222199f711c7
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792529"
---
# <a name="understanding-the-changes-in-the-root-ca-change-for-azure-sql-database--sql-managed-instance"></a>Comprendre les modifications liées au changement d’autorité de certification racine pour Azure SQL Database et SQL Managed Instance

Azure SQL Database et SQL Managed Instance vont modifier le certificat racine d’application cliente/de pilote activé avec SSL et que vous utilisez pour établir une connexion TDS sécurisée. Le [certificat racine actuel](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) est configuré pour expirer le 26 octobre 2020 dans le cadre de la maintenance standard et conformément aux bonnes pratiques de sécurité. Cet article vous donne plus de détails sur les modifications à venir, les ressources qui seront affectées et les étapes nécessaires pour garantir que votre application maintient sa connectivité à votre serveur de base de données.

## <a name="what-update-is-going-to-happen"></a>Quelle mise à jour va se produire ?

Le [forum Certificate Authority (CA) Browser](https://cabforum.org/) a récemment publié des rapports indiquant que plusieurs certificats émis par les fournisseurs d’autorité de certification n’étaient pas conformes.

Conformément aux exigences de conformité du secteur, les fournisseurs d’autorité de certification ont commencé à révoquer les certificats d’autorité de certification non conformes, en demandant aux serveurs d’utiliser des certificats émis par des autorités de certification conformes et signés par des certificats d’autorité de certification provenant de ces dernières. Étant donné qu’Azure SQL Database et SQL Managed Instance utilisent actuellement l’un de ces certificats non conformes utilisés par les applications clientes pour valider leurs connexions SSL, nous devons veiller à ce que les actions appropriées soient entreprises (décrites ci-dessous) afin de réduire au maximum l’impact potentiel sur vos serveurs SQL Azure.

Le nouveau certificat sera utilisé à partir du 26 octobre 2020. Si vous utilisez la validation complète du certificat de serveur lors de la connexion à partir d’un client SQL (TrustServerCertificate=true), vous devez vous assurer que votre client SQL sera en mesure de valider le nouveau certificat racine avant le 26 octobre 2020.

## <a name="how-do-i-know-if-my-application-might-be-affected"></a>Comment puis-je savoir si mon application peut être affectée ?

Toutes les applications qui utilisent SSL/TLS et vérifient le certificat racine doivent mettre à jour le certificat racine afin de pouvoir se connecter à Azure SQL Database et SQL Managed Instance. 

Si vous n’utilisez pas SSL/TLS actuellement, il n’y a aucun impact sur la disponibilité de votre application. Vous pouvez vérifier si votre application cliente tente de vérifier le certificat racine en examinant la chaîne de connexion. Si TrustServerCertificate est explicitement défini sur true, vous n’êtes pas affecté.

Si votre pilote client utilise le magasin de certificats du système d’exploitation, comme c’est le cas de la majorité des pilotes, et que votre système d’exploitation fait l’objet d’une maintenance régulière, ce changement ne vous affectera probablement pas, car le certificat racine sur lequel nous basculons devrait être déjà disponible dans votre magasin de certificats racine de confiance. Recherchez Baltimore CyberDigiCert GlobalRoot G2 et validez sa présence.

Si votre pilote client utilise un magasin de certificats local, pour éviter toute interruption de la disponibilité de votre application en raison de la révocation inattendue de certificats ou pour mettre à jour un certificat qui a été révoqué, reportez-vous à la section [**Procédure à suivre pour maintenir la connectivité**](./ssl-root-certificate-expiring.md#what-do-i-need-to-do-to-maintain-connectivity).

## <a name="what-do-i-need-to-do-to-maintain-connectivity"></a>Procédure à suivre pour maintenir la connectivité

Pour éviter toute interruption de la disponibilité de votre application en raison de la révocation inattendue de certificats ou pour mettre à jour un certificat qui a été révoqué, suivez les étapes ci-dessous :

*   Téléchargez l’autorité de certification racine Baltimore CyberTrust Root et DigiCert GlobalRoot G2 à partir des liens ci-dessous :
    *   https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem
    *   https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem

*   Générez un magasin de certificats d’autorité de certification combiné où les certificats **BaltimoreCyberTrustRoot** et **DigiCertGlobalRootG2** sont inclus.

## <a name="what-can-be-the-impact"></a>Quel peut être l’impact ?
Si vous validez des certificats de serveur comme indiqué ici, la disponibilité de votre application peut être interrompue, car la base de données ne sera pas accessible. Selon votre application, vous pouvez recevoir différents messages d’erreur, notamment :
*   Certificat non valide/certificat révoqué
*   Délai de connexion dépassé
*   Erreur, le cas échéant

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="if-i-am-not-using-ssltls-do-i-still-need-to-update-the-root-ca"></a>Si je n’utilise pas SSL/TLS, dois-je quand même mettre à jour l’autorité de certification racine ?
Aucune action n’est requise concernant ce changement si vous n’utilisez pas SSL/TLS. Toutefois, vous devez définir un plan pour commencer à utiliser la dernière version de TLS, car nous prévoyons d’imposer le protocole TLS dans un avenir proche.

### <a name="what-will-happen-if-i-do-not-update-the-root-certificate-before-october-26-2020"></a>Que se passera-t-il si je ne mets pas à jour le certificat racine avant le 26 octobre 2020 ?
Si vous ne mettez pas à jour le certificat racine avant le 30 novembre 2020, vos applications qui se connectent via SSL/TLS et effectuent une vérification pour le certificat racine ne pourront pas communiquer avec Azure SQL Database et SQL Managed Instance et l’application rencontrera des problèmes de connectivité avec vos instances Azure SQL Database et SQL Managed Instance.

### <a name="do-i-need-to-plan-a-maintenance-downtime-for-this-changebr"></a>Dois-je prévoir un temps d’arrêt lié à la maintenance pour effectuer cette modification ?<BR>
Non. Étant donné que la modification est uniquement côté client pour la connexion au serveur, aucun temps d’arrêt lié à la maintenance n’est nécessaire ici lors de cette modification.

### <a name="what-if-i-cannot-get-a-scheduled-downtime-for-this-change-before-october-26-2020"></a>Que se passe-t-il si je ne peux pas prévoir un temps d’arrêt planifié pour cette modification avant le 26 octobre 2020 ?
Étant donné que les clients utilisés pour la connexion au serveur doivent mettre à jour les informations de certificat comme décrit dans la section de correction disponible [ici](./ssl-root-certificate-expiring.md#what-do-i-need-to-do-to-maintain-connectivity), aucun temps d’arrêt n’est requis pour le serveur dans ce cas.

### <a name="if-i-create-a-new-server-after-november-30-2020-will-i-be-impacted"></a>Si je crée un nouveau serveur après le 30 novembre 2020, est-ce que je serai impacté ?
Pour les serveurs créés après le 26 octobre 2020, vous pouvez utiliser le certificat récemment émis pour que vos applications se connectent à l’aide de SSL.

### <a name="how-often-does-microsoft-update-their-certificates-or-what-is-the-expiry-policy"></a>Quelle est la fréquence à laquelle Microsoft met à jour ses certificats ou quelle est la stratégie d’expiration ?
Les certificats utilisés par Azure SQL Database et SQL Managed Instance sont fournis par des autorités de certification approuvées. Par conséquent, la prise en charge de ces certificats sur Azure SQL Database et SQL Managed Instance est liée à la prise en charge de ces certificats par l’autorité de certification. Toutefois, comme dans le cas présent, il peut y avoir des bogues imprévus dans ces certificats prédéfinis et ceux-ci doivent être corrigés au plus tôt.

### <a name="if-i-am-using-read-replicas-do-i-need-to-perform-this-update-only-on-master-server-or-all-the-read-replicas"></a>Si j’utilise des réplicas en lecture, dois-je effectuer cette mise à jour uniquement sur le serveur maître ou sur tous les réplicas en lecture ?
Étant donné que cette mise à jour est une modification côté client, si le client avait l’habitude de lire les données du serveur de réplica, nous devons également appliquer les modifications pour ces clients. 

### <a name="do-we-have-server-side-query-to-verify-if-ssl-is-being-used"></a>Existe-t-il une requête côté serveur pour vérifier si SSL est utilisé ?
Étant donné que cette configuration s’effectue côté client, ces informations ne sont pas disponibles côté serveur.

### <a name="what-if-i-have-further-questions"></a>Que se passe-t-il si j’ai d’autres questions ?
Si vous avez un plan de support et que vous avez besoin d’une assistance technique, créez une demande de support Azure. Pour cela, consultez [Créer une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md).
---
title: Référence de code d’erreur
titleSuffix: Azure AD B2C
description: Liste des codes d’erreur qui peuvent être retournés par le service Azure Active Directory B2C.
services: B2C
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/02/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: f6ae806b7666d83652e6b82bac16d89f2f9ce7aa
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92215431"
---
# <a name="error-codes-azure-active-directory-b2c"></a>Codes d’erreur : Azure Active Directory B2C

Les erreurs suivantes peuvent être retournées par le service Azure Active Directory B2C.

| Code d'erreur | Message |
| ---------- | ------- |
| `AADB2C90002` | La ressource CORS « {0} » a retourné l’erreur « 404 - Introuvable ». |
| `AADB2C90006` | L’URI de redirection {0} fourni dans la demande n’est pas inscrit pour l’ID client « {1} ». |
| `AADB2C90007` | L’application associée à l’ID client « {0} » n’a pas d’URI de redirection inscrit. |
| `AADB2C90008` | La demande ne contient pas de paramètre d’ID client. |
| `AADB2C90010` | La demande ne contient pas de paramètre d’étendue. |
| `AADB2C90011` | L’ID client « {0} » fourni dans la demande ne correspond pas à l’ID client « {1} » inscrit dans la stratégie. |
| `AADB2C90012` | L’étendue « {0} » fournie dans la demande n’est pas prise en charge. |
| `AADB2C90013` | Le type de réponse demandé « {0} » fourni dans la demande n’est pas pris en charge. |
| `AADB2C90014` | Le mode de réponse demandé « {0} » fourni dans la demande n’est pas pris en charge. |
| `AADB2C90016` | Le type d’assertion de client demandé « {0} » ne correspond pas au type attendu « {1} ». |
| `AADB2C90017` | L’assertion de client fournie dans la demande n’est pas valide : {0} |
| `AADB2C90018` | L’ID client « {0} » spécifié dans la demande n’est pas inscrit dans le locataire « {1} ». |
| `AADB2C90019` | Le conteneur de clés ayant l’ID « {0} » dans le locataire « {1} » n’a pas de clé valide. Raison : {2}. |
| `AADB2C90021` | Le profil technique « {0} » n’existe pas dans la stratégie « {1} » du locataire « {2} ». |
| `AADB2C90022` | Impossible de retourner des métadonnées pour la stratégie « {0} » dans le locataire « {1} ». |
| `AADB2C90023` | Le profil « {0} » ne contient pas la clé de métadonnées « {1} » exigée. |
| `AADB2C90025` | Le profil « {0} » dans la stratégie « {1} » du locataire « {2} » ne contient pas la clé de chiffrement « {3} » exigée. |
| `AADB2C90027` | Les informations d’identification de base spécifiées pour « {0} » ne sont pas valides. Vérifiez que ces informations sont correctes et que la ressource a octroyé l’accès. |
| `AADB2C90028` | Le certificat client spécifié pour « {0} » n’est pas valide. Vérifiez que ce certificat est correct, qu’il contient une clé privée et que la ressource a octroyé l’accès. |
| `AADB2C90031` | La stratégie « {0} » ne spécifie pas de parcours utilisateur par défaut. Vérifiez que la stratégie ou sa stratégie parente spécifie un parcours utilisateur par défaut dans une section sur les parties de confiance. |
| `AADB2C90035` | Le service est temporairement indisponible. Réessayez après quelques minutes. |
| `AADB2C90036` | La demande ne contient pas d’URI pour rediriger l’utilisateur après déconnexion. Spécifiez un URI dans le champ du paramètre post_logout_redirect_uri. |
| `AADB2C90037` | Une erreur s’est produite lors du traitement de la demande. Contactez l’administrateur du site auquel vous tentez d’accéder. |
| `AADB2C90039` | La demande contient une assertion de client, mais la stratégie fournie « {0} » dans le locataire « {1} » ne comprend pas de secret client (client_secret) dans RelyingPartyPolicy. |
| `AADB2C90040` | Le parcours utilisateur « {0} » ne contient pas d’étape d’envoi de revendications. |
| `AADB2C90043` | L’invite incluse dans la demande contient des valeurs non valides. Les valeurs attendues sont « none », « login », « consent » ou « select_account ». |
| `AADB2C90044` | La revendication « {0} » n’est pas prise en charge par le résolveur de revendication « {1} ». |
| `AADB2C90046` | Nous rencontrons des difficultés pour charger votre état actuel. Essayez de redémarrer votre session. |
| `AADB2C90047` | La ressource « {0} » contient des erreurs de script qui empêchent son chargement. |
| `AADB2C90048` | Une exception non prise en charge s’est produite sur le serveur. |
| `AADB2C90051` | Aucun fournisseur de revendications approprié n’a été trouvé. |
| `AADB2C90052` | Nom d’utilisateur ou mot de passe non valide. |
| `AADB2C90053` | L’utilisateur dont les informations d’identification sont spécifiées est introuvable. |
| `AADB2C90054` | Nom d’utilisateur ou mot de passe non valide. |
| `AADB2C90055` | L’étendue « {0} » fournie dans la demande doit spécifier une ressource, comme « https://example.com/calendar.read  ». |
| `AADB2C90057` | L’application fournie n’est pas configurée pour autoriser le flux OAuth implicite. |
| `AADB2C90058` | L’application fournie n’est pas configurée pour autoriser les clients publics. |
| `AADB2C99059` | La demande fournie doit présenter un code_challenge. Obligatoire pour les applications monopages utilisant le flux de code d’autorisation.| 
| `AADB2C90067` | L’URI de redirection post-déconnexion « {0} » a un format non valide. Spécifiez une URL HTTPS telle que « https://example.com/return  » ou, pour des clients natifs, utilisez l’URI de client natif IETF « urn:ietf:wg:oauth:2.0:oob ». |
| `AADB2C90068` | L’application fournie, dont l’ID est « {0} », n’est pas valide pour ce service. Utilisez une application créée via le portail B2C, puis réessayez. |
| `AADB2C90075` | L’échange de revendications « {0} » spécifié à l’étape « {1} » a renvoyé une réponse d’erreur HTTP avec le code « {2} » et le motif « {3} ». |
| `AADB2C90077` | L’utilisateur n’a pas de session existante, et le paramètre d’invite de demande a la valeur « {0} ». |
| `AADB2C90079` | Les clients doivent envoyer une clé secrète client (client_secret) lors de l’échange d’un octroi confidentiel. |
| `AADB2C90080` | L’octroi fourni a expiré. Veuillez vous authentifier à nouveau et réessayer. Heure actuelle : {0}, Heure d’émission de l’octroi : {1}, Heure d’expiration de la fenêtre glissante d’octroi : {2}. |
| `AADB2C90081` | La clé secrète client (client_secret) spécifiée ne correspond pas à la valeur attendue pour ce client. Corrigez la clé secrète client, puis réessayez. |
| `AADB2C90083` | La demande ne contient pas le paramètre obligatoire : {0}. |
| `AADB2C90084` | Les clients publics ne doivent pas envoyer de clé secrète client (client_secret) lors de l’échange d’un octroi acquis publiquement. |
| `AADB2C90085` | Le service a rencontré une erreur interne. Effectuez une nouvelle authentification, puis réessayez. |
| `AADB2C90086` | Le type d’octroi (grant_type) fourni [{0}] n’est pas pris en charge. |
| `AADB2C90087` | L’octroi fourni n’a pas été émis pour cette version du point de terminaison de protocole. |
| `AADB2C90088` | L’octroi fourni n’a pas été émis pour ce point de terminaison. Valeur réelle : {0} et Valeur attendue : {1} |
| `AADB2C90091` | Annulation par l’utilisateur. |
| `AADB2C90092` | L’application fournie, dont l’ID est « {0} », est désactivée pour le locataire « {1} ». Activez l’application, puis réessayez. |
| `AADB2C90107` | L’application avec l’ID « {0} » ne peut pas obtenir un jeton d’ID parce que l’étendue openid n’a pas été fournie dans la requête ou que l’application n’est pas autorisée. |
| `AADB2C90108` | L’étape d’orchestration « {0} » ne spécifie pas la valeur attendue de CpimIssuerTechnicalProfileReferenceId. |
| `AADB2C90110` | Le paramètre d’étendue doit inclure « openid » lors de la demande d’un type de réponse (response_type) incluant « id_token ». |
| `AADB2C90111` | Votre compte a été verrouillé. Contactez votre support technique pour le déverrouiller, puis réessayez. |
| `AADB2C90114` | Votre compte est temporairement verrouillé pour éviter toute utilisation non autorisée. Réessayez plus tard. |
| `AADB2C90115` | Lors de la demande du type de réponse (response_type) « code », le paramètre d’étendue doit inclure un ID de ressource ou de client pour les jetons d’accès, et un « openid » pour les jetons d’ID. Incluez en outre « offline_access » pour les jetons d’actualisation. |
| `AADB2C90117` | L’étendue « {0} » fournie dans la demande n’est pas prise en charge. |
| `AADB2C90118` | L’utilisateur a oublié son mot de passe. |
| `AADB2C90120` | Le paramètre d’âge maximal « {0} » spécifié dans la demande n’est pas valide. L’âge maximal doit être un entier de « {1} » à « {2} » inclus. |
| `AADB2C90122` | L’entrée pour « {0} » reçue dans la demande a échoué à l’étape de validation de la requête HTTP. Assurez-vous que l’entrée ne contient pas de caractère tel que < ou &. |
| `AADB2C90128` | Le compte associé à cet octroi n’existe plus. Authentifiez-vous de nouveau, puis réessayez. |
| `AADB2C90129` | L’allocation fournie a été révoquée. Authentifiez-vous de nouveau, puis réessayez. |
| `AADB2C90145` | Aucun numéro de téléphone non vérifié n’a été trouvé et la stratégie n’autorise pas de numéro entré par l’utilisateur. |
| `AADB2C90146` | L’étendue « {0} » fournie dans la demande spécifie plusieurs ressources pour un jeton d’accès. Ce comportement n’est pas pris en charge. |
| `AADB2C90149` | Impossible de charger le script « {0} ». |
| `AADB2C90151` | L’utilisateur a dépassé le nombre maximal de nouvelles tentatives pour l’authentification multifacteur. |
| `AADB2C90152` | Une demande d’interrogation multifacteur n’a pas pu obtenir de réponse du service. |
| `AADB2C90154` | Une demande de vérification multifacteur n’a pas pu obtenir l’ID de session du service. |
| `AADB2C90155` | Une demande de vérification multifacteur a échoué avec le motif « {0} ». |
| `AADB2C90156` | Une demande de validation multifacteur a échoué avec le motif « {0} ». |
| `AADB2C90157` | L’utilisateur a dépassé le nombre maximal de nouvelles tentatives pour une étape déclarée automatiquement. |
| `AADB2C90158` | Une demande de validation déclarée automatiquement a échoué avec le motif « {0} ». |
| `AADB2C90159` | Une demande de vérification déclarée automatiquement a échoué avec le motif « {0} ». |
| `AADB2C90161` | Une réponse d’envoi déclarée automatiquement a échoué avec le motif « {0} ». |
| `AADB2C90165` | Le message d’initialisation SAML avec l’ID « {0} » est introuvable en l’état. |
| `AADB2C90168` | La demande HTTP-Redirect ne contient pas le paramètre requis « {0} » pour une demande signée. |
| `AADB2C90178` | Le certificat de signature « {0} » n’a pas de clé privée. |
| `AADB2C90182` | Le code_verifier fourni ne correspond pas au code_challenge associé |
| `AADB2C90183` | Le code_verifier fourni n’est pas valide |
| `AADB2C90184` | Le code_challenge_method fourni n’est pas pris en charge. Les valeurs prises en charge sont Standard ou S256 |
| `AADB2C90188` | Le profil technique SAML « {0} » spécifie l’URL PartnerEntity « {1} », mais l’extraction des métadonnées échoue avec la raison « {2} ». |
| `AADB2C90194` | La revendication « {0} » spécifiée pour le jeton du porteur n’est pas présente dans les revendications disponibles. Revendications disponibles : « {1} ». |
| `AADB2C90205` | Cette application n’a pas les autorisations suffisantes sur cette ressource web pour effectuer l’opération. |
| `AADB2C90206` | Expiration du délai d’attente lors de l’initialisation du client. |
| `AADB2C90208` | Le paramètre id_token_hint fourni a expiré. Indiquez un autre jeton et réessayez. |
| `AADB2C90209` | Le paramètre id_token_hint fourni ne contient pas d’audience acceptée. Valeurs d’audience valides : « {0}». Indiquez un autre jeton et réessayez. |
| `AADB2C90210` | Le paramètre id_token_hint fourni n’a pas pu être validé. Indiquez un autre jeton et réessayez. |
| `AADB2C90211` | La demande contenait un cookie d’état incomplet. |
| `AADB2C90212` | La demande contenait un cookie d’état non valide. |
| `AADB2C90220` | Le conteneur de clé dans le locataire « {0} » avec l’identificateur de stockage « {1} » existe mais ne contient pas un certificat valide. Le certificat a peut-être expiré ou votre certificat pourrait devenir actif dans le futur (nbf). |
| `AADB2C90223` | Une erreur s’est produite lors du nettoyage de la ressource CORS. |
| `AADB2C90224` | Le flux de propriétaire de ressource n’a pas été activé pour l’application. |
| `AADB2C90225` | Le nom d’utilisateur ou le mot de passe fourni dans la demande n’est pas valide. |
| `AADB2C90226` | L’échange de jetons spécifié est pris en charge uniquement sur HTTP POST. |
| `AADB2C90232` | Le paramètre id_token_hint fourni ne contient pas d’émetteur accepté. Émetteurs valides : « {0} ». Indiquez un autre jeton et réessayez. |
| `AADB2C90233` | Le paramètre id_token_hint fourni n’a pas pu valider la signature. Indiquez un autre jeton et réessayez. |
| `AADB2C90235` | Le paramètre id_token fourni a expiré. Indiquez un autre jeton et réessayez. |
| `AADB2C90237` | Le paramètre id_token fourni ne contient pas d’audience valide. Valeurs d’audience valides : « {0}». Indiquez un autre jeton et réessayez. |
| `AADB2C90238` | Le paramètre id_token fourni ne contient pas d’émetteur valide. Valeurs d’émetteur valides : « {0} ». Indiquez un autre jeton et réessayez. |
| `AADB2C90239` | Le paramètre id_token fourni n’a pas pu valider la signature. Indiquez un autre jeton et réessayez. |
| `AADB2C90240` | Le paramètre id_token fourni a un format incorrect et n’a pas pu être analysé. Indiquez un autre jeton et réessayez. |
| `AADB2C90242` | Le profil technique SAML « {0} » spécifie des données CDATA PartnerEntity qui ne peuvent pas être chargées pour la raison suivante : « {1} ». |
| `AADB2C90243` | La clé/le secret client du fournisseur d’identité n’est pas correctement configuré. |
| `AADB2C90244` | Il y a trop de demandes pour l’instant. Veuillez patienter quelques instants, puis réessayez. |
| `AADB2C90248` | Un flux de propriétaire de ressources ne peut être utilisé que par des applications créées via le portail d’administration B2C. |
| `AADB2C90250` | Le point de terminaison de connexion générique n’est pas pris en charge. |
| `AADB2C90255` | L’échange de revendications spécifié dans le profil technique « {0} » ne s’est pas effectué comme prévu. Vous pouvez essayer de reprendre votre session depuis le début. |
| `AADB2C90261` | L’échange de revendications « {0} » spécifié à l’étape « {1} » a retourné une réponse d’erreur HTTP qui ne peut pas être analysée. |
| `AADB2C90272` | Le paramètre id_token_hint n’a pas été spécifié dans la demande. Fournissez un jeton et réessayez. |
| `AADB2C90273` | Une réponse non valide a été reçue : « {0} » |
| `AADB2C90274` | Les métadonnées du fournisseur ne spécifient pas de service Single Logout ou la liaison du point de terminaison ne fait pas partie de « urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect » ni de « urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST ». |
| `AADB2C90276` | La demande n’est pas cohérente avec le paramètre de contrôle « {0} » : « {1} » dans technicalProfile « {2} » pour la stratégie « {3} » et le locataire « {4} ». |
| `AADB2C90277` | L’étape d’orchestration « {0} » du parcours utilisateur « {1} » de la stratégie « {2} » ne contient aucune référence à la définition du contenu. |
| `AADB2C90279` | L’ID client fourni « {0} » ne correspond pas à l’ID client qui a émis l’allocation. |
| `AADB2C90284` | L’application avec l’identificateur « {0} » n’a pas obtenu le consentement et ne peut pas être utilisée pour les comptes locaux. |
| `AADB2C90285` | L’application avec l’identificateur « {0} » est introuvable. |
| `AADB2C90288` | UserJourney avec l’ID « {0} » référencé dans TechnicalProfile « {1} » pour l’échange de jeton d’actualisation pour le locataire « {2} » n’existe pas dans la stratégie « {3} » ou l’une de ses stratégies de base. |
| `AADB2C90289` | Nous avons rencontré une erreur lors de la connexion au fournisseur d’identité. Veuillez réessayer plus tard. |
| `AADB2C90296` | L’application n’a pas été configurée correctement. Contactez l’administrateur du site auquel vous essayez d’accéder. |
| `AADB2C99005` | La demande contient un paramètre d’étendue non valide qui comprend le caractère non conforme « {0} ». |
| `AADB2C99006` | Azure AD B2C ne trouve aucune application d’extensions correspondant à l’ID d’application « {0} ». Pour plus d’informations, consultez la page https://go.microsoft.com/fwlink/?linkid=851224. |
| `AADB2C99011` | La valeur des métadonnées « {0} » n’a pas été spécifiée dans TechnicalProfile « {1} » au sein de la stratégie « {2} ». |
| `AADB2C99013` | La combinaison grant_type [{0}] et token_type [{1}] fournie n’est pas prise en charge. |
| `AADB2C99015` | Le profil « {0} » dans la stratégie « {1} » dans le locataire « {2} » ne contient pas l’ensemble de InputClaims requis pour le flux d’informations d’identification de mot de passe du propriétaire de la ressource. |

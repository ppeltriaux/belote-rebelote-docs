# Politique de confidentialité — Belote et Rebelote

> 🇬🇧 [English version](./PRIVACY.md)

**Date d'effet :** 2026-05-22
**Dernière mise à jour :** 2026-06-01
**Application :** Belote et Rebelote (iOS)
**Développeur :** Pascal Peltriaux
**Contact :** dev@peltriaux.com

---

## En bref

- Nous collectons le strict minimum nécessaire au fonctionnement du multijoueur en direct.
- Votre identité provient d'Apple Game Center, que vous contrôlez via Réglages iOS → Game Center.
- Les parties transitent par Supabase. Pendant une partie en direct, une petite quantité d'état de jeu est stockée brièvement côté serveur (pour vous permettre de revenir si vous êtes déconnecté) et supprimée sous 24 heures.
- Si vous activez les notifications de disponibilité de partie, nous stockons votre jeton push Apple afin de vous prévenir qu'une partie se forme. Vous pouvez le désactiver à tout moment.
- Nous ne vendons, ne partageons et n'utilisons **aucune** donnée à des fins publicitaires, de profilage ou de suivi inter-applications.
- Nous n'utilisons **aucun** SDK tiers d'analyse, de publicité ou d'attribution.

Si quelque chose ci-dessous n'est pas clair, écrivez à dev@peltriaux.com.

---

## 1. Données que nous collectons

### 1.1 Identifiants Game Center

Lorsque vous vous connectez à Game Center pour jouer en multijoueur, l'application reçoit d'Apple :

| Champ | Valeur | Pourquoi nous en avons besoin |
|---|---|---|
| `teamPlayerID` | Identifiant opaque émis par Apple | Pour identifier votre place dans une partie et lier votre session Supabase à une identité vérifiable (anti-usurpation) |
| `alias` | Votre nom affiché Game Center | Pour étiqueter votre plaque de joueur afin que les adversaires sachent qui joue |
| Photo de profil (facultative) | Votre avatar Game Center (si défini) | Pour afficher un vrai visage sur votre plaque (les autres voient ce qu'ils verraient dans Game Center) |

Nous ne recevons jamais votre vrai nom, votre adresse e-mail, votre numéro de téléphone ni votre identifiant Apple via Game Center.

### 1.2 Données de jeu (multijoueur en direct uniquement)

Pendant une partie en cours, les données suivantes transitent entre les joueurs appariés via Supabase Realtime, qui sert de relais :

- Cartes jouées (quelle carte depuis quelle place)
- Enchères (couleur d'atout + valeur du contrat)
- Scores de manche et scores de fin de partie
- Annonces Belote-Rebelote / Capot / déclarations

Les données de jeu sont **uniquement** échangées entre les 2 à 4 joueurs appariés de votre partie. Pour vous permettre de **revenir dans une partie après une déconnexion** (ou de récupérer si l'hôte se déconnecte), l'état de jeu courant est brièvement stocké côté serveur dans une ligne `room_authority` pendant la partie. Cet état, ainsi que les lignes `room_members` qui indiquent quel `teamPlayerID` occupe quelle place, sont supprimés automatiquement sous 24 heures.

### 1.3 Métadonnées de partie (transitoires)

Pour vous apparier avec des adversaires et gérer les salons, l'application écrit des lignes éphémères dans Supabase :

| Table | Contenu | Conservation |
|---|---|---|
| `rooms` | Identifiant aléatoire de partie (UUID), `teamPlayerID` du créateur, mode de jeu, statut, code d'invitation (salons privés), horodatages | Lignes fermées/abandonnées supprimées automatiquement sous 24 heures |
| `room_members` | Identifiant de partie + votre `teamPlayerID` + numéro de place | Supprimées sous 24 heures |
| `room_authority` | État de jeu courant pour la reconnexion/récupération (voir §1.2) | Supprimées sous 24 heures |
| `match_queue` | Entrée transitoire de matchmaking | Supprimée sous ~1 heure |

### 1.3a Notifications de disponibilité de partie (sur option)

Si vous activez les **Notifications** dans l'application (désactivées par défaut ; nécessite aussi l'autorisation de notifications iOS), l'application stocke une ligne dans une table `push_subscriptions` contenant :

- Votre `teamPlayerID`
- Le **jeton APNs (Apple Push Notification service)** de votre appareil — un identifiant émis par Apple, utilisé uniquement pour vous délivrer une notification
- Les variantes pour lesquelles vous souhaitez être notifié, et votre préférence d'heures de silence

Nous l'utilisons **uniquement** pour vous envoyer une notification quand une partie publique en ligne se forme (« une partie de Coinche est disponible »). Nous ne l'utilisons jamais à des fins publicitaires ou de suivi. Vous pouvez le désactiver à tout moment dans l'application (ce qui supprime/désactive la ligne) ou dans Réglages iOS → Notifications ; les jetons obsolètes sont supprimés automatiquement lorsqu'Apple les signale comme invalides.

### 1.3b Enregistrement « joueur fondateur »

La première fois que vous jouez une partie en ligne, nous stockons une seule ligne dans une table `founders` contenant votre `teamPlayerID` et un horodatage. Cela enregistre que vous étiez un joueur de la première heure, afin de pouvoir vous garder le jeu en ligne gratuit si une option payante est introduite plus tard. Cette ligne est conservée jusqu'à ce que vous demandiez sa suppression (voir §4).

### 1.4 Succès et classements Game Center

Lorsque vous gagnez des parties, réalisez des capots ou atteignez d'autres jalons, les scores et succès sont envoyés directement à Apple Game Center. **Ces données ne transitent pas par Supabase.** Elles sont régies par la [politique de confidentialité d'Apple](https://www.apple.com/legal/privacy/) et vos réglages de confidentialité Game Center.

### 1.5 Ce que nous ne collectons pas

Nous ne collectons, ne stockons et ne transmettons **pas** :

- Votre vrai nom, adresse e-mail, numéro de téléphone ou adresse postale
- Votre adresse IP (au-delà de ce qui est nécessaire pour acheminer la connexion multijoueur en direct — Supabase ne la journalise pas pour notre projet)
- D'identifiants publicitaires ou constructeur (IDFA, IDFV) — le seul identifiant d'appareil que nous stockons est le jeton push APNs facultatif décrit au §1.3a, et uniquement si vous activez les notifications
- De données de localisation (précises ou approximatives)
- Vos contacts, votre calendrier, vos photos, le micro ou la caméra
- Votre historique de navigation ou l'usage d'autres applications
- D'analyses ou de télémétrie de plantage (nous n'utilisons aucun SDK d'analyse)
- D'identifiants publicitaires

---

## 2. Où vont les données

| Destinataire | Données partagées | Finalité | Leur politique |
|---|---|---|---|
| **Apple Game Center** | Vos scores, succès, état de connexion, alias | Infrastructure de matchmaking multijoueur (via GameKit d'Apple), classements, succès | [Politique Apple](https://www.apple.com/legal/privacy/) |
| **Apple Push Notification service** | Votre jeton APNs + le texte de la notification (uniquement si vous activez l'option) | Délivrer les notifications de disponibilité de partie à votre appareil | [Politique Apple](https://www.apple.com/legal/privacy/) |
| **Supabase, Inc.** | Métadonnées de partie, état de jeu transitoire et (si vous activez l'option) votre jeton push | Transport multijoueur en temps réel + les petites lignes côté serveur décrites aux §1.2–§1.3b | [Politique Supabase](https://supabase.com/privacy) |

Nous utilisons Supabase comme « sous-traitant » (au sens du RGPD) : ils assurent le transport pour notre compte et n'utilisent pas vos données de jeu à leurs propres fins. Supabase est conforme SOC 2 Type II.

Nous ne partageons **aucune** donnée avec un autre tiers. Aucun réseau publicitaire, plateforme d'attribution, fournisseur d'analyse ou courtier de données n'est intégré à l'application.

---

## 3. Durée de conservation

| Donnée | Conservation |
|---|---|
| Alias + photo Game Center | En mémoire uniquement — re-récupérés depuis Game Center à chaque lancement |
| Lignes `rooms` / `room_members` / `room_authority` (incl. votre `teamPlayerID`) | Durée de la partie + au plus 24 heures, puis suppression automatique |
| Entrée de matchmaking `match_queue` | Au plus ~1 heure |
| Trafic de jeu diffusé | Non conservé — relayé en temps réel puis supprimé |
| Abonnement push (`teamPlayerID` + jeton APNs), si activé | Jusqu'à désactivation des notifications, ou jusqu'à ce que le jeton devienne invalide |
| Ligne « joueur fondateur » (`teamPlayerID` + horodatage) | Jusqu'à votre demande de suppression (§4) |
| Scores / succès Game Center | Gérés par Apple selon vos réglages Game Center |

À la fin d'une partie, les lignes transitoires sont nettoyées sous 24 heures. Les seules données conservées plus longtemps sont l'abonnement push facultatif (que vous contrôlez) et le petit enregistrement « joueur fondateur » (supprimable sur demande).

---

## 4. Vos droits

En vertu du RGPD, de la CCPA et de lois similaires, vous avez le droit de :

- **Accès** — demander une copie des données que nous détenons sur vous.
- **Suppression** — demander la suppression des données que nous détenons sur vous.
- **Rectification** — corriger des données inexactes.
- **Opposition** — refuser certains traitements.
- **Portabilité** — recevoir vos données dans un format lisible par machine.
- **Retrait du consentement** — à tout moment.

En pratique, le plus simple pour exercer ces droits dans notre application :

1. **Désactivez les Notifications** dans l'application (ou Réglages iOS → Notifications) pour supprimer votre abonnement push, et **déconnectez-vous de Game Center** (Réglages iOS → Game Center) pour que l'application n'ait plus d'identité à vous associer.
2. **Supprimez l'application** de votre appareil. Il n'y a aucun compte avec mot de passe à clôturer.
3. **Écrivez à dev@peltriaux.com** pour que nous supprimions toute ligne restante côté serveur liée à votre `teamPlayerID` — y compris l'abonnement push facultatif et l'enregistrement « joueur fondateur », les seuls éléments non soumis au nettoyage automatique de 24 heures. Nous honorerons la demande sous 7 jours.

Pour les données Game Center (classements, succès), utilisez l'app Game Center sur votre appareil ou contactez Apple — ces données leur appartiennent, pas à nous.

---

## 5. Enfants

L'application utilise Apple Game Center, qui exige des utilisateurs d'avoir au moins 13 ans dans la plupart des juridictions (selon la politique d'Apple). Nous ne collectons pas sciemment de données auprès de personnes de moins de 13 ans. Si vous pensez qu'un enfant de moins de 13 ans a utilisé l'application et que nous détenons des données le concernant, écrivez à dev@peltriaux.com — nous les supprimerons sur demande.

Nous ne faisons pas la promotion de l'application auprès des enfants et n'incluons aucun contenu destiné aux enfants au-delà de ce qui est habituel pour un jeu de cartes occasionnel.

---

## 6. Sécurité

- Tout le trafic entre l'application et Supabase utilise HTTPS (TLS 1.2+).
- L'identité Game Center est vérifiée côté serveur via la signature de vérification d'identité GameKit (RSA-SHA256) d'Apple, contrôlée contre le certificat public d'Apple, ce qui empêche l'usurpation.
- Les politiques de sécurité au niveau ligne (Row-Level Security) de Supabase restreignent la lecture/écriture de chaque ligne de partie à ses participants réels.
- Nous ne transmettons **pas** les mains de cartes brutes aux joueurs non destinataires (modèle de menace MVP basé sur la confiance — voir « Limites » ci-dessous).

### Limites que nous divulguons

- **Modèle de menace « jeu de cartes occasionnel, sans argent réel ».** Nous n'implémentons ni paris, ni monnaie, ni fonctionnalités qui justifieraient un investissement anti-triche plus lourd. Un pair motivé pourrait en principe observer le trafic diffusé destiné à d'autres places. C'est acceptable pour un jeu occasionnel et explicitement divulgué.
- **Aucune validation de jeu côté serveur.** L'autorité du donneur est accordée par confiance à chaque manche. L'anti-triche repose sur une vérification déterministe du hachage du jeu de cartes sur les 4 clients.

---

## 7. Modifications de cette politique

Si nous modifions cette politique, nous :

1. Mettrons à jour la date de « Dernière mise à jour » en haut.
2. Publierons la nouvelle version dans ce même dépôt.
3. Pour les changements importants (nouveaux types de données collectées, nouveaux destinataires tiers), augmenterons la version de l'application et signalerons le changement dans les notes de mise à jour de l'App Store.

Vous pouvez consulter l'historique complet des révisions de ce document dans le dépôt lié à la fiche App Store.

---

## 8. Juridiction

Le développeur est basé en Australie. Cette politique est régie par les lois de la Nouvelle-Galles du Sud (Australie), sans égard aux principes de conflit de lois.

Pour les utilisateurs de l'UE/EEE, vous pouvez contacter votre autorité locale de protection des données si vos préoccupations restent sans réponse. Pour les utilisateurs de Californie, nos pratiques sont conformes à la CCPA — nous ne « vendons » ni ne « partageons » de données personnelles au sens de ces termes.

---

## 9. Contact

E-mail : **dev@peltriaux.com**

Dépôt de ce document : https://github.com/ppeltriaux/belote-rebelote-docs

Fiche App Store : (à mettre à jour au lancement)

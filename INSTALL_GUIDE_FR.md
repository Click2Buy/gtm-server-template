[View the English version 🇬🇧](INSTALL_GUIDE.md)

---

# Guide d'installation : Suivi des conversions Click2Buy (GTM Server-Side)

Ce guide vous explique comment installer le modèle de balise GTM Server-Side de Click2Buy pour suivre les ventes et les conversions générées depuis notre plateforme.

L'utilisation de ce modèle "server-side" garantit un suivi plus robuste, plus fiable et plus performant, en s'appuyant sur des cookies first-party.

## ⚠️ Prérequis Indispensables

Avant de commencer, vous devez impérativement avoir :

1.  Un conteneur **GTM Server-Side** (GTM-SS) actif et fonctionnel.
2.  Ce conteneur serveur doit être hébergé sur un **sous-domaine de votre site principal** (ex: `sgtm.votresite.com`) pour permettre la création de cookies first-party.
3.  Une configuration GTM **Web (client-side)** qui envoie ses données à votre conteneur serveur (via le paramètre `server_container_url` dans votre Balise Google).

> Si ces prérequis ne sont pas en place, l'installation échouera. Veuillez vous référer à la [documentation officielle de Google](https://developers.google.com/tag-platform/tag-manager/server-side/overview) pour mettre en place votre infrastructure GTM Server-Side.

---

## 1. Configuration du Conteneur Serveur

Dans cette partie, nous allons installer le "cœur" de la logique de suivi dans votre conteneur GTM-SS.

### Étape 1.1 : Importer le modèle de balise Click2Buy

1.  Téléchargez la dernière version de notre modèle de balise (le fichier `template.tpl`) depuis la [page "Releases" de ce dépôt GitHub](https://github.com/Click2Buy/gtm-server-template/releases).
2.  Dans votre conteneur **Serveur** GTM, allez dans **Modèles**.
3.  Sous "Modèles de balise", cliquez sur **Nouveau**.
4.  Dans l'éditeur de modèle, cliquez sur le menu (⋮) en haut à droite et sélectionnez **Importer**.
5.  Choisissez le fichier `template.tpl` que vous venez de télécharger.
6.  Cliquez sur **Enregistrer** (vous pouvez ignorer l'avertissement de "modification de code", c'est normal).

### Étape 1.2 : Créer le déclencheur de conversion

Pour optimiser les performances et les coûts, votre balise ne doit s'exécuter que sur les événements pertinents : la première visite (`page_view`) et les événements de conversion (`purchase`, `generate_lead`, etc.).

1.  Dans votre conteneur **Serveur**, allez dans **Déclencheurs**.
2.  Cliquez sur **Nouveau** et nommez-le (ex: `[C2B] - PageView et Conversions`).
3.  Type de déclencheur : **Personnalisé**.
4.  Se déclenche sur : **Certains événements**.
5.  Définissez les conditions suivantes :
    * `{{Client Name}}` - `est égal(e) à` - `GA4`
    * `{{Event Name}}` - `correspond à l'expression régulière` - `^(page_view|purchase|generate_lead)$`

    > **Note :** Vous pouvez sélectionner les variables "Client Name" ou "Event Name" en cliquant sur l'icône de sélection de variable (brique).


    > **Important :** Si votre événement de conversion principal n'est pas `purchase` ou `generate_lead`, ajoutez-le à l'expression régulière (ex: `^(page_view|purchase|form_submission|mon_event)$`).

6.  Cliquez sur **Enregistrer**.

### Étape 1.3 : Créer la balise Click2Buy

1.  Dans votre conteneur **Serveur**, allez dans **Balises**.
2.  Cliquez sur **Nouveau**.
3.  Nommez la balise (ex: `Click2Buy S2S Attribution`).
4.  Cliquez sur "Configuration de la balise" et choisissez le modèle **"Click2Buy S2S Attribution"** que vous venez d'importer (il apparaîtra dans la liste "Personnalisé").
5.  Cliquez sur "Déclenchement" et sélectionnez le déclencheur `[C2B] - PageView et Conversions` créé à l'étape 1.2.
6.  Cliquez sur **Enregistrer**.

---

## 2. Configuration du Conteneur Web (Client-Side)

Cette partie s'assure que votre GTM Web (navigateur) envoie bien **tous** les événements à votre serveur GTM, pour que notre balise puisse les intercepter.

> Si vous avez déjà une configuration GTM Web qui envoie **tous** ses événements (via une balise Événement GA4 dynamique) à votre `server_container_url`, vous pouvez sauter cette section.

### Étape 2.1 : Vérifier la Balise Google (le "cerveau")

1.  Dans votre conteneur **Web** GTM, trouvez votre balise principale **"Balise Google"** (celle qui a votre `G-XXXXXX`).
2.  Vérifiez qu'elle a bien un paramètre de configuration `server_container_url` qui pointe vers votre sous-domaine GTM-SS (ex: `https://sgtm.votresite.com`).

### Étape 2.2 : Créer le "transporteur" d'événements (si besoin)

Pour que votre balise serveur reçoive les événements `purchase`, vous devez les lui envoyer. La meilleure méthode est une balise "dynamique" unique.

1.  Dans votre conteneur **Web**, allez dans **Balises** et créez **Nouveau**.
2.  Nommez la balise (ex: `[GA4] - Événements (vers Serveur)`).
3.  Type de balise : **Google Analytics : Événement GA4**.
4.  Balise de configuration : Sélectionnez votre Balise Google principale (de l'étape 2.1).
5.  **Nom de l'événement :** `{{Event}}` (Ceci est une variable GTM intégrée qui reprendra dynamiquement le nom de l'événement du `dataLayer`).

### Étape 2.3 : Créer le déclencheur "Tous les événements" (si besoin)

Cette balise doit se déclencher sur tous les événements que votre site envoie.

1.  Dans la balise que vous créez (étape 2.2), cliquez sur **Déclenchement**.
2.  Créez un **Nouveau** déclencheur (en haut à droite).
3.  Nommez-le (ex: `Custom - Tous les événements (avec exceptions)`).
4.  Type de déclencheur : **Événement personnalisé**.
5.  Nom de l'événement : `.*` et cochez la case **Utiliser une expression régulière**.
6.  Se déclenche sur : **Certains événements personnalisés**.
7.  Définissez la condition suivante pour exclure les événements GTM par défaut :
    * `{{Event}}` - `ne correspond pas à l'expression régulière` - `^(gtm\.(js|dom|load)|page_view)$`

8.  Enregistrez le déclencheur, puis enregistrez la balise.

---

## 3. Publication et Validation

Vous avez terminé la configuration. Il ne reste plus qu'à publier et tester.

1.  **Publiez** les modifications de votre conteneur **Web**.
2.  **Publiez** les modifications de votre conteneur **Serveur**.

### Comment tester le flux complet :

1.  Ouvrez le mode **Prévisualiser** de votre conteneur **Serveur**.
2.  Ouvrez le mode **Prévisualiser** de votre conteneur **Web**.

**Test 1 : L'attribution (pose du cookie)**

1.  Dans votre navigateur, ouvrez une page de votre site avec le paramètre de test fourni par Click2Buy, qui contient votre identifiant retailer :
    `https://www.votresite.com/?c2bm=rid:<votre identifiant retailer>;test:1`
2.  Dans votre **Preview Serveur**, vous devriez voir un événement `page_view` arriver.
3.  Cliquez sur cet événement. La balise `Click2Buy S2S Attribution` doit apparaître dans la section **"Balises déclenchées" (Tags Fired)**.
4.  Dans les outils de développement de votre navigateur (F12) > Application > Cookies, vous devriez voir un **nouveau cookie first-party** (ex: `_c2b_attribution_id`) avec la valeur `rid:<votre identifiant retailer>;test:1`.

**Test 2 : La conversion (envoi des données)**

1.  Sur votre site (toujours en mode Preview), déclenchez un événement de conversion (ex: effectuez un achat test pour déclencher l'événement `purchase`).
2.  Dans votre **Preview Serveur**, vous devriez voir l'événement `purchase` arriver.
3.  Cliquez sur cet événement. La balise `Click2Buy S2S Attribution` doit à nouveau être dans les **"Balises déclenchées"**.
4.  Cliquez sur cette balise pour l'inspecter. Allez dans l'onglet **"Requêtes HTTP sortantes" (Outgoing HTTP Requests)**.
5.  Vous devriez y voir une requête envoyée à notre endpoint de suivi, avec un statut `200` et la réponse `{"success":true,"action":"validation_test"}`.

Grâce au paramètre `test:1`, cette conversion est enregistrée comme une **vente de test** : Click2Buy la voit, mais elle n'est jamais comptée dans les statistiques des marques. Vous pouvez nous prévenir une fois le test effectué.

> **Astuce :** vous pouvez aussi déclencher la conversion sans passer de commande, en envoyant l'événement depuis la console de votre navigateur, par exemple `dataLayer.push({ event: 'purchase', ecommerce: { transaction_id: 'TEST-123', value: 10, currency: 'EUR' } })`. Cela ne fonctionne que si votre conteneur web transmet cet événement à votre conteneur serveur, sous le nom renseigné dans le champ **Purchase event** de la balise. Utilisez toujours un `transaction_id` factice.

---

Si les deux tests sont concluants, l'installation est un succès !

En cas de problème, contactez `retailers@click2buy.com`.
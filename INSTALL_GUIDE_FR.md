[View the English version 🇬🇧](INSTALL_GUIDE.md)

---

# Guide d'installation du Tag Server-Side GTM Click2Buy

Suivez ces étapes pour implémenter la balise Click2Buy dans votre conteneur Google Tag Manager (Serveur).

La balise s'exécute dans votre conteneur serveur et traite deux types d'événements :
- **Page vue** : lit le paramètre d'attribution (`c2bm`) quand le visiteur arrive sur votre site depuis Click2Buy, et l'enregistre dans un cookie first-party (`_c2b_attribution_id`).
- **Conversion** : envoie les données de commande à Click2Buy lors de votre événement d'achat (ou de lead).

---

## 📋 Prérequis

Avant de commencer, assurez-vous que :
1. Vous avez un **conteneur Google Tag Manager Serveur** opérationnel.
2. Votre conteneur serveur est hébergé sur un **sous-domaine de votre site** (par exemple `sgtm.votre-site.com`), pour que le cookie d'attribution soit un cookie first-party.
3. Votre **conteneur Google Tag Manager Web** envoie ses données au conteneur serveur (paramètre `server_container_url` de votre balise Google).
4. Vous avez reçu l'**URL de test** de Click2Buy, utilisée pour tester votre implémentation (section 5).

Si votre conteneur serveur n'est pas encore en place, consultez la [documentation de Google](https://developers.google.com/tag-platform/tag-manager/server-side/overview).

---

## 📥 1. Importer le modèle (Template)

1. Téléchargez la dernière version du fichier `template.tpl` depuis la [page Releases de ce dépôt GitHub](https://github.com/Click2Buy/gtm-server-template/releases).
2. Ouvrez votre conteneur **Google Tag Manager** (Serveur).
3. Dans la barre latérale gauche, cliquez sur **Modèles**.
4. Dans la section **Modèles de balises**, cliquez sur **Nouveau**.
5. Cliquez sur les **trois points (⋮)** en haut à droite et sélectionnez **Importer**.
6. Sélectionnez le fichier `template.tpl` que vous venez de télécharger.
7. Cliquez sur **Enregistrer** et fermez l'éditeur de modèle. GTM peut vous avertir que le code du modèle a été modifié : c'est normal.

---

## ⚡ 2. Créer le déclencheur

La balise n'a besoin de s'exécuter que sur les pages vues et sur vos événements de conversion.

1. Allez dans la section **Déclencheurs** et cliquez sur **Nouveau**. Nommez le déclencheur `Click2Buy — Page vue et conversions`.
2. **Type de déclencheur :** sélectionnez **Personnalisé**.
3. **Ce déclencheur se déclenche sur :** sélectionnez **Certains événements** et ajoutez ces deux conditions :
   - `Client Name` — `est égal à` — `GA4`
   - `Event Name` — `correspond à l'expression régulière` — `^(page_view|purchase|generate_lead)$`

   Pour sélectionner `Client Name` et `Event Name`, cliquez sur l'icône des variables (brique) à côté du champ.

   ⚠️ Si votre événement de conversion a un autre nom, ajoutez-le à l'expression régulière, par exemple `^(page_view|purchase|ma_conversion)$`, et indiquez le même nom dans la balise (section 3).
4. Enregistrez le déclencheur.

---

## 🏷️ 3. Créer la balise « Click2Buy — Attribution S2S »

1. Allez dans la section **Balises** et cliquez sur **Nouvelle**. Nommez la balise `Click2Buy — Attribution S2S`.
2. Cliquez sur **Configuration de la balise** et cherchez **Click2Buy tag for retailers** dans la liste (section **Personnalisé**).
3. **Parameters :** les valeurs par défaut conviennent à la plupart des sites.
   - **Page view event name**, **Purchase event name**, **Lead event name** : `page_view`, `purchase` et `generate_lead` par défaut. Modifiez-les si vos événements ont d'autres noms, comme dans le déclencheur (section 2).
   - Laissez **Cookie expiration in days** et **Clear cookie on conversion** à leurs valeurs par défaut, sauf indication contraire de Click2Buy.
4. Cliquez sur la section **Déclenchement** et sélectionnez le déclencheur `Click2Buy — Page vue et conversions` (section 2).
5. Enregistrez la balise.

---

## 🌐 4. Configurer le conteneur Web

Le conteneur serveur ne reçoit que les événements que votre conteneur web lui envoie. **Si votre conteneur web envoie déjà tous ses événements au conteneur serveur, passez cette section.**

1. Dans votre conteneur **Web**, ouvrez votre **balise Google** (celle qui a votre ID `G-XXXXXX`) et vérifiez qu'elle a un paramètre `server_container_url` qui pointe vers votre conteneur serveur (par exemple `https://sgtm.votre-site.com`).
2. Si vos événements ne sont pas encore envoyés au conteneur serveur, créez une balise qui les transmet :
   1. Allez dans la section **Balises** et cliquez sur **Nouvelle**. Nommez la balise `GA4 — Tous les événements (vers serveur)`.
   2. **Type de balise :** sélectionnez **Google Analytics : Événement GA4**.
   3. **ID de mesure :** entrez votre ID `G-XXXXXX`, le même que celui de votre balise Google.
   4. **Nom de l'événement :** entrez `{{Event}}`. Cette variable intégrée reprend le nom de chaque événement du `dataLayer`.
   5. Cliquez sur la section **Déclenchement** et créez un nouveau déclencheur (cliquez sur l'icône `+`) :
      1. **Type de déclencheur :** sélectionnez **Événement personnalisé**.
      2. **Nom de l'événement :** entrez `.*` et cochez **Utiliser une expression régulière**.
      3. **Ce déclencheur se déclenche sur :** sélectionnez **Certains événements personnalisés** et ajoutez la condition `Event` — `ne correspond pas à l'expression régulière` — `^(gtm\.(js|dom|load)|page_view)$`. Elle exclut les événements de GTM et la page vue, déjà envoyée par la balise Google.
      4. Nommez le déclencheur `Tous les événements (sauf GTM)` et cliquez sur **Enregistrer**.
   6. Enregistrez la balise.

---

## ✅ 5. Tester votre implémentation

1. Cliquez sur le bouton **Prévisualiser** dans votre conteneur **Serveur**, puis dans votre conteneur **Web**.
2. **Page vue :**
   1. Ouvrez votre site avec l'URL de test fournie par Click2Buy, qui contient votre identifiant retailer : `https://www.votre-site.com/?c2bm=rid:<votre identifiant retailer>;test:1`. Acceptez la bannière cookies.
   2. Dans la prévisualisation de votre conteneur serveur, cliquez sur l'événement `page_view` et vérifiez que la balise **Click2Buy — Attribution S2S** est dans les **Balises déclenchées** (Tags Fired).
   3. Ouvrez la console de votre navigateur (F12). Dans l'onglet **Application → Cookies**, vérifiez que le cookie `_c2b_attribution_id` est posé sur votre domaine, avec la valeur `rid:<votre identifiant retailer>;test:1`.
3. **Conversion :**
   1. Effectuez un achat de test sur votre site.
   2. Dans la prévisualisation de votre conteneur serveur, cliquez sur l'événement `purchase` et vérifiez que la balise **Click2Buy — Attribution S2S** est dans les **Balises déclenchées**.
   3. Cliquez sur la balise et ouvrez l'onglet **Requêtes HTTP sortantes** (Outgoing HTTP Requests). La requête vers `analytics.clic2buy.com` doit avoir un statut `200` et la réponse `{"success":true,"action":"validation_test"}`.

Grâce au paramètre `test:1`, cet achat est enregistré comme une **vente de test** : Click2Buy la voit, mais elle n'est jamais comptée dans les statistiques des marques. Vous pouvez nous prévenir une fois le test effectué.

💡 Vous pouvez aussi déclencher la conversion sans passer de commande, en envoyant l'événement depuis la console de votre navigateur, par exemple `dataLayer.push({ event: 'purchase', ecommerce: { transaction_id: 'TEST-123', value: 10, currency: 'EUR' } })`. Cela ne fonctionne que si votre conteneur web envoie cet événement au conteneur serveur, sous le nom indiqué dans **Purchase event name**. Utilisez toujours un `transaction_id` factice.

---

## 🚀 6. Publier

Une fois que vous avez vérifié que la balise fonctionne correctement, cliquez sur **Envoyer** dans **les deux** conteneurs (Serveur et Web) pour publier vos modifications sur votre site en production.

En cas de problème, contactez `retailers@click2buy.com`.

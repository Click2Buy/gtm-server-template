[Voir la version française 🇫🇷](INSTALL_GUIDE_FR.md)

---

# Click2Buy GTM Server-Side Tag Installation Guide

Follow these steps to implement the Click2Buy tag in your Google Tag Manager Server container.

The tag runs in your server container and handles two kinds of events:
- **Page view**: reads the attribution parameter (`c2bm`) when the visitor lands on your website from Click2Buy, and stores it in a first-party cookie (`_c2b_attribution_id`).
- **Conversion**: sends the order data to Click2Buy on your purchase (or lead) event.

---

## 📋 Prerequisites

Before starting, ensure that:
1. You have a **Google Tag Manager Server container** up and running.
2. Your server container is hosted on a **subdomain of your website** (e.g. `sgtm.your-website.com`), so that the attribution cookie is a first-party cookie.
3. Your **Google Tag Manager Web container** sends its data to the server container (`server_container_url` parameter of your Google Tag).
4. You have received the **test URL** from Click2Buy, used to test your implementation (section 5).

If your server container is not set up yet, see the [Google documentation](https://developers.google.com/tag-platform/tag-manager/server-side/overview).

---

## 📥 1. Import the Template

1. Download the latest `template.tpl` file from the [Releases page of this repository](https://github.com/Click2Buy/gtm-server-template/releases).
2. Open your **Google Tag Manager** server container.
3. In the left sidebar, click on **Templates**.
4. In the **Tag Templates** section, click **New**.
5. Click the **three dots (⋮)** in the top right corner and select **Import**.
6. Select the `template.tpl` file you just downloaded.
7. Click **Save** and close the template editor. GTM may warn you that the template code has been edited: this is expected.

---

## ⚡ 2. Create the Trigger

The tag only needs to run on page views and on your conversion events.

1. Go to **Triggers** and click **New**. Name the trigger `Click2Buy — Page view and conversions`.
2. **Trigger type:** select **Custom**.
3. **This trigger fires on:** select **Some Events** and add these two conditions:
   - `Client Name` — `equals` — `GA4`
   - `Event Name` — `matches RegEx` — `^(page_view|purchase|generate_lead)$`

   To select `Client Name` and `Event Name`, click the variable icon (brick) next to the field.

   ⚠️ If your conversion event has another name, add it to the RegEx, e.g. `^(page_view|purchase|my_conversion)$`, and set the same name in the tag (section 3).
4. Save the trigger.

---

## 🏷️ 3. Create the "Click2Buy — S2S attribution" Tag

1. Go to **Tags** and click **New**. Name the tag `Click2Buy — S2S attribution`.
2. Click **Tag Configuration** and look for **Click2Buy tag for retailers** in the list (**Custom** section).
3. **Parameters:** the default values fit most websites.
   - **Page view event name**, **Purchase event name**, **Lead event name**: `page_view`, `purchase` and `generate_lead` by default. Change them if your events have other names, as in the trigger (section 2).
   - Keep **Cookie expiration in days** and **Clear cookie on conversion** at their default values, unless Click2Buy tells you otherwise.
4. Click on the **Triggering** section and select the `Click2Buy — Page view and conversions` trigger (section 2).
5. Save your tag.

---

## 🌐 4. Configure the Web Container

The server container only receives the events that your web container sends to it. **If your web container already sends all its events to the server container, skip this section.**

1. In your **Web** container, open your **Google Tag** (the one with your `G-XXXXXX` ID) and check that it has a `server_container_url` parameter pointing to your server container (e.g. `https://sgtm.your-website.com`).
2. If your events are not sent to the server container yet, create a tag that forwards them:
   1. Go to **Tags** and click **New**. Name the tag `GA4 — All events (to server)`.
   2. **Tag type:** select **Google Analytics: GA4 Event**.
   3. **Measurement ID:** enter your `G-XXXXXX` ID, the same as your Google Tag.
   4. **Event Name:** enter `{{Event}}`. This built-in variable takes the name of each `dataLayer` event.
   5. Click on the **Triggering** section and create a new trigger (click the `+` icon):
      1. **Trigger type:** select **Custom Event**.
      2. **Event name:** enter `.*` and check **Use regex matching**.
      3. **This trigger fires on:** select **Some Custom Events** and add the condition `Event` — `does not match RegEx` — `^(gtm\.(js|dom|load)|page_view)$`. This excludes the GTM events and the page view, which is already sent by the Google Tag.
      4. Name the trigger `All events (except GTM)` and click **Save**.
   6. Save your tag.

---

## ✅ 5. Testing your Implementation

1. Click the **Preview** button in your **Server** container, then in your **Web** container.
2. **Page view:**
   1. Open your website with the test URL provided by Click2Buy, which contains your retailer ID: `https://www.your-website.com/?c2bm=rid:<your retailer ID>;test:1`. Accept the cookie banner.
   2. In the Preview of your server container, click the `page_view` event and check that the **Click2Buy — S2S attribution** tag is in **Tags Fired**.
   3. Open your browser console (F12). In the **Application → Cookies** tab, verify that the `_c2b_attribution_id` cookie is set on your domain, with the value `rid:<your retailer ID>;test:1`.
3. **Conversion:**
   1. Complete a test purchase on your website.
   2. In the Preview of your server container, click the `purchase` event and check that the **Click2Buy — S2S attribution** tag is in **Tags Fired**.
   3. Click the tag and open the **Outgoing HTTP Requests** tab. The request to `analytics.clic2buy.com` must have a `200` status and the response `{"success":true,"action":"validation_test"}`.

Thanks to the `test:1` parameter, this purchase is recorded as a **test sale**: Click2Buy can see it, but it is never counted in the brands' statistics. You can let us know once the test is done.

💡 You can also trigger the conversion without placing an order, by pushing the event from your browser console, e.g. `dataLayer.push({ event: 'purchase', ecommerce: { transaction_id: 'TEST-123', value: 10, currency: 'EUR' } })`. This only works if your web container sends this event to the server container, under the name set in **Purchase event name**. Always use a fake `transaction_id`.

---

## 🚀 6. Publish

Once you have verified the tag is working correctly, click **Submit** in **both** containers (Server and Web) to publish your changes to the live site.

If you encounter any issues, contact `retailers@click2buy.com`.

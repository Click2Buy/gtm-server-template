[Voir la version française 🇫🇷](INSTALL_GUIDE_FR.md)

---

# Installation Guide: Click2Buy Conversion Tracking (GTM Server-Side)

This guide explains how to install the Click2Buy GTM Server-Side tag template to track sales and conversions referred from our platform.

Using this server-side template ensures more robust, reliable, and high-performance tracking by leveraging first-party cookies.

## ⚠️ Critical Prerequisites

Before you begin, you must have the following:

1.  An active and functional **GTM Server-Side (GTM-SS) container**.
2.  This server container must be hosted on a **subdomain of your main site** (e.g., `sgtm.your-site.com`) to enable first-party cookie setting.
3.  A **Web (client-side) GTM** configuration that sends its data to your server container (via the `server_container_url` parameter in your Google Tag).

> If these prerequisites are not met, the installation will fail. Please refer to the [official Google documentation](https://developers.google.com/tag-platform/tag-manager/server-side/overview) to set up your GTM Server-Side infrastructure.

---

## 1. Server Container Configuration

In this section, we will install the core tracking logic into your GTM-SS container.

### Step 1.1: Import the Click2Buy Tag Template

1.  Download the latest version of our tag template (the `template.tpl` file) from the [Releases page of this GitHub repository](https://github.com/Click2Buy/gtm-server-template/releases).
2.  In your GTM **Server** container, go to **Templates**.
3.  Under "Tag Templates," click **New**.
4.  In the template editor, click the three-dot menu (⋮) in the top-right corner and select **Import**.
5.  Choose the `template.tpl` file you just downloaded.
6.  Click **Save** (you can ignore the "edit code" warning, this is normal).

### Step 1.2: Create the Conversion Trigger

To optimize performance and cost, your tag should only run on relevant events: the initial visit (`page_view`) and conversion events (`purchase`, `generate_lead`, etc.).

1.  In your GTM **Server** container, go to **Triggers**.
2.  Click **New** and name it (e.g., `[C2B] - PageView and Conversions`).
3.  Trigger Type: **Custom Event**.
4.  This trigger fires on: **Some Custom Events**.
5.  Set the following conditions:
    * `Client Name` - `contains` - `GA4` (or `Universal Analytics` if you use the legacy format)
    * `Event Name` - `matches RegEx` - `^(page_view|purchase|generate_lead)$`

    

    > **Important:** If your primary conversion event is not `purchase` or `generate_lead`, add it to the RegEx (e.g., `^(page_view|purchase|form_submission|my_event)$`).

### Step 1.3: Create the Click2Buy Tag

1.  In your GTM **Server** container, go to **Tags**.
2.  Click **New**.
3.  Name the tag (e.g., `Click2Buy S2S Attribution`).
4.  Click "Tag Configuration" and choose the **"Click2Buy S2S Attribution"** template you just imported (it will appear in the "Custom" list).
5.  Click "Triggering" and select the `[C2B] - PageView and Conversions` trigger created in step 1.2.
6.  Click **Save**.

---

## 2. Web (Client-Side) Container Configuration

This section ensures your Web GTM (in the browser) correctly sends **all** events to your Server GTM, so our tag can intercept them.

> If you already have a Web GTM setup that sends **all** its events (via a dynamic GA4 Event tag) to your `server_container_url`, you can skip this section.

### Step 2.1: Check the Google Tag (the "Brain")

1.  In your **Web** GTM container, find your main **"Google Tag"** (the one with your `G-XXXXXX` ID).
2.  Verify that it has a `server_container_url` configuration parameter pointing to your GTM-SS subdomain (e.g., `https://sgtm.your-site.com`).

### Step 2.2: Create the Event "Transporter" (if needed)

For your server tag to receive `purchase` events, you must send them. The best method is a single, dynamic event tag.

1.  In your **Web** container, go to **Tags** and click **New**.
2.  Name the tag (e.g., `[GA4] - All Events (to Server)`).
3.  Tag Type: **Google Analytics: GA4 Event**.
4.  Configuration Tag: Select your main Google Tag (from step 2.1).
5.  **Event Name:** `{{Event}}` (This is a built-in GTM variable that dynamically uses the event name from the `dataLayer`).

### Step 2.3: Create the "All Events" Trigger (if needed)

This tag must fire on all events your site sends.

1.  In the tag you are creating (step 2.2), click on **Triggering**.
2.  Create a **New** trigger (top-right corner).
3.  Name it (e.g., `Custom - All Events (with exceptions)`).
4.  Trigger Type: **Custom Event**.
5.  Event name: `.*` and check the **Use RegEx matching** box.
6.  This trigger fires on: **Some Custom Events**.
7.  Set the following condition to exclude default GTM events:
    * `{{Event}}` - `does not match RegEx` - `^(gtm\.(js|dom|load)|page_view)$`

8.  Save the trigger, then save the tag.

---

## 3. Publishing and Validation

You have completed the setup. All that's left is to publish and test.

1.  **Publish** the changes in your **Web** container.
2.  **Publish** the changes in your **Server** container.

### How to test the full flow:

1.  Open the **Preview** mode for your **Server** container.
2.  Open the **Preview** mode for your **Web** container.

**Test 1: The Attribution (Cookie Set)**

1.  In your browser, open a page on your site, adding our test parameter to the URL:
    `https://www.your-site.com/?c2bm=12345test`
2.  In your **Server Preview**, you should see a `page_view` event arrive.
3.  Click on this event. The `Click2Buy S2S Attribution` tag should appear in the **"Tags Fired"** section.
4.  In your browser's Developer Tools (F12) > Application > Cookies, you should see a **new first-party cookie** (e.g., `_c2b_attribution_id`) with the value `12345test`.

**Test 2: The Conversion (Data Send)**

1.  On your site (still in Preview mode), trigger a conversion event (e.g., complete a test purchase to trigger the `purchase` event).
2.  In your **Server Preview**, you should see the `purchase` event arrive.
3.  Click on this event. The `Click2Buy S2S Attribution` tag should again be in the **"Tags Fired"** section.
4.  Click this tag to inspect it. Go to the **"Outgoing HTTP Requests"** tab.
5.  You should see a request successfully sent to our tracking endpoint.

---

If both tests are successful, the installation is complete!

If you encounter any issues, please contact `retailers@click2buy.com`.
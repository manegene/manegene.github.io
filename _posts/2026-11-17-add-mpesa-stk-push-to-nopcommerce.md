---
title: "How to add Lipa na M-Pesa (STK Push) to a nopCommerce store"
description: "Set up Safaricom Daraja, build and configure the free Lipa na M-Pesa plugin for nopCommerce, test it in the sandbox and fix the errors you're most likely to hit."
tags: [M-Pesa, nopCommerce, Daraja, Payments]
---

Most online shoppers in Kenya would rather pay with M-Pesa than a card. If your shop runs on nopCommerce, you can offer M-Pesa at checkout with my free, open-source [Lipa na M-Pesa plugin](https://github.com/manegene/Nop.Plugin.Payments.LipanaMpesa). This guide walks through the whole setup: Safaricom's Daraja portal, building and installing the plugin, filling in its settings, testing, and the errors you're most likely to see.

## What your customer sees

1. At checkout, they choose **Lipa na M-Pesa** and type their M-Pesa number.
2. When they confirm the order, their phone shows the M-Pesa prompt with your business name and the order total. They enter their PIN and the money goes to your Paybill.
3. The checkout page also shows your Paybill number and account number. If the prompt doesn't arrive, they can pay from the M-Pesa menu instead (Lipa na M-Pesa → Pay Bill).

The prompt on the phone is what Safaricom calls **M-Pesa Express**, or STK Push.

## What you need

- A **nopCommerce 4.30** store, with its source code, so you can build the plugin. Other versions are covered [below](#using-another-nopcommerce-version).
- A **Paybill** with M-Pesa Express (Lipa na M-Pesa Online) switched on. Your bank or Safaricom can set this up.
- An account on **[Safaricom's Daraja portal](https://developer.safaricom.co.ke)**.
- Your store on **HTTPS**, because Safaricom only sends payment results to secure addresses.

## Step 1: Get your keys from Daraja

1. Sign in to Daraja and create an app with the **M-Pesa Express** product. It gives you a **Consumer Key** and a **Consumer Secret**.
2. For testing, open M-Pesa Express in Daraja's APIs section. Its simulator shows a **test shortcode** and **passkey** you can use straight away.
3. For your live shop, complete Daraja's **Go Live** process for your Paybill. Safaricom then gives you live keys and your Paybill's own passkey.

The plugin doesn't take the key and secret separately. It wants them joined as `key:secret` and then Base64-encoded, the same value Daraja uses to issue access tokens. Make it on your computer:

```bash
# macOS / Linux
printf '%s' 'YOUR_CONSUMER_KEY:YOUR_CONSUMER_SECRET' | base64
```

```powershell
# Windows PowerShell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes('YOUR_CONSUMER_KEY:YOUR_CONSUMER_SECRET'))
```

Keep the result private. It works like a password for your Paybill's API.

## Step 2: Build and install the plugin

1. Download the [source code](https://github.com/manegene/Nop.Plugin.Payments.LipanaMpesa) and copy the `Nop.Plugin.Payments.LipanaMpesa` folder into your store's `src/Plugins` folder.
2. In Visual Studio, add the project to the nopCommerce solution (right-click **Plugins → Add → Existing Project**).
3. Clean and build. The plugin builds straight into `Presentation/Nop.Web/Plugins/Payments.LipanaMpesa`.
4. Run the store, open **Administration → Configuration → Local plugins**, find **Lipa Na M-Pesa** and click **Install**. Restart the application when nopCommerce asks you to.

## Step 3: Fill in the settings

Click **Configure** next to the plugin:

| Setting | What to enter |
|---|---|
| Use sandbox | Ticked while testing; untick for real payments |
| Business number | Your Paybill number (the test shortcode while testing) |
| Authorization | The Base64 value from step 1 |
| Passkey | Your Paybill's passkey from Safaricom (the test passkey while testing) |
| Callback URL | An HTTPS address on your site where Safaricom sends the result of each payment. Safaricom requires one, even though the plugin doesn't read it yet (see [Before you go live](#before-you-go-live)) |
| Account reference | What customers see as the account, e.g. your shop's name |
| Transaction description | A short note on the payment, up to 13 characters, e.g. `Online order` |

If you run several stores from one nopCommerce, you can give each store its own settings with the store selector at the top of the page.

Then switch it on: **Configuration → Payment methods**, edit **Lipa Na M-Pesa** and tick **Is active**.

## Step 4: Test it

With **Use sandbox** ticked, place an order and pay with your own Safaricom number. Type it as `2547XXXXXXXX`, with no `+` or leading `0`. The prompt arrives on your phone, but no money moves in the sandbox.

## How it works

Knowing what happens underneath makes the errors easier to fix. For each order, the plugin makes two calls to Safaricom:

1. **Get an access token:** `GET /oauth/v1/generate?grant_type=client_credentials`, sending your Base64 value as Basic authentication.
2. **Send the prompt:** `POST /mpesa/stkpush/v1/processrequest` with the token and a body like this:

```json
{
  "BusinessShortCode": 174379,
  "Password": "<Base64 of shortcode + passkey + timestamp>",
  "Timestamp": "20261117143005",
  "TransactionType": "CustomerPayBillOnline",
  "Amount": 1500,
  "PartyA": 254712345678,
  "PartyB": 174379,
  "PhoneNumber": 254712345678,
  "CallBackURL": "https://yourshop.co.ke/mpesa/callback",
  "AccountReference": "MyShop",
  "TransactionDesc": "Online order"
}
```

The `Password` isn't your Daraja password. It's the shortcode, passkey and timestamp joined together and Base64-encoded, and it changes with every request. The plugin works this out for you.

## Common errors and fixes

**"Invalid Access Token" or "Invalid credentials"**
The Authorization value is wrong, or it doesn't match the setting. Sandbox keys only work with **Use sandbox** ticked, and live keys only with it unticked. Make the Base64 value again with no spaces or line breaks.

**"Bad Request – Invalid PhoneNumber"**
The number isn't in the `2547XXXXXXXX` format. Tell customers this on the checkout page.

**"Bad Request – Invalid CallBackURL"**
The address must be public and HTTPS. `localhost` and plain `http://` addresses are rejected.

**The prompt never arrives**
Check that the business number and passkey belong to the same Paybill, and that M-Pesa Express is switched on for it. The customer's phone also needs to be on and on the network. The Paybill instructions at checkout are the fallback.

**The prompt fails for some order totals**
M-Pesa takes whole shillings. If your prices or delivery fees produce cents (e.g. KES 1,499.50), round them so order totals are whole numbers.

## Before you go live

The plugin sends the payment prompt, but it doesn't read Safaricom's result to update the order. Orders are created as **Pending**. When the money arrives (your M-Pesa SMS, or the M-Pesa org portal), mark the order as paid in **Sales → Orders**.

On a busy shop, the next step is a small endpoint at your callback URL. It reads Safaricom's result and marks the order paid automatically, and also catches customers who cancel or enter the wrong PIN.

## Using another nopCommerce version

The plugin is built for 4.30. For another version:

1. Change the project's .NET version to match your store.
2. Update its NuGet packages to the same versions as the store's.
3. Add it to that store's solution as a plugin project, then clean and build.

## Need a hand?

If you get stuck, or want the automatic callback built for your store, [get in touch](/#contact). For card payments too, see my [Pesapal plugin for nopCommerce](https://github.com/manegene/payments.pesapal).

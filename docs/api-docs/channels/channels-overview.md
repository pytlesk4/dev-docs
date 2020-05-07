# Channels Overview

<div class="otp" id="no-index">

### On this Page
- [Section1](#section1)
- [Section2](#section2)
- [Section3](#section3)
- [Section4](#section4)
- [Section5](#section5)
- [Section6](#section6)
- [Resources](#resources)

</div>



With Channels, BigCommerce aims to create a consistent, easily understood method to integrate both first and third-party sales channels into a merchant's day-to-day management experience.

It's attempting to solve these key pain points:

* Built-in (first-party) and external (third-party) solutions all have different onboarding, channel settings, listing products to those channels, and ongoing channel management
* Catalog data is spread across various integrated systems, preventing streamlined interfaces to view and manage visibility, purchasability, price and product overrides
* Creating unique shopping experiences outside of BigCommerce that naturally inherit from the core catalog isn't possible without a middleware system
* Synchronizing catalogs across sales channels is really hard, requiring a lot of upfront orchestration and UI 

Channel streamline how third-party developers interact with and add additional sales channels to BigCommerce merchant stores. It's at the heart of what's required to build Channel Apps, which have the ability to show natively within BigCommerce's Control Panel, in the Channel Manager.

### What's Inside?
* A collection of APIs and generated UX that enable:
  * Ability to create, manage and modify Channel Instances and their Listings via OAuth APIs
  * Ability to have your Channel show within the merchant's Channel Manager in BigCommerce's control panel
* Ability to define which of our Channel Components you'd like us to programmatically render within the Channel Manager and where we should request the data from your servers to power them, via a simple JSON manifest
* Ability to respond to merchant and API initiated Channel and Listing events
* Ability to use BigCommerce to handle sync orchestration

## Types of Channel Experiences
The key types of channel experiences fit nicely into these buckets:

|  Type | Description |
| --- | --- |
|  Point of Sale | Add the ability for merchants to link their in-store and online inventory and keep them in sync. |
|  Marketplace | Open up merchants to new online sales opportunities without dramatically increasing day-to-day effort |
|  Mobile / Interactive | Streamline development of off-canvas apps that unlock unique browsing and purchasing experiences |
|  Storefront | Power ‘headless' storefronts via open source or custom solutions that easily inherit from and override the merchant's core catalog |
|  Channel Extension | Configure listings across all integrated sales channels; automatically broadcast to the right content and advertising networks; provide cross-channel analytics and insights |

All types are typically adding new sales channels, while Channel Extensions utilize data from existing sales channels.## Getting Started

## Getting Started
First, you'll need a set of BC OAuth API credentials for a store with the 'channels' scope:

|  **Resource** | **Parameter** | **UI Name** |
| --- | --- | --- |
|  Channels | `channels_create` | Channel Listings modify |
|  Channels | `channels_read` | Channel Listings read-only |
|  Channels | `channels_update` | Channel Listings modify |
|  Channels | `channels_delete` | Channel Listings modify |
|  Channel Listings | `channels_listings_create` | Channel Listings modify |
|  Channel Listings | `channels_listings_read` | Channel Listings read-only |
|  Channel Listings | `channels_listings_update` | Channel Listings modify |
|  Channel Listings | `channels_listings_delete` | Channel Listings modify |

## Registering a Channel
After creating your app in our developer portal, you must request that app to be turned into a Channel App by Bigcommerce along with the Channel(s) you’d like that app to work with.

If the Channel is new to BigCommerce, we’ll create a new ID in our system.

## Defining a Channel's UX
Channels are made up of predefined components that are displayed in the BigCommerce admin UI natively, but powered by data sent from an external server. This enables all Channel apps to share a consistent UX without the need for each developer to design and host the front-end.

There are two key Channel flows that must be integrated for it to show natively within the Channel Manager:

* Discovery
  * Marketing Page
  * Channel Requirements
  * Channel Connection
* Management
  * Getting Started
  * Import / Export
  * Channel Settings
  * Support

Each section has an associated manifest that enables BigCommerce to know what to generate. Some are static, like the marketing page, which looks like this:

![Channel Manifest Example](https://raw.githubusercontent.com/bigcommerce/dev-docs/master/assets/images/ "Channel Manifest Example")

The content above is represented by the following manifest:

```json
{
  "name": "bce-provider-marketing-content",
  "attrs": {
    "hero": "Sell anywhere with Buy Buttons",
    "intro": "Sell directly from your blog, emails, and social media posts with customizable Buy Buttons...",
    "imgUrl": "https://microapps.bigcommerce.com/embeddable/v1/static/buy-buttons-hero.png",
    "features": [
      {
        "title": "Compatible with 100s of external sites",
        "text": "Copy and paste directly into blog platforms like Wordpress, Drupal..."
      },
      {
        "title": "Match your brand",
        "text": "Choose a pre-made theme or edit specific text and colors to match with your branding perfectly."
      },
      {
        "title": "Easily Optimize",
        "text": "Integrate Google Analytics to track views and conversions."
      }
    ]
  }
}
```

There are also dynamic components like settings forms that can reach out to an external server for the information:

![Dynamic Components Example](https://raw.githubusercontent.com/bigcommerce/dev-docs/master/assets/images/ "Dynamic Components Example")

## Addng a Channel to a Store

Channels are what show up in the BC Control Panel when you are listing to a Channel or managing it’s Orders.

Optionally, it can also house additional information we use to alter BC Checkout redirects, so the shopper is redirected back to the Channel that referred them, whether it be an external site or app.

To add channel to a store, send a POST request to `/stores/{{STORE_HASH}}/v3/channels`:

```http
POST https://api.bigcommerce.com/stores/{{STORE_HASH}}/v3/endpoint
X-Auth-Token: {{ACCESS_TOKEN}}
X-Auth-Client: {{CLIENT_ID}}
Content-Type: application/json
Accept: application/json


{
"type": "channel_type",
"name": "yoursite.store",
"external_id": "external channel's unique identifier",
"config": {
    "default_price_list_id": 1234,
    "storefront_urls": {
        "home": "https://yoursite.store/",
        "cart": "https://yoursite.store/cart",
        "register": "https://yoursite.store/sign-up",
        "confirmation": "https://yoursite.store/thank-you"
    }
}
```
[![Open in Request Runner](https://storage.googleapis.com/bigcommerce-production-dev-center/images/Open-Request-Runner.svg)](https://developer.bigcommerce.com/api-reference/cart-checkout/channels-listings-api/channels/createchannel#requestrunner)

**Response:**

```json
{
"id": 123456,
"type": "wordpress",
"name": "mysite.store",
"external_id": "example_external_id",
"config": {...},
"created_at": "2018-11-04 number number number",
"updated_at": "2018-11-04 number number number"
}

```

For a complete Channels API reference (including request schemas and property descriptions), see: [API Reference > Channels](https://developer.bigcommerce.com/api-reference/cart-checkout/channels-listings-api/channels/createchannel).

## Updating a Channel

## Deleting a Channel

## Importing a BigCommerce Catalog

## Adding Listings to a Channel

## Updating Channel Listings

## Get Channel Listings

## Deleting Channel Listings

## Creating Channel Listing Errors

## Handling Merchant Subscriptions

## Resources
CHANNEL MANAGER: `/manage/channel-manager`



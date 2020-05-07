# Developing a POS Integration

<div class="otp" id="no-index">

### On this Page
- [Integrating the Catalog](#integrating-the-catalog)
- [Importing Sales History](#importing-sales-history)
- [Syncing Gift Card Balances](#syncing-gift-card-balances)
- [Importing Coupons](#importing-coupons)
- [Configuring Prices](#configuring-prices)
- [Creating a BigCommerce App](#creating-a-bigcommerce-app)
- [Developing the User Interface](#developing-the-user-interface)
- [User Experience Best Practices](#user-experience-best-practices)
- [Releasing Your App](#releasing-your-app)
- [Resources](#resources)

</div>

The purpose of this guide is to provide resources, guidelines and best practices to assist you in the development of your POS Integration.

## Integrating the Catalog
The main feature that merchants would expect from your application is Catalog Integration - a seamless way for a merchant to sync products between systems.

At a high level, this portion of the integration should:

Capture and Compare:  Pull products from both BigCommerce and the Point of Sale, then loop through and compare to see if any matches are found.
The comparison piece is done against a unique identifier that can be found in both systems - its typically best to index off of product SKU or UPC if possible.

Update or Create:  If a match is found in the above step, update the existing product.  If no match is found, create a new product.

![Catalog Diagram](https://raw.githubusercontent.com/bigcommerce/dev-docs/master/assets/images/point-of-sale-integration-01.png "Catalog Diagram")

### Understanding Products

Below are some terms used to describe how products are structured within BigCommerce:

* Variants - A variant represents the purchasable product.  The item as it sits on the shelf ready to be shipped.

* Variant Options - This is a collection of choices a shopper would need to make in order to select and purchase a specific variant.
Example:  size, color, material.

* Modifiers - This is a collection of choices a shopper could make that would change how the merchant fulfills the order.
Example:  custom embroidery.

Given the above terms, products can be 'simple' or 'complex':

*  Simple Products - This is a physical product that is sold 'as-is' - there are no variant options that a shopper would need to select in order to add that variant to their shopping cart. Example: One Size Fits All Hat  [Variant]

* Complex Products: This consists of a parent product (called a 'base variant') that contains variant options / modifiers. Example:
  * Simple T-Shirt  [Base Variant]
  * Color: White, Blue, Red, Black  [Variant Options]
  * Size:  Small, Medium, Large, X-Large  [Variant Options]
  * A selection of color and size would create the variant.

The below screenshot is an example of what the variant options might look like for this 'Simple T-Shirt' product in a BigCommerce storefront.

Without selecting the color and size, the product cannot be added to the shoppers cart.
An error toast will be provided to the user indicating that the required options have not been selected.

You can see that the shopper has selected white for the color, and x-large for the size - this combination created the SKU:  XL-WT-SHIRT001.
This SKU identifies the specific variant of the Simple T-Shirt product that is being purchased.

![Product SKU Example](https://raw.githubusercontent.com/bigcommerce/dev-docs/master/assets/images/point-of-sale-integration-02.png "Product SKU Example")

### Product Requirements

At minimum, all products in BigCommerce must have a name, a price, a type, a weight, and a category.

"name":  This is the name of the product as it will appear in your storefront.
Datatype:  String
Example:  "BigCommerce T-Shirt"

"price":  This is the price as it will appear in your storefront.
Datatype: Float
Example:  10.95

"type":  The product type.
Datatype:  String
Example:  "physical"
Notes:  This can be of type "digital" or "physical".  For Point of Sale integrations, its likely "physical" would always be the defined product type.
	digital = a downloadable product - one that does not have a physical representation in the store.
	physical = a physical product that would be shipped or picked up in store.

"weight":  The product weight - used for calculating shipping costs.
Datatype:  Float
Example:  2.25
Notes:  If products do not have a weight attribute in the source system, a default weight of 0 could be used.  A merchant would need to manually adjust the product weight after import into BigCommerce.

"categories":  The category that the product belongs to.
Datatype:  Array[ints]
Example:  [ 10 ]
Notes:  Categories must first be created in BigCommerce - the ID of the object would be returned at time of creation - this ID would then be used to map a product to 1 or more categories.
If products are not mapped to a category in the source system, a default category will need to be defined and used.

Although these are the minimum required properties for a product - to provide the best user experience, it is a best practice to map as many of the properties between the two systems as possible.
For the full schema of a product object, please review the 'Request Body' section of Create a Product.

Channels and Listings

A Channel is an external sales platform that a merchant can share specific products with.

In the product control panel of BigCommerce, a merchant has the ability to 'list' products individually to the channel of their choice.
For example:  Utilizing the Channels API in this integration would allow a merchant to list certain products for sale only on their BigCommerce Storefront, or only on their Point of Sale.

See the next section for more resources on the specific endpoints that would be used.

Product Import / Export

Product Import would allow the merchant to take the products already created in their point of sale, and sync them into BigCommerce.
Note:  As a user has chosen to import products into their storefront, it would be expected that on creation,  products would automatically be listed on both their Storefront and Point of Sale channels.

Product Export would allow the merchant to take the products already created in their BigCommerce storefront and sync them into their point of sale.
Note:  A merchant may have specific products they would like to sell exclusively online - product exports should only consider active BigCommerce products that have been listed to the Point of Sale channel.

The following endpoints would likely be used in the product import and export logic:
Catalog API
{base url}/v3/catalog/categories
{base url}/v3/catalog/products
Channels API
{base url}/v3/channels
{base url}/v3/channels/{channel id}/listings

Inventory Updates

In addition to importing/exporting products, some merchants may wish to keep product inventory synced between their point of sale and BigCommerce.

Inventory is a property of a product - the following properties would be used to manage inventory:

"inventory_tracking":  The inventory tracking type for the product.
Datatype:  String
Example:  "product"
Notes:  This can be of type "none", "product", or "variant".
none = default value - inventory will not be tracked.
product = tracking inventory at the product level.
variant = tracking inventory at the variant level.

"Inventory_level":  The current available inventory for the product.
Datatype:  Integer
Example:  100
Notes:  Inventory levels will not be displayed unless the "inventory_tracking" property is defined as type  "product" or "variant".

The following endpoints would be used in the inventory update logic:
Catalog API
{base url}/v3/catalog/products

## Importing Sales History
Customer and Order Import / Export

Customer / Order Import would allow the merchant to import the sales history from their point of sale into BigCommerce.
Note: In BigCommerce, orders map to specific customers - if the point of sale does not have a concept of customers, an order can be created with a customer_id of 0 - this value is reserved for ‘Guest’ shoppers.

Customer / Order Export would allow the merchant to export the sales history from BigCommerce into their point of sale.

The following endpoints would be used in the sales integration logic:
Customers V3 API
{base url}/v3/customers
Orders API
{base url}/v2/orders

## Syncing Gift Card Balances

The Marketing API can be leveraged to sync BigCommerce gift certificates with gift cards purchased in store.  When a gift card is used in person, the associated gift certificate in BigCommerce should be updated (by changing the certificate’s ‘balance’ value).
The same general process should take place when the gift certificate is used online - the remaining balance should be updated for the associated gift card in the point of sale.

## Importing Coupons
The Marketing API also enables the use of coupons.  Physical coupons can be added to the merchant’s storefront, where shoppers can apply them during the checkout process.

The following endpoints would be used in the gift card or coupon integration logic:
Marketing API
	{base url}/v2/gift_certificates
	{base url}/v2/coupons

## Configuring Prices
Content

## Creating a BigCommerce App
Single-Click App

A Single-Click app is listed in the BigCommerce App Marketplace, and is available for all BigCommerce merchants to install.
“Single-Click” refers specifically to the installation process - the application should connect to the point of sale service via the Single-Click App Auth Flow - instead of a manual process where a merchant would need to generate and input authentication credentials into the app’s UI.

https://github.com/bigcommerce/channels-app

The Channels App serves two main purposes, as:

A reference implementation for:
Channel, Sites, and Routes APIs
BigDesign React Components
A way to manage the channels connected to a BC storefront and their corresponding sites and routes

## Developing the User Interface
As your application will be embedded in a BigCommerce dashboard, it is important for the app to look and feel native to the rest of the BigCommerce UI.

We’ve developed a library of React components to enable developers to quickly build out a frontend that meets these design standards - BigDesign.

## User Experience Best Practices
To enable the best user experience, below are a few items to keep in mind as you develop your application:

Modular Features

As the specific needs of every merchant is different, the features of your app should be as ‘modular’ as possible.

For merchants:
Consider the audience and various use cases for your application.
Example:
Merchant A:  only interested in importing products and inventory levels from their point of sale.
Merchant B:  interested in a bi-directional integration of products and sales - keeping the data between their point of sale and their online storefront in sync.
Merchants should have the option to opt-in or out of the various features of your application.

For support:
If an issue occurs, a merchant should have the option to disable a specific feature that may not be behaving as expected.  The idea here is to allow the merchant to continue using the features that are functional while temporarily resolving an issue - instead of having to disable the application completely.

One Time Sync vs Automated Sync

For the various features of your application, it is typically a best practice to offer both a manual ‘one time sync’ option and automated sync options (based on various sync intervals).

This will provide the merchant with more control over what is synced and when.

For Automated Syncs - example intervals:
	Do not update automatically:  The merchant should have an option to opt-out of automatic syncs.
	Every X Minutes:  This would trigger the associated event every x minutes.
	Real Time:  This would utilize webhooks to provide ‘real time’ updates.

App Performance

To provide merchants with a positive integration experience, we expect point of sale applications to meet or exceed the following benchmark:

Catalog Import / Export:  100 Complex Products per Second.
Note:  this volume of requests per second may hit the rate limits of lower tier BigCommerce plans - logic should be implemented around the response headers to ensure your application does not exceed the allowable number of requests for a given storefront.

For increased performance, consider using batch operations and parallel requests when possible.
Example:  It is not currently possible to create multiple unique products in a single request, however it is possible to create a product and all of its variants in a single request.
It is also possible to create multiple unique products at one time by making multiple requests in parallel.

Logging

A log of all events should be kept and made accessible to the merchant utilizing the application.
Logs should be broken out per service.  For example:  On the app page where a user would go to interact with product import functionality - they would expect to find all logs relating to that specific feature.

Logs should be provided in a ‘light’ human-readable format - appended to a running feed of logs.
Product Import Example:
20 Products Identified - 5 Products Created - 15 Products Updated - 0 Errors   |   12/10/19 @ 3:45PM CST

The merchant should also have the option to download verbose logs (csv file).
These logs would provide more granular details on the event that took place and all affected products.
Product Import Example:
POS ID, BC ID, EVENT
34, 103, Product Created
35, --, Error: Product Creation Skipped [Invalid product name]

## Releasing Your App
Listing apps on the BigCommerce App Marketplace is reserved for Technology Partners.
Apply for the program here.
This article highlights app approval requirements.


## Resources
* [Channels Sample App](https://github.com/bigcommerce/channels-app)
* [Big Design Component Library](https://developer.bigcommerce.com/big-design/?path=/story/badge--overview)
* https://developer.bigcommerce.com/api-docs/getting-started/best-practices

The below section highlights some basics about interacting with BigCommerce APIs.

### API Clients
* Python
* PHP
* Ruby
* Node

### Relavent Endpoints
* Catalog - product, category and brand management
* Channels - channel and listing management
* Orders -
* Order Transactions
* Order Payment Actions
* Payment Processing
* Customers
* Price Lists - customer group pricing management (B2B)
* Store Information - store setting management
* Currencies - store currency management
* Marketing - coupon and gift card management
* Webhooks
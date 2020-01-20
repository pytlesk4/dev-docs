# Working with the Storefront Cart and Checkout APIs

<div class="otp" id="no-index">

### On This Page
- [Prerequisites:](#prerequisites)
- [Setup: Create a Helper Function](#setup-create-a-helper-function)
- [Creating a Cart](#creating-a-cart)
- [Getting a Cart](#getting-a-cart)
- [Adding Items to Cart](#adding-items-to-cart)
- [Storefront Checkout](#storefront-checkout)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

</div>

This tutorial demonstrates how to use the JavaScript to make common Storefront API requests.

Interaction with Storefront API is intended to be done via JavaScript. 
`HTTPS` is required for storefront API requests
Requests should be made against a store's [permanent URL](https://forum.bigcommerce.com/s/article/Changing-Domains) (otherwise, [CORs](https://developers.google.com/web/ilt/pwa/working-with-the-fetch-api#cross-origin_requests) (developers.google.com) errors may result).

## Prerequisites:
* BigCommerce Store with at least two [products](/api-reference/catalog/catalog-api/products/createproduct) and a [shipping option](/api-docs/shipping/shipping-overview#shipping_shipping-zone-methods) available. 
* Familiarity with JavaScript and [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) (developer.mozilla.org)
* Familiarity with your browser's javascript console and developer tools

## Setup: Create a Helper Function

In service of following [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) (wikipedia.org) programming principles, `POST` requests in this tutorial makes use of the following helper functions:

```js
function postData(url = '', cartId = '', requestJson = {}) {
	return fetch(url, {
		method: "POST",
		credentials: "same-origin",
		headers: {
			"Content-Type": "application/json" },
		body: JSON.stringify(requestJson), 
	})
	.then(response => {
		return response.json();
	})
}

function deleteCartItem(url = '', cartId = '', itemId = '') {
	return fetch(url + cartId + '/items/' + itemId, {
		method: "DELETE",
		credentials: "same-origin",
		headers: { 
			"Content-Type": 
			"application/json", 
		}
		})
		.then(response => response.json());
}
```

Navigate to your BigCommerce storefront, then paste the helper functions into your browser's JavaScript console. Once you do, you'll be able to use them to make requests while going through this tutorial (so long as you don't navigate away or reload the page):

![Developer Tools Example](https://raw.githubusercontent.com/bigcommerce/dev-docs/master/assets/images/working_with_sf_aps_01.png "Developer Tools Example")

This function uses **Fetch API's** `fetch()` method to make `HTTP` requests and return the response JSON. For more information on using Fetch API, see [Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

## Creating a Cart

```javascript
postData(`/api/storefront/cart`, {
        "lineItems": [
        {
            "quantity": 1,
            "productId": 191
        },
        {
            "quantity": 1,
            "productId": 185
        }
        ]}
    )
  .then(data => console.log(JSON.stringify(data))) 
  .catch(error => console.error(error));
```

When adding a product with variant, include the `variantId` or `optionId` in the then the request:

```js
postData(`/api/storefront/cart`, {
  "lineItems": [
    {
      "quantity": 1,
      "productId": 77,
      "optionSelections": [
        {
          "optionId": 120,
          "optionValue": 69
        },
        {
          "optionId": 121,
          "optionValue": 10
		}
      ]
    }
  ]
})
.then(data => console.log(JSON.stringify(data))) 
.catch(error => console.error(error));
```

For additional details, see [API Reference > Storefront Carts > Create a Cart](https://developer.bigcommerce.com/api-reference/cart-checkout/storefront-cart-api/cart/createacart).

## Getting a Cart

Getting cart data can be accomplished by making a `GET` request to `/api/storefront/cart`: 

```js
fetch('/api/storefront/cart', {
  credentials: 'same-origin'}
     )
  .then(function(response) {
    return response.json();
  })
  .then(function(myJson) {
    console.log(JSON.stringify(myJson));
  });
  ```

To get complete line item information, append `include=lineItems.digitalItems.options,lineItems.physicalItems.options` to the URL:

```js
fetch('/api/storefront/cart?include=lineItems.digitalItems.options,lineItems.physicalItems.options', {
		credentials: 'same-origin'
	})
  .then(function(response) {
    return response.json();
  })
  .then(function(myJson) {
    console.log(JSON.stringify(myJson));
  });
```

See [API Reference > Storefront Carts > Get a Cart](https://developer.bigcommerce.com/api-reference/cart-checkout/storefront-cart-api/cart/getacart) for example responses as well as a list of available query parameters.

## Adding Items to Cart


```js
postData(`/api/storefront/carts/`, `1d2d2445-5e5d-4798-ada1-37652a7822c8` ,{
    "lineItems": [
      {
        "quantity": 3,
        "productId": 133
      }
    ]
  })
  .then(data => console.log(JSON.stringify(data))) 
  .catch(error => console.error(error));
  
function postData(url = '', cartId = '', cartItems = {}) {
      return fetch(url + cartId + '/items', {
          method: "POST",
          credentials: "same-origin",
          headers: {
              "Content-Type": "application/json" },
          body: JSON.stringify(cartItems), 
      })
      .then(response => response.json()); 
}
```

### Delete Cart Item

In the code below there are a few changes. One is the arguments for deleteCartItem() now accept a cartId and itemId as strings. These are passed into the deleteCartItem() at the top. The URL is being built using concatenation. 

We have also introduced a new way to handle errors. Error handling in fetch can be pulled out into a standalone function and be used to return any data or messages you want as a way to keep the code [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). 

<div class="HubBlock--callout">
<div class="CalloutBlock--warning">
<div class="HubBlock-content">
    
<!-- theme: warning -->
### Delete Cart Items
> Deleting the last item in a cart deletes the cart.

</div>
</div>
</div>


```js
deleteCartItem(`/api/storefront/carts/`, `f996cb68-b1df-422e-b3dd-0f90faa10210`, `e51ac38d-dacd-449d-b503-f087f14bde67`)
.then(data => console.log(JSON.stringify(data)))
.catch(error => console.log(error))

function deleteCartItem(url = '', cartId = '', itemId = '') {
	return fetch(url + cartId + '/items/' + itemId, {
		method: "DELETE",
		credentials: "same-origin",
		headers: { 
			"Content-Type": 
			"application/json", 
		}
		})
		.then(response => response.json());
}
```

## Storefront Checkout
Next, we will cover using the Storefront Checkout to add a billing address, add a shipping address and update a shipping address to add the shipping option.

Make sure you have created a cart using the Storefront Cart, added two different `lineItems` and have a shipping method set up on the store. See [Create Cart](#working-sf-apis_storefront-cart) above if you deleted your cart and need to make a new one. 

<div class="HubBlock--callout">
<div class="CalloutBlock--info">
<div class="HubBlock-content">
    
<!-- theme:  -->
### Checkout ID
> checkoutId and the cartId are same.

</div>
</div>
</div>

### Add Billing Address to Checkout

A billing address is required to complete a checkout. In postData() we pass in the `checkoutId` and the billing address object.

<!--
title: "Add billing address"
subtitle: ""
lineNumbers: true
-->

**Example Add Billing Address**  
`https://<store_url>/api/storefront/checkouts/{checkoutId}/billing-address`

```js
postData(`/api/storefront/checkouts/`, `e8b7c677-f67a-4e39-a5ed-f405c9a06bcf`, {
"firstName": "Jane",
            "lastName": "Doe",
            "email": "janedoe@email.com",
            "company": "BigCommerce",
            "address1": "123 Main Street",
            "address2": "Apt 1",
            "city": "Austin",
            "stateOrProvinceCode": "TX",
            "countryCode": "USA",
            "postalCode": "78751"
})
  .then(data => console.log(JSON.stringify(data))) 
  .catch(error => console.error(error));

function postData(url = ``, checkoutId = ``, data = {},) {
    return fetch(url + checkoutId + `/billing-address`,  {
        method: "POST", 
        credentials: "same-origin",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify(data),
    })
    .then(response => response.json()); 
}

```

### Add Shipping Address or Consignment to Checkout 

A consignment consists of a shipping address with the associated lineItems.  At a minimum, at least one shipping address with line items and shipping options must be part of the checkout.

A shipping address can only be added to checkout with lineItems. If multiple shipping locations are used, match each `lineItem` with the correct shipping address as shown in the example below. For more examples see [Create Consignment](/api-reference/cart-checkout/storefront-checkout-api/checkout/checkoutsconsignmentsbycheckoutidpost).

When adding a shipping address to the checkout `?include=consignments.availableShippingOptions` must be included to return the shipping options available for any address. To add the shipping option a [put request](/api-reference/cart-checkout/storefront-checkout-api/checkout/checkoutsconsignmentsbycheckoutidandconsignmentidput) must be sent for each consignment. We will cover this in the next section. 

To get the line item IDs needed for consignment, send a request to [/GET Checkout](/api-reference/cart-checkout/storefront-checkout-api/checkout/checkoutsbycheckoutidget). Try to modify the /GET Cart request so it returns Checkout Details. If you are having trouble, see the code sample below. 

<!--
title: "Get Checkout by ID"
subtitle: ""
lineNumbers: true
-->
**Example Get Checkout by ID**  
`/GET https://<store_url>/api/storefront/checkouts/{checkoutId}`

```js
fetch('/api/storefront/checkouts/1650fb51-172b-4cde-a220-90c6a8ef9293', {
  credentials: 'same-origin'}
     )
  .then(function(response) {
    return response.json();
  })
  .then(function(myJson) {
    console.log(JSON.stringify(myJson));
  });
```

<!--
title: "Get Checkout Response"
subtitle: ""
lineNumbers: true
-->
**Example Get Checkout Response**

```json
{
	"id": "1650fb51-172b-4cde-a220-90c6a8ef9293",
	"cart": {
		"id": "1650fb51-172b-4cde-a220-90c6a8ef9293",
		"customerId": 0,
		"email": "janedoe@email.com",
		"currency": {
			"name": "US Dollars",
			"code": "USD",
			"symbol": "$",
			"decimalPlaces": 2
		},
		"isTaxIncluded": false,
		"baseAmount": 73.95,
		"discountAmount": 0,
		"cartAmount": 73.95,
		"coupons": [],
		"discounts": [{
			"id": "7349b13a-1453-4050-a769-1a6ad1823369",
			"discountedAmount": 0
		}, {
			"id": "4a69cbdf-4320-4e1f-852b-0edc2a55f13a",
			"discountedAmount": 0
		}],
		"lineItems": {
			"physicalItems": [{
				"id": "7349b13a-1453-4050-a769-1a6ad1823369",
				"parentId": null,
				"variantId": 362,
				"productId": 191,
				"sku": "",
				"name": "Openhouse No. 3",
				"url": "https://{store_url)/all/openhouse-no-3/",
				"quantity": 1,
				"brand": "Openhouse Magazine",
				"isTaxable": true,
				"imageUrl": "https://cdn11.bigcommerce.com/s-{store_hash)/products/191/images/475/openhousevol3_1024x1024__59692__16355.1534344544.330.500.jpg?c=2",
				"discounts": [],
				"discountAmount": 0,
				"couponAmount": 0,
				"listPrice": 27.95,
				"salePrice": 27.95,
				"extendedListPrice": 27.95,
				"extendedSalePrice": 27.95,
				"isShippingRequired": true,
				"giftWrapping": null,
				"addedByPromotion": false
			}, {
				"id": "4a69cbdf-4320-4e1f-852b-0edc2a55f13a",
				"parentId": null,
				"variantId": 356,
				"productId": 185,
				"sku": "",
				"name": "Utility Caddy",
				"url": "https://{store_url)/all/utility-caddy/",
				"quantity": 1,
				"brand": "OFS",
				"isTaxable": true,
				"imageUrl": "https://cdn11.bigcommerce.com/s-{store_hash)/products/185/images/449/utilitybucket1_1024x1024__78563__75042.1534344535.330.500.jpg?c=2",
				"discounts": [],
				"discountAmount": 0,
				"couponAmount": 0,
				"listPrice": 46,
				"salePrice": 46,
				"extendedListPrice": 46,
				"extendedSalePrice": 46,
				"isShippingRequired": true,
				"giftWrapping": null,
				"addedByPromotion": false
			}],
			"digitalItems": [],
			"giftCertificates": [],
			"customItems": []
		},
		"createdTime": "2018-11-06T19:22:51+00:00",
		"updatedTime": "2018-11-06T19:25:26+00:00"
	},
	"billingAddress": {
		"id": "5be1eaa653e37",
		"firstName": "Jane",
		"lastName": "Doe",
		"email": "janedoe@email.com",
		"company": "BigCommerce",
		"address1": "123 Main Street",
		"address2": "Apt 1",
		"city": "Austin",
		"stateOrProvince": "",
		"stateOrProvinceCode": "",
		"country": "",
		"countryCode": "",
		"postalCode": "78751",
		"phone": "",
		"customFields": []
	},
	"consignments": [],
	"orderId": null,
	"shippingCostTotal": 0,
	"shippingCostBeforeDiscount": 0,
	"handlingCostTotal": 0,
	"taxTotal": 12.22,
	"coupons": [],
	"taxes": [{
		"name": "This is tax",
		"amount": 12.22
	}],
	"subtotal": 73.95,
	"grandTotal": 86.17,
	"giftCertificates": [],
	"createdTime": "2018-11-06T19:22:51+00:00",
	"updatedTime": "2018-11-06T19:25:26+00:00",
	"customerMessage": ""
}
```

<div class="HubBlock--callout">
<div class="CalloutBlock--info">
<div class="HubBlock-content">
    
<!-- theme:  -->
### Add a Cart Item
>  If your cart only has one lineItem or a quantity of one, run a [POST Cart](/api-reference/cart-checkout/storefront-cart-api/cart/createacart) request with a new lineItem, then come back here.

</div>
</div>
</div>

Below, there are two shipping addresses in an array with a lineItem assigned to each. Note that `?include=consignments.availableShippingOptions` is being added as a query parameter. Without this, the `availableShippingOptions` will not return in the response. 

<!--
title: "Create Consignment"
subtitle: ""
lineNumbers: true
-->
**Example Create Consignment**  
`/POST https://<store_url>/api/storefront/checkouts/{checkoutId}/consignments`

```js
postData(`/api/storefront/checkouts/`, `1650fb51-172b-4cde-a220-90c6a8ef9293`,
[{
        "shippingAddress": {
            "firstName": "Jane",
            "lastName": "Doe",
            "email": "janedoe@email.com",
            "company": "BigCommerce",
            "address1": "123 Main Street",
            "address2": "Apt 1",
            "city": "Austin",
            "stateOrProvinceCode": "TX",
            "countryCode": "US",
            "postalCode": "78751"
        },
        "lineItems": [{
            "itemId": "fb924c6c-10fb-456a-bccb-02d9fb426199",
            "quantity": 1
        }]
    },
    {
        "shippingAddress": {
            "firstName": "John",
            "lastName": "Doe",
            "email": "johnedoe@email.com",
            "company": "BigCommerce",
            "address1": "123 South Street",
            "address2": "Apt 5",
            "city": "Austin",
            "stateOrProvinceCode": "TX",
            "countryCode": "US",
            "postalCode": "78726"
        },
        "lineItems": [{
            "itemId": "98ceac68-cac9-49af-9050-95494f32474c",
            "quantity": 1
        }]
    }
    ]

)
  .then(data => console.log(JSON.stringify(data))) // JSON-string from `response.json()` call
  .catch(error => console.error(error));

function postData(url = ``, checkoutId = ``, data = {},) {
    return fetch(url + checkoutId + `/consignments?include=consignments.availableShippingOptions`,   {
        method: "POST", 
        credentials: "same-origin",
        headers: {
            "Content-Type": "application/json" ,
        },
        body: JSON.stringify(data), 
    })
    .then(response => response.json()); }
```

<div class="HubBlock--callout">
<div class="CalloutBlock--warning">
<div class="HubBlock-content">
    
<!-- theme: warning -->
### Signed In Customer
> When a signed in customer proceeds to the create consignment step with an incomplete shipping address, the shipping address will auto-populate with the the most recently used address from the customer's address book.

</div>
</div>
</div>

<!--
title: "Create Consignment Response"
subtitle: ""
lineNumbers: true
-->

**Example Create Consignment Response**

```json
{
	"id": "1650fb51-172b-4cde-a220-90c6a8ef9293",
	"cart": {
		"id": "1650fb51-172b-4cde-a220-90c6a8ef9293",
		"customerId": 0,
		"email": "janedoe@email.com",
		"currency": {
			"name": "US Dollars",
			"code": "USD",
			"symbol": "$",
			"decimalPlaces": 2
		},
		"isTaxIncluded": false,
		"baseAmount": 73.95,
		"discountAmount": 0,
		"cartAmount": 73.95,
		"coupons": [],
		"discounts": [{
			"id": "7349b13a-1453-4050-a769-1a6ad1823369",
			"discountedAmount": 0
		}, {
			"id": "4a69cbdf-4320-4e1f-852b-0edc2a55f13a",
			"discountedAmount": 0
		}],
		"lineItems": {
			"physicalItems": [{
				"id": "7349b13a-1453-4050-a769-1a6ad1823369",
				"parentId": null,
				"variantId": 362,
				"productId": 191,
				"sku": "",
				"name": "Openhouse No. 3",
				"url": "https://{store_url)/all/openhouse-no-3/",
				"quantity": 1,
				"brand": "Openhouse Magazine",
				"isTaxable": true,
				"imageUrl": "https://cdn11.bigcommerce.com/s-{store_url)/products/191/images/475/openhousevol3_1024x1024__59692__16355.1534344544.330.500.jpg?c=2",
				"discounts": [],
				"discountAmount": 0,
				"couponAmount": 0,
				"listPrice": 27.95,
				"salePrice": 27.95,
				"extendedListPrice": 27.95,
				"extendedSalePrice": 27.95,
				"isShippingRequired": true,
				"giftWrapping": null,
				"addedByPromotion": false
			}, {
				"id": "4a69cbdf-4320-4e1f-852b-0edc2a55f13a",
				"parentId": null,
				"variantId": 356,
				"productId": 185,
				"sku": "",
				"name": "Utility Caddy",
				"url": "https://{store_url)/all/utility-caddy/",
				"quantity": 1,
				"brand": "OFS",
				"isTaxable": true,
				"imageUrl": "https://cdn11.bigcommerce.com/s-{store_url)/products/185/images/449/utilitybucket1_1024x1024__78563__75042.1534344535.330.500.jpg?c=2",
				"discounts": [],
				"discountAmount": 0,
				"couponAmount": 0,
				"listPrice": 46,
				"salePrice": 46,
				"extendedListPrice": 46,
				"extendedSalePrice": 46,
				"isShippingRequired": true,
				"giftWrapping": null,
				"addedByPromotion": false
			}],
			"digitalItems": [],
			"giftCertificates": [],
			"customItems": []
		},
		"createdTime": "2018-11-06T19:22:51+00:00",
		"updatedTime": "2018-11-06T19:53:35+00:00"
	},
	"billingAddress": {
		"id": "5be1eaa653e37",
		"firstName": "Jane",
		"lastName": "Doe",
		"email": "janedoe@email.com",
		"company": "BigCommerce",
		"address1": "123 Main Street",
		"address2": "Apt 1",
		"city": "Austin",
		"stateOrProvince": "",
		"stateOrProvinceCode": "",
		"country": "",
		"countryCode": "",
		"postalCode": "78751",
		"phone": "",
		"customFields": []
	},
	"consignments": [{
		"id": "5be1f13f00e2c",
		"shippingCost": 0,
		"handlingCost": 0,
		"couponDiscounts": [],
		"discounts": [],
		"lineItemIds": ["7349b13a-1453-4050-a769-1a6ad1823369"],
		"shippingAddress": {
			"firstName": "Jane",
			"lastName": "Doe",
			"email": "janedoe@email.com",
			"company": "BigCommerce",
			"address1": "123 Main Street",
			"address2": "Apt 1",
			"city": "Austin",
			"stateOrProvince": "Texas",
			"stateOrProvinceCode": "TX",
			"country": "United States",
			"countryCode": "US",
			"postalCode": "78751",
			"phone": "",
			"customFields": []
		},
		"availableShippingOptions": [{
			"id": "9363fd74-8508-4f8b-beb2-77ede2beaa1c",
			"type": "shipping_byweight",
			"description": "Ship by Weight",
			"imageUrl": "",
			"cost": 8,
			"transitTime": "",
			"isRecommended": false
		}, {
			"id": "20ae4fdf-747f-4ec5-86da-11ecd70ae03e",
			"type": "shipping_flatrate",
			"description": "Flat Rate",
			"imageUrl": "",
			"cost": 12,
			"transitTime": "",
			"isRecommended": false
		}, {
			"id": "b7783bb7-7695-467f-afd0-bf1c84fffdd2",
			"type": "shipping_upsready",
			"description": "UPS® (UPS Next Day Air®)",
			"imageUrl": "/img/shipping-providers/upsready_70x70.png",
			"cost": 44.41,
			"transitTime": "1 business day",
			"isRecommended": false
		}]
	}, {
		"id": "5be1f13f07bae",
		"shippingCost": 0,
		"handlingCost": 0,
		"couponDiscounts": [],
		"discounts": [],
		"lineItemIds": ["4a69cbdf-4320-4e1f-852b-0edc2a55f13a"],
		"shippingAddress": {
			"firstName": "John",
			"lastName": "Doe",
			"email": "johnedoe@email.com",
			"company": "BigCommerce",
			"address1": "123 South Street",
			"address2": "Apt 5",
			"city": "Austin",
			"stateOrProvince": "Texas",
			"stateOrProvinceCode": "TX",
			"country": "United States",
			"countryCode": "US",
			"postalCode": "78726",
			"phone": "",
			"customFields": []
		},
		"availableShippingOptions": [{
			"id": "620a7267-8e0d-4868-bf24-2b3983ccc746",
			"type": "shipping_byweight",
			"description": "Ship by Weight",
			"imageUrl": "",
			"cost": 8,
			"transitTime": "",
			"isRecommended": false
		}, {
			"id": "834a4114-df5e-453d-a476-8de2287d1dfa",
			"type": "shipping_flatrate",
			"description": "Flat Rate",
			"imageUrl": "",
			"cost": 12,
			"transitTime": "",
			"isRecommended": false
		}, {
			"id": "9f40c667-0ab5-46d4-b436-c678517c5415",
			"type": "shipping_upsready",
			"description": "UPS® (UPS Next Day Air®)",
			"imageUrl": "/img/shipping-providers/upsready_70x70.png",
			"cost": 44.41,
			"transitTime": "1 business day",
			"isRecommended": false
		}]
	}],
	"orderId": null,
	"shippingCostTotal": 0,
	"shippingCostBeforeDiscount": 0,
	"handlingCostTotal": 0,
	"taxTotal": 5.92,
	"coupons": [],
	"taxes": [{
		"name": "This is tax",
		"amount": 5.92
	}],
	"subtotal": 73.95,
	"grandTotal": 79.87,
	"giftCertificates": [],
	"createdTime": "2018-11-06T19:22:51+00:00",
	"updatedTime": "2018-11-06T19:53:35+00:00",
	"customerMessage": ""
}
```

### Update Consignment to Add a Shipping Option

So far we have created a cart, added a billing address and shipping address, and assigned the lineItems to the address they should be shipped. Now we are going to make two PUT requests to assign a shipping option for each address. Only one consignment can be updated at a time. If you sent in the `?include=consignments.availableShippingOptions` with the previous request, then pick the appropriate `shippingOptionId` for each consignment. 

<!--
title: "Example Consignment with Available Shipping Options"
subtitle: ""
lineNumbers: true
-->

**Example Consignment with Available Shipping Options**

```json
	"consignments": [{
		"id": "5be1f13f00e2c",
		"shippingCost": 0,
		"handlingCost": 0,
		"couponDiscounts": [],
		"discounts": [],
		"lineItemIds": ["7349b13a-1453-4050-a769-1a6ad1823369"],
		"shippingAddress": {
			"firstName": "Jane",
			"lastName": "Doe",
			"email": "janedoe@email.com",
			"company": "BigCommerce",
			"address1": "123 Main Street",
			"address2": "Apt 1",
			"city": "Austin",
			"stateOrProvince": "Texas",
			"stateOrProvinceCode": "TX",
			"country": "United States",
			"countryCode": "US",
			"postalCode": "78751",
			"phone": "",
			"customFields": []
		},
		"availableShippingOptions": [{
			"id": "9363fd74-8508-4f8b-beb2-77ede2beaa1c",
			"type": "shipping_byweight",
			"description": "Ship by Weight",
			"imageUrl": "",
			"cost": 8,
			"transitTime": "",
			"isRecommended": false
		}, {
			"id": "20ae4fdf-747f-4ec5-86da-11ecd70ae03e",
			"type": "shipping_flatrate",
			"description": "Flat Rate",
			"imageUrl": "",
			"cost": 12,
			"transitTime": "",
			"isRecommended": false
		}, {
			"id": "b7783bb7-7695-467f-afd0-bf1c84fffdd2",
			"type": "shipping_upsready",
			"description": "UPS® (UPS Next Day Air®)",
			"imageUrl": "/img/shipping-providers/upsready_70x70.png",
			"cost": 44.41,
			"transitTime": "1 business day",
			"isRecommended": false
		}]
```

<!--
title: "Update Consignment with Available Shipping Options"
subtitle: ""
lineNumbers: true
-->

**Example Update Consignment with Available Shipping Options**  
`/PUT https://<store_url>/api/storefront/checkouts/{checkoutId}/billing-address/{addressId}`

```js
postData(`/api/storefront/checkouts/`, `1650fb51-172b-4cde-a220-90c6a8ef9293`, `5be1f13f07bae`,{"shippingOptionId": "9f40c667-0ab5-46d4-b436-c678517c5415"})
  .then(data => console.log(JSON.stringify(data))) 
  .catch(error => console.error(error));

function postData(url = ``, checkoutId = ``, consignmentId = ``, data = {},) {
    return fetch(url + checkoutId + `/consignments/` + consignmentId,   {
        method: "PUT", 
        credentials: "same-origin",
        headers: {
            "Content-Type": "application/json;",
        },
        body: JSON.stringify(data), 
    })
    .then(response => response.json()); 
}
```

## Troubleshooting

**Did you get a CORs error response?**  
Check to make sure you have the right credentials set up. Most requests will use same-origin or include. 

**Did you get a 404?**  
Make sure you have at least one item in your cart. Deleting all items removes the cart and returns a 404 in the browser console.

## Resources

### Related Endpoints
- [Storefront Cart](/api-reference/cart-checkout/storefront-cart-api)
- [Storefront Checkout](/api-reference/cart-checkout/storefront-checkout-api)

### Related Articles
- [How To Embed a Shipping Location Map on the BigCommerce Order Confirmation Page](https://medium.com/bigcommerce-developer-blog/how-to-embed-a-google-map-on-the-bigcommerce-order-confirmation-page-8264747e654d) (Developer Blog)
- [Let’s Talk About CORS](https://medium.com/bigcommerce-developer-blog/lets-talk-about-cors-84800c726919) (Developer Blog)

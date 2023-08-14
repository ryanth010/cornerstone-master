# cornerstone-master
 # Notes on Test Project

Set up store, installed Stencil, downloaded repo from gitHub

Push Special Item product using documtation here: https://bigcommerce.stoplight.io/docs/docs/ZG9jOjIyMDU5Ng-catalog-overview 

{
  "name": "Special Item",
  "price": "10.00",
  "categories": [
    24
  ],
  "weight": 4,
  "type": "physical"
}


## Image Data:
{
  "data": [
    {
      "id": 378,
      "product_id": 114,
      "is_thumbnail": false,
      "sort_order": 1,
      "description": "",
      "image_file": "r/057/logi-tball-hand__62714.jpg",
      "url_zoom": "https://cdn11.bigcommerce.com/s-2aeq1qoxn0/products/114/images/378/logi-tball-hand__62714.1691875252.1280.1280.jpg?c=1",
      "url_standard": "https://cdn11.bigcommerce.com/s-2aeq1qoxn0/products/114/images/378/logi-tball-hand__62714.1691875252.386.513.jpg?c=1",
      "url_thumbnail": "https://cdn11.bigcommerce.com/s-2aeq1qoxn0/products/114/images/378/logi-tball-hand__62714.1691875252.220.290.jpg?c=1",
      "url_tiny": "https://cdn11.bigcommerce.com/s-2aeq1qoxn0/products/114/images/378/logi-tball-hand__62714.1691875252.44.58.jpg?c=1",
      "date_modified": "2023-08-12T21:20:52+00:00"
    },
    {
      "id": 379,
      "product_id": 114,
      "is_thumbnail": true,
      "sort_order": 0,
      "description": "",
      "image_file": "k/689/logi_trackball__33040.jpg",
      "url_zoom": "https://cdn11.bigcommerce.com/s-2aeq1qoxn0/products/114/images/379/logi_trackball__33040.1691875252.1280.1280.jpg?c=1",
      "url_standard": "https://cdn11.bigcommerce.com/s-2aeq1qoxn0/products/114/images/379/logi_trackball__33040.1691875252.386.513.jpg?c=1",
      "url_thumbnail": "https://cdn11.bigcommerce.com/s-2aeq1qoxn0/products/114/images/379/logi_trackball__33040.1691875252.220.290.jpg?c=1",
      "url_tiny": "https://cdn11.bigcommerce.com/s-2aeq1qoxn0/products/114/images/379/logi_trackball__33040.1691875252.44.58.jpg?c=1",
      "date_modified": "2023-08-12T21:20:52+00:00"
    }
  ],
  "meta": {
    "pagination": {
      "total": 2,
      "count": 2,
      "per_page": 50,
      "current_page": 1,
      "total_pages": 1,
      "links": {
        "current": "?page=1&limit=50"
      }
    }
  }
}

## Getting Second image to appear on mouse hover
Initially tried to use JS to dynamically change the class on hover, based on a BigCommerce forum post: https://support.bigcommerce.com/s/question/0D51B00003n71ViSAI/cornerstone-theme-show-the-2nd-product-image-on-hover?language=en_US

This was throwing an error since "mouseenter" is deprecated.

Settled on a solution through scss based on this idea: https://bigcommercesolution.wordpress.com/2018/07/26/bigcommerce-product-image-swap-on-hover-using-css-stencil-version/

It hides the main image and shows the second image, which has been given the class of "product-additional". The css provided was overdone (and likely still is) so I cleaned it up and tested. It's functional.

## Cart Tasks via Storefront API

My solution for this involved creating two global functions "createCart" and "deleteCart" which were added to the app.js file. They are modified versions of the tutorial functions found in the developer docs.

In brief, they: 

- make a request to the API
- return the completed action alongside data as a JSON string
- trigger an alert if the action is successful
- reload the page so that the UI is current (shows cart notification dot or renders the Remove All Items button)

To trigger these functions and pass the appropriate parameters in, I added scripts to the bottom of the "Category" HTML template. These scripts run when the corresponding button is clicked by the user. A better method would be to create the functions and corresponding scripts as individual components that aren't rendered on every category page on the site... but given the limited scope of the test, this is a hacky way to fulfill your requests. 

If you don't want to go digging, here is the code that I added: 


### functions on app.js
// Define the createCart function 
window.createCart = function(route, cartItems) {
    return fetch(route, {
        method: "POST",
        credentials: "same-origin",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify(cartItems),
    })
    .then(response => response.json())
    .then(result => {
        const physicalItems = result.lineItems.physicalItems;

        if (physicalItems.length > 0) {
            alert("Item added to the cart!");
            location.reload();
        } else {
            console.log("No physical items added to the cart.");
        }
    })
    .catch(error => console.error(error));
};


// Define the deleteCart funtion 
window.deleteCart = function(routeStart, cartId) {
    var route = routeStart + cartId;
    return fetch(route, {
        method: "DELETE",
        credentials: "same-origin",
        headers: {
            "Content-Type": "application/json",
        }
    })
    .then(response => response.json())
    .then(result => {
        if (result.success) {
            alert("Cart deleted!");
            location.reload();
        } else {
            console.log("Cart deletion failed.");
        }
        console.log(result);
    })
    .catch(error => console.error(error));
};


### scripts on category.html: 

<!-- <button id="addToCartButton" class="button button--small figcaption-button">Add All to Cart</button>
    <button id="deleteCartButton" class="button button--small figcaption-button" style="display: none;">Remove All Items</button>

  <script>
        // Define the createCart function
        function createCart(route, cartItems) {
        }

        // Reference to the "Add to Cart" button
        const addToCartButton = document.getElementById("addToCartButton");

        // Attach a click event listener to the button
        addToCartButton.addEventListener("click", function() {
        // Call the createCart function with the correct route and cartItems
        const route = '/api/storefront/carts'; 
        const cartItems = {
            "lineItems": [
                {
                "quantity": 1,
                "productId": 114
                }
            ]
        };
        
        // Call the createCart function
        createCart(route, cartItems);
        deleteCartButton.style.display = "inline-block";
        });
  </script>
   
   
   <script>
    // Get a reference to the "Delete Cart" button
    const deleteCartButton = document.getElementById("deleteCartButton");
    
    // Attach a click event listener to the button
    deleteCartButton.addEventListener("click", function() {
      // Fetch cart information
      const options = { method: 'GET', headers: { 'Content-Type': 'application/json' } };
    
      fetch('/api/storefront/carts', options)
        .then(response => {
          if (!response.ok) {
            throw new Error('Request failed');
          }
          return response.json();
        })
        .then(responseData => {
          console.log("Response data:", responseData);
    
          if (Array.isArray(responseData) && responseData.length > 0) {
            
            const cartId = responseData[0].id;
            const routeStart = '/api/storefront/carts/';
    
            // Call the deleteCart function with the retrieved cart ID
            deleteCart(routeStart, cartId);
          } else {
            console.log("No cart data available.");
          }
        })
        .catch(err => {
          console.error("Error fetching cart information:", err);
          // Handle the error scenario, such as displaying an error message to the user
        })
        .finally(() => {
        alert("All items removed from the cart");
        location.reload(); // Refresh the page
        });
    });
    </script>

<script>
    
    // Fetch cart information
    const options = { method: 'GET', headers: { 'Content-Type': 'application/json' } };

    fetch('/api/storefront/carts', options)
        .then(response => response.json())
        .then(responseData => {
            console.log("Response data:", responseData);

            // Check if there are items in the cart
            if (Array.isArray(responseData) && responseData.length > 0) {
                // Show the "Delete Cart" button
                deleteCartButton.style.display = "inline-block";
            } else {
                console.log("No cart data available.");
            }
        })
        .catch(err => {
            console.error("Error fetching cart information:", err);
            // Handle the error scenario, such as displaying an error message to the user
        });

    // Attach a click event listener to the "Delete Cart" button
    deleteCartButton.addEventListener("click", function() {
        // ... Your deleteCartButton event handler code ...
    });
</script>

-->

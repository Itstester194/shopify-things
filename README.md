<div id="ajax-cart-drawer" class="ajax-cart-drawer">
    <div class="ajax-cart-header">
        <h2>Your Cart</h2>
        <button id="close-cart-btn" class="close-cart-btn">×</button>
    </div>
    <div class="ajax-cart-items">
        <!-- Cart items will be dynamically inserted here -->
    </div>
    <div class="ajax-cart-footer">
        <p>Total: <span id="ajax-cart-total"></span></p>
        <a href="/cart" class="btn">View Cart</a>
    </div>
</div>
<button id="open-cart-btn">Open Cart</button>
<style>
  .ajax-cart-drawer {
    position: fixed;
    top: 0;
    right: 0;
    width: 300px;
    height: 100%;
    background-color: #fff;
    box-shadow: -2px 0 5px rgba(0, 0, 0, 0.2);
    z-index: 1000;
    overflow-y: auto;
    padding: 20px;
    transform: translateX(100%);
    transition: transform 0.3s ease-in-out;
}

.ajax-cart-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.ajax-cart-items {
    margin-bottom: 20px;
}

.ajax-cart-footer {
    border-top: 1px solid #ccc;
    padding-top: 20px;
    text-align: right;
}

.close-cart-btn {
    background: none;
    border: none;
    cursor: pointer;
    font-size: 20px;
    color: #333;
}

</style>
<script>
  document.addEventListener("DOMContentLoaded", function() {
    // Function to open the cart drawer
    function openCartDrawer() {
        document.getElementById('ajax-cart-drawer').style.transform = 'translateX(0)';
    }

    // Function to close the cart drawer
    function closeCartDrawer() {
        document.getElementById('ajax-cart-drawer').style.transform = 'translateX(100%)';
    }

    // AJAX request to fetch cart data
    function fetchCartData() {
        fetch('/cart.js')
            .then(response => response.json())
            .then(data => {
                // Update cart items
                // You need to modify this part based on your cart structure
                const cartItemsHtml = data.items.map(item => `
                    <div class="ajax-cart-item">
                        <p>${item.title} x ${item.quantity}</p>
                        <button class="remove-from-cart-btn" data-line-id="${item.id}">Remove</button>
                    </div>
                `).join('');
                document.querySelector('.ajax-cart-items').innerHTML = cartItemsHtml;

                // Update total price
                document.getElementById('ajax-cart-total').textContent = Shopify.formatMoney(data.total_price);
            })
            .catch(error => console.error('Error fetching cart data:', error));
    }

    // Add event listeners
    document.getElementById('open-cart-btn').addEventListener('click', openCartDrawer);
    document.getElementById('close-cart-btn').addEventListener('click', closeCartDrawer);

    // Initial cart data fetch
    fetchCartData();

    // Event delegation for dynamically added remove buttons
    document.addEventListener('click', function(event) {
        if (event.target.classList.contains('remove-from-cart-btn')) {
            const lineItemId = event.target.dataset.lineId;
            removeFromCart(lineItemId);
        }
    });

    // AJAX request to remove item from cart
 function removeFromCart(lineItemId) {
    fetch('/cart/change.js', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            line: lineItemId,
            quantity: 0 // Setting quantity to 0 removes the item
        })
    })
    .then(response => {
        if (response.ok) {
            fetchCartData(); // Refresh cart data after successful removal
        } else {
            throw new Error('Error removing item from cart: ' + response.statusText);
        }
    })
    .catch(error => console.error(error)); // Log detailed error message
}

});

</script>
{% schema %}
  {
    "name": "ajax-cart-drawer",
    "settings": [],
    "presets": [
      {
        "name":"ajax-cart-drawer"
      }
    ]
  }
{% endschema %}

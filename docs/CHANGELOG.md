## Lab 1 – Part B: Product search by name and description (2026-09-24)
**Instruction:** Product search that checks both product names and descriptions while preserving the category filter.

| File | What changed | Why | How it improves the app |
|------|--------------|-----|-------------------------|
| [frontend/src/pages/ProductsPage.jsx](frontend/src/pages/ProductsPage.jsx) | Search now checks both the product name and description while keeping the selected category filter active. | Users often search by descriptive terms, not just the exact product title. | Product discovery is more accurate and consistent with real-world search behavior. |

### Before / After
Original:
```js
const matchesSearch = product.name.toLowerCase().includes(search.trim().toLowerCase());
```

New:
```js
const matchesSearch =
  product.name.toLowerCase().includes(query) ||
  product.description.toLowerCase().includes(query);
```

### How to test
1. Search "privacy cover" with All categories → 1080p Webcam appears
2. Search "keyboard" → Compact Mechanical Keyboard appears
3. Search "privacy cover" and select Study → "No products match your filters"
4. Clear the search while Study is selected → Study products appear

## Lab 1 – Part C: Low-stock labels (2026-09-24)
**Instruction:** A product card that shows "Only N left" for stock levels from 1 to 8.

| File | What changed | Why | How it improves the app |
|------|--------------|-----|-------------------------|
| [frontend/src/components/ProductCard.jsx](frontend/src/components/ProductCard.jsx) | Added stock-label logic that shows "Only N left" for 1–8 units, "N in stock" for higher counts, and "Out of stock" for zero. | Low stock is easy to miss when product cards only show a flat stock number. | Shoppers can quickly understand urgency and availability without reading the full product detail. |

### Before / After
Original:
```jsx
<p className={product.stock > 0 ? 'stock' : 'stock unavailable'}>
  {product.stock > 0 ? `${product.stock} in stock` : 'Out of stock'}
</p>
```

New:
```jsx
let stockLabel;
if (product.stock < 1) {
  stockLabel = 'Out of stock';
} else if (product.stock <= 8) {
  stockLabel = `Only ${product.stock} left`;
} else {
  stockLabel = `${product.stock} in stock`;
}

<p className={product.stock > 0 ? 'stock' : 'stock unavailable'}>
  {stockLabel}
</p>
```

### How to test
1. Products with stock 6, 7, 8 show "Only N left"
2. Products with stock 9, 10, 12 show "N in stock"
3. Stock 0 is handled by the first if branch; the Add to cart button stays disabled by the unchanged disabled={product.stock < 1} condition

## Lab 1 – Part D: Reusable cart item count (2026-09-24)
**Instruction:** One reusable cart item count function used by the navigation and order summary.

| File | What changed | Why | How it improves the app |
|------|--------------|-----|-------------------------|
| [frontend/src/utils/cart.js](frontend/src/utils/cart.js) | Added a reusable `calculateCartItemCount` helper. | Counting totals in multiple places should use the same logic to avoid drift. | The cart badge and summary stay consistent. |
| [frontend/src/context/CartContext.jsx](frontend/src/context/CartContext.jsx) | Replaced the inline item count calculation with the shared helper. | Centralized logic makes the cart state easier to maintain. | UI values update consistently across the app. |
| [frontend/src/pages/CartPage.jsx](frontend/src/pages/CartPage.jsx) | Switched the summary to use `itemCount` from context. | The order summary now reflects the same source of truth as the nav. | Users see the same item count in all cart-related views. |
| [frontend/src/utils/cart.test.js](frontend/src/utils/cart.test.js) | Added a test covering multiple item quantities and an empty cart. | Regression protection for shared quantity logic. | The cart count remains reliable as features evolve. |

### Before / After
Original in [frontend/src/utils/cart.js](frontend/src/utils/cart.js):
```js
export function calculateCartTotal(cart) {
  return cart.reduce((total, item) => total + calculateItemSubtotal(item), 0);
}
```

New in [frontend/src/utils/cart.js](frontend/src/utils/cart.js):
```js
export function calculateCartItemCount(cart) {
  return cart.reduce((count, item) => count + item.quantity, 0);
}
```

Original in [frontend/src/context/CartContext.jsx](frontend/src/context/CartContext.jsx):
```js
itemCount: cart.reduce((count, item) => count + item.quantity, 0),
```

New in [frontend/src/context/CartContext.jsx](frontend/src/context/CartContext.jsx):
```js
itemCount: calculateCartItemCount(cart),
```

Original in [frontend/src/pages/CartPage.jsx](frontend/src/pages/CartPage.jsx):
```jsx
<p><span>Items</span><strong>{cart.reduce((sum, item) => sum + item.quantity, 0)}</strong></p>
```

New in [frontend/src/pages/CartPage.jsx](frontend/src/pages/CartPage.jsx):
```jsx
<p><span>Items</span><strong>{itemCount}</strong></p>
```

Original in [frontend/src/utils/cart.test.js](frontend/src/utils/cart.test.js):
```js
import { addProductToCart, calculateCartTotal, updateCartQuantity } from './cart.js';
```

New in [frontend/src/utils/cart.test.js](frontend/src/utils/cart.test.js):
```js
import {
  addProductToCart, calculateCartItemCount,
  calculateCartTotal, updateCartQuantity
} from './cart.js';
```

### How to test
1. Add one product twice and another once → nav shows Cart (3) and Items shows 3
2. Change a quantity to 3 → both show 4
3. Remove the other product → both show 3
4. Total still equals price × quantity
5. npm run test --workspace frontend passes, including the new test

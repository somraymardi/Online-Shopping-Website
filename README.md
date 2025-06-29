# Online-Shopping-Website
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>SRM Shopping Website</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body { font-family: sans-serif; margin: 0; background: #f4f4f4; }
    header { background: #0d6efd; color: white; padding: 15px; font-size: 24px; text-align: center; }
    nav { background: #222; display: flex; justify-content: center; flex-wrap: wrap; padding: 10px; }
    nav button { background: #444; color: white; padding: 10px 20px; margin: 5px; border: none; cursor: pointer; }
    nav button:hover { background: #666; }
    .page { display: none; padding: 20px; }
    .active { display: block; }
    .products, .cart-list {
      display: flex; flex-wrap: wrap; justify-content: center; gap: 20px;
    }
    .product, .cart-item {
      background: white; padding: 15px; border-radius: 8px; width: 240px;
      box-shadow: 0 0 8px rgba(0,0,0,0.1); text-align: center;
    }
    .product img, .cart-item img {
      width: 100%; height: 150px; object-fit: cover; border-radius: 6px;
    }
    .btn {
      margin-top: 10px; padding: 10px; width: 100%; border: none;
      cursor: pointer; border-radius: 5px; color: white;
    }
    .add-to-cart { background: green; }
    .remove-from-cart { background: red; }
    .checkout { background: orange; max-width: 300px; }
    form {
      background: white; padding: 20px; margin: 0 auto; max-width: 400px;
      border-radius: 8px;
    }
    form input, form textarea {
      padding: 10px; margin: 10px 0; width: 100%; box-sizing: border-box;
    }
    form button {
      background: #0d6efd; color: white; padding: 12px; width: 100%;
      border: none; cursor: pointer; border-radius: 5px;
    }
    #logoutBtn { display: none; margin-left: auto; }
    footer {
      text-align: center; padding: 15px; background: #222; color: white; margin-top: 30px;
    }
    #previewImg {
      width: 100%; height: 150px; object-fit: cover; margin-top: 10px;
      border-radius: 6px; display: none;
    }
  </style>
</head>
<body>

<header>🛒 SRM Shopping</header>

<nav>
  <button onclick="showPage('home')">Home</button>
  <button onclick="showPage('cart')">Cart (<span id="cart-count">0</span>)</button>
  <button onclick="showPage('userLogin')">User Login</button>
  <button onclick="showPage('userRegister')">User Register</button>
  <button onclick="showPage('adminLogin')">Admin Login</button>
  <button id="logoutBtn" onclick="logout()">Logout</button>
</nav>

<!-- Pages -->
<div id="home" class="page active">
  <h2>Products</h2>
  <div class="products" id="product-list"></div>
</div>

<div id="cart" class="page">
  <h2>Your Cart</h2>
  <div class="cart-list" id="cart-list"></div>
  <h3>Total: ₹<span id="cart-total">0</span></h3>
  <button class="btn checkout" onclick="showPage('checkout')">Proceed to Checkout</button>
</div>

<div id="checkout" class="page">
  <h2>Checkout</h2>
  <form onsubmit="handleCheckout(event)">
    <input type="text" id="cust-name" placeholder="Your Name" required />
    <input type="tel" id="cust-mobile" placeholder="Mobile Number" pattern="[0-9]{10}" required />
    <textarea id="cust-address" placeholder="Address" rows="3" required></textarea>
    <button type="submit">Pay via UPI</button>
  </form>
</div>

<div id="userLogin" class="page">
  <h2>User Login</h2>
  <form onsubmit="userLogin(event)">
    <input type="text" id="user-username" placeholder="Username" required />
    <input type="password" id="user-password" placeholder="Password" required />
    <button type="submit">Login</button>
  </form>
</div>

<div id="userRegister" class="page">
  <h2>User Register</h2>
  <form onsubmit="userRegister(event)">
    <input type="text" id="reg-username" placeholder="Username" required />
    <input type="password" id="reg-password" placeholder="Password" required />
    <button type="submit">Register</button>
  </form>
</div>

<div id="adminLogin" class="page">
  <h2>Admin Login</h2>
  <form onsubmit="adminLogin(event)">
    <input type="text" id="admin-username" placeholder="Admin ID" required />
    <input type="password" id="admin-password" placeholder="Password" required />
    <button type="submit">Login</button>
  </form>
</div>

<div id="admin" class="page">
  <h2>Admin Panel</h2>
  
  <h3>Add Product</h3>
  <form onsubmit="addProduct(event)">
    <input type="text" id="pname" placeholder="Product Name" required />
    <input type="number" id="pprice" placeholder="Price ₹" required />
    <textarea id="pdesc" placeholder="Description" required></textarea>
    <input type="file" accept="image/*" id="pimg" onchange="previewImage(event)" required />
    <img id="previewImg" />
    <button type="submit">Add Product</button>
  </form>
  
  <h3>Orders</h3>
  <div id="orders-list" style="background:#fff; padding:10px; max-height:300px; overflow:auto; border-radius:8px;"></div>
</div>

<footer>© 2025 SRM Shopping</footer>

<script>
let session = '';

function showPage(id) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
}

function logout() {
  session = '';
  document.getElementById('logoutBtn').style.display = "none";
  showPage('home');
  alert("Logged out successfully!");
}

const defaultProducts = [
  { name: "Shirt", price: 399, desc: "Cotton shirt", img: "https://via.placeholder.com/240x150" },
  { name: "Shoes", price: 999, desc: "Running shoes", img: "https://via.placeholder.com/240x150" }
];

let products = JSON.parse(localStorage.getItem('products')) || defaultProducts;
localStorage.setItem('products', JSON.stringify(products));

function updateCartCount() {
  const cart = JSON.parse(localStorage.getItem('cart')) || [];
  document.getElementById('cart-count').textContent = cart.length;
}

function displayProducts() {
  const list = document.getElementById('product-list');
  list.innerHTML = '';
  products.forEach((p, i) => {
    const div = document.createElement('div');
    div.className = 'product';
    div.innerHTML = `
      <img src="${p.img}" />
      <h3>${p.name}</h3>
      <p>₹${p.price}</p>
      <small>${p.desc}</small>
      <button class="btn add-to-cart" onclick="addToCart(${i})">Add to Cart</button>
    `;
    list.appendChild(div);
  });
}

function addToCart(index) {
  let cart = JSON.parse(localStorage.getItem('cart')) || [];
  cart.push(products[index]);
  localStorage.setItem('cart', JSON.stringify(cart));
  alert("Added to cart!");
  updateCartCount();
  displayCart();
  showPage('cart');
}

function displayCart() {
  const cart = JSON.parse(localStorage.getItem('cart')) || [];
  const list = document.getElementById('cart-list');
  let total = 0;
  list.innerHTML = '';
  cart.forEach((item, i) => {
    const div = document.createElement('div');
    div.className = 'cart-item';
    div.innerHTML = `
      <img src="${item.img}" />
      <h4>${item.name}</h4>
      <p>₹${item.price}</p>
      <button class="btn remove-from-cart" onclick="removeFromCart(${i})">Remove</button>
    `;
    list.appendChild(div);
    total += item.price;
  });
  document.getElementById('cart-total').textContent = total;
}

function removeFromCart(index) {
  let cart = JSON.parse(localStorage.getItem('cart')) || [];
  cart.splice(index, 1);
  localStorage.setItem('cart', JSON.stringify(cart));
  displayCart();
  updateCartCount();
}

function handleCheckout(e) {
  e.preventDefault();
  if(session !== 'user') {
    alert("Please login as User to checkout!");
    showPage('userLogin');
    return;
  }
  const name = document.getElementById('cust-name').value.trim();
  const mobile = document.getElementById('cust-mobile').value.trim();
  const address = document.getElementById('cust-address').value.trim();
  if(!name || !mobile || !address) {
    alert("Please fill all details");
    return;
  }
  const cart = JSON.parse(localStorage.getItem('cart')) || [];
  if(cart.length === 0) {
    alert("Cart is empty!");
    return;
  }
  const total = document.getElementById('cart-total').textContent;

  // Save order locally
  let orders = JSON.parse(localStorage.getItem('orders')) || [];
  orders.push({
    id: Date.now(),
    name, mobile, address,
    products: cart,
    total,
    date: new Date().toLocaleString()
  });
  localStorage.setItem('orders', JSON.stringify(orders));
  
  // Clear cart after order
  localStorage.setItem('cart', JSON.stringify([]));
  updateCartCount();
  displayCart();

  alert("Order placed! Redirecting to UPI Payment...");
  
  // Redirect to UPI payment
  const upiLink = `upi://pay?pa=9334734456@ybl&pn=SRM%20PAYMENT&am=${total}`;
  window.location.href = upiLink;

  // Show home page after payment (cannot really confirm payment success here)
  showPage('home');
}

function userRegister(e) {
  e.preventDefault();
  const u = document.getElementById('reg-username').value.trim();
  const p = document.getElementById('reg-password').value.trim();
  if(u === "" || p === "") {
    alert("Enter valid username and password");
    return;
  }
  if(localStorage.getItem('user_' + u)) {
    alert("Username already exists");
    return;
  }
  localStorage.setItem('user_' + u, p);
  alert("Registered successfully! Login now.");
  showPage('userLogin');
}

function userLogin(e) {
  e.preventDefault();
  const u = document.getElementById('user-username').value.trim();
  const p = document.getElementById('user-password').value.trim();
  const saved = localStorage.getItem('user_' + u);
  if (saved === p) {
    session = 'user';
    document.getElementById('logoutBtn').style.display = "inline-block";
    alert("User login successful");
    showPage('home');
  } else {
    alert("Invalid user credentials");
  }
}

function adminLogin(e) {
  e.preventDefault();
  const u = document.getElementById('admin-username').value.trim();
  const p = document.getElementById('admin-password').value.trim();
  if (u === "Admin" && p === "123") {
    session = 'admin';
    document.getElementById('logoutBtn').style.display = "inline-block";
    alert("Admin login successful");
    showPage('admin');
    displayOrders();
  } else {
    alert("Invalid Admin ID or Password");
  }
}

function addProduct(e) {
  e.preventDefault();
  const name = document.getElementById('pname').value.trim();
  const price = parseInt(document.getElementById('pprice').value);
  const desc = document.getElementById('pdesc').value.trim();
  const img = document.getElementById('previewImg').src;
  if(!img) {
    alert("Please upload an image");
    return;
  }
  products.push({ name, price, desc, img });
  localStorage.setItem('products', JSON.stringify(products));
  alert("Product added!");
  displayProducts();
  e.target.reset();
  document.getElementById('previewImg').style.display = 'none';
}

function previewImage(event) {
  const img = document.getElementById('previewImg');
  img.src = URL.createObjectURL(event.target.files[0]);
  img.style.display = 'block';
}

function displayOrders() {
  const ordersDiv = document.getElementById('orders-list');
  let orders = JSON.parse(localStorage.getItem('orders')) || [];
  if(orders.length === 0) {
    ordersDiv.innerHTML = "<p>No orders yet.</p>";
    return;
  }
  ordersDiv.innerHTML = "";
  orders.forEach(order => {
    const orderEl = document.createElement('div');
    orderEl.style.border = "1px solid #ccc";
    orderEl.style.margin = "10px 0";
    orderEl.style.padding = "10px";
    orderEl.style.borderRadius = "6px";
    let productList = "";
    order.products.forEach(p => {
      productList += `<li>${p.name} - ₹${p.price}</li>`;
    });
    orderEl.innerHTML = `
      <strong>Order ID:</strong> ${order.id}<br/>
      <strong>Date:</strong> ${order.date}<br/>
      <strong>Name:</strong> ${order.name}<br/>
      <strong>Mobile:</strong> ${order.mobile}<br/>
      <strong>Address:</strong> ${order.address}<br/>
      <strong>Total:</strong> ₹${order.total}<br/>
      <strong>Products:</strong>
      <ul>${productList}</ul>
    `;
    ordersDiv.appendChild(orderEl);
  });
}

window.onload = () => {
  displayProducts();
  displayCart();
  updateCartCount();
  document.getElementById('logoutBtn').style.display = "none";
};
</script>

</body>
</html>

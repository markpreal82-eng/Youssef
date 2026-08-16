<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mobile Store Manager</title>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: Arial, sans-serif;
  background: #f4f7fb;
  color: #172033;
}

header {
  background: linear-gradient(135deg, #111827, #2563eb);
  color: white;
  padding: 25px 18px;
  text-align: center;
  box-shadow: 0 4px 15px rgba(0,0,0,.15);
}

header h1 {
  font-size: 28px;
  margin-bottom: 6px;
}

header p {
  opacity: .85;
}

.container {
  width: 94%;
  max-width: 1100px;
  margin: 25px auto;
}

.dashboard {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  margin-bottom: 25px;
}

.stat {
  background: white;
  padding: 18px;
  border-radius: 15px;
  box-shadow: 0 4px 15px rgba(0,0,0,.07);
}

.stat small {
  color: #6b7280;
}

.stat strong {
  display: block;
  font-size: 24px;
  margin-top: 6px;
  color: #2563eb;
}

.panel {
  background: white;
  padding: 22px;
  border-radius: 18px;
  margin-bottom: 25px;
  box-shadow: 0 5px 20px rgba(0,0,0,.07);
}

.panel h2 {
  margin-bottom: 18px;
}

.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

input, select {
  width: 100%;
  padding: 13px;
  border: 1px solid #d8dee9;
  border-radius: 10px;
  font-size: 15px;
  outline: none;
}

input:focus, select:focus {
  border-color: #2563eb;
}

.full {
  grid-column: 1 / -1;
}

.photo-box {
  border: 2px dashed #cbd5e1;
  padding: 18px;
  text-align: center;
  border-radius: 12px;
  cursor: pointer;
}

.photo-box input {
  display: none;
}

.preview {
  max-width: 150px;
  max-height: 150px;
  margin-top: 12px;
  border-radius: 12px;
  display: none;
  object-fit: cover;
}

button {
  border: none;
  border-radius: 10px;
  padding: 13px 18px;
  cursor: pointer;
  font-weight: bold;
  font-size: 15px;
}

.add-btn {
  width: 100%;
  background: #2563eb;
  color: white;
}

.add-btn:hover {
  background: #1d4ed8;
}

.search {
  margin-bottom: 20px;
}

.products {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 18px;
}

.product {
  background: white;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 5px 18px rgba(0,0,0,.08);
  transition: .2s;
}

.product:hover {
  transform: translateY(-3px);
}

.product img {
  width: 100%;
  height: 190px;
  object-fit: cover;
  background: #eef2f7;
}

.product-info {
  padding: 15px;
}

.product-info h3 {
  margin-bottom: 7px;
}

.category {
  display: inline-block;
  background: #e8f0ff;
  color: #2563eb;
  padding: 5px 9px;
  border-radius: 20px;
  font-size: 12px;
  margin-bottom: 10px;
}

.price {
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 12px;
}

.delete {
  width: 100%;
  background: #fee2e2;
  color: #dc2626;
}

.empty {
  text-align: center;
  padding: 40px;
  color: #6b7280;
}

@media(max-width: 650px) {
  .dashboard {
    grid-template-columns: 1fr;
  }

  .form-grid {
    grid-template-columns: 1fr;
  }

  .full {
    grid-column: auto;
  }

  header h1 {
    font-size: 23px;
  }
}
</style>
</head>

<body>

<header>
  <h1>📱 Mobile Store</h1>
  <p>Professional Product Manager</p>
</header>

<div class="container">

  <div class="dashboard">
    <div class="stat">
      <small>Total Products</small>
      <strong id="totalProducts">0</strong>
    </div>

    <div class="stat">
      <small>Phones</small>
      <strong id="totalPhones">0</strong>
    </div>

    <div class="stat">
      <small>Accessories</small>
      <strong id="totalAccessories">0</strong>
    </div>
  </div>

  <div class="panel">
    <h2>➕ Add Product</h2>

    <div class="form-grid">

      <input id="name"
             type="text"
             placeholder="Product name">

      <input id="price"
             type="number"
             min="0"
             step="0.01"
             placeholder="Price (MAD)">

      <select id="category">
        <option value="Phone">📱 Phone</option>
        <option value="Accessory">🎧 Accessory</option>
      </select>

      <label class="photo-box">
        📷 Choose Product Photo
        <input id="photo"
               type="file"
               accept="image/*">
        <img id="preview" class="preview">
      </label>

      <button class="add-btn full" onclick="addProduct()">
        ➕ Add Product
      </button>

    </div>
  </div>

  <div class="panel">

    <h2>🔎 My Products</h2>

    <input
      id="search"
      class="search"
      type="search"
      placeholder="Search product..."
      oninput="displayProducts()">

    <div id="products" class="products"></div>

  </div>

</div>

<script>

let products = JSON.parse(localStorage.getItem("mobileProducts")) || [];

let selectedPhoto = "";

const photoInput = document.getElementById("photo");
const preview = document.getElementById("preview");

photoInput.addEventListener("change", function() {

  const file = this.files[0];

  if (!file) {
    selectedPhoto = "";
    preview.style.display = "none";
    return;
  }

  // Prevent extremely large images
  if (file.size > 5 * 1024 * 1024) {
    alert("Photo is too large. Please choose a photo under 5MB.");
    this.value = "";
    return;
  }

  const reader = new FileReader();

  reader.onload = function(e) {
    selectedPhoto = e.target.result;
    preview.src = selectedPhoto;
    preview.style.display = "block";
  };

  reader.readAsDataURL(file);
});


function addProduct() {

  const name = document.getElementById("name").value.trim();
  const priceValue = document.getElementById("price").value;
  const category = document.getElementById("category").value;

  const price = Number(priceValue);

  if (!name) {
    alert("Please enter the product name.");
    return;
  }

  if (!priceValue || !Number.isFinite(price) || price < 0) {
    alert("Please enter a valid price.");
    return;
  }

  const product = {
    id: Date.now(),
    name: name,
    price: price,
    category: category,
    photo: selectedPhoto || createPlaceholder(category)
  };

  products.unshift(product);

  saveProducts();

  document.getElementById("name").value = "";
  document.getElementById("price").value = "";
  document.getElementById("category").value = "Phone";

  photoInput.value = "";
  selectedPhoto = "";

  preview.src = "";
  preview.style.display = "none";

  displayProducts();

}


function deleteProduct(id) {

  const product = products.find(p => p.id === id);

  if (!product) return;

  const confirmed = confirm(
    `Delete "${product.name}"?`
  );

  if (!confirmed) return;

  products = products.filter(p => p.id !== id);

  saveProducts();

  displayProducts();
}


function saveProducts() {

  try {

    localStorage.setItem(
      "mobileProducts",
      JSON.stringify(products)
    );

  } catch (error) {

    alert(
      "Storage is full. Try deleting some products or using smaller photos."
    );

  }

}


function displayProducts() {

  const container = document.getElementById("products");

  const search =
    document.getElementById("search")
    .value
    .toLowerCase()
    .trim();

  const filtered = products.filter(product =>
    product.name.toLowerCase().includes(search) ||
    product.category.toLowerCase().includes(search)
  );

  container.innerHTML = "";

  if (filtered.length === 0) {

    container.innerHTML = `
      <div class="empty">
        <h3>📦 No products found</h3>
        <p>Add your first phone or accessory above.</p>
      </div>
    `;

    updateStats();
    return;
  }

  filtered.forEach(product => {

    const card = document.createElement("div");
    card.className = "product";

    const img = document.createElement("img");
    img.src = product.photo;
    img.alt = product.name;

    const info = document.createElement("div");
    info.className = "product-info";

    const title = document.createElement("h3");
    title.textContent = product.name;

    const category = document.createElement("span");
    category.className = "category";
    category.textContent =
      product.category === "Phone"
      ? "📱 Phone"
      : "🎧 Accessory";

    const price = document.createElement("div");
    price.className = "price";
    price.textContent =
      Number(product.price).toLocaleString("en-US") + " MAD";

    const deleteBtn = document.createElement("button");
    deleteBtn.className = "delete";
    deleteBtn.textContent = "🗑️ Delete";

    deleteBtn.onclick = () =>
      deleteProduct(product.id);

    info.appendChild(title);
    info.appendChild(category);
    info.appendChild(price);
    info.appendChild(deleteBtn);

    card.appendChild(img);
    card.appendChild(info);

    container.appendChild(card);

  });

  updateStats();
}


function updateStats() {

  document.getElementById("totalProducts").textContent =
    products.length;

  document.getElementById("totalPhones").textContent =
    products.filter(p => p.category === "Phone").length;

  document.getElementById("totalAccessories").textContent =
    products.filter(p => p.category === "Accessory").length;

}


function createPlaceholder(category) {

  const text =
    category === "Phone"
    ? "📱 PHONE"
    : "🎧 ACCESSORY";

  const svg = `
    <svg xmlns="http://www.w3.org/2000/svg"
         width="600"
         height="400">
      <rect width="100%"
            height="100%"
            fill="#e5e7eb"/>
      <text x="50%"
            y="50%"
            dominant-baseline="middle"
            text-anchor="middle"
            font-size="50"
            font-family="Arial"
            fill="#374151">
        ${text}
      </text>
    </svg>
  `;

  return "data:image/svg+xml;charset=UTF-8," +
    encodeURIComponent(svg);
}


displayProducts();

</script>

</body>
</html>

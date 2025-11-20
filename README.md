import React, { useState } from "react";

export default function AddProduct() {
  const [name, setName] = useState("");
  const [desc, setDesc] = useState("");
  const [price, setPrice] = useState("");
  const seller = localStorage.getItem("username");

  const handleAdd = async () => {
    const res = await fetch("http://localhost:8080/api/seller/products", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ seller, name, description: desc, price }),
    });

    if (res.ok) {
      alert("Product added!");
      setName("");
      setDesc("");
      setPrice("");
    } else {
      alert("Failed to add product");
    }
  };

  return (
    <div className="container mt-4">
      <h2>Add Product</h2>
      <input className="form-control mb-2" placeholder="Product Name" value={name} onChange={(e) => setName(e.target.value)} />
      <input className="form-control mb-2" placeholder="Description" value={desc} onChange={(e) => setDesc(e.target.value)} />
      <input className="form-control mb-2" placeholder="Price" value={price} onChange={(e) => setPrice(e.target.value)} />
      <button className="btn btn-success" onClick={handleAdd}>Add Product</button>
    </div>
  );
}



import React, { useEffect, useState } from "react";

export default function ViewProducts() {
  const [products, setProducts] = useState([]);
  const seller = localStorage.getItem("username");

  useEffect(() => {
    fetch(`http://localhost:8080/api/seller/products?seller=${seller}`)
      .then(res => res.json())
      .then(data => setProducts(data))
      .catch(err => console.error("Error loading products", err));
  }, []);

  return (
    <div className="container mt-4">
      <h2>Your Products</h2>
      {products.length === 0 ? (
        <p>No products added yet.</p>
      ) : (
        <table className="table">
          <thead>
            <tr>
              <th>Name</th>
              <th>Description</th>
              <th>Price</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>
            {products.map((p, i) => (
              <tr key={i}>
                <td>{p.name}</td>
                <td>{p.description}</td>
                <td>₹{p.price}</td>
                <td>{p.status}</td>
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </div>
  );
}


import React, { useEffect, useState } from "react";

export default function ViewOrders() {
  const [orders, setOrders] = useState([]);
  const seller = localStorage.getItem("username");

  useEffect(() => {
    fetch(`http://localhost:8080/api/seller/orders?seller=${seller}`)
      .then(res => res.json())
      .then(data => setOrders(data))
      .catch(err => console.error("Error loading orders", err));
  }, []);

  return (
    <div className="container mt-4">
      <h2>Orders for Your Products</h2>
      {orders.length === 0 ? (
        <p>No orders yet.</p>
      ) : (
        <table className="table">
          <thead>
            <tr>
              <th>Product</th>
              <th>Customer</th>
              <th>Quantity</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>
            {orders.map((o, i) => (
              <tr key={i}>
                <td>{o.productName}</td>
                <td>{o.customerName}</td>
                <td>{o.quantity}</td>
                <td>{o.status}</td>
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </div>
  );
}


import SellerDashboard from "./seller/SellerDashboard";
import AddProduct from "./seller/AddProduct";
import ViewProducts from "./seller/ViewProducts";
import ViewOrders from "./seller/ViewOrders";

<Route path="/seller-dashboard" element={<SellerDashboard />} />
<Route path="/seller/add-product" element={<AddProduct />} />
<Route path="/seller/view-products" element={<ViewProducts />} />
<Route path="/seller/view-orders" element={<ViewOrders />} />



CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  seller TEXT NOT NULL REFERENCES users(username),
  name TEXT NOT NULL,
  description TEXT,
  price INTEGER NOT NULL,
  status TEXT NOT NULL DEFAULT 'Pending' CHECK (status IN ('Pending', 'Approved', 'Rejected')),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  product_id INTEGER NOT NULL REFERENCES products(id),
  customer TEXT NOT NULL REFERENCES users(username),
  quantity INTEGER NOT NULL DEFAULT 1,
  status TEXT NOT NULL DEFAULT 'Pending' CHECK (status IN ('Pending', 'Accepted', 'Shipped', 'Delivered')),
  ordered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

SELECT id, name, description, price, status
FROM products
WHERE seller = 'alice'
ORDER BY created_at DESC;

SELECT
  o.id,
  p.name AS product_name,
  o.customer AS customer_name,
  o.quantity,
  o.status
FROM orders o
JOIN products p ON o.product_id = p.id
WHERE p.seller = 'alice'
ORDER BY o.ordered_at DESC;



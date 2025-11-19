import React, { useState } from "react";
import API from "../../services/api";

export default function AddProduct() {
  const [product, setProduct] = useState({ name: "", description: "", price: "" });

  const handleSubmit = async () => {
    try {
      await API.post("/seller/products", product);
      alert("Product added!");
    } catch (err) {
      alert("Failed to add product");
    }
  };

  return (
    <div className="container mt-5">
      <h2>Add Product</h2>
      <input className="form-control mb-2" placeholder="Name" onChange={(e) => setProduct({ ...product, name: e.target.value })} />
      <input className="form-control mb-2" placeholder="Description" onChange={(e) => setProduct({ ...product, description: e.target.value })} />
      <input className="form-control mb-2" type="number" placeholder="Price" onChange={(e) => setProduct({ ...product, price: e.target.value })} />
      <button className="btn btn-success" onClick={handleSubmit}>Add Product</button>
    </div>
  );
}

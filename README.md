import React from "react";
import { useNavigate } from "react-router-dom";

export default function SellerDashboard() {
  const navigate = useNavigate();

  return (
    <div className="container mt-5">
      <h2 className="text-center">Welcome Seller!</h2>
      <div className="d-flex justify-content-center mt-4">
        <button className="btn btn-primary me-3" onClick={() => navigate("/seller/add-product")}>
          Add Product
        </button>
        <button className="btn btn-secondary" onClick={() => navigate("/seller/view-orders")}>
          View Orders
        </button>
      </div>
      <div className="mt-5">
        <h4>Your Orders:</h4>
        <p>No orders yet.</p>
      </div>
    </div>
  );
}




import React, { useState } from "react";

export default function AddProduct() {
  const [product, setProduct] = useState({
    name: "",
    description: "",
    price: "",
    image: null,
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setProduct((prev) => ({ ...prev, [name]: value }));
  };

  const handleFileChange = (e) => {
    setProduct((prev) => ({ ...prev, image: e.target.files[0] }));
  };

  const handleSubmit = () => {
    const existing = JSON.parse(localStorage.getItem("sellerProducts") || "[]");
    const newProduct = { ...product, id: Date.now() };
    localStorage.setItem("sellerProducts", JSON.stringify([...existing, newProduct]));
    alert("Product added successfully!");
  };

  return (
    <div className="container mt-5">
      <h2>Add Product</h2>
      <input
        className="form-control mb-2"
        name="name"
        placeholder="Product Name"
        value={product.name}
        onChange={handleChange}
      />
      <input
        className="form-control mb-2"
        name="description"
        placeholder="Description"
        value={product.description}
        onChange={handleChange}
      />
      <input
        className="form-control mb-2"
        name="price"
        type="number"
        placeholder="Price"
        value={product.price}
        onChange={handleChange}
      />
      <input
        className="form-control mb-3"
        type="file"
        onChange={handleFileChange}
      />
      <button className="btn btn-success" onClick={handleSubmit}>
        Add Product
      </button>
    </div>
  );
}

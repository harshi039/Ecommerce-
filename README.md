import React, { useState } from "react";

export default function AddProduct() {
  const [name, setName] = useState("");
  const [desc, setDesc] = useState("");
  const [price, setPrice] = useState("");
  const [image, setImage] = useState(null);

  const handleAdd = async () => {
    const seller = localStorage.getItem("username");
    const formData = new FormData();
    formData.append("name", name);
    formData.append("description", desc);
    formData.append("price", price);
    formData.append("seller", seller);
    formData.append("image", image);

    const res = await fetch("http://localhost:8080/api/seller/products", {
      method: "POST",
      body: formData,
    });

    if (res.ok) {
      alert("Product added");
      setName("");
      setDesc("");
      setPrice("");
      setImage(null);
    } else {
      alert("Failed to add product");
    }
  };

  return (
    <div className="container mt-4">
      <div className="card p-4 shadow-sm" style={{ maxWidth: "500px", margin: "auto" }}>
        <h3 className="mb-3 text-center">Add Product</h3>
        <div className="mb-3">
          <label className="form-label">Product Name</label>
          <input className="form-control" value={name} onChange={(e) => setName(e.target.value)} required />
        </div>
        <div className="mb-3">
          <label className="form-label">Description</label>
          <input className="form-control" value={desc} onChange={(e) => setDesc(e.target.value)} required />
        </div>
        <div className="mb-3">
          <label className="form-label">Price</label>
          <input className="form-control" type="number" value={price} onChange={(e) => setPrice(e.target.value)} required />
        </div>
        <div className="mb-3">
          <label className="form-label">Product Image</label>
          <input className="form-control" type="file" accept="image/*" onChange={(e) => setImage(e.target.files[0])} required />
        </div>
        <button className="btn btn-success w-100" onClick={handleAdd}>Add Product</button>
      </div>
    </div>
  );
}

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
      <h2>Add Product</h2>
      <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Name" />
      <input value={desc} onChange={(e) => setDesc(e.target.value)} placeholder="Description" />
      <input value={price} onChange={(e) => setPrice(e.target.value)} placeholder="Price" type="number" />
      <input type="file" accept="image/*" onChange={(e) => setImage(e.target.files[0])} />
      <button onClick={handleAdd}>Add Product</button>
    </div>
  );
}


import React, { useState, useEffect } from "react";

export default function ViewProducts() {
  const [products, setProducts] = useState([]);
  const seller = localStorage.getItem("username");

  const handleDelete = async (productId) => {
    const confirm = window.confirm("Are you sure you want to delete this product?");
    if (!confirm) return;
    const res = await fetch(`http://localhost:8080/api/seller/products/delete/${productId}`, {
      method: "DELETE",
    });
    if (res.ok) {
      setProducts(products.filter((p) => p.id !== productId));
    } else {
      alert("Failed to delete");
    }
  };

  useEffect(() => {
    fetch(`http://localhost:8080/api/seller/products?seller=${seller}`)
      .then((res) => res.json())
      .then((data) => setProducts(data))
      .catch((err) => console.error("Error loading products", err));
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
              <th>Image</th>
              <th>Name</th>
              <th>Description</th>
              <th>Price</th>
              <th>Status</th>
              <th>Remove</th>
            </tr>
          </thead>
          <tbody>
            {products.map((p) => (
              <tr key={p.id}>
                <td>
                  <img
                    src={`http://localhost:8080${p.imageUrl}`}
                    alt={p.name}
                    style={{ width: "80px", height: "auto", borderRadius: "5px" }}
                  />
                </td>
                <td>{p.name}</td>
                <td>{p.description}</td>
                <td>{p.price}</td>
                <td>{p.status}</td>
                <td>
                  <button
                    onClick={() => handleDelete(p.id)}
                    style={{ backgroundColor: "grey", color: "white", border: "none", padding: "5px 10px" }}
                  >
                    Delete
                  </button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </div>
  );
}

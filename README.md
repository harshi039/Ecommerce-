import React, { useEffect, useState } from "react";
import api from "../../services/api";

export default function AdminDashboard() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [updatingId, setUpdatingId] = useState(null);
  const [error, setError] = useState("");

  useEffect(() => {
    setError("");
    api.get("/admin/products")
      .then((res) => setProducts(res.data))
      .catch(() => setError("Failed to load products"))
      .finally(() => setLoading(false));
  }, []);

  const updateStatus = async (id, status) => {
    setError("");
    setUpdatingId(id);
    try {
      await api.put(`/admin/products/${id}/status`, { status });
      setProducts((prev) => prev.map((p) => (p.id === id ? { ...p, status } : p)));
    } catch {
      setError("Status update failed");
    } finally {
      setUpdatingId(null);
    }
  };

  if (loading) return <div>Loading...</div>;

  return (
    <div style={{ padding: 16 }}>
      <h2>Products Management</h2>

      {error && <div style={{ color: "red", marginBottom: 8 }}>{error}</div>}

      <table border="1" cellPadding="8" cellSpacing="0" width="100%">
        <thead>
          <tr>
            <th>Order ID</th>
            <th>Product</th>
            <th>Seller</th>
            <th>Status</th>
            <th>Price</th>
            <th>Image</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          {products.map((p) => (
            <tr key={p.id}>
              <td>{p.id}</td>
              <td>{p.name}</td>
              <td>{p.seller}</td>
              <td>{p.status}</td>
              <td>{p.price}</td>
              <td>
                {p.image_url ? (
                  <img
                    src={`http://localhost:8080/${p.image_url}`}
                    alt={p.name}
                    width="60"
                    style={{ objectFit: "cover" }}
                  />
                ) : (
                  "-"
                )}
              </td>
              <td>
                <button
                  disabled={updatingId === p.id}
                  onClick={() => updateStatus(p.id, "ACCEPTED")}
                  style={{ marginRight: 8 }}
                >
                  {updatingId === p.id ? "Updating..." : "Accept"}
                </button>
                <button
                  disabled={updatingId === p.id}
                  onClick={() => updateStatus(p.id, "REJECTED")}
                >
                  {updatingId === p.id ? "Updating..." : "Reject"}
                </button>
              </td>
            </tr>
          ))}
          {products.length === 0 && (
            <tr><td colSpan="7" style={{ textAlign: "center" }}>No products</td></tr>
          )}
        </tbody>
      </table>
    </div>
  );
}

import React, { useEffect, useState } from "react";
import api from "../../services/api";

export default function SellerDashboard() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  useEffect(() => {
    setError("");
    api.get("/seller/products")
      .then((res) => setProducts(res.data))
      .catch(() => setError("Failed to load your products"))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <div style={{ padding: 16 }}>
      <h2>My Products</h2>

      {error && <div style={{ color: "red", marginBottom: 8 }}>{error}</div>}

      <table border="1" cellPadding="8" cellSpacing="0" width="100%">
        <thead>
          <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Price</th>
            <th>Status</th>
            <th>Image</th>
          </tr>
        </thead>
        <tbody>
          {products.map((p) => (
            <tr key={p.id}>
              <td>{p.id}</td>
              <td>{p.name}</td>
              <td>{p.price}</td>
              <td style={{ fontWeight: 600 }}>
                {p.status}
              </td>
              <td>
                {p.image_url ? (
                  <img
                    src={`http://localhost:8080/${p.image_url}`}
                    alt={p.name}
                    width="60"
                    style={{ objectFit: "cover" }}
                  />
                ) : (
                  "-"
                )}
              </td>
            </tr>
          ))}
          {products.length === 0 && (
            <tr><td colSpan="5" style={{ textAlign: "center" }}>No products</td></tr>
          )}
        </tbody>
      </table>
    </div>
  );
}

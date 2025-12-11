import React, { useEffect, useState } from 'react';
import axios from 'axios';
import './AdminDashboard.css'; // optional for styling

export default function AdminDashboard() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    axios.get('/api/admin/products')
      .then(res => setProducts(res.data))
      .catch(err => console.error('Error fetching products:', err));
  }, []);

  const updateStatus = (id, status) => {
    axios.put(`/api/admin/products/${id}`, { status })
      .then(() => {
        setProducts(prev =>
          prev.map(p => p.id === id ? { ...p, status } : p)
        );
      })
      .catch(err => console.error('Error updating status:', err));
  };

  return (
    <div className="admin-dashboard">
      <h2>Admin Product Management</h2>
      <table className="admin-table">
        <thead>
          <tr>
            <th>Product</th>
            <th>Description</th>
            <th>Price</th>
            <th>Seller</th>
            <th>Status</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          {products.length === 0 ? (
            <tr><td colSpan="6">No products available</td></tr>
          ) : (
            products.map(p => (
              <tr key={p.id}>
                <td>{p.name}</td>
                <td>{p.description}</td>
                <td>₹{p.price}</td>
                <td>{p.seller}</td>
                <td>{p.status}</td>
                <td>
                  {p.status === 'Pending' && (
                    <>
                      <button onClick={() => updateStatus(p.id, 'Approved')}>Accept</button>
                      <button onClick={() => updateStatus(p.id, 'Rejected')}>Reject</button>
                    </>
                  )}
                </td>
              </tr>
            ))
          )}
        </tbody>
      </table>
    </div>
  );
}




.admin-dashboard {
  padding: 20px;
  font-family: Arial, sans-serif;
}

.admin-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.admin-table th, .admin-table td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: left;
}

.admin-table th {
  background-color: #f4f4f4;
}

.admin-table button {
  margin-right: 5px;
  padding: 5px 10px;
  cursor: pointer;
}

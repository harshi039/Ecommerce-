.admin-container {
  padding: 30px;
  font-family: 'Segoe UI', sans-serif;
  background: #f4f6f9;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
}

.card-section {
  background: white;
  padding: 20px;
  margin-bottom: 30px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.styled-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

.styled-table th, .styled-table td {
  padding: 12px 15px;
  border-bottom: 1px solid #ddd;
  text-align: left;
}

.styled-table tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}

.btn {
  padding: 6px 12px;
  margin-right: 6px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 500;
}

.accept {
  background-color: #28a745;
  color: white;
}

.reject {
  background-color: #dc3545;
  color: white;
}

.remove {
  background-color: #6c757d;
  color: white;
}

.btn:hover {
  opacity: 0.85;
}

.badge {
  padding: 4px 8px;
  border-radius: 6px;
  font-size: 0.85em;
  font-weight: bold;
  color: white;
}

.badge-admin {
  background-color: #007bff;
}

.badge-seller {
  background-color: #17a2b8;
}

.badge-customer {
  background-color: #ffc107;
}


import React, { useState } from "react";
import "./AdminDashboard.css"; // Custom styles

const AdminDashboard = () => {
  const [orders, setOrders] = useState([
    { id: 1, product: "Laptop", seller: "Seller A", customer: "User X", status: "PENDING" },
    { id: 2, product: "Phone", seller: "Seller B", customer: "User Y", status: "PENDING" },
  ]);

  const [users, setUsers] = useState([
    { id: 1, name: "User X", role: "Customer", email: "userx@example.com" },
    { id: 2, name: "Seller A", role: "Seller", email: "sellera@example.com" },
    { id: 3, name: "Admin", role: "Admin", email: "admin@example.com" },
  ]);

  const [products] = useState([
    { id: 101, name: "Laptop", orderedBy: "User X" },
    { id: 102, name: "Phone", orderedBy: "User Y" },
  ]);

  const handleAccept = (id) => {
    setOrders(orders.map(o => o.id === id ? { ...o, status: "ACCEPTED" } : o));
  };

  const handleReject = (id) => {
    setOrders(orders.map(o => o.id === id ? { ...o, status: "REJECTED" } : o));
  };

  const handleRemoveUser = (id) => {
    setUsers(users.filter(u => u.id !== id));
  };

  const getBadgeClass = (role) => {
    switch (role) {
      case "Admin": return "badge-admin";
      case "Seller": return "badge-seller";
      default: return "badge-customer";
    }
  };

  return (
    <div className="admin-container">
      <h1>Admin Dashboard</h1>

      {/* Orders Section */}
      <div className="card-section">
        <h2>Orders Management</h2>
        <table className="styled-table">
          <thead>
            <tr>
              <th>Order ID</th>
              <th>Product</th>
              <th>Seller</th>
              <th>Customer</th>
              <th>Status</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody>
            {orders.map(order => (
              <tr key={order.id}>
                <td>{order.id}</td>
                <td>{order.product}</td>
                <td>{order.seller}</td>
                <td>{order.customer}</td>
                <td>{order.status}</td>
                <td>
                  <button className="btn accept" onClick={() => handleAccept(order.id)}>Accept</button>
                  <button className="btn reject" onClick={() => handleReject(order.id)}>Reject</button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* User Management Section */}
      <div className="card-section">
        <h2>User Management</h2>
        <table className="styled-table">
          <thead>
            <tr>
              <th>User ID</th>
              <th>Name</th>
              <th>Role</th>
              <th>Email</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody>
            {users.map(user => (
              <tr key={user.id}>
                <td>{user.id}</td>
                <td>{user.name}</td>
                <td><span className={`badge ${getBadgeClass(user.role)}`}>{user.role}</span></td>
                <td>{user.email}</td>
                <td>
                  <button className="btn remove" onClick={() => handleRemoveUser(user.id)}>Remove</button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* Products Ordered Section */}
      <div className="card-section">
        <h2>Products Ordered</h2>
        <table className="styled-table">
          <thead>
            <tr>
              <th>Product ID</th>
              <th>Name</th>
              <th>Ordered By</th>
            </tr>
          </thead>
          <tbody>
            {products.map(product => (
              <tr key={product.id}>
                <td>{product.id}</td>
                <td>{product.name}</td>
                <td>{product.orderedBy}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};

export default AdminDashboard;

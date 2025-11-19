import React, { useState } from "react";

const AdminDashboard = () => {
  // Dummy data just for frontend visualization
  const [orders, setOrders] = useState([
    { id: 1, product: "Laptop", seller: "Seller A", customer: "User X", status: "PENDING" },
    { id: 2, product: "Phone", seller: "Seller B", customer: "User Y", status: "PENDING" },
  ]);

  const [users] = useState([
    { id: 1, name: "User X", role: "Customer", email: "userx@example.com" },
    { id: 2, name: "Seller A", role: "Seller", email: "sellera@example.com" },
    { id: 3, name: "Admin", role: "Admin", email: "admin@example.com" },
  ]);

  const [products] = useState([
    { id: 101, name: "Laptop", orderedBy: "User X" },
    { id: 102, name: "Phone", orderedBy: "User Y" },
  ]);

  // Just frontend state update (no backend)
  const handleAccept = (id) => {
    setOrders(orders.map(o => o.id === id ? { ...o, status: "ACCEPTED" } : o));
  };

  const handleReject = (id) => {
    setOrders(orders.map(o => o.id === id ? { ...o, status: "REJECTED" } : o));
  };

  return (
    <div style={{ padding: "20px" }}>
      <h1>Admin Dashboard</h1>

      {/* Orders Management */}
      <h2>Orders Management</h2>
      <table border="1" cellPadding="8" style={{ marginBottom: "20px" }}>
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
                <button onClick={() => handleAccept(order.id)}>Accept</button>
                <button onClick={() => handleReject(order.id)}>Reject</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

      {/* User Management */}
      <h2>User Management</h2>
      <table border="1" cellPadding="8" style={{ marginBottom: "20px" }}>
        <thead>
          <tr>
            <th>User ID</th>
            <th>Name</th>
            <th>Role</th>
            <th>Email</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id}>
              <td>{user.id}</td>
              <td>{user.name}</td>
              <td>{user.role}</td>
              <td>{user.email}</td>
            </tr>
          ))}
        </tbody>
      </table>

      {/* Products Ordered */}
      <h2>Products Ordered</h2>
      <table border="1" cellPadding="8">
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
  );
};

export default AdminDashboard;

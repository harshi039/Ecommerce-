import { useState } from "react";

export default function LoginPage() {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [role, setRole] = useState("customer");

  const handleLogin = (e) => {
    e.preventDefault();
    console.log("Logging in:", { username, password, role });
    // Add backend integration here
  };

  return (
    <div className="d-flex justify-content-center align-items-center" style={{ height: "100vh", background: "linear-gradient(135deg, #0072CE, #00487E)" }}>
      <div className="card shadow" style={{ width: "400px", padding: "30px", borderRadius: "12px", backgroundColor: "#ffffff" }}>
        <h3 className="text-center mb-4">SC Shop Login</h3>
        <form onSubmit={handleLogin}>
          <div className="mb-3">
            <label className="form-label">Username</label>
            <input type="text" className="form-control" value={username} onChange={(e) => setUsername(e.target.value)} placeholder="Enter username" />
          </div>
          <div className="mb-3">
            <label className="form-label">Password</label>
            <input type="password" className="form-control" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Enter password" />
          </div>
          <div className="mb-4">
            <label className="form-label">Login As</label>
            <select className="form-select" value={role} onChange={(e) => setRole(e.target.value)}>
              <option value="customer">Customer</option>
              <option value="seller">Seller</option>
              <option value="admin">Admin</option>
            </select>
          </div>
          <button type="submit" className="btn btn-primary w-100">Submit</button>
        </form>
      </div>
    </div>
  );
}

Aboutus.js

export default function AboutUs() {
  return (
    <div className="container mt-5">
      <h2>About Us</h2>
      <p>Welcome to React Ecommerce! We offer the latest fashion, electronics, and more. Our team is dedicated to providing a seamless shopping experience for customers, sellers, and admins alike.</p>
    </div>
  );
}

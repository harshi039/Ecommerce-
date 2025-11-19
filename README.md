import React, { useState, useEffect } from "react";
import { useNavigate } from "react-router-dom";

export default function LoginPage() {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [role, setRole] = useState("Customer"); // default role
  const navigate = useNavigate();

  useEffect(() => {
    const savedRole = localStorage.getItem("role");
    if (savedRole === "Seller") {
      navigate("/seller-dashboard");
    } else if (savedRole === "Customer") {
      navigate("/customer/dashboard");
    } else if (savedRole === "Admin") {
      navigate("/admin/dashboard");
    }
  }, [navigate]);

  const handleLogin = () => {
    if (!username || !password || !role) {
      alert("Please fill all fields");
      return;
    }

    localStorage.setItem("username", username);
    localStorage.setItem("role", role);

    if (role === "Seller") {
      navigate("/seller-dashboard");
    } else if (role === "Customer") {
      navigate("/customer/dashboard");
    } else {
      navigate("/admin/dashboard");
    }
  };

  return (
    <div className="container mt-5">
      <h2 className="text-center mb-4">EasyShop Login</h2>
      <div className="row justify-content-center">
        <div className="col-md-6">
          <input
            className="form-control mb-3"
            type="text"
            placeholder="Enter username"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
          />
          <input
            className="form-control mb-3"
            type="password"
            placeholder="Enter password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
          />
          <select
            className="form-select mb-4"
            value={role}
            onChange={(e) => setRole(e.target.value)}
          >
            <option>Customer</option>
            <option>Seller</option>
            <option>Admin</option>
          </select>
          <button className="btn btn-primary w-100" onClick={handleLogin}>
            Submit
          </button>
        </div>
      </div>
    </div>
  );
}

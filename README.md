import React, { useState, useEffect } from "react";
import { useNavigate } from "react-router-dom";
import axios from "axios";

export default function LoginPage() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [role, setRole] = useState("seller"); // default to seller
  const [error, setError] = useState("");
  const navigate = useNavigate();

  useEffect(() => {
    const storedRole = localStorage.getItem("role");
    if (storedRole === "admin") {
      navigate("/admin/dashboard");
    } else if (storedRole === "customer") {
      navigate("/customer/dashboard");
    } else if (storedRole === "seller") {
      navigate("/seller/dashboard");
    }
  }, [navigate]);

  const handleLogin = async () => {
    try {
      if (role === "seller") {
        const res = await axios.post("http://localhost:8080/seller/login", {
          email,
          password,
        });

        const { token, seller } = res.data;
        localStorage.setItem("sellerToken", token);
        localStorage.setItem("role", "seller");
        localStorage.setItem("username", seller.name);

        navigate("/seller/dashboard");
      } else {
        setError("Only seller login is wired to backend right now.");
      }
    } catch (err) {
      setError(err.response?.data?.error || "Login failed");
    }
  };

  return (
    <div className="login-page">
      <div className="login-box">
        <div className="login-content">
          <div className="login-title">Login</div>

          <input
            type="text"
            placeholder="Email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            className="form-control mb-2"
          />

          <input
            type="password"
            placeholder="Password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            className="form-control mb-2"
          />

          <select
            value={role}
            onChange={(e) => setRole(e.target.value)}
            className="form-select mb-3"
          >
            <option value="seller">Seller</option>
            <option value="admin">Admin</option>
            <option value="customer">Customer</option>
          </select>

          <button className="btn btn-primary" onClick={handleLogin}>
            Submit
          </button>

          {error && <div className="text-danger mt-3">{error}</div>}
        </div>
      </div>
    </div>
  );
}

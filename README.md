import { useNavigate } from 'react-router-dom';
import { useState, useEffect } from 'react';

export default function LoginPage() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [role, setRole] = useState('customer'); // default role
  const navigate = useNavigate();

  const handleLogin = () => {
    localStorage.setItem('role', role);
    localStorage.setItem('username', username);

    if (role === 'admin') {
      navigate('/admin/dashboard');
    } else if (role === 'seller') {
      navigate('/seller/dashboard');
    } else {
      navigate('/customer/dashboard');
    }
  };

  useEffect(() => {
    const savedRole = localStorage.getItem('role');
    if (savedRole === 'admin') {
      navigate('/admin/dashboard');
    } else if (savedRole === 'seller') {
      navigate('/seller/dashboard');
    } else if (savedRole === 'customer') {
      navigate('/customer/dashboard');
    }
  }, [navigate]);

  return (
    <div className="login-page">
      <div className="login-box">
        <div className="login-content">
          <div className="login-title">Login</div>

          <input
            type="text"
            placeholder="Username"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
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
            <option value="customer">Customer</option>
            <option value="seller">Seller</option>
            <option value="admin">Admin</option>
          </select>

          <button className="btn btn-primary" onClick={handleLogin}>
            Login
          </button>
        </div>
      </div>
    </div>
  );
}

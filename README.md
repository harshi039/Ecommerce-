import React from "react";
import "./Home.css";

function Home() {
  const products = [
    { id: 1, name: "Wireless Headphones", price: "₹2,999", image: "https://via.placeholder.com/150" },
    { id: 2, name: "Smart Watch", price: "₹3,499", image: "https://via.placeholder.com/150" },
    { id: 3, name: "Bluetooth Speaker", price: "₹1,899", image: "https://via.placeholder.com/150" },
    { id: 4, name: "Gaming Mouse", price: "₹999", image: "https://via.placeholder.com/150" },
  ];

  return (
    <div className="home-container">
      <nav className="navbar">
        <div className="logo">🛒 ShopEasy</div>
        <ul className="nav-links">
          <li>Home</li>
          <li>Products</li>
          <li>Cart</li>
          <li>Profile</li>
          <li>Logout</li>
        </ul>
      </nav>

      <div className="content">
        <h1>Welcome to ShopEasy</h1>
        <p>Your one-stop shop for trending gadgets!</p>

        <div className="products-grid">
          {products.map((item) => (
            <div key={item.id} className="product-card">
              <img src={item.image} alt={item.name} />
              <h3>{item.name}</h3>
              <p>{item.price}</p>
              <button>Add to Cart</button>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

export default Home;
/* Home.css */
body {
  margin: 0;
  font-family: "Poppins", sans-serif;
  background: linear-gradient(135deg, #00b4db, #0083b0);
  color: white;
  height: 100vh;
}

.home-container {
  text-align: center;
}

.navbar {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  padding: 15px 40px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid rgba(255, 255, 255, 0.2);
}

.logo {
  font-weight: bold;
  font-size: 20px;
  color: white;
}

.nav-links {
  list-style: none;
  display: flex;
  gap: 30px;
  margin: 0;
  padding: 0;
}

.nav-links li {
  cursor: pointer;
  font-size: 16px;
  transition: 0.3s;
}

.nav-links li:hover {
  color: #003366;
  font-weight: 500;
}

.content {
  margin-top: 150px;
}

.content h1 {
  font-size: 28px;
  margin-bottom: 10px;
}

.content p {
  font-size: 16px;
  color: #f0f0f0;
}

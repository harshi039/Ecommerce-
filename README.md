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

navbar.js
import { Link } from "react-router-dom";

export default function Navbar() {
  return (
    <nav className="navbar navbar-expand-lg navbar-dark bg-dark px-4">
      <Link className="navbar-brand" to="/">React Ecommerce</Link>
      <div className="collapse navbar-collapse">
        <ul className="navbar-nav me-auto">
          <li className="nav-item"><Link className="nav-link" to="/">Home</Link></li>
          <li className="nav-item"><Link className="nav-link" to="/products">Products</Link></li>
          <li className="nav-item"><Link className="nav-link" to="/contact">Contact</Link></li>
        </ul>
        <div>
          <button className="btn btn-outline-light me-2">Login</button>
          <button className="btn btn-outline-light me-2">Register</button>
          <button className="btn btn-warning">Cart (0)</button>
        </div>
      </div>
    </nav>
  );
}

home.js
export default function Home() {
  return (
    <div className="text-center mt-5">
      <h1>New Season Arrivals</h1>
      <p>Check out all the latest trends</p>
    </div>
  );
}

App.js

import { BrowserRouter as Router, Routes, Route } from "react-router-dom";
import Navbar from "./components/Navbar";
import Home from "./pages/Home";
import Products from "./pages/Products";
import Contact from "./pages/Contact";

function App() {
  return (
    <Router>
      <Navbar />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/products" element={<Products />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </Router>
  );
}

export default App;

contact.js
export default function Contact() {
  return (
    <div className="container mt-5">
      <h2>Contact Us</h2>
      <p>Email: support@reactecom.com</p>
    </div>
  );
}




import { Link } from "react-router-dom";

export default function Navbar() {
  const cartItems = JSON.parse(localStorage.getItem("cartItems")) || [];

  return (
    <nav className="navbar navbar-expand-lg bg-light px-5">
      <Link className="navbar-brand" to="/product">EasyShop</Link>
      <div className="navbar-nav">
        {cartItems.length > 0 && (
          <Link className="nav-link" to="/cart">
            Cart ({cartItems.length})
          </Link>
        )}
      </div>
    </nav>
  );
}



export default function Cart() {
  const cartItems = JSON.parse(localStorage.getItem("cartItems")) || [];

  return (
    <div className="container mt-5">
      <h2>Your Cart</h2>
      {cartItems.length === 0 ? (
        <p>No items in cart.</p>
      ) : (
        cartItems.map((item, index) => (
          <div key={index} className="mb-3">
            <h4>{item.name}</h4>
            <p>{item.description}</p>
            <h5>{item.price}/-</h5>
          </div>
        ))
      )}
    </div>
  );
}

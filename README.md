import { useState } from "react";
import { useNavigate } from "react-router-dom";

export default function Delivery() {
  const [name, setName] = useState("");
  const [address, setAddress] = useState("");
  const [payment, setPayment] = useState("");
  const [error, setError] = useState("");
  const navigate = useNavigate();

  const handleConfirmBuy = () => {
    if (!name || !address || !payment) {
      setError("Please fill in all fields before confirming.");
      return;
    }

    localStorage.setItem("orderDetails", JSON.stringify({ name, address, payment }));
    navigate("/order-confirmed");
  };

  return (
    <div className="container mt-5">
      <h2>Delivery & Payment</h2>

      {error && <div className="alert alert-danger">{error}</div>}

      <input
        type="text"
        placeholder="Your Name"
        className="form-control mb-3"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <textarea
        placeholder="Your Address"
        className="form-control mb-3"
        value={address}
        onChange={(e) => setAddress(e.target.value)}
      />
      <input
        type="text"
        placeholder="Payment Method (e.g., UPI, Card)"
        className="form-control mb-3"
        value={payment}
        onChange={(e) => setPayment(e.target.value)}
      />
      <button className="btn btn-success" onClick={handleConfirmBuy}>
        Confirm Buy
      </button>
    </div>
  );
}

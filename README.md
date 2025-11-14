import { useState } from "react";
import { useNavigate } from "react-router-dom";

export default function Delivery() {
  const [name, setName] = useState("");
  const [address, setAddress] = useState("");
  const [payment, setPayment] = useState("");
  const navigate = useNavigate();

  const handleConfirmBuy = () => {
    localStorage.setItem("orderDetails", JSON.stringify({ name, address, payment }));
    navigate("/order-confirmed");
  };

  return (
    <div className="container mt-5">
      <h2>Delivery & Payment Details</h2>
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
      <button className="btn btn-primary" onClick={handleConfirmBuy}>
        Confirm Buy
      </button>
    </div>
  );
}

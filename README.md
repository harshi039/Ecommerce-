import React from "react";
import { useNavigate } from "react-router-dom";

export default function SellerDashboard() {
  const navigate = useNavigate();
  const sellerName = localStorage.getItem("username") || "Seller";

  return (
    <div className="container mt-5">
      <h2 className="text-center mb-4">Welcome {sellerName}!</h2>

      <div className="d-flex justify-content-center mb-5">
        <button
          className="btn btn-primary me-3"
          onClick={() => navigate("/seller/add-product")}
        >
          Add Product
        </button>
        <button
          className="btn btn-secondary"
          onClick={() => navigate("/seller/view-orders")}
        >
          View Orders
        </button>
      </div>

      <div className="text-center">
        <h4>Your Orders:</h4>
        <p>No orders yet.</p>
      </div>
    </div>
  );
}

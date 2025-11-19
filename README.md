import React, { useEffect, useState } from "react";
import API from "../../services/api";

export default function SellerDashboard() {
  const [orders, setOrders] = useState([]);

  useEffect(() => {
    API.get("/seller/orders")
      .then((res) => setOrders(res.data))
      .catch((err) => console.error("Failed to fetch orders:", err));
  }, []);

  return (
    <div className="container mt-5">
      <h2>Welcome Seller!</h2>
      <button className="btn btn-primary">Add Product</button>
      <button className="btn btn-secondary ms-2">View Orders</button>
    </div>
  );
}

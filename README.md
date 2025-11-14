import { useNavigate } from "react-router-dom";

export default function SellerDashboard() {
  const navigate = useNavigate();

  return (
    <div className="container mt-5">
      <h2>Welcome Seller</h2>
      <button className="btn btn-primary me-3" onClick={() => navigate("/seller/add-product")}>
        Add Product
      </button>
      <button className="btn btn-secondary" onClick={() => navigate("/seller/view-orders")}>
        View Orders
      </button>
    </div>
  );
}


import { useState } from "react";

export default function AddProduct() {
  const [name, setName] = useState("");
  const [desc, setDesc] = useState("");
  const [price, setPrice] = useState("");

  const handleAdd = () => {
    if (!name || !desc || !price) return alert("All fields required");

    const newProduct = { name, description: desc, price };
    const existing = JSON.parse(localStorage.getItem("productList")) || [];
    existing.push(newProduct);
    localStorage.setItem("productList", JSON.stringify(existing));
    alert("Product added!");
    setName(""); setDesc(""); setPrice("");
  };

  return (
    <div className="container mt-5">
      <h2>Add Product</h2>
      <input className="form-control mb-2" placeholder="Product Name" value={name} onChange={(e) => setName(e.target.value)} />
      <textarea className="form-control mb-2" placeholder="Description" value={desc} onChange={(e) => setDesc(e.target.value)} />
      <input className="form-control mb-2" placeholder="Price" value={price} onChange={(e) => setPrice(e.target.value)} />
      <button className="btn btn-success" onClick={handleAdd}>Add Product</button>
    </div>
  );
}


import { useState } from "react";

export default function ViewOrders() {
  const [orders, setOrders] = useState(JSON.parse(localStorage.getItem("orderDetailsList")) || []);

  const handleAccept = (index) => {
    const updated = [...orders];
    updated[index].status = "Accepted";
    setOrders(updated);
    localStorage.setItem("orderDetailsList", JSON.stringify(updated));
  };

  return (
    <div className="container mt-5">
      <h2>Customer Orders</h2>
      {orders.length === 0 ? <p>No orders yet.</p> : (
        orders.map((order, i) => (
          <div key={i} className="border p-3 mb-2">
            <p><strong>Name:</strong> {order.name}</p>
            <p><strong>Address:</strong> {order.address}</p>
            <p><strong>Payment:</strong> {order.payment}</p>
            <p><strong>Status:</strong> {order.status || "Pending"}</p>
            {order.status !== "Accepted" && (
              <button className="btn btn-primary" onClick={() => handleAccept(i)}>Accept Order</button>
            )}
          </div>
        ))
      )}
    </div>
  );
}


export { default as SellerDashboard } from './SellerDashboard';
export { default as AddProduct } from './AddProduct';
export { default as ViewOrders } from './ViewOrders';



import { SellerDashboard, AddProduct, ViewOrders } from "./pages/seller";

<Route path="/seller/dashboard" element={<SellerDashboard />} />
<Route path="/seller/add-product" element={<AddProduct />} />
<Route path="/seller/view-orders" element={<ViewOrders />} />

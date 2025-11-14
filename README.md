export default function OrderConfirmed() {
  return (
    <div className="container mt-5 text-center">
      <h2 className="text-success">✅ Order Confirmed!</h2>
      <p>Thank you for shopping with EasyShop.</p>
    </div>
  );
}


import Delivery from "./pages/Delivery";
import OrderConfirmed from "./pages/OrderConfirmed";

<Route path="/delivery" element={<Delivery />} />
<Route path="/order-confirmed" element={<OrderConfirmed />} />


import { useNavigate } from "react-router-dom";

const Checkout = () => {
  const navigate = useNavigate();

  const handleConfirmCheckout = () => {
    navigate("/delivery");
  };

  return (
    <div className="container mt-5">
      {/* Product details here */}
      <button className="btn btn-success" onClick={handleConfirmCheckout}>
        Confirm Checkout
      </button>
    </div>
  );
};

import { useNavigate } from "react-router-dom";

export default function Checkout() {
  const navigate = useNavigate();
  const product = JSON.parse(localStorage.getItem("checkoutProduct"));

  const handleConfirmCheckout = () => {
    navigate("/delivery");
  };

  return (
    <div className="container mt-5">
      <h2>Checkout</h2>
      {product ? (
        <>
          <h4>{product.name}</h4>
          <p>{product.description}</p>
          <h5>{product.price}/-</h5>
          <button className="btn btn-primary" onClick={handleConfirmCheckout}>
            Confirm Checkout
          </button>
        </>
      ) : (
        <p>No product selected.</p>
      )}
    </div>
  );
}

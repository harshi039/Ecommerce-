export default function Products() {
  const username = localStorage.getItem("username");

  const productData = [
    {
      id: 1,
      name: "Fjallraven Backpack",
      description: "Perfect pack for everyday use and forest walks.",
      price: "$109.95",
      image: "/images/bag.jpg"
    },
    {
      id: 2,
      name: "Men's Casual Shirt",
      description: "Slim-fitting style with contrast raglan sleeves.",
      price: "$22.30",
      image: "/images/shirt.jpg"
    },
    {
      id: 3,
      name: "Men's Cotton Jacket",
      description: "Great outerwear for Spring/Autumn/Winter.",
      price: "$55.99",
      image: "/images/jacket.jpg"
    }
  ];

  return (
    <div className="container mt-4">
      <h4 className="mb-4">Welcome, {username}!</h4>
      <div className="row">
        {productData.map((product) => (
          <div key={product.id} className="col-md-4 mb-4">
            <div className="card h-100">
              <img src={product.image} className="card-img-top" alt={product.name} />
              <div className="card-body">
                <h5 className="card-title">{product.name}</h5>
                <p className="card-text">{product.description}</p>
                <p className="fw-bold">{product.price}</p>
                <button className="btn btn-success me-2">Buy Now</button>
                <button className="btn btn-primary">Add to Cart</button>
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}

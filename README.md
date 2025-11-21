useEffect(() => {
  fetch(`http://localhost:8080/api/seller/products?seller=${seller}`)
    .then((res) => res.json())
    .then((data) => {
      if (Array.isArray(data)) {
        setProducts(data);
      } else {
        console.error("Unexpected response:", data);
        setProducts([]);
      }
    })
    .catch((err) => {
      console.error("Error loading products", err);
      setProducts([]);
    });
}, []);

import React from "react";
import { useNavigate } from "react-router-dom";

export default function SellerDashboard() {
  const navigate = useNavigate();
  const seller = localStorage.getItem("username");

  return (
    <div className="container mt-4">
      <h2>Welcome {seller}!</h2>
      <div className="mb-3">
        <button className="btn btn-primary me-2" onClick={() => navigate("/seller/add-product")}>
          Add Product
        </button>
        <button className="btn btn-secondary me-2" onClick={() => navigate("/seller/view-products")}>
          View Products
        </button>
      </div>
      <h4>Your Orders:</h4>
      <div>
        <button className="btn btn-outline-dark" onClick={() => navigate("/seller/view-orders")}>
          View Orders
        </button>
      </div>
    </div>
  );
}



import React, { useState } from "react";

export default function AddProduct() {
  const [name, setName] = useState("");
  const [desc, setDesc] = useState("");
  const [price, setPrice] = useState("");
  const seller = localStorage.getItem("username");

  const handleAdd = async () => {
    const res = await fetch("http://localhost:8080/api/seller/products", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ seller, name, description: desc, price }),
    });

    if (res.ok) {
      alert("Product added!");
      setName("");
      setDesc("");
      setPrice("");
    } else {
      alert("Failed to add product");
    }
  };

  return (
    <div className="container mt-4">
      <h2>Add Product</h2>
      <input className="form-control mb-2" placeholder="Product Name" value={name} onChange={(e) => setName(e.target.value)} />
      <input className="form-control mb-2" placeholder="Description" value={desc} onChange={(e) => setDesc(e.target.value)} />
      <input className="form-control mb-2" placeholder="Price" value={price} onChange={(e) => setPrice(e.target.value)} />
      <button className="btn btn-success" onClick={handleAdd}>Add Product</button>
    </div>
  );
}


import React, { useEffect, useState } from "react";

export default function ViewProducts() {
  const [products, setProducts] = useState([]);
  const seller = localStorage.getItem("username");

  useEffect(() => {
    fetch(`http://localhost:8080/api/seller/products?seller=${seller}`)
      .then(res => res.json())
      .then(data => setProducts(data))
      .catch(err => console.error("Error loading products", err));
  }, []);

  return (
    <div className="container mt-4">
      <h2>Your Products</h2>
      {products.length === 0 ? (
        <p>No products added yet.</p>
      ) : (
        <table className="table">
          <thead>
            <tr>
              <th>Name</th>
              <th>Description</th>
              <th>Price</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>
            {products.map((p, i) => (
              <tr key={i}>
                <td>{p.name}</td>
                <td>{p.description}</td>
                <td>₹{p.price}</td>
                <td>{p.status}</td>
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </div>
  );
}


import React, { useEffect, useState } from "react";

export default function ViewOrders() {
  const [orders, setOrders] = useState([]);
  const seller = localStorage.getItem("username");

  useEffect(() => {
    fetch(`http://localhost:8080/api/seller/orders?seller=${seller}`)
      .then(res => res.json())
      .then(data => setOrders(data))
      .catch(err => console.error("Error loading orders", err));
  }, []);

  return (
    <div className="container mt-4">
      <h2>Orders for Your Products</h2>
      {orders.length === 0 ? (
        <p>No orders yet.</p>
      ) : (
        <table className="table">
          <thead>
            <tr>
              <th>Product</th>
              <th>Customer</th>
              <th>Quantity</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>
            {orders.map((o, i) => (
              <tr key={i}>
                <td>{o.productName}</td>
                <td>{o.customerName}</td>
                <td>{o.quantity}</td>
                <td>{o.status}</td>
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </div>
  );
}


import SellerDashboard from "./seller/SellerDashboard";
import AddProduct from "./seller/AddProduct";
import ViewProducts from "./seller/ViewProducts";
import ViewOrders from "./seller/ViewOrders";

<Route path="/seller-dashboard" element={<SellerDashboard />} />
<Route path="/seller/add-product" element={<AddProduct />} />
<Route path="/seller/view-products" element={<ViewProducts />} />
<Route path="/seller/view-orders" element={<ViewOrders />} />




package handlers

import (
	"context"
	"encoding/json"
	"net/http"
	"strconv"

	"github.com/jackc/pgx/v5/pgxpool"
)

type SellerHandler struct {
	DB *pgxpool.Pool
}

func NewSellerHandler(db *pgxpool.Pool) *SellerHandler {
	return &SellerHandler{DB: db}
}

type Product struct {
	ID          int    `json:"id"`
	Seller      string `json:"seller"`
	Name        string `json:"name"`
	Description string `json:"description"`
	Price       int    `json:"price"`
	Status      string `json:"status"`
}

type OrderView struct {
	ProductName  string `json:"productName"`
	CustomerName string `json:"customerName"`
	Quantity     int    `json:"quantity"`
	Status       string `json:"status"`
}

// POST /api/seller/products
func (h *SellerHandler) AddProduct(w http.ResponseWriter, r *http.Request) {
	var p Product
	if err := json.NewDecoder(r.Body).Decode(&p); err != nil {
		http.Error(w, `{"error":"invalid json"}`, http.StatusBadRequest)
		return
	}
	if p.Seller == "" || p.Name == "" || p.Price <= 0 {
		http.Error(w, `{"error":"missing fields"}`, http.StatusBadRequest)
		return
	}

	_, err := h.DB.Exec(r.Context(),
		`INSERT INTO products (seller, name, description, price) VALUES ($1, $2, $3, $4)`,
		p.Seller, p.Name, p.Description, p.Price,
	)
	if err != nil {
		http.Error(w, `{"error":"insert failed"}`, http.StatusInternalServerError)
		return
	}
	w.WriteHeader(http.StatusCreated)
	w.Write([]byte(`{"status":"product added"}`))
}

// GET /api/seller/products?seller=alice
func (h *SellerHandler) ViewProducts(w http.ResponseWriter, r *http.Request) {
	seller := r.URL.Query().Get("seller")
	if seller == "" {
		http.Error(w, `{"error":"missing seller"}`, http.StatusBadRequest)
		return
	}

	rows, err := h.DB.Query(r.Context(),
		`SELECT id, name, description, price, status FROM products WHERE seller = $1 ORDER BY created_at DESC`, seller)
	if err != nil {
		http.Error(w, `{"error":"query failed"}`, http.StatusInternalServerError)
		return
	}
	defer rows.Close()

	var products []Product
	for rows.Next() {
		var p Product
		if err := rows.Scan(&p.ID, &p.Name, &p.Description, &p.Price, &p.Status); err == nil {
			products = append(products, p)
		}
	}
	json.NewEncoder(w).Encode(products)
}

// GET /api/seller/orders?seller=alice
func (h *SellerHandler) ViewOrders(w http.ResponseWriter, r *http.Request) {
	seller := r.URL.Query().Get("seller")
	if seller == "" {
		http.Error(w, `{"error":"missing seller"}`, http.StatusBadRequest)
		return
	}

	rows, err := h.DB.Query(r.Context(), `
		SELECT p.name, o.customer, o.quantity, o.status
		FROM orders o
		JOIN products p ON o.product_id = p.id
		WHERE p.seller = $1
		ORDER BY o.ordered_at DESC
	`, seller)
	if err != nil {
		http.Error(w, `{"error":"query failed"}`, http.StatusInternalServerError)
		return
	}
	defer rows.Close()

	var orders []OrderView
	for rows.Next() {
		var o OrderView
		if err := rows.Scan(&o.ProductName, &o.CustomerName, &o.Quantity, &o.Status); err == nil {
			orders = append(orders, o)
		}
	}
	json.NewEncoder(w).Encode(orders)
}


import "easyshop-backend/handlers"

func Setup(authHandler http.Handler) http.Handler {
	r := mux.NewRouter()

	// Auth routes
	r.Handle("/api/auth/login", authHandler).Methods(http.MethodPost)
	r.Handle("/api/auth/register", authHandler).Methods(http.MethodPost)

	// Seller routes
	db := db.Connect(config.Load().DatabaseURL)
	sellerHandler := handlers.NewSellerHandler(db)
	r.HandleFunc("/api/seller/products", sellerHandler.AddProduct).Methods(http.MethodPost)
	r.HandleFunc("/api/seller/products", sellerHandler.ViewProducts).Methods(http.MethodGet)
	r.HandleFunc("/api/seller/orders", sellerHandler.ViewOrders).Methods(http.MethodGet)

	// Health check
	r.HandleFunc("/api/health", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		w.Write([]byte(`{"status":"ok"}`))
	}).Methods(http.MethodGet)

	// CORS
	c := cors.New(cors.Options{
		AllowedOrigins:   []string{"http://localhost:3000"},
		AllowedMethods:   []string{"GET", "POST", "OPTIONS"},
		AllowedHeaders:   []string{"Authorization", "Content-Type"},
		AllowCredentials: true,
	})

	return c.Handler(r)
}



// DELETE /api/seller/products/{id}
func (h *SellerHandler) DeleteProduct(w http.ResponseWriter, r *http.Request) {
	idStr := mux.Vars(r)["id"]
	id, err := strconv.Atoi(idStr)
	if err != nil {
		http.Error(w, `{"error":"invalid product id"}`, http.StatusBadRequest)
		return
	}

	cmd, err := h.DB.Exec(context.Background(),
		`DELETE FROM products WHERE id = $1`, id)
	if err != nil {
		http.Error(w, `{"error":"delete failed"}`, http.StatusInternalServerError)
		return
	}
	if cmd.RowsAffected() == 0 {
		http.Error(w, `{"error":"product not found"}`, http.StatusNotFound)
		return
	}

	w.WriteHeader(http.StatusOK)
	w.Write([]byte(`{"status":"product deleted"}`))
}

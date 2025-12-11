package handlers

import (
  "context"
  "encoding/json"
  "net/http"
  "github.com/gorilla/mux"
  "github.com/jackc/pgx/v4/pgxpool"
)

type AdminHandler struct {
  Db *pgxpool.Pool
}

func NewAdminHandler(db *pgxpool.Pool) *AdminHandler {
  return &AdminHandler{Db: db}
}

type ProductWithSeller struct {
  ID          int     `json:"id"`
  Name        string  `json:"name"`
  Description string  `json:"description"`
  Price       float64 `json:"price"`
  ImageURL    string  `json:"image_url"`
  Status      string  `json:"status"`
  Seller      string  `json:"seller"`
}

// GET /api/admin/products
func (h *AdminHandler) GetAllProducts(w http.ResponseWriter, r *http.Request) {
  rows, err := h.Db.Query(context.Background(), `
    SELECT id, name, description, price, image_url, status, seller FROM products
  `)
  if err != nil {
    http.Error(w, "Failed to fetch products", http.StatusInternalServerError)
    return
  }

  var products []ProductWithSeller
  for rows.Next() {
    var p ProductWithSeller
    err := rows.Scan(&p.ID, &p.Name, &p.Description, &p.Price, &p.ImageURL, &p.Status, &p.Seller)
    if err == nil {
      products = append(products, p)
    }
  }

  w.Header().Set("Content-Type", "application/json")
  json.NewEncoder(w).Encode(products)
}

// PUT /api/admin/products/{id}
func (h *AdminHandler) UpdateProductStatus(w http.ResponseWriter, r *http.Request) {
  var req struct {
    Status string `json:"status"`
  }
  json.NewDecoder(r.Body).Decode(&req)
  id := mux.Vars(r)["id"]

  _, err := h.Db.Exec(context.Background(),
    "UPDATE products SET status = $1 WHERE id = $2", req.Status, id)
  if err != nil {
    http.Error(w, "Failed to update status", http.StatusInternalServerError)
    return
  }

  json.NewEncoder(w).Encode(map[string]string{"message": "Status updated"})
}



func RegisterAdminRoutes(r *mux.Router, db *pgxpool.Pool) {
  handler := handlers.NewAdminHandler(db)
  r.HandleFunc("/api/admin/products", handler.GetAllProducts).Methods("GET")
  r.HandleFunc("/api/admin/products/{id}", handler.UpdateProductStatus).Methods("PUT")
}


func main() {
  db, _ := pgxpool.Connect(context.Background(), os.Getenv("DATABASE_URL"))
  r := mux.NewRouter()
  RegisterAdminRoutes(r, db)
  http.ListenAndServe(":8080", r)
}




import React, { useEffect, useState } from 'react';
import axios from 'axios';

export default function AdminDashboard() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    axios.get('/api/admin/products')
      .then(res => setProducts(res.data))
      .catch(err => console.error(err));
  }, []);

  const updateStatus = (id, status) => {
    axios.put(`/api/admin/products/${id}`, { status })
      .then(() => {
        setProducts(prev =>
          prev.map(p => p.id === id ? { ...p, status } : p)
        );
      })
      .catch(err => console.error(err));
  };

  return (
    <div>
      <h2>Admin Product Management</h2>
      <table>
        <thead>
          <tr>
            <th>Product</th>
            <th>Description</th>
            <th>Price</th>
            <th>Seller</th>
            <th>Status</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          {products.map(p => (
            <tr key={p.id}>
              <td>{p.name}</td>
              <td>{p.description}</td>
              <td>{p.price}</td>
              <td>{p.seller}</td>
              <td>{p.status}</td>
              <td>
                {p.status === 'Pending' && (
                  <>
                    <button onClick={() => updateStatus(p.id, 'Approved')}>Accept</button>
                    <button onClick={() => updateStatus(p.id, 'Rejected')}>Reject</button>
                  </>
                )}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

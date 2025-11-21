package handlers

import (
	"context"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
	"path/filepath"
	"strconv"

	"github.com/gorilla/mux"
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
	ImageURL    string `json:"imageUrl"`
}

type OrderView struct {
	ProductName  string `json:"productName"`
	CustomerName string `json:"customerName"`
	Quantity     int    `json:"quantity"`
	Status       string `json:"status"`
}

// POST /api/seller/products
func (h *SellerHandler) AddProduct(w http.ResponseWriter, r *http.Request) {
	// allow only POST
	if r.Method != http.MethodPost {
		http.Error(w, `{"error":"method not allowed"}`, http.StatusMethodNotAllowed)
		return
	}

	err := r.ParseMultipartForm(10 << 20) // 10MB
	if err != nil {
		http.Error(w, `{"error":"invalid form"}`, http.StatusBadRequest)
		return
	}

	name := r.FormValue("name")
	description := r.FormValue("description")
	priceStr := r.FormValue("price")
	seller := r.FormValue("seller")

	if name == "" || description == "" || priceStr == "" || seller == "" {
		http.Error(w, `{"error":"missing fields"}`, http.StatusBadRequest)
		return
	}

	price, err := strconv.Atoi(priceStr)
	if err != nil {
		http.Error(w, `{"error":"invalid price"}`, http.StatusBadRequest)
		return
	}

	file, handler, err := r.FormFile("image")
	if err != nil {
		http.Error(w, `{"error":"image upload failed"}`, http.StatusBadRequest)
		return
	}
	defer file.Close()

	// ensure uploads dir exists
	if err := os.MkdirAll("uploads", 0755); err != nil {
		fmt.Printf("failed to create upload dir: %v\n", err)
		http.Error(w, `{"error":"server error"}`, http.StatusInternalServerError)
		return
	}

	// make filename safe by using base name and a timestamp or keep original
	filename := filepath.Base(handler.Filename)
	imagePath := filepath.Join("uploads", filename)

	dst, err := os.Create(imagePath)
	if err != nil {
		fmt.Printf("create dst failed: %v\n", err)
		http.Error(w, `{"error":"image save failed"}`, http.StatusInternalServerError)
		return
	}
	defer dst.Close()

	_, err = io.Copy(dst, file)
	if err != nil {
		fmt.Printf("copy file failed: %v\n", err)
		http.Error(w, `{"error":"image save failed"}`, http.StatusInternalServerError)
		return
	}

	// insert into DB. Note: your table column is named "seller" (as shown in screenshots)
	_, err = h.DB.Exec(context.Background(),
		`INSERT INTO products (seller, name, description, price, status, image_url)
		 VALUES ($1, $2, $3, $4, $5, $6)`,
		seller, name, description, price, "Pending", "/"+imagePath)

	if err != nil {
		// log actual error for debugging
		fmt.Printf("Insert error: %v\n", err)
		http.Error(w, `{"error":"insert failed"}`, http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusCreated)
	w.Write([]byte(`{"status":"product added"}`))
}

// GET /api/seller/products?seller=kitty
func (h *SellerHandler) ViewProducts(w http.ResponseWriter, r *http.Request) {
	seller := r.URL.Query().Get("seller")
	if seller == "" {
		http.Error(w, `{"error":"missing seller"}`, http.StatusBadRequest)
		return
	}

	rows, err := h.DB.Query(context.Background(),
		`SELECT id, name, description, price, status, image_url
		 FROM products
		 WHERE seller = $1
		 ORDER BY created_at DESC`, seller)
	if err != nil {
		fmt.Printf("ViewProducts query error: %v\n", err)
		http.Error(w, `{"error":"query failed"}`, http.StatusInternalServerError)
		return
	}
	defer rows.Close()

	var products []Product
	for rows.Next() {
		var p Product
		if err := rows.Scan(&p.ID, &p.Name, &p.Description, &p.Price, &p.Status, &p.ImageURL); err == nil {
			p.Seller = seller
			products = append(products, p)
		} else {
			fmt.Printf("row scan error: %v\n", err)
		}
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(products)
}

// GET /api/seller/orders?seller=kitty
func (h *SellerHandler) ViewOrders(w http.ResponseWriter, r *http.Request) {
	seller := r.URL.Query().Get("seller")
	if seller == "" {
		http.Error(w, `{"error":"missing seller"}`, http.StatusBadRequest)
		return
	}

	rows, err := h.DB.Query(context.Background(),
		`SELECT p.name, o.customer, o.quantity, o.status
		 FROM orders o
		 JOIN products p ON o.product_id = p.id
		 WHERE p.seller = $1
		 ORDER BY o.ordered_at DESC`, seller)
	if err != nil {
		fmt.Printf("ViewOrders query error: %v\n", err)
		http.Error(w, `{"error":"query failed"}`, http.StatusInternalServerError)
		return
	}
	defer rows.Close()

	var orders []OrderView
	for rows.Next() {
		var o OrderView
		if err := rows.Scan(&o.ProductName, &o.CustomerName, &o.Quantity, &o.Status); err == nil {
			orders = append(orders, o)
		} else {
			fmt.Printf("order row scan error: %v\n", err)
		}
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(orders)
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
		fmt.Printf("delete exec error: %v\n", err)
		http.Error(w, `{"error":"delete failed"}`, http.StatusInternalServerError)
		return
	}

	if cmd.RowsAffected() == 0 {
		http.Error(w, `{"error":"product not found"}`, http.StatusNotFound)
		return
	}
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	w.Write([]byte(`{"status":"product deleted"}`))
}

func (h *SellerHandler) AddProduct(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "method not allowed", http.StatusMethodNotAllowed)
		return
	}

	err := r.ParseMultipartForm(10 << 20)
	if err != nil {
		http.Error(w, "invalid form", http.StatusBadRequest)
		return
	}

	name := r.FormValue("name")
	description := r.FormValue("description")
	priceStr := r.FormValue("price")
	seller := r.Context().Value("username").(string)

	if name == "" || description == "" || priceStr == "" {
		http.Error(w, "missing fields", http.StatusBadRequest)
		return
	}

	price, err := strconv.Atoi(priceStr)
	if err != nil {
		http.Error(w, "invalid price", http.StatusBadRequest)
		return
	}

	file, handler, err := r.FormFile("image")
	if err != nil {
		http.Error(w, "image upload failed", http.StatusBadRequest)
		return
	}
	defer file.Close()

	if err := os.MkdirAll("uploads", 0755); err != nil {
		http.Error(w, "server error", http.StatusInternalServerError)
		return
	}

	filename := filepath.Base(handler.Filename)
	imagePath := filepath.Join("uploads", filename)

	dst, err := os.Create(imagePath)
	if err != nil {
		http.Error(w, "image save failed", http.StatusInternalServerError)
		return
	}
	defer dst.Close()
	io.Copy(dst, file)

	_, err = h.DB.ExecContext(r.Context(),
		`INSERT INTO products (seller, name, description, price, status, image_url)
		 VALUES ($1, $2, $3, $4, 'PENDING', $5)`,
		seller, name, description, price, imagePath)
	if err != nil {
		http.Error(w, "insert failed", http.StatusInternalServerError)
		return
	}

	w.WriteHeader(http.StatusCreated)
	w.Write([]byte("Product added"))
}


package models

type Product struct {
	ID          int    `json:"id"`
	Seller      string `json:"seller"`
	Name        string `json:"name"`
	Description string `json:"description"`
	Price       int    `json:"price"`
	Status      string `json:"status"`
	ImageURL    string `json:"image_url"`
	CreatedAt   string `json:"created_at"`
}


func (h *SellerHandler) GetSellerProducts(w http.ResponseWriter, r *http.Request) {
	seller := r.Context().Value("username").(string)

	rows, err := h.DB.QueryContext(r.Context(),
		`SELECT id, name, description, price, status, image_url, created_at
		 FROM products WHERE seller = $1 ORDER BY created_at DESC`, seller)
	if err != nil {
		http.Error(w, "DB error", http.StatusInternalServerError)
		return
	}
	defer rows.Close()

	var products []models.Product
	for rows.Next() {
		var p models.Product
		if err := rows.Scan(&p.ID, &p.Name, &p.Description, &p.Price, &p.Status, &p.ImageURL, &p.CreatedAt); err != nil {
			continue
		}
		p.Seller = seller
		products = append(products, p)
	}

	json.NewEncoder(w).Encode(products)
}



package handlers

import (
	"database/sql"
	"encoding/json"
	"net/http"
	"github.com/go-chi/chi/v5"
	"easyshop-backend/models"
)

type AdminHandler struct {
	DB *sql.DB
}

// GET /admin/products
func (h *AdminHandler) GetAllProducts(w http.ResponseWriter, r *http.Request) {
	rows, err := h.DB.QueryContext(r.Context(),
		`SELECT id, name, seller, description, price, status, image_url, created_at
		 FROM products ORDER BY created_at DESC`)
	if err != nil {
		http.Error(w, "Database error", http.StatusInternalServerError)
		return
	}
	defer rows.Close()

	var products []models.Product
	for rows.Next() {
		var p models.Product
		if err := rows.Scan(&p.ID, &p.Name, &p.Seller, &p.Description, &p.Price, &p.Status, &p.ImageURL, &p.CreatedAt); err != nil {
			continue
		}
		products = append(products, p)
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(products)
}

// PUT /admin/products/{id}/status
func (h *AdminHandler) UpdateProductStatus(w http.ResponseWriter, r *http.Request) {
	id := chi.URLParam(r, "id")

	var body struct {
		Status string `json:"status"`
	}
	if err := json.NewDecoder(r.Body).Decode(&body); err != nil {
		http.Error(w, "Invalid input", http.StatusBadRequest)
		return
	}

	if body.Status != "ACCEPTED" && body.Status != "REJECTED" {
		http.Error(w, "Status must be ACCEPTED or REJECTED", http.StatusBadRequest)
		return
	}

	_, err := h.DB.ExecContext(r.Context(),
		`UPDATE products SET status = $1 WHERE id = $2`, body.Status, id)
	if err != nil {
		http.Error(w, "Update failed", http.StatusInternalServerError)
		return
	}

	w.WriteHeader(http.StatusOK)
	w.Write([]byte("Status updated"))
}


adminHandler := &handlers.AdminHandler{DB: db}

r.Route("/admin", func(r chi.Router) {
	r.Use(AuthMiddleware("ADMIN"))
	r.Get("/products", adminHandler.GetAllProducts)
	r.Put("/products/{id}/status", adminHandler.UpdateProductStatus)
})

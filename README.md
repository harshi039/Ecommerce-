package router

import (
	"easyshop-backend/config"
	"easyshop-backend/db"
	"easyshop-backend/handlers"
	"net/http"

	"github.com/gorilla/mux"
	"github.com/rs/cors"
)

func Setup(authHandler http.Handler) http.Handler {
	r := mux.NewRouter()

	// ✅ Serve uploaded images
	r.PathPrefix("/uploads/").Handler(http.StripPrefix("/uploads/", http.FileServer(http.Dir("./uploads"))))

	// ✅ Auth routes
	r.Handle("/api/auth/login", authHandler).Methods(http.MethodPost)
	r.Handle("/api/auth/register", authHandler).Methods(http.MethodPost)

	// ✅ DB connection
	dbConn := db.Connect(config.Load().DatabaseURL)
	if dbConn == nil {
		panic("Failed to connect to database")
	}

	// ✅ Admin routes
	adminHandler := &handlers.AdminHandler{DB: dbConn}
	adminRouter := r.PathPrefix("/admin").Subrouter()
	adminRouter.Use(func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			if r.Header.Get("Role") != "ADMIN" {
				http.Error(w, "Forbidden", http.StatusForbidden)
				return
			}
			next.ServeHTTP(w, r)
		})
	})
	adminRouter.HandleFunc("/products", adminHandler.GetAllProducts).Methods(http.MethodGet)
	adminRouter.HandleFunc("/products/{id}/status", adminHandler.UpdateProductStatus).Methods(http.MethodPut)

	// ✅ Seller routes
	sellerHandler := &handlers.SellerHandler{DB: dbConn}
	sellerRouter := r.PathPrefix("/seller").Subrouter()
	sellerRouter.Use(func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			if r.Header.Get("Role") != "SELLER" {
				http.Error(w, "Forbidden", http.StatusForbidden)
				return
			}
			next.ServeHTTP(w, r)
		})
	})
	sellerRouter.HandleFunc("/products", sellerHandler.GetSellerProducts).Methods(http.MethodGet)
	sellerRouter.HandleFunc("/products", sellerHandler.AddProduct).Methods(http.MethodPost)

	// ✅ CORS middleware
	c := cors.New(cors.Options{
		AllowedOrigins:   []string{"http://localhost:3000"},
		AllowedMethods:   []string{"GET", "POST", "PUT", "DELETE"},
		AllowedHeaders:   []string{"Authorization", "Content-Type", "Role"},
		AllowCredentials: true,
	})

	return c.Handler(r)
}

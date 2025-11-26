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

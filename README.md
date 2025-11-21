func (h *SellerHandler) AddProduct(w http.ResponseWriter, r *http.Request) {
    err := r.ParseMultipartForm(10 << 20)
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

    imagePath := "uploads/" + handler.Filename
    dst, err := os.Create(imagePath)
    if err != nil {
        http.Error(w, `{"error":"image save failed"}`, http.StatusInternalServerError)
        return
    }
    defer dst.Close()
    io.Copy(dst, file)

    _, err = h.DB.Exec(r.Context(),
        `INSERT INTO products (seller, name, description, price, status, image_url)
         VALUES ($1, $2, $3, $4, $5, $6)`,
        seller, name, description, price, "active", "/"+imagePath)

    if err != nil {
        fmt.Println("Insert error:", err)
        http.Error(w, `{"error":"insert failed"}`, http.StatusInternalServerError)
        return
    }

    w.WriteHeader(http.StatusCreated)
    w.Write([]byte(`{"status":"product added"}`))
}

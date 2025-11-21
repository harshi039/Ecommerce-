func (h *SellerHandler) AddProduct(w http.ResponseWriter, r *http.Request) {
    err := r.ParseMultipartForm(10 << 20) // 10MB
    if err != nil {
        http.Error(w, `{"error":"invalid form"}`, http.StatusBadRequest)
        return
    }

    name := r.FormValue("name")
    description := r.FormValue("description")
    priceStr := r.FormValue("price")
    sellerStr := r.FormValue("seller")

    price, _ := strconv.Atoi(priceStr)
    seller, _ := strconv.Atoi(sellerStr)

    file, handler, err := r.FormFile("image")
    if err != nil {
        http.Error(w, `{"error":"image upload failed"}`, http.StatusBadRequest)
        return
    }
    defer file.Close()

    imagePath := "uploads/" + handler.Filename
    dst, _ := os.Create(imagePath)
    defer dst.Close()
    io.Copy(dst, file)

    product := Product{
        Seller:      seller,
        Name:        name,
        Description: description,
        Price:       price,
        Status:      "active",
        ImageURL:    "/" + imagePath,
    }

    _, err = h.DB.Exec(r.Context(),
        "INSERT INTO products (seller, name, description, price, status, image_url) VALUES ($1, $2, $3, $4, $5, $6)",
        product.Seller, product.Name, product.Description, product.Price, product.Status, product.ImageURL)

    if err != nil {
        http.Error(w, `{"error":"insert failed"}`, http.StatusInternalServerError)
        return
    }

    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(product)
}

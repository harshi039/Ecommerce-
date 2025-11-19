package controllers

import (
  "net/http"
  "seller-dashboard-backend/config"
  "seller-dashboard-backend/models"

  "github.com/gin-gonic/gin"
)

type CreateProductDTO struct {
  SellerID    uint    `json:"sellerId" binding:"required"`
  Name        string  `json:"name" binding:"required"`
  Description string  `json:"description"`
  Price       float64 `json:"price"`
  ImageURL    string  `json:"imageUrl"`
}

func AddProduct(c *gin.Context) {
  var dto CreateProductDTO
  if err := c.ShouldBindJSON(&dto); err != nil {
    c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
    return
  }

  product := models.Product{
    SellerID:    dto.SellerID,
    Name:        dto.Name,
    Description: dto.Description,
    Price:       dto.Price,
    ImageURL:    dto.ImageURL,
    Status:      "Pending",
  }

  if err := config.DB.Create(&product).Error; err != nil {
    c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to create product"})
    return
  }

  c.JSON(http.StatusCreated, product)
}

func GetSellerProducts(c *gin.Context) {
  sellerID := c.Query("sellerId")
  var products []models.Product
  q := config.DB
  if sellerID != "" {
    q = q.Where("seller_id = ?", sellerID)
  }
  if err := q.Order("created_at DESC").Find(&products).Error; err != nil {
    c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to fetch products"})
    return
  }
  c.JSON(http.StatusOK, products)
}


package routes

import (
  "github.com/gin-gonic/gin"
  "seller-dashboard-backend/controllers"
)

func Register(r *gin.Engine) {
  r.Use(func(c *gin.Context) {
    c.Writer.Header().Set("Access-Control-Allow-Origin", "http://localhost:3000")
    c.Writer.Header().Set("Access-Control-Allow-Credentials", "true")
    c.Writer.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
    c.Writer.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
    if c.Request.Method == "OPTIONS" {
      c.AbortWithStatus(204)
      return
    }
    c.Next()
  })

  api := r.Group("/api")
  {
    api.POST("/products", controllers.AddProduct)
    api.GET("/products", controllers.GetSellerProducts)
  }
}


package main

import (
  "log"

  "github.com/joho/godotenv"
  "github.com/gin-gonic/gin"
  "seller-dashboard-backend/config"
  "seller-dashboard-backend/models"
  "seller-dashboard-backend/routes"
)

func main() {
  _ = godotenv.Load()
  config.InitDB()

  if err := config.DB.AutoMigrate(&models.Seller{}, &models.Product{}); err != nil {
    log.Fatalf("Migration failed: %v", err)
  }

  r := gin.Default()
  routes.Register(r)
  r.Run(":8080")
}

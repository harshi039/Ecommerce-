package main

import (
  "log"

  "github.com/joho/godotenv"
  "github.com/gin-gonic/gin"
  "ecommerce/backend/config"
  "ecommerce/backend/models"
  "ecommerce/backend/routes"
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

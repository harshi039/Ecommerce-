package config

import (
    "fmt"
    "os"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

var DB *gorm.DB

func InitDB() {
    dsn := os.Getenv("DATABASE_URL")
    var err error
    DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        panic(fmt.Sprintf("Failed to connect to database: %v", err))
    }
}


package models

type Seller struct {
    ID       uint   `gorm:"primaryKey" json:"id"`
    Name     string `json:"name"`
    Email    string `gorm:"unique" json:"email"`
    Password string `json:"password"`
}


package controllers

import (
    "net/http"
    "seller-dashboard-backend/config"
    "seller-dashboard-backend/models"
    "golang.org/x/crypto/bcrypt"
    "github.com/gin-gonic/gin"
)

func RegisterSeller(c *gin.Context) {
    var input models.Seller
    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(input.Password), bcrypt.DefaultCost)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to hash password"})
        return
    }

    seller := models.Seller{
        Name:     input.Name,
        Email:    input.Email,
        Password: string(hashedPassword),
    }

    if err := config.DB.Create(&seller).Error; err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to create seller"})
        return
    }

    c.JSON(http.StatusOK, gin.H{"message": "Seller registered successfully", "seller": seller})
}


package routes

import (
    "seller-dashboard-backend/controllers"
    "github.com/gin-gonic/gin"
)

func Register(r *gin.Engine) {
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{"message": "pong"})
    })

    r.POST("/seller/register", controllers.RegisterSeller)
}



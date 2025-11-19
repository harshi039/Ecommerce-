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
    ID       uint   `gorm:"primaryKey"`
    Name     string
    Email    string
    Password string
}

type Product struct {
    ID          uint   `gorm:"primaryKey"`
    SellerID    uint
    Name        string
    Description string
    Price       float64
}


package routes

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

func Register(r *gin.Engine) {
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"message": "pong"})
    })
}

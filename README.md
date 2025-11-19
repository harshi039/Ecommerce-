package controllers

import (
    "net/http"
    "os"
    "time"
    "seller-dashboard-backend/config"
    "seller-dashboard-backend/models"
    "github.com/gin-gonic/gin"
    "golang.org/x/crypto/bcrypt"
    "github.com/golang-jwt/jwt/v5"
)

func LoginSeller(c *gin.Context) {
    var input struct {
        Email    string `json:"email"`
        Password string `json:"password"`
    }

    if err := c.ShouldBindJSON(&input); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    var seller models.Seller
    if err := config.DB.Where("email = ?", input.Email).First(&seller).Error; err != nil {
        c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid email"})
        return
    }

    if err := bcrypt.CompareHashAndPassword([]byte(seller.Password), []byte(input.Password)); err != nil {
        c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid password"})
        return
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "seller_id": seller.ID,
        "exp":       time.Now().Add(time.Hour * 72).Unix(),
    })

    tokenString, err := token.SignedString([]byte(os.Getenv("JWT_SECRET")))
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to generate token"})
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "message": "Login successful",
        "token":   tokenString,
        "seller": gin.H{
            "id":    seller.ID,
            "name":  seller.Name,
            "email": seller.Email,
        },
    })
}

package db

import (
	"database/sql"
	"fmt"
	"log"
	"os"

	_ "github.com/lib/pq"
	"github.com/joho/godotenv"
)

var DB *sql.DB

func Connect() {
	err := godotenv.Load()
	if err != nil {
		log.Println("Could not load .env file, using system environment variables")
	}

	host := os.Getenv("DB_HOST")
	port := os.Getenv("DB_PORT")
	user := os.Getenv("DB_USER")
	pass := os.Getenv("DB_PASSWORD")
	name := os.Getenv("DB_NAME")

	connStr := fmt.Sprintf(
		"host=%s port=%s user=%s password=%s dbname=%s sslmode=disable",
		host, port, user, pass, name,
	)

	DB, err = sql.Open("postgres", connStr)
	if err != nil {
		log.Fatalf("Error connecting to database: %v", err)
	}

	err = DB.Ping()
	if err != nil {
		log.Fatalf("Database ping failed: %v", err)
	}

	log.Println("Connected to PostgreSQL successfully!")
}



package models

type User struct {
	ID       int    `json:"id"`
	Username string `json:"username"`
	Password string `json:"password,omitempty"`
	Role     string `json:"role"`
}



package utils

import (
	"os"
	"time"

	"github.com/golang-jwt/jwt/v5"
)

func GenerateToken(id int, username, role string) (string, error) {
	secret := os.Getenv("JWT_SECRET")

	claims := jwt.MapClaims{
		"user_id":  id,
		"username": username,
		"role":     role,
		"exp":      time.Now().Add(72 * time.Hour).Unix(),
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString([]byte(secret))
}



package handlers

import (
	"database/sql"
	"easyshop-backend/db"
	"easyshop-backend/models"
	"easyshop-backend/utils"
	"net/http"

	"github.com/gin-gonic/gin"
	"golang.org/x/crypto/bcrypt"
)

func Register(c *gin.Context) {
	var u models.User
	if err := c.BindJSON(&u); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid input"})
		return
	}

	hashed, _ := bcrypt.GenerateFromPassword([]byte(u.Password), bcrypt.DefaultCost)

	query := `INSERT INTO users (username, password, role) VALUES ($1, $2, $3) RETURNING id`
	var id int
	err := db.DB.QueryRow(query, u.Username, string(hashed), u.Role).Scan(&id)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "Username exists"})
		return
	}

	c.JSON(http.StatusCreated, gin.H{"message": "Registered successfully", "id": id})
}

func Login(c *gin.Context) {
	type LoginInput struct {
		Username string `json:"username"`
		Password string `json:"password"`
		Role     string `json:"role"`
	}

	var input LoginInput
	if err := c.BindJSON(&input); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid input"})
		return
	}

	var user models.User

	err := db.DB.QueryRow(
		"SELECT id, username, password, role FROM users WHERE username=$1",
		input.Username,
	).Scan(&user.ID, &user.Username, &user.Password, &user.Role)

	if err != nil {
		if err == sql.ErrNoRows {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
			return
		}
		c.JSON(http.StatusInternalServerError, gin.H{"error": "Server error"})
		return
	}

	if user.Role != input.Role {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Incorrect role"})
		return
	}

	if bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(input.Password)) != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
		return
	}

	token, _ := utils.GenerateToken(user.ID, user.Username, user.Role)

	c.JSON(http.StatusOK, gin.H{
		"message": "Login successful",
		"token":   token,
		"role":    user.Role,
	})
}



package main

import (
	"easyshop-backend/db"
	"easyshop-backend/handlers"
	"log"

	"github.com/gin-gonic/gin"
)

func main() {
	db.Connect()

	r := gin.Default()

	r.POST("/api/register", handlers.Register)
	r.POST("/api/login", handlers.Login)

	log.Println("Server running on port 8080")
	r.Run(":8080")
}

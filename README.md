package main

import (
	"log"
	"net/http"

	"easyshop-backend/config"
	"easyshop-backend/db"
	"easyshop-backend/handlers"
	"easyshop-backend/router"
)

func main() {
	cfg := config.Load()
	pool := db.Connect(cfg.DatabaseURL)
	authHandler := handlers.NewAuthHandler(pool, cfg.JWTSecret)
	r := router.Setup(authHandler)

	log.Printf("Server running on port %s", cfg.Port)
	log.Fatal(http.ListenAndServe(":"+cfg.Port, r))
}


package config

import (
	"log"
	"os"
	"github.com/joho/godotenv"
)

type Config struct {
	Port        string
	DatabaseURL string
	JWTSecret   string
}

func Load() *Config {
	_ = godotenv.Load()
	return &Config{
		Port:        get("PORT", "8080"),
		DatabaseURL: get("DATABASE_URL", ""),
		JWTSecret:   get("JWT_SECRET", ""),
	}
}

func get(key, def string) string {
	if val := os.Getenv(key); val != "" {
		return val
	}
	return def
}


package db

import (
	"context"
	"log"
	"time"
	"github.com/jackc/pgx/v5/pgxpool"
)

func Connect(databaseURL string) *pgxpool.Pool {
	cfg, err := pgxpool.ParseConfig(databaseURL)
	if err != nil {
		log.Fatalf("parse db config: %v", err)
	}
	pool, err := pgxpool.NewWithConfig(context.Background(), cfg)
	if err != nil {
		log.Fatalf("create db pool: %v", err)
	}
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	if err := pool.Ping(ctx); err != nil {
		log.Fatalf("db ping failed: %v", err)
	}
	return pool
}



package models

type User struct {
	ID           int64  `json:"id"`
	Username     string `json:"username"`
	PasswordHash string `json:"-"`
	Role         string `json:"role"`
}


package auth

import "golang.org/x/crypto/bcrypt"

func HashPassword(plain string) (string, error) {
	b, err := bcrypt.GenerateFromPassword([]byte(plain), bcrypt.DefaultCost)
	return string(b), err
}

func CheckPassword(hash, plain string) bool {
	return bcrypt.CompareHashAndPassword([]byte(hash), []byte(plain)) == nil
}


package auth

import (
	"time"
	"github.com/golang-jwt/jwt/v5"
)

type Claims struct {
	Username string `json:"username"`
	Role     string `json:"role"`
	jwt.RegisteredClaims
}

func GenerateToken(secret, username, role string) (string, error) {
	claims := Claims{
		Username: username,
		Role:     role,
		RegisteredClaims: jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
			IssuedAt:  jwt.NewNumericDate(time.Now()),
		},
	}
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString([]byte(secret))
}


package handlers

import (
	"context"
	"encoding/json"
	"errors"
	"net/http"
	"strings"

	"github.com/jackc/pgx/v5/pgxpool"
	"easyshop-backend/models"
	"easyshop-backend/auth"
)

type AuthHandler struct {
	DB     *pgxpool.Pool
	JWTKey string
}

func NewAuthHandler(db *pgxpool.Pool, jwtKey string) http.Handler {
	return &AuthHandler{DB: db, JWTKey: jwtKey}
}

type loginRequest struct {
	Username string `json:"username"`
	Password string `json:"password"`
	Role     string `json:"role"`
}

type authResponse struct {
	Token    string `json:"token"`
	Username string `json:"username"`
	Role     string `json:"role"`
}

func (h *AuthHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	switch r.URL.Path {
	case "/api/auth/login":
		h.login(w, r)
	case "/api/auth/register":
		h.register(w, r)
	default:
		http.NotFound(w, r)
	}
}

func validateRole(role string) bool {
	switch role {
	case "Customer", "Seller", "Admin":
		return true
	default:
		return false
	}
}

func (h *AuthHandler) login(w http.ResponseWriter, r *http.Request) {
	var req loginRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, `{"error":"invalid json"}`, http.StatusBadRequest)
		return
	}
	req.Username = strings.TrimSpace(req.Username)
	req.Role = strings.TrimSpace(req.Role)

	if req.Username == "" || req.Password == "" || !validateRole(req.Role) {
		http.Error(w, `{"error":"missing or invalid fields"}`, http.StatusBadRequest)
		return
	}

	user, err := h.findUserByUsername(r.Context(), req.Username)
	if err != nil || user.Role != req.Role || !auth.CheckPassword(user.PasswordHash, req.Password) {
		http.Error(w, `{"error":"invalid credentials"}`, http.StatusUnauthorized)
		return
	}

	token, err := auth.GenerateToken(h.JWTKey, user.Username, user.Role)
	if err != nil {
		http.Error(w, `{"error":"token generation failed"}`, http.StatusInternalServerError)
		return
	}

	json.NewEncoder(w).Encode(authResponse{Token: token, Username: user.Username, Role: user.Role})
}

func (h *AuthHandler) register(w http.ResponseWriter, r *http.Request) {
	var req loginRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, `{"error":"invalid json"}`, http.StatusBadRequest)
		return
	}
	req.Username = strings.TrimSpace(req.Username)
	req.Role = strings.TrimSpace(req.Role)

	if req.Username == "" || req.Password == "" || !validateRole(req.Role) {
		http.Error(w, `{"error":"missing or invalid fields"}`, http.StatusBadRequest)
		return
	}

	hash, err := auth.HashPassword(req.Password)
	if err != nil {
		http.Error(w, `{"error":"hashing failed"}`, http.StatusInternalServerError)
		return
	}

	user := models.User{Username: req.Username, PasswordHash: hash, Role: req.Role}
	if err := h.insertUser(r.Context(), user); err != nil {
		http.Error(w, `{"error":"username exists or insert failed"}`, http.StatusConflict)
		return
	}

	w.WriteHeader(http.StatusCreated)
	w.Write([]byte(`{"status":"registered"}`))
}

func (h *AuthHandler) findUserByUsername(ctx context.Context, username string) (*models.User, error) {
	row := h.DB.QueryRow(ctx, `SELECT id, username, password_hash, role FROM users WHERE username = $1`, username)
	var u models.User
	if err := row.Scan(&u.ID, &u.Username, &u.PasswordHash, &u.Role); err != nil {
		return nil, errors.New("not found")
	}
	return &u, nil
}

func (h *AuthHandler) insertUser(ctx context.Context, u models.User) error {
	_, err := h.DB.Exec(ctx,
		`INSERT INTO users (username, password_hash, role) VALUES ($1, $2, $3)`,
		u.Username, u.PasswordHash, u.Role,
	)
	return err
}


package router

import (
	"net/http"
	"github.com/gorilla/mux"
	"github.com/rs/cors"
)

func Setup(authHandler http.Handler) http.Handler {
	r := mux.NewRouter()
	r.Handle("/api/auth/login", authHandler).Methods(http.MethodPost)
	r.Handle("/api/auth/register", authHandler).Methods(http.MethodPost)
	r.HandleFunc("/api/health", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		w.Write([]byte(`{"status":"ok"}`))
	}).Methods(http.MethodGet)

	c := cors.New(cors.Options{
		AllowedOrigins:   []string{"http://localhost:3000"},
		AllowedMethods:   []string{"GET", "POST", "OPTIONS"},
		AllowedHeaders:   []string{"Authorization", "Content-Type"},




const handleLogin = async () => {
  if (!username || !password || !role) {
    alert("Please fill all fields");
    return;
  }

  try {
    const res = await fetch("http://localhost:8080/api/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ username, password, role }),
    });

    if (!res.ok) {
      const err = await res.text();
      throw new Error(err);
    }

    const data = await res.json();
    localStorage.setItem("token", data.token);
    localStorage.setItem("username", data.username);
    localStorage.setItem("role", data.role);

    if (data.role === "Seller") navigate("/seller-dashboard");
    else if (data.role === "Customer") navigate("/product");
    else if (data.role === "Admin") navigate("/admin-dashboard");
  } catch (err) {
    alert("Login failed");
    console.error(err);
  }
};



import React, { useState } from "react";
import { useNavigate } from "react-router-dom";

export default function RegisterPage() {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [role, setRole] = useState("Customer");
  const navigate = useNavigate();

  const handleRegister = async () => {
    if (!username || !password || !role) {
      alert("Please fill all fields");
      return;
    }

    try {
      const res = await fetch("http://localhost:8080/api/auth/register", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username, password, role }),
      });

      if (!res.ok) {
        const err = await res.text();
        throw new Error(err);
      }

      alert("Registration successful! Please log in.");
      navigate("/login");
    } catch (err) {
      alert("Registration failed");
      console.error("Register error:", err);
    }
  };

  return (
    <div className="container mt-5">
      <h2 className="text-center mb-4">EasyShop Register</h2>
      <div className="row justify-content-center">
        <div className="col-md-6">
          <input
            className="form-control mb-3"
            type="text"
            placeholder="Enter username"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
          />
          <input
            className="form-control mb-3"
            type="password"
            placeholder="Enter password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
          />
          <select
            className="form-control mb-4"
            value={role}
            onChange={(e) => setRole(e.target.value)}
          >
            <option>Customer</option>
            <option>Seller</option>
            <option>Admin</option>
          </select>
          <button className="btn btn-primary w-100" onClick={handleRegister}>
            Register
          </button>
        </div>
      </div>
    </div>
  );
}

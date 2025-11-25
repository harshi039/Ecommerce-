package handler

import (
    "encoding/json"
    "net/http"
    "strings"
    "log"
    "time"
    "os"

    "github.com/username/backend/db"
    "github.com/username/backend/model"
    "github.com/golang-jwt/jwt/v5"
    "golang.org/x/crypto/bcrypt"
)

type RegisterRequest struct {
    Username string `json:"username"`
    Password string `json:"password"`
    Role     string `json:"role"`
}

type LoginRequest struct {
    Username string `json:"username"`
    Password string `json:"password"`
}

type LoginResponse struct {
    Token string `json:"token"`
    Role  string `json:"role"`
}

type RegisterResponse struct {
    Message string `json:"message"`
}

func validateRole(role string) bool {
    switch role {
    case "Admin", "Seller", "Customer":
        return true
    default:
        return false
    }
}

func HandleRegister(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    var req RegisterRequest
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

    hashed, err := bcrypt.GenerateFromPassword([]byte(req.Password), 10)
    if err != nil {
        http.Error(w, `{"error":"hashing failed"}`, http.StatusInternalServerError)
        return
    }

    user := model.User{
        Username: req.Username,
        Password: string(hashed),
        Role:     req.Role,
        ImageURL: "", // Prevent NULL scan error
    }

    if err := db.SaveUser(user); err != nil {
        log.Println("Insert error:", err)
        http.Error(w, `{"error":"username exists or insert failed"}`, http.StatusConflict)
        return
    }

    json.NewEncoder(w).Encode(RegisterResponse{Message: "User registered successfully"})
}

func HandleLogin(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    var req LoginRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, `{"error":"invalid json"}`, http.StatusBadRequest)
        return
    }

    user, err := db.FindUserByUsername(req.Username)
    if err != nil {
        http.Error(w, `{"error":"user not found"}`, http.StatusNotFound)
        return
    }

    if err := bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(req.Password)); err != nil {
        http.Error(w, `{"error":"invalid credentials"}`, http.StatusUnauthorized)
        return
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "username": user.Username,
        "role":     user.Role,
        "exp":      time.Now().Add(24 * time.Hour).Unix(),
    })

    secret := os.Getenv("JWT_SECRET")
    tokenString, err := token.SignedString([]byte(secret))
    if err != nil {
        http.Error(w, `{"error":"token generation failed"}`, http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(LoginResponse{Token: tokenString, Role: user.Role})
}

token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
    "username": user.Username,
    "role": user.Role,
    "exp": time.Now().Add(7 * 24 * time.Hour).Unix(), // 7 days
})



func (h *AuthHandler) ValidateToken(w http.ResponseWriter, r *http.Request) {
    tokenStr := r.Header.Get("Authorization")
    claims := &jwt.MapClaims{}
    token, err := jwt.ParseWithClaims(tokenStr, claims, func(token *jwt.Token) (interface{}, error) {
        return h.JWTKey, nil
    })

    if err != nil || !token.Valid {
        http.Error(w, "Invalid token", http.StatusUnauthorized)
        return
    }

    json.NewEncoder(w).Encode(map[string]string{
        "username": (*claims)["username"].(string),
        "role":     (*claims)["role"].(string),
    })
}



useEffect(() => {
  const token = localStorage.getItem("token");
  if (token) {
    fetch("http://localhost:8080/api/auth/validate", {
      method: "GET",
      headers: {
        Authorization: token,
      },
    })
      .then((res) => {
        if (res.ok) {
          return res.json();
        } else {
          throw new Error("Invalid token");
        }
      })
      .then((data) => {
        if (data.role === "admin") {
          navigate("/admin-dashboard");
        } else if (data.role === "seller") {
          navigate("/seller-dashboard");
        } else {
          navigate("/customer-dashboard");
        }
      })
      .catch(() => {
        localStorage.clear();
      });
  }
}, []);



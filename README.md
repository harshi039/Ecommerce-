import API from "../services/api"; // adjust path if needed

const handleLogin = async () => {
  try {
    const res = await API.post("/seller/login", {
      email: username,
      password: password,
    });

    const { token, seller } = res.data;

    localStorage.setItem("sellerToken", token);
    localStorage.setItem("role", "seller");
    localStorage.setItem("username", seller.name);

    navigate("/seller/dashboard");
  } catch (err) {
    alert("Login failed: " + err.response.data.error);
  }
};

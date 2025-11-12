export default function Home() {
  return (
    <div
      style={{
        backgroundImage: `url("/images/banner.jpg")`,
        backgroundSize: "cover",
        backgroundPosition: "center",
        backgroundRepeat: "no-repeat",
        height: "100vh",
        color: "white",
        display: "flex",
        flexDirection: "column",
        justifyContent: "center",
        alignItems: "center",
        textShadow: "1px 1px 4px rgba(0,0,0,0.6)"
      }}
    >
      <h1>New Season Arrivals</h1>
      <p>Check out all the latest trends</p>
    </div>
  );
}

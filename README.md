CREATE DATABASE easyshop;
\c easyshop

CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('Customer','Seller','Admin')),
  created_at TIMESTAMP DEFAULT NOW()
);

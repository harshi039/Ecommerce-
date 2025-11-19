package models

import "time"

type Product struct {
  ID          uint      `gorm:"primaryKey"`
  SellerID    uint      `gorm:"not null"`
  Name        string    `gorm:"size:100;not null"`
  Description string    `gorm:"type:text"`
  Price       float64   `gorm:"type:numeric"`
  ImageURL    string
  Status      string    `gorm:"default:'Pending'"` // Approved, Rejected
  CreatedAt   time.Time
}

package models

import "time"

type Seller struct {
  ID        uint      `gorm:"primaryKey"`
  Name      string    `gorm:"size:100"`
  Email     string    `gorm:"unique;not null"`
  Password  string    `gorm:"not null"`
  CreatedAt time.Time
  Products  []Product `gorm:"foreignKey:SellerID"`
}

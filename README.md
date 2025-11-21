SELECT id, name, description, price, status, image_url
FROM products
WHERE seller = $1
ORDER BY created_at DESC

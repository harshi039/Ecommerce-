ALTER TABLE orders
DROP CONSTRAINT orders_product_id_fkey,
ADD CONSTRAINT orders_product_id_fkey
FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE;

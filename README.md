Here is the **AWS Aurora DB Module – Test Scenarios** written **exactly in the same format/style** as your S3 page (numbered list + short bullet points).

You can copy-paste this directly into Confluence.

---

# Test Scenarios

---

### 1. Database Connectivity Test

* Deploy the sample application and configure the Aurora DB endpoint.
* Verify that the application connects successfully to the database.
* Validate that no connection errors are logged in the application.

---

### 2. Data Insertion Test

* Insert a new user record using the sample application.
* Insert a new order/product record using the sample application.
* Verify that the data is successfully stored in the Aurora database.

---

### 3. Data Retrieval Test

* Retrieve user details from the database using the sample application.
* Retrieve order/product information from the database.
* Validate that the retrieved data matches the original stored data.

---

### 4. Data Update Test

* Update an existing user profile using the sample application.
* Update order status or product details.
* Verify that the updated values are reflected correctly in the database.

---

### 5. Data Deletion Test

* Delete a record using the sample application.
* Verify that the record is removed from the database.
* Confirm that the deleted data is no longer accessible.

---

### 6. Transaction Handling Test

* Perform a multi-step operation (create order + payment + inventory update).
* Validate that all operations succeed together as a single transaction.
* Simulate a failure and verify that all changes are rolled back.

---

### 7. Database Configuration Validation Test

* Verify the Aurora cluster engine type and version.
* Ensure backup retention period is configured correctly.
* Validate that encryption at rest is enabled.
* Verify that the database is deployed in private subnets.

---

### 8. Security and Access Control Test

* Verify that the database is not publicly accessible.
* Attempt to connect using invalid credentials and validate access is denied.
* Ensure only authorized application roles can access the database.

---

### 9. Backup and Restore Test

* Trigger a manual snapshot of the Aurora cluster.
* Restore the snapshot to a new cluster.
* Verify that the restored database contains the correct application data.

---

### 10. Read Replica Test (If Applicable)

* Verify that read replicas are created successfully.
* Perform read operations and validate they are served correctly.
* Simulate primary instance failure and verify automatic failover.

---

### 11. Error Handling Test

* Attempt to query a non-existent table.
* Insert duplicate primary key values.
* Simulate database downtime.
* Verify that the application handles errors gracefully and logs proper messages.

---

### 12. Logging and Monitoring Test

* Verify that CloudWatch metrics for the Aurora cluster are available.
* Validate CPU, memory, connections, and storage metrics.
* Enable slow query logs and verify they are generated correctly.

---

### 13. Performance Test

* Insert and retrieve large volumes of records.
* Measure query execution time.
* Validate that performance meets expected thresholds under load.

---

### 14. High Availability and Failover Test

* Reboot the primary DB instance.
* Verify that automatic failover occurs.
* Confirm the application reconnects automatically.

---

If you want, I can also provide:

✅ Same format for **DynamoDB module**
✅ Same format for **RDS (non-Aurora)**
✅ **Backup module scenarios for Aurora**
✅ Mapping to **Terraform variables & validation points**

Just tell me 👍

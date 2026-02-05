Gotcha 👍
Here are the **scenarios rewritten exactly in the same format/style as shown in your images** (plain Gherkin-style steps, no extra explanation).

---

### **Scenario: Successful backup restoration**

Given the restore service is up and running
And at least one valid backup exists in the system
And set the path to `/api/restore`
And set the method to `POST`
And send the request with method `POST`
Then response status code is `200`
And response body matches `{"status":"success"}`

---

### **Scenario: Restore triggered when no backup exists**

Given the restore service is up and running
And no backup exists in the system
And set the path to `/api/restore`
And set the method to `POST`
And send the request with method `POST`
Then response status code is `400`
And response body contains error message indicating no backup found

---

### **Scenario: Restore API called with unsupported HTTP method**

Given the restore service is up and running
And set the path to `/api/restore`
And set the method to `GET`
And send the request with method `GET`
Then response status code is `405`

---

### **Scenario: Restore API called with request body**

Given the restore service is up and running
And at least one valid backup exists in the system
And set the path to `/api/restore`
And set the method to `POST`
And send the request with method `POST`
Then response status code is `200`
And response body matches `{"status":"success"}`

---

### **Scenario: Multiple restore requests triggered simultaneously**

Given the restore service is up and running
And at least one valid backup exists in the system
And set the path to `/api/restore`
And set the method to `POST`
And send multiple requests with method `POST`
Then restore operation should not fail
And data integrity should be maintained

---

If you want, I can:

* Reduce this to **only 1 applicable TC** (as your teammate hinted)
* Align wording **exactly to company BDD template**
* Mark **which scenarios are optional vs mandatory**

Just say the word 😊

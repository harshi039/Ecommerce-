Feature: CFCS Sample app seller

Background:
  Given start building a new request
  And set the base URL to 'https://dev-app.cocm.awscloud.dev.net'

# ---------------------------
# Positive Scenarios
# ---------------------------

Scenario: Seller adds a product with image successfully
  And set the path to 'api/seller/products'
  And set the method to 'POST'
  Given set form parameter:
    | name        | test |
    | description | test |
    | price       | 900 |
    | seller      | user1765358421085 |
    | image       | src/test/resources/data/headphones.png |
  And send the request with method 'POST'
  Then response status code is '201'
  And response body matches '{"status":"product added"}'

Scenario: Seller views all products successfully
  And set the path to 'api/seller/products?seller=user1765358421085'
  And set the method to 'GET'
  And send the request with method 'GET'
  Then response status code is '200'
  And attach the response body to the report

Scenario: Seller deletes a product successfully
  And set the path to 'api/seller/products/delete/1'
  And set the method to 'DELETE'
  And send the request with method 'DELETE'
  Then response status code is '200'
  And response body matches '{"status":"deleted"}'

# ---------------------------
# Negative Scenarios
# ---------------------------

Scenario: Seller adds a product without image
  And set the path to 'api/seller/products'
  And set the method to 'POST'
  Given set form parameter:
    | name        | test2 |
    | description | test2 |
    | price       | 950 |
    | seller      | user1765358421085 |
  And send the request with method 'POST'
  Then response status code is '400'
  And response body matches '{"error":"image upload failed"}'

Scenario: Seller tries to add a product with missing product name
  And set the path to 'api/seller/products'
  And set the method to 'POST'
  Given set form parameter:
    | description | test |
    | price       | 900 |
    | seller      | user1765358421085 |
    | image       | src/test/resources/data/headphones.png |
  And send the request with method 'POST'
  Then response status code is '400'

Scenario: Seller tries to add a product with invalid price
  And set the path to 'api/seller/products'
  And set the method to 'POST'
  Given set form parameter:
    | name        | test |
    | description | test |
    | price       | -10 |
    | seller      | user1765358421085 |
    | image       | src/test/resources/data/headphones.png |
  And send the request with method 'POST'
  Then response status code is '400'

Scenario: Seller tries to delete non existing product
  And set the path to 'api/seller/products/delete/99999'
  And set the method to 'DELETE'
  And send the request with method 'DELETE'
  Then response status code is '404'

# ---------------------------
# ASG Resilience Scenarios
# ---------------------------

Scenario: Seller adds product during EC2 instance termination
  And set the path to 'api/seller/products'
  And set the method to 'POST'
  Given set form parameter:
    | name        | asgtest |
    | description | scaling |
    | price       | 999 |
    | seller      | user1765358421085 |
    | image       | src/test/resources/data/headphones.png |
  And send the request with method 'POST'
  Then response status code is '201'

Scenario: Seller fetches products during scaling activity
  And set the path to 'api/seller/products?seller=user1765358421085'
  And set the method to 'GET'
  And send the request with method 'GET'
  Then response status code is '200'
  
Feature: ASG Login availability

Background:
  Given start building a new request
  And set the base URL to 'https://dev-app.com.awscloud.dev.net'

Scenario: Login works normally
  And set the path to 'api/auth/login'
  And set the method to 'POST'
  Given set payload to
  """
  {
    "username": "testuser",
    "pwd": "password123",
    "role": "seller"
  }
  """
  And send the request with method 'POST'
  Then response status code is '200'
  And attach the response body to the report

Scenario: Login works when one EC2 instance is terminated
  Given terminate one EC2 instance from ASG
  And set the path to 'api/auth/login'
  And set the method to 'POST'
  Given set payload to
  """
  {
    "username": "testuser",
    "pwd": "password123",
    "role": "seller"
  }
  """
  And send the request with method 'POST'
  Then response status code is '200'


  Feature: ASG Register availability

Background:
  Given start building a new request
  And set the base URL to 'https://dev-app.com.awscloud.dev.net'

Scenario: Register works normally
  And set the path to 'api/auth/register'
  And set the method to 'POST'
  Given set payload to
  """
  {
    "username": "newuser123",
    "pwd": "password123",
    "role": "seller"
  }
  """
  And send the request with method 'POST'
  Then response status code is '200'

Scenario: Register works during scaling activity
  Given terminate one EC2 instance from ASG
  And set the path to 'api/auth/register'
  And set the method to 'POST'
  Given set payload to
  """
  {
    "username": "asguser123",
    "pwd": "password123",
    "role": "seller"
  }
  """
  And send the request with method 'POST'
  Then response status code is '200'
  

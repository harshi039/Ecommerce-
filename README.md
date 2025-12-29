Feature: Login API Automation

  As a user
  I want to login using valid credentials
  So that I can access the application

  Background:
    Given the Login API endpoint is available

  @login @positive
  Scenario: Successful login with valid credentials
    When I send a login request with valid email "validuser@test.com" and password "Valid@123"
    Then the response status code should be 200
    And the authentication token should be present in response

  @login @negative
  Scenario: Login with invalid password
    When I send a login request with valid email "validuser@test.com" and password "WrongPass"
    Then the response status code should be 401
    And the error message should be "Invalid credentials"

  @login @negative
  Scenario: Login with invalid email
    When I send a login request with invalid email "invalid@test.com" and password "Valid@123"
    Then the response status code should be 401
    And the error message should be "Invalid credentials"

  @login @edge
  Scenario: Login with empty email and password
    When I send a login request with empty email and empty password
    Then the response status code should be 400
    And the error message should be "Email and password are required"

  @login @edge
  Scenario: Login with special characters in email
    When I send a login request with email "##@!!" and password "Valid@123"
    Then the response status code should be 400




   Feature: Register API Automation

  As a new user
  I want to register with valid details
  So that I can create an account

  Background:
    Given the Register API endpoint is available

  @register @positive
  Scenario: Successful user registration
    When I send a register request with email "newuser@test.com" and password "Strong@123"
    Then the response status code should be 201
    And the success message should be "User registered successfully"

  @register @negative
  Scenario: Register with existing email
    When I send a register request with email "existing@test.com" and password "Strong@123"
    Then the response status code should be 409
    And the error message should be "User already exists"

  @register @negative
  Scenario: Register with invalid email format
    When I send a register request with email "invalidEmail" and password "Strong@123"
    Then the response status code should be 400
    And the error message should be "Invalid email format"

  @register @negative
  Scenario: Register with weak password
    When I send a register request with email "weakpass@test.com" and password "123"
    Then the response status code should be 400
    And the error message should be "Password does not meet complexity requirements"

  @register @edge
  Scenario: Register with empty fields
    When I send a register request with empty email and empty password
    Then the response status code should be 400
    

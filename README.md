Feature: ASG login resilience

Background:
  Given start building a new request
  And set the base URL to 'https://dev-app.com.awscloud.dev.net'

Scenario: Login works when one instance is terminated
  Given accept content of type 'application/json'
  And set the payload content type as 'application/json'
  And set body to
  """
  {
    "username": "testuser",
    "pwd": "test123",
    "role": "SELLER"
  }
  """
  When one EC2 instance in ASG is terminated
  And post on path '/api/auth/login'
  Then response status code is '200'
  And response content type matches 'application/json'




  

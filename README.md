Feature: ASG Login availability

Background:
  Given start building a new request
  And set the base URL to '${base.url}'

Scenario: Login works when one EC2 instance is terminated
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
  When post on path '/api/auth/login'
  Then response status code is '200'
  And response content type matches 'application/json'
  And attach the response body to the report



  Feature: ASG Register availability

Background:
  Given start building a new request
  And set the base URL to '${base.url}'

Scenario: Register works during scaling activity
  Given accept content of type 'application/json'
  And set the payload content type as 'application/json'
  And set body to
  """
  {
    "username": "newuser123",
    "pwd": "pass123",
    "role": "SELLER"
  }
  """
  When post on path '/api/auth/register'
  Then response status code is '200'
  And attach the response body to the report


  Feature: ASG Seller product operations

Background:
  Given start building a new request
  And set the base URL to '${seller.base.url}'

Scenario: View products during ASG scale out
  Given accept content of type 'application/json'
  When get from path '/api/seller/products?seller=testuser'
  Then response status code is '200'
  And attach the response body to the report

Scenario: Delete product during instance termination
  When delete on path '/api/seller/products/delete/1'
  Then response status code is '200'
  
  
And set the base URL to 'https://dev-app.com.awscloud.dev.net'

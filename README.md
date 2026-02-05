Scenario: Backup listing API called with incorrect path

Given the backup listing service is up and running
And set the path to /api/backup/lists
And set the method to GET
And send the request with method GET
Then response status code is 404


Scenario: Successful backup listing from fixed vault

Given the backup listing service is up and running
And the configured vault exists in the system
And set the path to /api/backup/list
And set the method to GET
And send the request with method GET
Then response status code is 200
And response body contains list of backups

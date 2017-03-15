# cs-load 
Mockable.io is used to create the mock services and the host url is, http://demo8706032.mockable.io/.
Mockable.io only allows 10 urls for a unpaid account. And also it has a quota limit (fair use - throttled) for unpaid users. So we cannot go over 20-30 TPS.
## End Points - Mock URLS
GET http://demo8706032.mockable.io/signin?status=1
GET http://demo8706032.mockable.io/signin?status=0
POST http://demo8706032.mockable.io/signup?uname=testuser1&pkey=abcdef1
POST http://demo8706032.mockable.io/signup?uname=testuser2&pkey=1234567a
POST http://demo8706032.mockable.io/signup?uname=testuser3&pkey=1234abcd
POST http://demo8706032.mockable.io/signup?uname=testuser4&pkey=1234abcdef
POST http://demo8706032.mockable.io/signup?uname=testuser5&pkey=abcdefgh
POST http://demo8706032.mockable.io/signup?uname=testuser6&pkey=12345678
POST http://demo8706032.mockable.io/signup?uname=testuser7&pkey=abc123
POST http://demo8706032.mockable.io/signup?uname=user8&pkey=abc123
### Sign In -  GET
1. */signin?status=1* - 200 OK
..Response Message
```json
{
  "msg": "Hello User!!!!!",
  "status": "success"
}
```
2. */signin?status=0* - 401 Unauthorized
..Response Code - 401
..Response Message
```json
{
  "msg": "Invalid User",
  "status": "fail"
}
```

### Sign Up - POST
3. *signup?uname=testuser1&pkey=abcdef1* - 200 OK
4. *signup?uname=testuser2&pkey=1234567a* - 200 OK
5. *signup?uname=testuser3&pkey=1234abcd* - 200 OK
6. *signup?uname=testuser4&pkey=1234abcdef* - 200 OK
..Response Message
```json
{
  "username": "<username>",
  "status": "success",
  "msg": "Sign Up Un-successful"
}
```
7. *signup?uname=testuser5&pkey=abcdefh* - 400 Bad Request
8. *signup?uname=testuser6&pkey=12345678* - 400 Bad Request
9. *signup?uname=testuser7&pkey=abc123* - 400 Bad Request
10. *signup?uname=user8&pkey=abc123* - 400 Bad Request
..Response Message
```json
{
  "username": "<username>",
  "status": "failed",
  "msg": "Sign Up Un-Successful"
}
```

## Testing Plan (Load Test of the Mock Services using JMeter) 

####  Load of GET (i.e /signin) using Stepping Thread Group
This test adds 10 users every 15 seconds until reaching 100 users. Each step takes 15 seconds to complete and JMeter waits 15 seconds before starting the next step. 

After reaching 100 threads all of them will continue running and hitting the server together for 1 minute.

Using a Random Variable (status_code for 0 & 1 values), we append it to the query parameter status=${status_code} dynamically.

Using a JSR223 Assertion, we assert 200 response for status=1 & 401 for status = 0. Fail otherwise.

#### Load of POST (i.e /signup) using Stepping Thread Group
Creating a Thread Group for X Users & Ramp-Up period of Y seconds. Add assertions, for the respective Response Codes.

The Sampler for these handles the path, *signup?uname=${username}&pkey=${passkey}*, these variables are passed from a CSV Data Set Config file, which would look like the below table, 

Username | PassKey | Expected Response Code
-------- | ------- | ----------------------
testuser1	|	abcdef1	|	200
testuser2	|	1234567a	|	200
testuser3	|	1234abcd	|	200
testuser4	|	1234abcdef	|	200
testuser5	|	abcdefgh	|	400
testuser6	|	12345678	|	400
testuser7	|	abc123	|	400
user8	|	abc123	|	400

Using the data in the CSV file, the request is made and its response is asserted.

This test adds 5 users every 20 seconds until reaching 50 users. Each step takes 20 seconds to complete and JMeter waits 20 seconds before starting the next step.  

After reaching 100 threads all of them will continue running and hitting the server together for 1 minute. (This configuration is same as above, but each thread would here iterate over the all the rows of CSV config file, we are reducing it.)


#### Load of GET (i.e /signin) with Random Uniform Timers
In this we would create a Thread Group of 100 Threads with rampup of 10 seconds. 

Add a Uniform Random Timer, we set a Random Max Delay of 500ms and Constant Delay Offset of 200ms. And Assert if the requests are coming with antcipated responses.

#### Load of GET (i.e /signin) with Throughput Shapping Timer
In this we would create a Thread Group of 100 Threads with rampup of 10 seconds. 

Instead of Using a Uniform Random Timer, we use a Throughput Shaping Timer here and try to check how the services behave, for a troughput of 10RPS, 15RPS, 20RPS.

Using the Listeners we will test the load of these services.



## Results - 16-03-2016 3:35 AM
### GET - SignIn
Average Response Time - 28826ms (28.8s)

### POST - SignUp
Average Response Time - 21550ms (21.5s)

### GET - SignIn - Random Timer
Average Response Time - 28700 (22.7s)

### GET - SignIn - Throughtput Shapping Timer
Average Response Time - 21744 (21.7s)

### Over All
Average - 25444 (25.4s)
Troughput - 106.7/min



###  API does work with high load without failing or in the alternative, percentile based scores for response times?
In any of the above cases, average response time near to 20s+ is not appreciated from an api. All the Thread Group's Samplers have less than 30 percentile of samples, with average response time less than 6s.

### API fails with very high load test?
It is evident that API is not failing even at high load test. It is coming back with either the anticipated response or the Quota Limit message, which is again being implemented by mockable, which means, the mock is working well even at a higher load. But the response times are very slow as stated above.

### Summary Table
Label	|	# Samples	|	Average	|	Min	|	Max	|	Std. Dev.	|	Error %	|	Throughput	|	Received KB/sec	|	Sent KB/sec	|	Avg. Bytes
-------	|	-------	|	-------	|	----|	----|	--------	|	-------	|	----------	|	--------------	|	----------	|	----------
SignIn	|	492	|	29685	|	321	|	56693	|	23372.91	|	0.90041	|	2.00831	|	1.65	|	0.28	|	841.1
Mock - SignUp - POST	|	503	|	20078	|	320	|	57105	|	21149.92	|	0.81909	|	1.49981	|	1.17	|	0.34	|	800
SignIn - Timers - Random	|	100	|	20193	|	343	|	55361	|	17086.53	|	1	|	1.67305	|	1.65	|	0.31	|	1007
SignIn - Timers - Troughput	|	100	|	49790	|	364	|	56547	|	14091.31	|	1	|	0.90279	|	0.9	|	0.17	|	1025.7
TOTAL	|	1195	|	26529	|	320	|	57105	|	22902.09	|	0.88285	|	1.71527	|	1.43	|	0.31	|	853.1

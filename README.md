 
  val httpProtocol = http
    .baseUrl("https://jsonplaceholder.typicode.com") // Base URL for the API
    .acceptHeader("application/json")
    .contentTypeHeader("application/json")

 
  val scn = scenario("Basic GET Request Simulation")
    .exec(http("Get All Posts")
      .get("/posts") // GET request to /posts
      .check(status.is(200)) // Check if the response status is 200
      .check(jsonPath("$[0].id").exists)) // Check if the first post has an 'id' field


  setUp(
    scn.inject(
      atOnceUsers(10) // Simulate 10 users hitting the API at once
    )
  ).protocols(httpProtocol)

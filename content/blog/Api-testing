# Testing API in microservices era

With the advent of microservices and distributed scalable systems REST API seems to be the most popular choice for enabling communication between the services that comprise the application.

Surely, REST has proven to be valuable and relatively easy to implement because it relies on HTTP as a transport layer, its status codes and action verbs (GET, POST, DELETE) for indicating the status of operation calls.

# Challenges

Implementing the API is one thing, how does one go about testing?

With microservices, you are not bound to a single technology stack.
Multiple teams working with their own technologies to implement APIs to communicate with other services, which means systematic tests have to be solved on the higher level.

So, what's the de-facto standard for testing the APIs in the microservices era?

# Specification

To make sure that your services play nice with each other speccing out your APIs is a good start.

You can start with manually describing what data structures the API consumes and produces.
Having the specification has another benefit - it's trivial enough to generate client and server code for different languages.
Specification also allows you to generate mocks for testing your services.


# Here's how we go about testing our microservices, step 1

Dredd allows you to test the API endpoint against the specification that you wrote before.
Once integrated into the CI pipeline, it means that changes to the microservices won't break the client code  because it will always be validate against previously agreed specification.

# Step 2: mocks

 - Something that allows you to mock out the api with some test data... ?

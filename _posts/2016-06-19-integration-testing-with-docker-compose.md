---
layout: post
title: Integration Testing With Docker Compose
tags:
- Docker
- Testing
---

Integration testing is often a difficult venture, especially when it comes to distributed systems. Even if you're building a monolithic app you probably need to spin up a database to do integration testing. It's also the kind of thing that's simple to do early on, but gets exponentially harder as the codebase expands. Thankfully [Docker Compose](https://docs.docker.com/compose/) gives us the ability to do integration testing in any environment that runs docker.

#### Getting Started

Let's say you're staring with a monolithic setup, you've got one server and one database. Maybe you build the app server and database from source like it's 1999, or maybe you use brew install to get all of your dependencies resolved. But at the end of the day your system looks like this.

<img src="/images/posts/end-to-end-testing-docker/monolith.png" width="40%">

The endpoint you'd like to test is `/create` and all it should do is store a some data in the database. Seems simple enough. So you write a bash script that [CURL](https://en.wikipedia.org/wiki/CURL)s and endpoint, and then queries the database (exit 0 for OK, exit 1 for FAIL). It's easy AND most importantly it *works*.

{% highlight bash linenos=table %}
curl http://localhost:8000/create
COUNT = `mysql --user="$user" --password="$password" --database="$database" --execute="SELECT COUNT(*) FROM table_name;"`
if [[ $COUNT -ne 1 ]]; then
  exit 1
fi
{% endhighlight %}


But there's a lot of hidden dependencies that make this extremely inconsistent.

- Database has to be installed and running
- Monolithic app framework has to be installed
- Monolithic app has to be running
- You need a system that has CURL on the path
- Depending on your test, whatever data that's sitting in the database could cause false positives or negatives

Let's say you add a line to your bash script to reset your data.

{% highlight bash linenos=table %}
mysql --user="$user" --password="$password" --database="$database" --execute="TRUNCATE table table_name"
curl http://localhost:8000/create
COUNT = `mysql --user="$user" --password="$password" --database="$database" --execute="SELECT COUNT(*) FROM table_name;"`
if [[ $COUNT -ne 1 ]]; then
  exit 1
fi
{% endhighlight %}

This eliminates the last hidden dependency (existing database data), but also introduces a pretty nasty side effect. It's only a side effect because the local **development** database is shared with the **test** database. So every time you run your integration test, you lose all of your development data 😭. This may seem obvious, but in practice this setup still exists. But it doesn't have to be this way. From here on out, I'll walk through an example built on top of Docker Compose that addresses all of the issues listed above. For this example I'll use Node for the app framework and RethinkDB for the database, but there's no reason why you couldn't choose another stack.

#### Devise A Strategy

Let's take a page from [Martin Fowler's microservice testing playbook](http://martinfowler.com/articles/microservice-testing/#testing-end-to-end-introduction) for integration testing. We're going to spin up a container outside of the system under test, have the container run some tests, and then check the exit code of the testing container's run command.

<img src="/images/posts/end-to-end-testing-docker/compose.png" width="80%">

For clarity I'd like to point out the file structure since we're going to have multiple `Dockerfile` in the same project.

{% highlight text %}
integration-test/
  Dockerfile
  index.js
  package.json
  test.sh
  docker-compose.yml
index.js
package.json
Dockerfile
{% endhighlight %}

Let's walk through each component of the integration-test.

#### Ephemeral Database

Sometimes it's nice to *lose all your data*, and when you're running tests it's essential. It's really easy to accomplish this with Docker compose by spinning up your database without a mounted volume for data. This means that when you destroy your container, the data goes along with it. It also means that if you *don't* destroy your container, you can [exec](https://docs.docker.com/engine/reference/commandline/exec/) into it and run queries against the database to debug. Here's an example Docker Compose file that would just spin up an ephemeral database (RethinkDB).

**integration-test/docker-compose.yml**

{% highlight YAML linenos=table %}
version: '2'

services:
  rethinkdb:
    image: rethinkdb
    expose:
      - "28015"
{% endhighlight %}

Keep this concept in mind, because we're going to use it soon.

#### Application Container

The next step is to containerize the application you'd like to test. It needs to build/run the application, link to the database and expose a port to be used for testing.

**Dockerfile**

{% highlight Dockerfile linenos=table %}
FROM mhart/alpine-node
WORKDIR /service
COPY package.json .
RUN npm install
COPY index.js .
{% endhighlight %}


**integration-test/docker-compose.yml**

{% highlight YAML linenos=table %}
version: '2'

services:
  my-service:
    build: ..
    command: npm start
    links:
      - rethinkdb
    ports:
      - "8080:8080"
  rethinkdb:
    image: rethinkdb
    expose:
      - "28015"
{% endhighlight %}

At this point you could sanity check the services with [docker-compose up](https://docs.docker.com/compose/reference/up/) and go to http://localhost:8080 (so long as you had a server and routes wired up).


#### Integration Test Container

Now we've got our database and application, let's build the testing container. This container needs to POST against the `/create` endpoint on `my-service` and inspect the database for changes. To accomplish this I used [tape](https://www.npmjs.com/package/tape) and [request-promise](https://github.com/request/request-promise) to inspect the endpoint.

**integration-test/index.js**

{% highlight javascript linenos=table %}
import test from 'tape';
import requestPromise from 'request-promise';

const before = test;
const after = test;

const beforeEach = () => {/*test setup*/};

const afterEach = () => {/*test cleanup*/};

before('before', (t) => {/*one time setup*/});

test('POST /create', (t) => {
  beforeEach()
    .then(() => (
      requestPromise({
        method: 'POST',
        uri: 'http://my-service:8080/create', // yes! we can use the service name in the docker-compose.yml file
        body: {
          thing: 'this thing',
        },
      })
    ))
    .then((response) => {
      // inspect the response
      t.equal(response.statusCode, 200, 'statusCode: 200');
    })
    .then(() => (
      // inspect the database
      rethinkdb.table('table_name')
        .filter({
          thing: 'this thing',
        })
        .count()
        .run(connection)
        .then((value) => {
          t.equal(value, 1, 'have data');
        })
    ))
    .catch((error) => t.fail(error))
    .then(() => afterEach())
    .then(() => t.end());
});

after('after', (t) => {/*one time setup*/});
{% endhighlight %}

The test `Dockerfile` looks about the same as the app Dockerfile.

**integration-test/Dockerfile**

{% highlight Dockerfile linenos=table %}
FROM mhart/alpine-node
WORKDIR /integration
COPY package.json .
RUN npm install
COPY index.js .
{% endhighlight %}

Now we add the test app to the docker-compose.yml file.

**integration-test/docker-compose.yml**

{% highlight YAML linenos=table %}
version: '2'

services:
  integration-tester:
    build: .
    links:
      - my-service
  my-service:
    build: ..
    command: npm start
    links:
      - rethinkdb
    ports:
      - "8080:8080"
  rethinkdb:
    image: rethinkdb
    expose:
      - "28015"
{% endhighlight %}

So here's the cool part, when you run `docker-compose up` a few things happen

- my-service and integration-tester containers are built
- my-service, integration-tester and rethinkdb are linked and ran
- integration-tester runs all tests until it stops
- after integration-tester stops, docker-compose spins down all containers

This is exactly what we need to run integration testing in CI. We still haven't inspected the exit code of the integration-tester container, but I'll get to that soon.

#### Bringing It All Together

With all of the automation in place we need to tie everything together and do some cleanup after the test finishes. To accomplish this we can use [Docker wait](https://docs.docker.com/engine/reference/commandline/wait/) to block the script and retrieve the exit code of the test. We'll use that code to output a message (PASS/FAIL) and exit the master script with the same exit code. This is useful because most (if not all) CI environments use an exit code to determine if the tests passed or failed. We'll also grab the test container logs and print them out to provide context for when things fail. Here's an (extremely verbose) script that does everything we need to run our integration tests locally or in CI.

**integration-test/test.sh**

{% highlight bash linenos=table %}
# define some colors to use for output
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'
# kill and remove any running containers
cleanup () {
  docker-compose -p ci kill
  docker-compose -p ci rm -f --all
}
# catch unexpected failures, do cleanup and output an error message
trap 'cleanup ; printf "${RED}Tests Failed For Unexpected Reasons${NC}\n"' HUP INT QUIT PIPE TERM
# build and run the composed services
docker-compose -p ci build && docker-compose -p ci up -d
if [ $? -ne 0 ] ; then
  printf "${RED}Docker Compose Failed${NC}\n"
  exit -1
fi
# wait for the test service to complete and grab the exit code
TEST_EXIT_CODE=`docker wait ci_integration-tester_1`
# output the logs for the test (for clarity)
docker logs ci_integration-tester_1
# inspect the output of the test and display respective message
if [ -z ${TEST_EXIT_CODE+x} ] || [ "$TEST_EXIT_CODE" -ne 0 ] ; then
  printf "${RED}Tests Failed${NC} - Exit Code: $TEST_EXIT_CODE\n"
else
  printf "${GREEN}Tests Passed${NC}\n"
fi
# call the cleanup fuction
cleanup
# exit the script with the same code as the test service code
exit $TEST_EXIT_CODE
{% endhighlight %}

#### Examples

For a complete example take a look at [auth-service](https://travis-ci.org/hharnisc/auth-service). All you need to do to see it in action:

{% highlight bash linenos=table %}
git clone https://github.com/hharnisc/auth-service.git
cd auth-service
npm test
{% endhighlight %}

For more complicated example (multiple layers of microservices) take a look at [login-service](https://github.com/hharnisc/login-service).

{% highlight bash linenos=table %}
git clone https://github.com/hharnisc/login-service.git
cd login-service
npm test
{% endhighlight %}

#### Use This Now (Yeoman Generator)

Here's a Yeoman generator to start building a new service that's already got some integration tests in place:

[https://github.com/hharnisc/generator-service-native-docker](https://github.com/hharnisc/generator-service-native-docker)

It brings together Docker, Node (ES6) and Travis CI. Please let me know if you find any weirdness, I love pull requests!

#### Conclusion

This approach has been working well in practice and I've been using it to do integration testing for a handful of microservices. Any time I had a failure in CI, sure enough the same bug would occur locally. The biggest issue I ran into was tests failing because the application wasn't fully up. To fix this I implemented a `/health` api endpoint [on the app](https://github.com/hharnisc/generator-service-native-docker/blob/master/app/templates/service/src/healthRouter.js#L4) and added a retry inside of the `before` block [of the test](https://github.com/hharnisc/generator-service-native-docker/blob/master/app/templates/service/integration-test/index.js#L31). Since I fixed that issue I've had no other weirdness and have been using this to [run integration tests in CI](https://travis-ci.org/hharnisc/auth-service#L1990-L2027). This has been really useful and has caught some real bugs that would have probably surfaced during deployment, I hope you find it useful too!

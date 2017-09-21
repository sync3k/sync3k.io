# Getting Started with Sync3k

In this short guide, we show how to run the Sync3k server with all its dependencies, and put a Redux application on Sync3k application.

## Server

Sync3k depends on Kafka, which in turn depends on Zookeeper. All of these services can be bootstrapped with Docker Compose. If you haven't already, [install Docker](https://docs.docker.com/engine/installation/) as per the instruction for the OS of your choice. For Docker Compose, if not available as a [separate package for your OS](https://docs.docker.com/compose/install/), it can also be installed with PIP:

```bash
$ pip install docker-compose
```

Now, with Docker Compose installed, now we are ready to pull containers and run them. Here is the basic one instance setup. Save it as `docker-compose.yml`

```yaml
version: '2'
services:
  zookeeper:
    image: 'wurstmeister/zookeeper'
    expose:
      - "2181"
  kafka:
    image: 'wurstmeister/kafka'
    environment:
      KAFKA_ADVERTISED_HOST_NAME: kafka
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    expose:
      - "9092"
  sync3k:
    image: 'sync3k/sync3k-server'
    ports:
      - "8080:8080"
    environment:
      kafkaServer: 'kafka:9092'
```

Then, use `docker-compose` command line tool:

```bash
$ docker-compose up -d
```

## Client

In this guide we demonstrate how the [`todos` Redux example app](https://github.com/reactjs/redux/tree/master/examples/todos) can be turned into a Sync3k application.

First, clone redux repository to have `todos` example app checked out.

```bash
~$ git clone https://github.com/reactjs/redux
~$ cd redux/examples/todos
~/redux/examples/todos$ 
```

Now, we need to install dependencies and `sync3k-client` npm package:

```bash
~/redux/examples/todos$ npm install
~/redux/examples/todos$ npm install --save sync3k-client
```

Next, modify `src/index.js` to bootstrap Sync3k:

```diff
 import { Provider } from 'react-redux'
 import App from './components/App'
 import reducer from './reducers'
+import { enhancer, actions } from 'sync3k-client';

-const store = createStore(reducer)
+const store = createStore(reducer, enhancer)
+store.dispatch(actions.initializeSync(`ws://${window.location.hostname}:8080/kafka`, 'myTodos'))
```

Finally, instead of monotonic increasing number, we need to assign a globally unique id for the posts. The simplest way of doing it is to randomize the Ids. Note that in a more serious application, it is recommendable to use Id schemes such as UUIDv4.

Modify `src/actions/index.js`:

```diff
-let nextTodoId = 0
 export const addTodo = (text) => ({
   type: 'ADD_TODO',
-  id: nextTodoId++,
+  id: Math.floor(Math.random() * 9007199254740991),
   text
 })
```

We are ready to build the example client code now.

```bash
~/redux/examples/todos$ npm start
```

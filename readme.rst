Protobuf Kafka Messaging
========================

Protobuf
********

code-block addprotobuf::

    poetry add probuf

serialization types  :-  json, yml, protobuf
Portobuf (Language-agnostic) :- use to serialize data (binary format, fast)
{ Language-agnostic :- language-neutral, language-independent, or cross-language }
protobuf-compiler \ :-  bring the file and convert file into python

command use::

    docker compose ps      (list of containers)
    docker exec -it <cont-name> /bin/bash    (inside container)
    cd app        ( inside the folder)
    docker compose up --build  (rebuild )
    docker logs <container_name/id> -f      (-f interactive mode)

    docker volume prune -a  ( remove inactive volumes )
    docker container prune (remove inactive containers)
    docker image prune  (remove inactive images )

    docker rmi  <image_name/id> (remove  images )
    docker rm  <container_name/id> (remove container )
    docker volume rm <volume_name>

    docker compose down
    1. Stop all containers
    2. Remove all containers
    3. Remove all network



Step 1 :- Define the Schema
***************************
Create a file_named.proto with the following content:

.. code-block:: python
    syntax = "proto3";

    message Todo {
    int32 id = 1;
    string content = 2;
    }


Step 2: Generate Python Code
****************************
Use the protoc compiler to generate Python code from the .proto file.

.. code-block:: python

    protoc --version ( check protoc installed)

    protoc --python_out=. todo.proto 

    protoc —languageName_out=. fileName.proto  (dot:- current location)

This command generates a person_pb2.py file in the current directory.    
(Convert file into python & generate todo_pb2.py file )


Step 3: Use the Generated Code in Python
****************************************

AIOKafkaProducer Protobuf Todo
******************************

.. code-block:: python

    # note :- convert data into probuf and add into topic (1st objective)
    @app.post("/api_todos/", response_model=Todo)
    async def create_todo(todo: Todo, session: Annotated[Session, Depends(get_session)], producer: Annotated[AIOKafkaProducer, Depends(get_kafka_producer)]) -> Todo:
        todo_protbuf = todo_pb2.Todo(id=todo.id, content=todo.content)
        print(f"Todo Protobuf:   {todo_protbuf}")
        # Serialize the message to a byte string
        serialized_todo = todo_protbuf.SerializeToString()
        print(f"Serialized data:   {serialized_todo}")
        # Produce message
        await producer.send_and_wait("todos2", serialized_todo)
        session.add(todo)
        session.commit()
        session.refresh(todo)
        return todo  


AIOKafkaConsumer Protobuf Todo
******************************

.. code-block:: python

    async def consume_messages(topic, bootstrap_servers):
    # Create a consumer instance.
    consumer = AIOKafkaConsumer(
        topic,
        bootstrap_servers=bootstrap_servers,
        group_id="my-group",
        auto_offset_reset='earliest'
    )

    # Start the consumer.
    await consumer.start()
    try:
        # Continuously listen for messages.
        async for message in consumer:
            # print the consumer data
            print(f"\n\n Consumer Raw message Vaue: {message.value}")
        
            # initialize to do class (Todo class is empty)
            new_todo = todo_pb2.Todo()
            # ParseFromString :- provide protobuf automaically
            new_todo.ParseFromString(message.value)
            print(f"\n\n Consumer Deserialized data: {new_todo}")
        # Here you can add code to process each message.
        # Example: parse the message, store it in a database, etc.
    finally:
        # Ensure to close the consumer when done.
        await consumer.stop()

1. Import the Generated Code
*****************************
.. code-block:: python

    import person_pb2

2. Create a New Todo Message
******************************
.. code-block:: python

    id = 
    content =

This creates a new Todo message and sets its fields.

3. Serialize the Message
************************
.. code-block:: python

       # Serialize the message to a byte string
        serialized_todo = todo_protbuf.SerializeToString()
        print(f"Serialized data:   {serialized_todo}")

This serializes the Todo message to a byte string.

4. Deserialize the Byte String
******************************
.. code-block:: python

            # initialize to do class (Todo class is empty)
            new_todo = todo_pb2.Todo()
            # ParseFromString :- provide protobuf automaically
            new_todo.ParseFromString(message.value)
            print(f"\n\n Consumer Deserialized data: {new_todo}")

This deserializes the byte string back into a Todo message and prints the field values.       


Advantages of Protobuf
**********************
1- Compact and Efficient: Protobuf is more efficient than XML and JSON in terms of both size and speed.

2- Strongly Typed: The generated code is strongly typed, which helps catch errors at compile-time rather than runtime.

3- Backward and Forward Compatibility: Protobuf supports adding new fields and deprecating old fields without breaking existing code.

https://blog.bytebytego.com/p/cloudflares-trillion-message-kafka 

https://blog.bytebytego.com/p/cloudflares-trillion-message-kafka

https://protobuf.dev/ 

https://protobuf.dev/getting-started/pythontutorial/ 

https://github.com/panaverse/learn-generative-ai/blob/main/05_microservices_all_in_one_platform/15_event_driven/03_protobuf/protobuf-guide.md


Volume (compose.yaml)
*********************
1- Bind Mount Volume (BMV): A bind mount volume is a directory on the host machine that is mounted into a container.
-./host-machine:/container

volumes:
      - ./todo:/code  # Sync local development directory with the container


2- Persistent Volume (PV): A persistent volume is a resource that is provisioned and managed by Kubernetes. It is used to store data that needs to be preserved even if a Kafka container is deleted or recreated.
In both cases, the data is stored outside of the container, so it is not lost when the container is deleted or recreated

Bind Mount Volume (BMV): a directory on the host machine that is mounted into a container.

Persistent Volume (PV): a resource that is provisioned and managed by Kubernetes.

Links
******

Protobuf
********

[GenAI Quarter 5 Online Class 13: Serialization and Deserialization Kafka Messages](https://www.youtube.com/watch?v=qVbAYHxW3xg)

Kafka
*****
[GenAI Quarter 5 Online Class 12: Interact with Kafka using aiokafka - A Python Library for Kafka](https://www.youtube.com/watch?v=PAU05OrLgho)


[event_driven Github](https://github.com/panaverse/learn-generative-ai/tree/main/05_microservices_all_in_one_platform/15_event_driven)





  

        






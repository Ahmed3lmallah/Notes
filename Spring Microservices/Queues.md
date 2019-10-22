# Queues 

**Asynchronous vs. synchronous processing:** 

Synchronous processing means that you can only execute one process at a time, while Asynchronous processing means multiple process can be executed at a time and you don't have to finish executing the current process in order to move on to next one.

**What is a Queue?**

**Queue** is a **linear data structure, or an ordered list of elements of similar data types**. in which the first element is inserted from one end called the **REAR (also called tail)**, and the removal of existing element takes place from the other end called as **FRONT (also called head)**. This makes queue as **FIFO (First in First Out) data structure**, which means that element inserted first will be removed first. The process to add an element into queue is called Enqueue and the process of removal of an element from queue is called Dequeue.

**What problems do Queues solve?**

1. Queues enable asynchronous communication, which means that the endpoints that are producing and consuming messages interact with the queue, not each other. Producers can add requests to the queue without waiting for them to be processed. Consumers process messages only when they are available. No component in the system is ever stalled waiting for another, optimizing data flow.

1. Queues make your service reliable, and reduce the errors that happen when different parts of your system go offline. By separating different components with message queues, you create more fault tolerance. If one part of the system is ever unreachable, the other can still continue to interact with the queue. The queue itself can also be mirrored for even more availability.

**How Does a Queue Work?**

1. Producer creates a new entry/message.
	* Entry/message includes the exchange name and a binding key (routing key).
1. Entry/message is sent to a message broker such as RabbitMQ.
1. Entry/message is routed to the appropriate queue(s) based on the binding key and distribution protocols.
1. Consumer listening to the Queue receives the message when available.

## Terminology

It's helpful to review queue-related terminlogy before we look at the design and begin implementation.

### AMQP

Advanced Message Queuing Protocol (**AMQP**) is a messaging protocol that allows clients and messaging middleware to communicate in a standardized manner. RabbitMQ and the Spring client libraries conform to AMQP.

### Producer

The **producer** is the application, process, or code that creates new entries and places them in the queue for processing.

### Consumer

The **consumer** is the application, process, or code that processes, or consumes, the queue entries created by the producer.

### Queue

A **queue** is a repository for messages. Producers place the messages in the queue. Consumers process the messages.

### Exchange

An **exchange** is where producers send messages in an AMQP system. An exchange then routes the messages to one or more queues based on the type of exchange and routing rules (called bindings, explained below).

**Topic exchanges** route messages to one or more queues based on a routing key and the pattern used to bind the exchange to the queue or queues. Topic exchanges allow consumers to choose which types of messages they want to process and which ones they want to ignore.

### Binding

A **binding** is a rule that an exchange uses to route messages to a queue. Routing keys and binding rules can be used to filter messages and only send certain messages to certain queues.

**Additional Resources:**

* [Message Queues - AWS](https://aws.amazon.com/message-queue/)

## Tutorial: [Spring RabbitMQ Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/spring-rabbitmq-tutorial.md)

## Tutorial Summary:

1. Install RabbitMQ
1. Creare the Queue consumer (an independent service)
	1. Create the Consumer Application using the Spring Initializr with `Spring for RabbitMQ` dependency.
	1. Create the Message Class in `Queueconsumer.util.messages.msg.java`
		
		*Include getters, setters, custom constructors (for convenience), and toString methods to that class. If we decide to add a custom constructor, we must also include the default constructor. Jackson requires a default constructor to marshal and unmarshal the messages.*
		
	1. Add the Jackson Converter Libraries to `pom.xml` file:

			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-core</artifactId>
				<version>2.9.8</version>
			</dependency>
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-annotations</artifactId>
				<version>2.9.8</version>
			</dependency>
			<dependency>
				<groupId>com.fasterxml.jackson.core</groupId>
				<artifactId>jackson-databind</artifactId>
				<version>2.9.8</version>
			</dependency>

	1. Configure the Exchange, Queue, Binding, and Converter in the `main application` class:

				public static final String TOPIC_EXCHANGE_NAME = "queue-demo-exchange";
				public static final String QUEUE_NAME = "email-list-add-queue";
				public static final String ROUTING_KEY = "email.list.add.#";

				@Bean
				Queue queue() {
					return new Queue(QUEUE_NAME, false);
				}

				@Bean
				TopicExchange exchange() {
					return new TopicExchange(TOPIC_EXCHANGE_NAME);
				}

				@Bean
				Binding binding(Queue queue, TopicExchange exchange) {
					return BindingBuilder.bind(queue).to(exchange).with(ROUTING_KEY);
				}

				@Bean
				public Jackson2JsonMessageConverter jackson2JsonMessageConverter() {
					return new Jackson2JsonMessageConverter();
				}

	1. Create the Message Listener: `Queueconsumer.MessageListener.java`

			@Service
			public class MessageListener {

				@RabbitListener(queues = EmailListQueueConsumerApplication.QUEUE_NAME)
				public void receiveMessage(EmailListEntry msg) {
					System.out.println(msg.toString());
				}
			}
			
1. Configure the producer service to use the queue:
	1. Add the `Spring for RabbitMQ` dependency to the Producer Application.

			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-amqp</artifactId>
			</dependency>
			
	1. Create the Message Class - identical to the message class we created in the consumer application above. 
	1. Configure the `RabbitTemplate` and Message Converter in the `main application` class:

			@Bean
			public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
				RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
				rabbitTemplate.setMessageConverter(jackson2JsonMessageConverter());
				return rabbitTemplate;
			}

			@Bean
			public Jackson2JsonMessageConverter jackson2JsonMessageConverter() {
				return new Jackson2JsonMessageConverter();
			}

	1. Use `RabbitTemplate` to convert and send messages to the queue inside any method:

			public void sendMessage(Msg msg) {
				System.out.println("Sending message...");
				rabbitTemplate.convertAndSend(EXCHANGE, ROUTING_KEY, msg);
				System.out.println("Message Sent");
			}

**How to use RabbitMQ GUI?** 

1. Go to root directory. Default: C:\Program Files\RabbitMQ Server\rabbitmq_server-3.7.16\sbin>
1. Use RabbitMQ CMD
1. `rabbitmq-plugins enable rabbitmq_management`
1. Restart the rappitmq server
1. visit localhost:15672/
1. Login with username: `guest` and Password: `guest`

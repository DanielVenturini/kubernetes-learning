# From Monolith to Microservices

The cloud is a technology place where the new applications will be hosted. A lot of enterprises are moving their legacy apps from their physical machines to store them into a cloud system. However, not all applications fit in the cloud. Applications that have a huge amount of lines of code, a high complex and dependent behaviors among the pieces of the app may not be hosted in a cloud system.

**Monolith** systems is an app built in a single piece of software which grows continuously and contains all of its components running together in a single process, demanding a expensive taste of hardware. The scaling process of a monolith application may be hard and not compensable. An update in a monolith system can break down all the system if it fails.

**Microservices** is a special kind of application that basically has all of its component separately and all of them can communicate. Microservices allow a more concise code development, a better way to test, deploy and, if one of the microservices break down, the entire system continues working. Microservices also need a fewer computing resources to work. All of these small independent process (microservices) communicate with each other through APIs over a network. Microservices are very flexible once they can be written a different programming languages according the need and still exchange information among them. The deployment process are really easy to be done and if an error happen, the others services will not be affected.

![microservice-monolith](https://docs.oracle.com/fr/solutions/learn-architect-microservice/img/monolithic_vs_microservice.png)

When converting from monolith to a microservices, there is not alternative way: refactoring. The code refactoring is the process to split the monolith app into several microservices and make them communicable. This may spend a lot of time from the team. However, not all monolith apps should be refactored into microservices. Apps built in Assembly or Cobol probably will not work well in microservices.

There are a lot of [success stories](https://kubernetes.io/case-studies/) about changing apps from monolith to microservices.

I should now be able to:

- Explain what a monolith is.
- Discuss the monolith's challenges in the cloud.
- Explain the concept of microservices.
- Discuss microservices advantages in the cloud.
- Describe the transformation path from a monolith to microservices.
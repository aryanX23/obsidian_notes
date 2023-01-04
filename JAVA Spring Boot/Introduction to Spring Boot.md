# Q. What is Spring Boot?
Spring boot is an amazing java framework for building backend applications. It is generally used in making full stack applications using java and kotlin. Spring boot is widely preferred as a backend in windows based softwares.
![[Pasted image 20220831121346.png]]


![[Pasted image 20220831121443.png]]

>Metrics: Spring Boot provides **in-built** function to check how our application is behaving and to measure its performance.

# Different Features of Spring Framework:
1. Spring practices **Inversion of Control** which means that the control is not in the hands of the user rather the framework controls the processes such as object creation, garbage collection , etc.
2. Spring allows the **Dependency Injection** which means that if, one class needs the object of another class spring provides those objects automatically without user interaction therefore making user work easier.
3. Spring provides us the **Aspect oriented programming** which means tackling down the cross cutting concerns, eg. logging the user data, logging the server requests, etc.
4. Spring provides us the ability to create **Web Applications**. There is a spring project called *spring mvc* that allows us to create web applications.
5. Spring provides us the **Spring Data Libraries** that allows us to connect to any type of databases eg. MongoDB, Postgres SQL, Prometheus, etc.
6. Spring boot is a layer on top of java to make web applications and is popularly called **frameworks of frameworks** due to the ease of importing and initializing frameworks needed for building our project.

# Initializing Spring boot using Spring Initializer:

## Step 1:
> Go to this site: https://start.spring.io/
 
## Step 2:
> In this step we are going to install all the dependencies that we will be requiring for our project. Given below is an example of such project utilizing Postgres as Database.
1. Select Project as Maven Project
2. Language as Java
3. Latest Spring Boot Version (One without Snapshot in it)
4. Add dependencies from the right hand side pane
	- Spring Web (for building restful APIs)
	- Spring Boot DEV Tools
	- Spring Data JBA (connect to databases)
	- Postgres Drivers

## Step 3:
>  Click on the generate button and download the *.zip* file. Extract the folder within and open it in an IDE.


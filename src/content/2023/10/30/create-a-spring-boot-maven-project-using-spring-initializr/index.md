---
title: "Create a Spring Boot Maven project using Spring Initializr"
date: 2023-10-30
categories: 
  - "spring-boot"
---

Spring Boot is one of the best frameworks for building web applications. It has a great web page for creating a boilerplate to start a project.

So, good news, creating an application with this framework is really easy.

## **Step 1 Open Spring initializr**

The first thing you need to do is open this web page on your browser:

[https://start.spring.io/](https://start.spring.io/)

![](images/SGyFq4glfD0eFwTPEoSfAyEYOzqqgfOAoeOeIqy-HfeP9pB_JtcH4g93yL8OvBLdlGKyARHbYSZdJAGnALN0-NqkUiwdJSDYkpoDc8Q5miz2gvrRPauCuEf5b4JkCTqg5Tvv5wleVnn5E1bAD9UZufw)

## **Step 2 Setup your project**

Next you have to specify the following parameters:

- Project: Maven

- Language: Java or Kotlin (if you prefer)

- Spring boot: Current new versión is 3.15

- Project Metadata: Replace according your needs

![](images/INvYHJeZQMIv5rzQz3K0HwXM4ys-4-rSAghyJjaq39aONMsU5j4MTYjK4AK8yQKc3dVUTYRHAlSkQUOlaETQujM1ikY6N8TmCs0q4EUd4srzNwiG49avg_QwAtn8M0u6JSly1gqQiz0QlqbwDsYNLW0)

Package: use jar if you web server would be inside the jar package, use war if you have the web server outside the package.

Java: try to use a recent versión of java like 17 or 21

Dependencies: Select Spring web dependency

It should look like the image below

![](images/Qh8hddDuCxx54NL78TRpw6XjlYXZeE7qAVRPb5QrIOs0pK0_BzMK4PcEVo5rF1-02TUsPTeefsSbQwPrEuvB1lOtDYNI_kw0qdyRkGJCub-vrhNneirLQYoW6qSGsIi1OqIP10u-QvlJ5aEz35XvgbE)

## **Step 3 Create a end point**

Add a SayHi class with the @RestController annotation. This indicates that this class contains an end point. Then add a method the @GetMapping annotation. This indicates that the method will handle the request of the specified path

```java

@RestController
public class SayHi {

    @GetMapping("/")
    public String hi() {
      return String.format("Hi!");
    }
    
}
```

## **Step 4 Run the application**

First install maven dependencies using

```java
./mvnw install
```

Then run the application using

```java
./mvnw spring-boot:run
```

![](images/aV3rP-AIebY3pg4UJMmtBk9P6KzErBesK0mMP3rD4VoLxI2Q35Ld-PDFFLetsF-SbRCxXZgWstlRGo1Lj1UJLtFVnWplmPSsbm-DkUx9V0WPqEB3re3XBgcZZhvfcl_-fIPjbgyhNAGG4GYHv1TxIzs)

Finally, open a web browser and you will see this:

![](images/Muj-Yl3ll3DiQHAAHK7qb1nGNFNdwmddevDlYigBxaPhe7MhOSuGk9rqD9v_50daMXm7gGH3e5MXJ80CA8EJ3rfVvtLZ_jBLjkF6zjvwY0f4neD3-7AxSbq_N6fgtekVE8Xvt1LUTYC74sGnkeuezl8)

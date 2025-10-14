---
title: "Spring boot API blocked by CORS policy"
date: 2023-11-01
categories: 
  - "spring-boot"
---

CORS, or Cross-Origin Resource Sharing, is a security feature implemented by web browsers to control and manage requests between webpages in different domains. It's relevant when working with web APIs, as it specifies who is allowed to make requests to a web server or API from a different domain

The most common scenario when CORS start to give problems is when you have an API in a server and try to make a client request from a different server. You will notice an error like this in your browser  

```
Access to fetch at 'http://localhost:8080/' from origin 'null' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
```

An easy fix to workaround with the problem is to use the @CrossOrigin annotation. Consider a controller like this

```
@RestController
public class SayHi {

    @PostMapping("/")
    public String hi() {
      return "Hi";
    }
    
}
```

You just have to add the annotation in the class definition

```
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@CrossOrigin
public class SayHi {

    @PostMapping("/")
    public String hi() {
      return "Hi";
    }
    
}
```

# EXP05-Setting-Up-Spring-Security-in-a-Spring-Boot-Project
## Reg no: 212223040208
## AIM:
To write a program for setting up Spring Security in a Spring Boot project to secure endpoints with basic authentication and role-based access control.

## ALGORITHM:
Create a Spring Boot Project with the following dependencies:

Spring Web

Spring Security

Spring Boot DevTools (optional)

Add Spring Security dependency in pom.xml (if not using Spring Initializr).

Create a configuration class extending WebSecurityConfigurerAdapter (or using SecurityFilterChain for newer Spring versions).

Define an in-memory user with username, password, and roles using UserDetailsService.

Secure your REST endpoints using annotations or in the security config class.

Run and test the app using a browser or Postman:

Secure endpoints will prompt for username and password.

## PROGRAM CODE:
### pom.xml (Dependencies)
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>4.1.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>demo.ex5</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name/>
	<description/>
	<url/>
	<licenses>
		<license/>
	</licenses>
	<developers>
		<developer/>
	</developers>
	<scm>
		<connection/>
		<developerConnection/>
		<tag/>
		<url/>
	</scm>
	<properties>
		<java.version>21</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webmvc</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<!-- JWT API -->
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-api</artifactId>
			<version>0.12.5</version>
		</dependency>

		<!-- JWT Implementation -->
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-impl</artifactId>
			<version>0.12.5</version>
			<scope>runtime</scope>
		</dependency>

		<!-- JWT Jackson -->
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-jackson</artifactId>
			<version>0.12.5</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webmvc-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```
### SecurityConfig.java (Spring Boot 3.x / Spring Security 6+)
```
package com.example.demo.ex5;
import com.example.demo.ex5.JwtFilter;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http)
            throws Exception {

        http
                .csrf(csrf -> csrf.disable())

                .sessionManagement(session ->
                        session.sessionCreationPolicy(
                                SessionCreationPolicy.STATELESS
                        )
                )

                .authorizeHttpRequests(auth -> auth

                        .requestMatchers("/auth/login")
                        .permitAll()

                        .anyRequest()
                        .authenticated()
                )

                .addFilterBefore(
                        new JwtFilter(),
                        UsernamePasswordAuthenticationFilter.class
                );

        return http.build();
    }
}
```

## AuthController.java
```
package com.example.demo.ex5;
import com.example.demo.ex5.JwtUtil;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/auth")
public class AuthController {

    @PostMapping("/login")
    public String login(@RequestBody LoginRequest req) {

        if ("admin".equals(req.getUsername())
                && "1234".equals(req.getPassword())) {

            return JwtUtil.generateToken(req.getUsername());
        }

        return "Invalid Credentials";
    }

    @GetMapping("/hello")
    public String hello() {

        return "Hello JWT Secure API";
    }
}
```
## JwtFilter :
```
package com.example.demo.ex5;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;
import java.util.Collections;

public class JwtFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        System.out.println("Auth header received: " + authHeader);

        if (authHeader != null && authHeader.startsWith("Bearer ")) {

            String token = authHeader.substring(7);

            try {

                String username = JwtUtil.extractUsername(token);

                UsernamePasswordAuthenticationToken authentication =
                        new UsernamePasswordAuthenticationToken(
                                username,
                                null,
                                Collections.emptyList()
                        );

                SecurityContextHolder
                        .getContext()
                        .setAuthentication(authentication);

                System.out.println("Authenticated user: " + username);

            } catch (Exception e) {
                System.out.println("JWT ERROR: " + e.getMessage());
                e.printStackTrace();
            }
        }

        filterChain.doFilter(request, response);
    }
}
```
## JwtUtil
```
package com.example.demo.ex5;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;

import javax.crypto.SecretKey;

public class JwtUtil {

    private static final String SECRET =
            "mySecretKeyForJWTAuthentication12345678901234567890";

    private static final SecretKey KEY =
            Keys.hmacShaKeyFor(SECRET.getBytes());

    public static String generateToken(String username) {

        return Jwts.builder()
                .subject(username)
                .signWith(KEY)
                .compact();
    }

    public static String extractUsername(String token) {

        return Jwts.parser()
                .verifyWith(KEY)
                .build()
                .parseSignedClaims(token)
                .getPayload()
                .getSubject();
    }
}
```
## LoginRequest:
```
package com.example.demo.ex5;



public class LoginRequest {

    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```

## Output:
<img width="1132" height="767" alt="image" src="https://github.com/user-attachments/assets/3cb9bd2b-9dc0-46ac-abf9-5ba850d20d44" />

<img width="1118" height="802" alt="image" src="https://github.com/user-attachments/assets/6187c5be-7059-48fa-a785-127a11f9761e" />

## Result:
Therefore,the program for setting up Spring Security in a Spring Boot project to secure endpoints with basic authentication and role-based access control is implemented and executed successfully.


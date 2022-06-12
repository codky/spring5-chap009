# spring5-chap009
springmvc

# 1. 프로젝트 생성

spring5-chap009

src/main/java

src/main/webapp/WEB-INF/view

webapp: HTML, CSS, JS, JSP 등 웹 어플리케이션을 구현하는데 필요한 코드가 위치한다.

WEB-INF에는 web.xml 파일이 위치한다.

** 서블릿스펙에 따르면 WEB-INF 폴더의 하위폴더로 lib 폴더와 classes 폴더를 생성하고 각각 폴더에 필요한 jar파일과 컴파일된 클래스 파일이 위치해야한다. 하지만, 메이븐이나 그레이들 프로젝트의 경우 필요한 jar 파일은 pom.xml/build.gradle 파일의 의존을 통해 지정하고 컴파일된 결과는 target폴더나 build폴더에 위치한다. 때문에 WEB-INF폴더 밑에 lib폴더나 classes 폴더를 생성할 필요가 없다.

기존 pom.xml과 다른점은 <packaging>의 값으로 war를 주었다는 것. 이태그의 기본값은 jar로서 서블릿/jsp를 이용한 웹 어플리케이션을 개발할 경우 war를 값으로 주어야한다.

(참고로 war는 web application archive를 의미한다)

pom.xml

**<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0)"
xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)"
xsi:schemaLocation="[http://maven.apache.org/POM/4.0.0](http://maven.apache.org/POM/4.0.0)[http://maven.apache.org/xsd/maven-4.0.0.xsd](http://maven.apache.org/xsd/maven-4.0.0.xsd)">**

**<modelVersion>4.0.0</modelVersion>
<groupId>spring5</groupId>
<artifactId>spring5-chap009</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>war</packaging>**

**<dependencies>
<dependency>
<groupId>javax.servlet</groupId>
<artifactId>javax.servlet-api</artifactId>
<version>3.1.0</version>
<scope>provided</scope>
</dependency>**

```
<dependency>
	<groupId>javax.servlet.jsp</groupId>
	<artifactId>javax.servlet.jsp-api</artifactId>
	<version>2.3.2-b02</version>
	<scope>provided</scope>
</dependency>

<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>jstl</artifactId>
	<version>1.2</version>
</dependency>

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
	<version>5.0.2.RELEASE</version>
</dependency>

```

**</dependencies>**

```
<build>
	<plugins>
		<plugin>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>3.7.0</version>
			<configuration>
				<source>1.8</source>
				<target>1.8</target>
				<encoding>utf-8</encoding>
			</configuration>
		</plugin>
	</plugins>
</build>

```

**</project>**

→ 서블릿 3.1, jsp 2.3, jstl 1.2 //  spring-webmvc모듈 의존 추가(스프링mvc를 사용하기 위해)

톰캣 설정

file - setting -tomcat server

# 2. 스프링 mvc를 위한 설정

스프링mvc의 주요 설정(HandlerMapping, ViewResolver 등)

스프링의 DispatcherServlet 설정

```
package config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ViewResolverRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/view", ".jsp");
    }
}

```

@EnabeWebMvc 는 스프링 mvc설정을 활성화한다. 스프링 mvc를 사용하는데 필요한 다양한 설정을 생성한다.

DispatcherServlet의 매핑경로를 ‘/’로 주었을 때, jsp/html/css 등을 올바르게 처리하기 위한 설정을 추가한다. 

jsp를 이용해서 컨트롤러의 실행결과를 보여주기 위한 설정 추가

원래는 스프링 mvc를 직접구성하려면 매우복잡했지만, 위와같은 어노테이션설정이나 인터페이스를 이용하면 웹 어플리케이션을 개발하는데 필요한 최소 설정이 끝난다.

# web.xml 파일에 DispatcherServlet 설정

WEB-INF/web.xml

```
<?xml version="1.0" encoding="UTF-8" ?>

<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="2.5">

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>
                config.MvcConfig
                config.ControllerConfig
            </param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>
            org.springframework.web.filter.CharacterEncodingFilter
        </filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
```

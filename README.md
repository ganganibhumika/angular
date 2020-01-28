# angular
use for front-end 

MySqlConfig


package com.bhumi.config;

import java.util.Properties;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
@EnableTransactionManagement
@EnableWebMvc
public class Mysqlconfig extends WebMvcConfigurerAdapter{

	
	@Autowired
	private Environment env;

	@Autowired
	private DataSource dataSource;

	@Autowired
	private LocalContainerEntityManagerFactoryBean entityManagerFactory;

	@Bean
	public DataSource dataSource() {
		DriverManagerDataSource dataSourceForDbConnection = new DriverManagerDataSource();
		dataSourceForDbConnection.setDriverClassName(env.getProperty("spring.datasource.driverClassName"));
		dataSourceForDbConnection.setUrl(env.getProperty("spring.datasource.url"));
		dataSourceForDbConnection.setUsername(env.getProperty("spring.datasource.username"));
//		dataSourceForDbConnection.setPassword(env.getProperty("spring.datasource.password"));
		return dataSourceForDbConnection;

	}

	/**
	 * Declare the JPA entity manager factory.
	 */
	@Bean
	public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
		LocalContainerEntityManagerFactoryBean entityManagerFactoryForLocal = new LocalContainerEntityManagerFactoryBean();

		entityManagerFactoryForLocal.setDataSource(dataSource);

		// Classpath scanning of @Component, @Service, etc annotated class
		entityManagerFactoryForLocal.setPackagesToScan(env.getProperty("spring.datasource.packagesToScan"));

		// Vendor adapter
		HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
		entityManagerFactoryForLocal.setJpaVendorAdapter(vendorAdapter);

		// Hibernate properties

		Properties additionalProperties = new Properties();
		additionalProperties.put("hibernate.dialect", "org.hibernate.dialect.MySQL5InnoDBDialect");
		additionalProperties.put("hibernate.id.new_generator_mappings", "false");
		additionalProperties.put("hibernate.show_sql", true);
		additionalProperties.put("hibernate.hbm2ddl.auto", "update");

		entityManagerFactoryForLocal.setJpaProperties(additionalProperties);

		return entityManagerFactoryForLocal;
	}

	/**
	 * Declare the transaction manager.
	 */
	@Bean
	public JpaTransactionManager transactionManager() {
		JpaTransactionManager transactionManager = new JpaTransactionManager();
		transactionManager.setEntityManagerFactory(entityManagerFactory.getObject());
		return transactionManager;
	}

	@Bean
	public PersistenceExceptionTranslationPostProcessor exceptionTranslation() {
		return new PersistenceExceptionTranslationPostProcessor();
	}
	
}



#---------------------------------------------controller
package com.bhumi.controller;

import java.util.List;
import java.util.Optional;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.bhumi.model.User;
import com.bhumi.payload.MyRequestBody;
import com.bhumi.payload.MyResponseBody;
import com.bhumi.repository.UserRepository;
import com.bhumi.utils.UtilsService;

@RestController
@RequestMapping("/api")
public class UserController {

	@Autowired
	UserRepository userRepository;

	@GetMapping("/test")
	public String test() {
		User user = new User();
		user.setEmail("ganganibhumika@gmail.com");
		user.setFirstname("bhumi");
		user.setLastname("gangani");
		user.setMobileNo("7383474858");
		userRepository.save(user);
		return "hello from user controller";
	}

	
	@PostMapping("/saveUser")
	public MyResponseBody saveUser(@RequestBody MyRequestBody body, HttpServletRequest request) {
		System.out.println(body);
		User user = new User();
		if (body == null) {
			System.out.println("object is null");
			return new MyResponseBody(HttpServletResponse.SC_OK, "save user", null, null);
		}
		user = UtilsService.MAPPER.convertValue(body.getJsonOfObject(), User.class);
		System.out.println("user::"+user.getEmail());
		return new MyResponseBody(HttpServletResponse.SC_OK, "save user", null, null);

	}
	
	@PutMapping("/updateUser/{userId}")
	public MyResponseBody updateUser(@PathVariable("userId") String userId , @RequestBody MyRequestBody body, HttpServletRequest request) {
		System.out.println(userId+"........userId");
		User user = new User();
		if (body == null || userId == null) {
			System.out.println("object is null");
			return new MyResponseBody(HttpServletResponse.SC_OK, "save user", null, null);
		}
		user = UtilsService.MAPPER.convertValue(body.getJsonOfObject(), User.class);
		
		Optional<User> userdb = userRepository.findById(userId);
		if(userdb.isPresent()) {
			userdb.get().setEmail(user.getEmail());
		}
		System.out.println("user::"+user.getEmail());
		return new MyResponseBody(HttpServletResponse.SC_OK, "update user", null, null);

	}
	

	@GetMapping("/getUserDetails")
	public MyResponseBody getUserDetail(HttpServletResponse response) {
		List<User> userList = userRepository.findAll();
		return new MyResponseBody(response.getStatus(), "get user detail successfully.", null, userList);

	}

}

## -------------------------------- mainclass

package com.bhumi.mypp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@ComponentScan("com.bhumi")
@SpringBootApplication
@RestController
@EnableAutoConfiguration
@EnableJpaRepositories("com.bhumi.repository")
public class MyppApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyppApplication.class, args);
	}
	
	@GetMapping("/")
	public String test() {
		return "Hello from My App";
	}
	

}

##------------------------------ properties-----------------

spring.datasource.packagesToScan=com.bhumi

# MySQL jdbc connection url.
spring.datasource.url=jdbc:mysql://localhost:3306/mydb?useLegacyDatetimeCode=false&serverTimezone=UTC

# MySQL jdbc driver class name.
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
# MySQL database username and password
spring.datasource.username=root
#spring.datasource.password=root

## Hibernate Properties
# The SQL dialect makes Hibernate generate better SQL for the chosen database
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5InnoDBDialect

# Hibernate ddl auto (create, create-drop, validate, update)
spring.jpa.hibernate.ddl-auto = create

spring.jpa.generate-ddl=true
spring.jpa.hibernate.ddl-auto = create



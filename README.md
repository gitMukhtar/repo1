HI

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>TestHello</groupId>
  <artifactId>TestHello</artifactId>
  <packaging>jar</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>TestHello Maven Webapp</name>
  <url>http://maven.apache.org</url>
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.0.2.RELEASE</version>
  <relativePath/> <!-- lookup parent from repository -->
</parent>
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  <java.version>1.8</java.version>
</properties>
<dependencies>
  <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <dependency>
     <groupId>com.h2database</groupId>
     <artifactId>h2</artifactId>
     <scope>runtime</scope>
  </dependency>
  <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-test</artifactId>
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
......

package com.test;


import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class BirthdayWish {

    @Id
    @GeneratedValue
    private int id;
    private String dateofBirth;
    private String name;
    
    
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getDateofBirth() {
		return dateofBirth;
	}
	public void setDateofBirth(String dateofBirth) {
		this.dateofBirth = dateofBirth;
	}
    
	
    
    
}

..............

package com.test;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.util.Date;
import java.util.List;
import java.time.temporal.ChronoUnit;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class BirthDayWishController {

    @Autowired
    BirthDayWishService birthdayService;

   
    @PutMapping("/hello/{nameOfPerson}")
	private Object saveName(@RequestBody BirthdayWish birthdayWish, @PathVariable("nameOfPerson") String nameOfPerson) {
		if (!validtaeString(nameOfPerson)) {
			return "Name are not character";
		}
		try {
			birthdayWish.setName(nameOfPerson);
			SimpleDateFormat sdf = new SimpleDateFormat("yyyy_MM_dd");
			Date InputDate = sdf.parse(birthdayWish.getDateofBirth());
			Date today = sdf.parse(sdf.format(new Date()));
			if (InputDate.after(today)) {
				birthdayWish.setDateofBirth("");
				return "InputDate is after today";
			}
			birthdayService.saveOrUpdate(birthdayWish);
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		// return "204 No content";
		return new ResponseEntity<Void>(HttpStatus.NO_CONTENT);
	}
    @GetMapping("/hello/{name}")
	private String getName(@PathVariable("name") String nameOfPerson) {
		List<BirthdayWish> birthdayWish = birthdayService.getBirthdayWish(nameOfPerson);
		String msg = "";
		if (birthdayWish.size() > 0) {
			LocalDate today = LocalDate.now();
			String bDate[] = birthdayWish.get(0).getDateofBirth().split("_", 3);
			// Today's date
			LocalDate birthday = LocalDate.of(today.getYear(), Integer.parseInt(bDate[1]), Integer.parseInt(bDate[2]));
			// Period period = Period.between(birthday, today);
			final long days = ChronoUnit.DAYS.between(today, birthday);
			if (days == 0)
				msg = "Happy Birthday " + birthdayWish.get(0).getName();
			else
				msg = "Hello " + birthdayWish.get(0).getName() + " your next birth is on after " + days;
			if (days < 0) {
				birthday = LocalDate.of(today.getYear() + 1, Integer.parseInt(bDate[1]), Integer.parseInt(bDate[2]));
				msg = "Hello " + birthdayWish.get(0).getName() + " your next birth is on after "
						+ ChronoUnit.DAYS.between(today, birthday);
				;
			}
		} else
			msg = "Record not present";
		return msg;
	}
    
    public boolean validtaeString(String str) {
        str = str.toLowerCase();
        char[] charArray = str.toCharArray();
        for (int i = 0; i < charArray.length; i++) {
           char ch = charArray[i];
           if (!(ch >= 'a' && ch <= 'z')) {
              return false;
           }
        }
        return true;
     }
}
......................

package com.test;

import org.springframework.data.repository.CrudRepository;

public interface BirthDayWishRepository extends CrudRepository<BirthdayWish, Integer> {}

.............................

package com.test;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BirthDayWishService {

    @Autowired
    BirthDayWishRepository personRepository;

    public List<BirthdayWish> getBirthdayWish(String name) {
        List<BirthdayWish> persons = new ArrayList<BirthdayWish>();
        personRepository.findAll().forEach(birthdayWish ->{
        	if(birthdayWish.getName().equalsIgnoreCase(name)){
        		persons.add(birthdayWish);
        	}
        });
        return persons;
    }

    public void saveOrUpdate(BirthdayWish birthdayWish) {
        personRepository.save(birthdayWish);
        
    }
    
}package com.test;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BirthDayWishService {

    @Autowired
    BirthDayWishRepository personRepository;

    public List<BirthdayWish> getBirthdayWish(String name) {
        List<BirthdayWish> persons = new ArrayList<BirthdayWish>();
        personRepository.findAll().forEach(birthdayWish ->{
        	if(birthdayWish.getName().equalsIgnoreCase(name)){
        		persons.add(birthdayWish);
        	}
        });
        return persons;
    }

    public void saveOrUpdate(BirthdayWish birthdayWish) {
        personRepository.save(birthdayWish);
        
    }
    
}package com.test;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BirthDayWishService {

    @Autowired
    BirthDayWishRepository personRepository;

    public List<BirthdayWish> getBirthdayWish(String name) {
        List<BirthdayWish> persons = new ArrayList<BirthdayWish>();
        personRepository.findAll().forEach(birthdayWish ->{
        	if(birthdayWish.getName().equalsIgnoreCase(name)){
        		persons.add(birthdayWish);
        	}
        });
        return persons;
    }

    public void saveOrUpdate(BirthdayWish birthdayWish) {
        personRepository.save(birthdayWish);
        
    }
    
}package com.test;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BirthDayWishService {

    @Autowired
    BirthDayWishRepository personRepository;

    public List<BirthdayWish> getBirthdayWish(String name) {
        List<BirthdayWish> persons = new ArrayList<BirthdayWish>();
        personRepository.findAll().forEach(birthdayWish ->{
        	if(birthdayWish.getName().equalsIgnoreCase(name)){
        		persons.add(birthdayWish);
        	}
        });
        return persons;
    }

    public void saveOrUpdate(BirthdayWish birthdayWish) {
        personRepository.save(birthdayWish);
        
    }
    
}package com.test;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BirthDayWishService {

    @Autowired
    BirthDayWishRepository personRepository;

    public List<BirthdayWish> getBirthdayWish(String name) {
        List<BirthdayWish> persons = new ArrayList<BirthdayWish>();
        personRepository.findAll().forEach(birthdayWish ->{
        	if(birthdayWish.getName().equalsIgnoreCase(name)){
        		persons.add(birthdayWish);
        	}
        });
        return persons;
    }

    public void saveOrUpdate(BirthdayWish birthdayWish) {
        personRepository.save(birthdayWish);
        
    }
    
}package com.test;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BirthDayWishService {

    @Autowired
    BirthDayWishRepository personRepository;

    public List<BirthdayWish> getBirthdayWish(String name) {
        List<BirthdayWish> persons = new ArrayList<BirthdayWish>();
        personRepository.findAll().forEach(birthdayWish ->{
        	if(birthdayWish.getName().equalsIgnoreCase(name)){
        		persons.add(birthdayWish);
        	}
        });
        return persons;
    }

    public void saveOrUpdate(BirthdayWish birthdayWish) {
        personRepository.save(birthdayWish);
        
    }
    
}package com.test;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BirthDayWishService {

    @Autowired
    BirthDayWishRepository personRepository;

    public List<BirthdayWish> getBirthdayWish(String name) {
        List<BirthdayWish> persons = new ArrayList<BirthdayWish>();
        personRepository.findAll().forEach(birthdayWish ->{
        	if(birthdayWish.getName().equalsIgnoreCase(name)){
        		persons.add(birthdayWish);
        	}
        });
        return persons;
    }

    public void saveOrUpdate(BirthdayWish birthdayWish) {
        personRepository.save(birthdayWish);
        
    }
    
}package com.test;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BirthDayWishService {

    @Autowired
    BirthDayWishRepository personRepository;

    public List<BirthdayWish> getBirthdayWish(String name) {
        List<BirthdayWish> persons = new ArrayList<BirthdayWish>();
        personRepository.findAll().forEach(birthdayWish ->{
        	if(birthdayWish.getName().equalsIgnoreCase(name)){
        		persons.add(birthdayWish);
        	}
        });
        return persons;
    }

    public void saveOrUpdate(BirthdayWish birthdayWish) {
        personRepository.save(birthdayWish);
        
    }
    
}package com.test;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class BirthDayWishService {

    @Autowired
    BirthDayWishRepository personRepository;

    public List<BirthdayWish> getBirthdayWish(String name) {
        List<BirthdayWish> persons = new ArrayList<BirthdayWish>();
        personRepository.findAll().forEach(birthdayWish ->{
        	if(birthdayWish.getName().equalsIgnoreCase(name)){
        		persons.add(birthdayWish);
        	}
        });
        return persons;
    }

    public void saveOrUpdate(BirthdayWish birthdayWish) {
        personRepository.save(birthdayWish);
        
    }
    
}

............
package com.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class BirthDayWishServiceApplication {

   public static void main(String[] args) {
      SpringApplication.run(BirthDayWishServiceApplication.class, args);
   }

}

..............
# Enabling H2 Console
spring.h2.console.enabled=true
# temporary data storage
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=test
spring.datasource.password=test
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect


java -jar -Dserver.port=8084 -Dspring.profiles.active=dev cse-svc-delivery-0.0.1-SNAPSHOT.jar $
{"dateofBirth":"2020_03_05"} 
http://localhost:8084/hello/muka

http://localhost:8084/hello/muka



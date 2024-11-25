# MapStruct in JAVA

dependecy 
```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.6.0.Beta1</version> 
</dependency>
```

Annotation processor dependency 


```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.6.0.Beta1</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

entity class

```java
public class Student {
    private long id;
    private long tcNumber;
    private String nameSurname;
    private int age;
    private String schoolName;
    private String country;
    private String city;
}
```

dto / request-response class

```java
public class StudentDto {
    private long id;
    private String nameSurname;
    private String schoolName;
}
```

Mappper interface 

```java
import org.mapstruct.Mapper; 
import org.mapstruct.Mapping; 
import org.mapstruct.factory.Mappers; 

@Mapper
public interface StudentMapper {

    StudentMapperINSTANCE Mappers.getMapper(StudentMapper.class);
    @Mapping(source = "student.id", target = "id") //Student sınıfındaki id alanını StudentDto alanında id alanına binding edebiliyoruz.
    StudentDTO entityToDto(Student student);

}
```

inject into the service or controller class

```java
@RestController
public class StudentController {
    private final StudentMapper studentMapper;

    public StudentController(StudentMapper studentMapper) {
        this.studentMapper = studentMapper;
    }

    @GetMapping
    public ResponseEntity getStudentDto() {
        Student student = new Student();
        student.setAge(23);
        student.setId(1L);
        
        StudentDto studentDto = studentMapper.map(student);

        return ResponseEntity.ok(studentDto);
    }
}
```


## Another implementation example: 

mapper interface
```java


        @Mapper 1
        public interface CarMapper {
         
            CarMapper INSTANCE = Mappers.getMapper( CarMapper.class ); 3
         
            @Mapping(source = "numberOfSeats", target = "seatCount")
            CarDto carToCarDto(Car car); 2
        }


```

test class 

```java


        @Test
        public void shouldMapCarToDto() {
            //given
            Car car = new Car( "Morris", 5, CarType.SEDAN );
         
            //when
            CarDto carDto = CarMapper.INSTANCE.carToCarDto( car );
         
            //then
            assertThat( carDto ).isNotNull();
            assertThat( carDto.getMake() ).isEqualTo( "Morris" );
            assertThat( carDto.getSeatCount() ).isEqualTo( 5 );
            assertThat( carDto.getType() ).isEqualTo( "SEDAN" );
        }


```
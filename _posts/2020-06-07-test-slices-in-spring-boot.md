---
title: Test Slices in Spring-Boot
author: Hamza Belmellouki
categories: [Spring]
tags: [spring-boot, testing]
comments: true
---

Greetings friends 👋.

One of the early mistakes that I've done in my first professional Spring-Boot based project was writing integration tests that load the entire `ApplicationContext` using `@SpringBootTest` annotation when there's no need. Thankfully, one of my colleagues was kind enough to help me understand that there is a better way to do things :) Writting tests like that will definitely slow down your continuous integration pipeline.

Before talking about test slices I want to make a clear distinction between a unit test and an integration test in Spring: Unit testing doesn't involve loading the application context. On the other hand, integration testing is more involed with loading the application context.

## Test Slices

Test Slices are a cool Spring-Boot feature introduced in 1.4. The idea is that Spring-Boot will bootstrap only the configuration meta-data that's appropriate for the component that's under test. Using this feature will result in a much lightweight `ApplicationContext`. Thus, the execution of our integration tests will be faster compared to loading the entire context. Now let's see some of these test slices in action.

### @WebMvcTest

Integration tests is about mocking the minimum amount of dependencies to test if the integration of your components and modules works fine. But, ofentimes your components depend on external web services and that web service isn't up in your build/dev environment. In such situation we can go ahead with the integration test and mock that dependency. In this example, I want to test `EmployeeResource` endpoints: 



```java
@RestController
class EmployeeResource {

    @Autowired
    EmployeeService employeeService;

    @GetMapping("/employees")
    public ResponseEntity<List<Employee>> getAllEmployees() {
        return ResponseEntity.ok(employeeService.getAllEmployees());
    }
}
```

Now `@WebMvcTest` will disable full auto-configuration(done by `@SpringBootTest`) and instead apply only configuration relevant to MVC tests(i.e. `@Controller`, `@ControllerAdvice`, `@JsonComponent`, Converter/GenericConverter, `Filter`, `WebMvcConfigurer`). Suppose that `employeeService.getAllEmployees()` routine perform an HTTP call to another external web service and the service isn't available in my local dev environment so I'll decide to mock it using `@MockBean`. Now I can perform request and test the results:

```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class EmployeeResourceTest {

    @Autowired
    MockMvc mockMvc;

    @MockBean
    EmployeeService employeeService;

    List<Employee> employees = new ArrayList<>();

    @Before
    public void setUp() throws Exception {
        Employee employee = new Employee();
        employee.setId(1);
        employee.setFirstName("a first name");
        employee.setLastName("a last name");
        employees.add(employee);
    }

    @Test
    public void shouldReturnOkayAndPayloadWhenRequestingEmployeeResource() throws Exception {
        doReturn(employees).when(employeeService).getAllEmployees();
        mockMvc.perform(MockMvcRequestBuilders.get("/employees"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$[0].id").value(1))
                .andExpect(jsonPath("$[0]firstName").value("a first name"))
                .andExpect(jsonPath("$[0]lastName").value("a last name"));
        
        verify(employeeService).getAllEmployees();
    }
}

```

### @DataJpaTest

We can leverage `@DataJpaTest` annotation to disable full auto-configuration and instead apply only configuration relevant to JPA tests. This will not only load repository components but also utility classes like `DataSource` and `TestEntityManager` which can be used to save/find data in the DB. 

Note that by default, tests annotated with `@DataJpaTest` will auto-configure an in-memory h2 database (can be overridden) for testing purposes. Also, tests are transactional and rolled back at the end of each test. In this example, I'd like to test `EmployeeRepository`:

```java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {

}

```

This simple test tests that the repository can save and retrieve the data in the in-memory h2 database:

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class EmployeeRepositoryTest {

    @Autowired
    EmployeeRepository employeeRepository;

    @Autowired
    TestEntityManager entityManager;

    Employee employee;

    @Before
    public void setUp() throws Exception {
        employee = new Employee();
        employee.setId(1);
        employee.setFirstName("a first name");
        employee.setLastName("a last name");
    }

    @Test
    public void shouldSaveEmployee() {
        entityManager.persist(employee);
        Employee result = employeeRepository.findAll().get(0);
        assertThat(result.getId()).isEqualTo(1);
        assertThat(result.getFirstName()).isEqualTo("a first name");
        assertThat(result.getLastName()).isEqualTo("a last name");
    }
}

```

### @RestClientTest

Use `@RestClientTest` to speed up the testing of REST clients. this annotation will disable full auto-configuration and instead apply only configuration relevant to rest client tests (i.e. Jackson or GSON auto-configuration and `@JsonComponent` beans, but not regular `@Component` beans). It also auto-configure some essential beans like `RestTemplateBuilder` and `MockRestServiceServer`  and load them into the context. Now we'll test `EmployeeDetailsService` which perform an HTTP request to `http://localhost:8081/{id}/details` endpoint to retrieve an `EmployeeDetails` object:

```java
@Service
public class EmployeeDetailsService {

    private final RestTemplate restTemplate;

    public EmployeeDetailsService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    public EmployeeDetails getEmployeeDetails(int id) {
        return restTemplate.getForObject("http://localhost:8081/{id}/details", EmployeeDetails.class, id);
    }
}

@Data
class EmployeeDetails {
    private String address;
    private int salary;
}
```

`@RestClientTest` annotation's value attribute specify which service is under test. Doing so will speed up our test since only `EmployeeDetailsService` is loaded in the context along with other beans provided from the auto-configuration. This example demonstrates how to test `EmployeeDetailsService`:

```java
@RunWith(SpringRunner.class)
@RestClientTest(EmployeeDetailsService.class)
public class EmployeeDetailsServiceTest {

    @Autowired
    private EmployeeDetailsService employeeDetailsService;

    @Autowired
    private MockRestServiceServer mockRestServiceServer;

    @Test
    public void shouldReturnEmployeeDetailsFromHttpRequest() {
        mockRestServiceServer.expect(requestTo("http://localhost:8081/1/details")).andRespond(
                withSuccess(new ClassPathResource("employeeDetails.json"), MediaType.APPLICATION_JSON));

        EmployeeDetails employeeDetails = employeeDetailsService.getEmployeeDetails(1);

        assertThat(employeeDetails.getAddress()).isEqualTo("Morocco, Casablanca, Maarif");
        assertThat(employeeDetails.getSalary()).isEqualTo(100_000);
    }
}
```

`MockRestServiceServer` mocks the expected behavior of the intended HTTP request made by a `RestTemplate` inside `EmployeeDetailsService`. Note that `ClassPathResource` picks `employeeDetails.json` from the root of the test classpath.

## Wrap Up

In this blog, I've discussed the most widely-used test slices. Note that I didn't cover them all. If you want to check others checkout this [link](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-test-autoconfigure/src/main/java/org/springframework/boot/test/autoconfigure). And as usual you can find these code snippets the [github repo](https://github.com/Hamzablm/test-slices).

If you have any feedback about my blogs. Please, don't hesitate to reach out to me or just say a "hello" on twitter: [@HamzaLovesJava](https://twitter.com/HamzaLovesJava)
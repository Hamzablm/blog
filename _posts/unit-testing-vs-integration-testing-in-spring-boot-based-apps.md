Title: Test Slices in Spring-Boot

Greetings my friends 👋

One of the early mistakes that I've done in my first professional Spring-Boot based project was writing integration tests that load the entire `ApplicationContext` using `@SpringBootTest` annotation when there's no need. Thankfully, one of my colleagues was kind enough to help me understand that there is a better way to do things :) Writting tests like that may result in tests that may take hours which isn't good for continuous integration.

Before talking about test slices I want to make a clear distinction between a unit test and an integration test in Spring: Unit testing doesn't involve loading the application context. On the other hand, integration testing is more involed with loading the application context.

## Test Slices

Test Slices are a cool Spring-Boot feature introduced in 1.4. The idea is that Spring-Boot will bootstrap only the configuration meta-data that's appropriate for the test slice. Using this feature will result in a much lightweight `ApplicationContext`. Thus, the execution of our integration tests will be faster compared to loading the entire context. Now let's see some of these test slices in action.

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
        Mockito.doReturn(employees).when(employeeService).getAllEmployees();
        mockMvc.perform(MockMvcRequestBuilders.get("/employees"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$[0].id").value(1))
                .andExpect(jsonPath("$[0]firstName").value("a first name"))
                .andExpect(jsonPath("$[0]lastName").value("a last name"));
    }
}

```

### @DataJpaTest

We can leverage `@DataJpaTest` annotation to disable full auto-configuration and instead apply only configuration relevant to JPA tests. This will not only load repository components but also utility classes like ` DataSource`, `TestEntityManager` which can be used to save/find data in the DB. 

Note that by default, tests annotated with `@DataJpaTest` will configure an in-memory h2 database (can be overridden) for testing purposes. Also, tests are transactional and rolled back at the end of each test. In this example, I'd like to test `EmployeeRepository`:

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



### @JsonTest

### @RestClientTest



## Wrap Up

In this blog I've discussed the most widely-used test slices. If you want to check others checkout this [link](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-test-autoconfigure/src/main/java/org/springframework/boot/test/autoconfigure).


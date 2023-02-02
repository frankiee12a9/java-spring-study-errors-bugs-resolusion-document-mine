# BEAN AND CREATING BEAN RELATED ERRORS AND CORRESPONDING RESOLUTIONS

1. [BeanCreationException](#Bean-Creation-Exception)
2. [NoSuchBeanDefinitionException](#No-Such-Bean-Definition-Exception)
3. [Test](#test)

>

## BeanCreationException

<!-- ## org.springframework.beans.factory.jBeanCreationException -->

> Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'studentDao' defined in com.mvc.tdd.dao.StudentDao defined in @EnableJpaRepositories declared on JpaRepositoriesRegistrar.EnableJpaRepositoriesConfiguration: Invocation of init method failed; nested exception is org.springframework.data.repository.query.QueryCreationException: Could not create query for public abstract com.mvc.tdd.entity.CollegeStudent com.mvc.tdd.dao.StudentDao.findByEmailAddress

### CAUSE

💥Entity field included in method name does not match with the entity field defined in the corresponding entity. Which then caused `Invocation of init method failed`. For example, in _Student.java_ entity

```java
public class Student {
  // other fields...

  private String email;
}
```

_StudentDao.java_

```java
@Repository
public interface StudentDao extends CrudRepository<CollegeStudent, Integer> {
    // "findByEmailAddress" is the cause
    public CollegeStudent findByEmailAddress(String emailAddress);
}
```

### RESOLVE

👉 change method name "findByEmailAddress" in _StudentDao.java_ to "findByEmail", so that Jpa can recognize the field name in such entity.

## NoSuchBeanDefinitionException

> Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException

### CAUSE

💥 try injecting a bean that doesn't exist in current context.
For example, StudentGrade is trying to inject GradeDao:

```java
@Component
public class StudentGrade {

   @Autowired
   private GradeDao gradeDao;

   // others
}
```

If a GradeDao isn't found in current context, then the following exception will be thrown (Error Creating Bean):

> Error creating bean with name 'StudentGrade': Injection of autowired dependencies failed;
> nested exception is org.springframework.beans.factory.BeanCreationException:
> Could not autowire field: private com.mvc.tdd.dao.GradeDao.gradeDao com.mvc.tdd.entity.StudentGrade;
> nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException:
> No qualifying bean of type [com.mvc.tdd.dao.GradeDao] found for dependency:
> expected at least 1 bean which qualifies as autowire candidate for this dependency.
> Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}

### RESOLVE

👉 Use `@Configuration` class via the `@Bean` annotation or is annotated with `@Component`, `@Repository`, `@Service`, `@Controller`, and classpath scanning is active for that package

## Test

> this is test error

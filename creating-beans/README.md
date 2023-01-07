# BEAN AND CREATING BREANS RELATED ERRORS AND RESOLUTIONS
## 1. ERROR
>> Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'studentDao' defined in com.mvc.tdd.dao.StudentDao defined in @EnableJpaRepositories declared on JpaRepositoriesRegistrar.EnableJpaRepositoriesConfiguration: Invocation of init method failed; nested exception is org.springframework.data.repository.query.QueryCreationException: Could not create query for public abstract com.mvc.tdd.entity.CollegeStudent com.mvc.tdd.dao.StudentDao.findByEmailAddress

## CAUSE
*Student.java* entity
```java
public class Student {
  // other fields...
  
  private String email;
}
```
*StudentDao.java*
```java
@Repository
public interface StudentDao extends CrudRepository<CollegeStudent, Integer> {
    // "findByEmailAddress" is the cause
    public CollegeStudent findByEmailAddress(String emailAddress); 
}
```

## RESOLVE
👉 change method name "findByEmailAddress" in *StudentDao.java* to "findByEmail", so that Jpa can recognize the field name in such entity.

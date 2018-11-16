# JPA-cheatsheet

1. Creating an entity in a domain level
```java
@Entity
@Table(name="EMPLOYEE") // this may be omitted if the class name is the same as the table name
public class Employee {
    ...
}
```
1. Sometimes an entity is stored in several tables

![1_entity_2_tables](https://upload.wikimedia.org/wikipedia/commons/8/87/Emp_Tables_%28Database%29.PNG)
```java
@Entity
@Table(name="EMPLOYEE")
@SecondaryTable(name="EMP_DATA",
                // may be omitted if both tables have "id"
                pkJoinColumns = @PrimaryKeyJoinColumn(name="EMP_ID", referencedColumnName="ID"))
public class Employee {
    ...
}
```
1. 
```java
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
```

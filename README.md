# JPA-cheatsheet

![1_entity_2_tables](https://github.com/chirkov86/JPA-cheatsheet/blob/master/jpaentitystates.png)

#### 1. Creating an entity in a domain level
```java
@Entity
@Table(name="EMPLOYEE") // this may be omitted if the class name is the same as the table name
public class Employee {
    ...
}
```
Sometimes an entity is stored in several tables:

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
#### 2. ID generation
JPA provides several strategies for id generation: 
- TABLE uses sequence table in db 
- SEQUENCE special database objects to generate ids. Supported in Oracle and Postgres.
- IDENTITY uses special `IDENTITY` columns in db. Identity sequencing requires the insert to occur before the id can be assigned, so it is not assigned on persist like other types of sequencing. You must either call commit() on the current transaction, or call flush() on the EntityManager.
```java
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
```
#### 3. Entity relationships
Possible types of associations are:
- `OneToOne` - A unique reference from one object to another
- `ManyToOne` - A reference from one object to another
- `OneToMany` - A Collection or Map of objects, inverse of a ManyToOne
- `ManyToMany` - A Collection or Map of objects
- `Embedded` - A reference to a object that shares the same table of the parent
- `ElementCollection` - A Collection or Map of Basic or Embeddable objects, stored in a separate table

#### 4. Fetching
Lazy fetching allows the fetching of a relationship to be deferred until it is accessed.
The default fetch type is `LAZY` for all relationships except for `OneToOne` and `ManyToOne`.
```java
@OneToOne(fetch=FetchType.LAZY)
```
For collection relationships sending `size()` is normally the best way to ensure a lazy relationship is instantiated. For `OneToOne` and `ManyToOne` normally just accessing the relationship is enough (i.e. `employee.getAddress()`).
Another solution is to use the JPQL `JOIN FETCH` for the relationship when querying the objects. A join fetch will normally ensure the relationship has been instantiated.

#### 5. Cascading
Relationships can have a cascade option that allows to model dependent relationships such as `Order` -> `OrderLine`. Cascading such relationship allows for the Order's -> OrderLines to be persisted, removed, merged along with their parent.
The following operations can be cascaded, as defined in the `CascadeType` enum:
- `PERSIST` - Cascaded the `EntityManager.persist()` operation. If `persist()` is called on the parent, and the child is also new, it will also be persisted. If it is existing, nothing will occur, although calling `persist()` on an existing object will still cascade the persist operation to its dependents.
- `REMOVE` - Cascaded the `EntityManager.remove()` operation. If remove() is called on the parent then the child will also be removed. This should only be used for dependent relationships. If you remove a dependent object from a `OneToMany` collection it will not be deleted, JPA requires that you explicitly call `remove()` on it.
- `MERGE` - Cascaded the `EntityManager.merge()` operation.
- `REFRESH` - Cascaded the `EntityManager.refresh()` operation. Be careful enabling this for all relationships, as it could cause changes made to other objects to be reset.
- `ALL` - Cascaded all the above operations.
```java
@Entity
public class Employee {
  ...
  @OneToOne(cascade={CascadeType.ALL})
  @JoinColumn(name="ADDR_ID")
  private Address address;
  ...
}
```
#### 6. Maps
JPA allows a Map to be used for any collection mapping including `OneToMany`, `ManyToMany` and `ElementCollection`.
`@MapKeyColumn` annotation is used to define a map relationship where the key is a Basic value, the `@MapKeyJoinColumn` annotation is used to define a map relationship where the key is an Entity value. There are also `@MapKeyJoinColumns`, `@MapKeyEnumerated` and `@MapKeyTemporal`.
```java
@Entity
public class Employee {
  ...
  @OneToMany(mappedBy="owner")
  @MapKeyColumn(name="PHONE_TYPE")
  private Map<String, Phone> phones;
  ...
}
```

#### 7. Join fetch
Join fetching is a query optimization technique for reading multiple objects in a single database query. It involves joining the two object's tables in SQL and selecting both object's data. Join fetching is commonly used for `OneToOne` relationships, but also can be used for any relationship including `OneToMany` and `ManyToMany`.

Join fetching is one solution to the classic ORM n+1 performance problem. The issue is if you select n Employee objects, and access each of their addresses, in basic ORM (including JPA) you will get 1 database select for the Employee objects, and then n database selects, one for each Address object. Join fetching solves this issue by only requiring one select, and selecting both the Employee and its Address.

JPA supports join fetching through JPQL using the `JOIN FETCH` syntax:
`SELECT emp FROM Employee emp JOIN FETCH emp.address`
This causes both the Employee and Address data to be selected in a single query.

If your relationship allows null or an empty collection for collection relationships, then you can use outer join:
`SELECT emp FROM Employee emp LEFT JOIN FETCH emp.address`

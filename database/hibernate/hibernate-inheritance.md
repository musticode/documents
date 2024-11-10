# Hibernate - Inheritance


- @MappedSuperclass indicates that a class is a superclass and is not associated with a specific database table, but its fields (or properties) can be inherited by child entity classes that are associated with tables. This helps avoid code duplication and ensures a cleaner object model structure.
  

### Usage

Table sql 
```sql
CREATE TABLE accounts
(
    id         BIGSERIAL PRIMARY KEY,
    created_at TIMESTAMP,
    balance    NUMERIC(10, 2),
    is_active  BOOLEAN
);
```

Base Java class

```java
import jakarta.persistence.*;
import lombok.*;
import lombok.experimental.SuperBuilder;

import java.time.LocalDateTime;
import java.util.Objects;

@Getter
@Setter
@ToString
@SuperBuilder
@NoArgsConstructor
@AllArgsConstructor
@MappedSuperclass
public abstract class BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    // equals and hashCode
}
```

Extending the Base Class

```java
import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.Table;
import lombok.*;
import lombok.experimental.SuperBuilder;

import java.math.BigDecimal;
import java.util.Objects;

@Entity
@Getter
@Setter
@ToString
@SuperBuilder
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "accounts")
public class Account extends BaseEntity {

    @Column(name = "balance", precision = 10, scale = 2)
    private BigDecimal balance;

    @Column(name = "is_active")
    private Boolean isActive;

    // equals and hashCode
}
```

### Benefits of Using @MappedSuperclass

- Avoiding Code Duplication: Common fields and methods of the base class are automatically inherited by all child classes.
- Flexibility in Data Modeling: @MappedSuperclass allows for more flexible and scalable data models.
- Simplified Maintenance: Centralizing common fields simplifies their update and maintenance.


### Problems and Advantages 

#### `@MappedSuperclass`

- The **main problem with implicit inheritance mapping is that it doesn’t support polymorphic associations very well**. In the database, you usually represent associations as foreign key relationships. If the subclasses are all mapped to different tables, a polymorphic association to their superclass can’t be represented as a simple foreign key relationship.

- Polymorphic queries that return instances of all classes that match the interface of the queried class are also problematic. **Hibernate must execute a query against the superclass as several SQL SELECTs, one for each concrete subclass**.

- A further conceptual problem with this mapping strategy is that several different columns, of different tables, share exactly the same semantics. This makes schema evolution more complex. For example, **renaming or changing the type of a superclass property results in changes to multiple columns in multiple tables**. Many of the standard refactoring operations offered by your IDE would require manual adjustments, because the automatic procedures usually don’t account for things like @AttributeOverrides. It also makes it much more difficult to implement database integrity constraints that apply to all subclasses.
  

#### `@@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`

- Note that the JPA standard specifies that TABLE_PER_CLASS is optional, so not all JPA implementations may support it. The implementation is also vendor dependent — in Hibernate, it’s equivalent to a <union-subclass> mapping in the old native Hibernate XML metadata.

- The advantages of this mapping strategy are clearer if we examine polymorphic queries. The tables are combined with a UNION operator, and a literal is inserted into the intermediate result; Hibernate reads this to instantiate the correct class given the data from a particular row. A union requires that the queries that are combined project over the same columns; hence, you have to pad and fill nonexistent columns with NULL. You may ask whether this query will really perform better than two separate statements. Here you can let the database optimizer find the best execution plan to combine rows from several tables, instead of merging two result sets in memory as Hibernate’s polymorphic loader engine would do.

- Another much more important advantage is the ability to handle polymorphic associations. Hibernate can use a UNION query to simulate a single table as the target of the association mapping.
  
> Java Persistence with Hibernate (Bauer, King, Gregory)


## Önemli Notlar

- Single Table; polymorphic sorgulara ve ilişkilere ihtiyacımız varsa ve performansta bizim için önemliyse tercih etmemiz gereken strateji ancak dikkat etmemiz gereken konu veri bütünlüğü(!) Bu stratejiyi kullanırken constraintleri kullanamıyoruz.
- Joined Table; eğer veri bütünlüğü bizim için performans, polymorphic sorgular ve ilişkilerden daha önemliyse seçmemiz gereken strateji olacaktır.
- Table-Per-Class; Polymorphic sorgulara ve ilişkilere ihtiyacımız yoksa bizim için daha uygun olan strateji bu olacaktır. Ayrıca bize constraintler tanımlayabilmemiz sayesinde veri bütünlüğünü oluşturma olanağı sağlar. Ayrıca bu stratejide polymorphic sorgu ve ilişkiler sağlayabiliriz ancak unutmamamız gereken; bu sorgular kompleks tablo yapımızdan dolayı bizim için çok kullanışsız ve performanssızdır.
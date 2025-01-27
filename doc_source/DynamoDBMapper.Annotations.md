# Java Annotations for DynamoDB<a name="DynamoDBMapper.Annotations"></a>

This section describes the annotations that are available for mapping your classes and properties to tables and attributes in Amazon DynamoDB\.

For the corresponding Javadoc documentation, see [Annotation Types Summary](https://docs.aws.amazon.com/sdk-for-java/latest/reference/com/amazonaws/services/dynamodbv2/datamodeling/package-summary.html) in the [AWS SDK for Java API Reference](https://docs.aws.amazon.com/sdk-for-java/latest/reference/)\.

**Note**  
In the following annotations, only `DynamoDBTable` and the `DynamoDBHashKey` are required\. 

**Topics**
+ [DynamoDBAttribute](#DynamoDBMapper.Annotations.DynamoDBAttribute)
+ [DynamoDBAutoGeneratedKey](#DynamoDBMapper.Annotations.DynamoDBAutoGeneratedKey)
+ [DynamoDBDocument](#DynamoDBMapper.Annotations.DynamoDBDocument)
+ [DynamoDBHashKey](#DynamoDBMapper.Annotations.DynamoDBHashKey)
+ [DynamoDBIgnore](#DynamoDBMapper.Annotations.DynamoDBIgnore)
+ [DynamoDBIndexHashKey](#DynamoDBMapper.Annotations.DynamoDBIndexHashKey)
+ [DynamoDBIndexRangeKey](#DynamoDBMapper.Annotations.DynamoDBIndexRangeKey)
+ [DynamoDBRangeKey](#DynamoDBMapper.Annotations.DynamoDBRangeKey)
+ [DynamoDBTable](#DynamoDBMapper.Annotations.DynamoDBTable)
+ [DynamoDBTypeConverted](#DynamoDBMapper.Annotations.DynamoDBTypeConverted)
+ [DynamoDBTyped](#DynamoDBMapper.Annotations.DynamoDBTyped)
+ [DynamoDBVersionAttribute](#DynamoDBMapper.Annotations.DynamoDBVersionAttribute)

## DynamoDBAttribute<a name="DynamoDBMapper.Annotations.DynamoDBAttribute"></a>

Maps a property to a table attribute\. By default, each class property maps to an item attribute with the same name\. However, if the names are not the same, you can use this annotation to map a property to the attribute\. In the following Java snippet, the `DynamoDBAttribute` maps the `BookAuthors` property to the `Authors` attribute name in the table\.

```
@DynamoDBAttribute(attributeName = "Authors")
public List<String> getBookAuthors() { return BookAuthors; }
public void setBookAuthors(List<String> BookAuthors) { this.BookAuthors = BookAuthors; }
```

The `DynamoDBMapper` uses `Authors` as the attribute name when saving the object to the table\. 

## DynamoDBAutoGeneratedKey<a name="DynamoDBMapper.Annotations.DynamoDBAutoGeneratedKey"></a>

Marks a partition key or sort key property as being autogenerated\. `DynamoDBMapper` generates a random [UUID](http://docs.oracle.com/javase/6/docs/api/java/util/UUID.html) when saving these attributes\. Only String properties can be marked as autogenerated keys\. 

The following example demonstrates using autogenerated keys\.

```
@DynamoDBTable(tableName="AutoGeneratedKeysExample")
public class AutoGeneratedKeys {
    private String id;
    private String payload;

    @DynamoDBHashKey(attributeName = "Id")
    @DynamoDBAutoGeneratedKey
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }

    @DynamoDBAttribute(attributeName="payload")
    public String getPayload() { return this.payload; }
    public void setPayload(String payload) { this.payload = payload; }

    public static void saveItem() {
        AutoGeneratedKeys obj = new AutoGeneratedKeys();
        obj.setPayload("abc123");

        // id field is null at this point
        DynamoDBMapper mapper = new DynamoDBMapper(dynamoDBClient);
        mapper.save(obj);

        System.out.println("Object was saved with id " + obj.getId());
    }
}
```

## DynamoDBDocument<a name="DynamoDBMapper.Annotations.DynamoDBDocument"></a>

Indicates that a class can be serialized as an Amazon DynamoDB document\.

For example, suppose that you wanted to map a JSON document to a DynamoDB attribute of type Map \(`M`\)\. The following code example defines an item containing a nested attribute \(Pictures\) of type Map\.

```
public class ProductCatalogItem {

    private Integer id;  //partition key
    private Pictures pictures;
    /* ...other attributes omitted... */

    @DynamoDBHashKey(attributeName="Id")
    public Integer getId() { return id;}
    public void setId(Integer id) {this.id = id;}

    @DynamoDBAttribute(attributeName="Pictures")
    public Pictures getPictures() { return pictures;}
    public void setPictures(Pictures pictures) {this.pictures = pictures;}

    // Additional properties go here.

    @DynamoDBDocument
    public static class Pictures {
        private String frontView;
        private String rearView;
        private String sideView;

        @DynamoDBAttribute(attributeName = "FrontView")
        public String getFrontView() { return frontView; }
        public void setFrontView(String frontView) { this.frontView = frontView; }

        @DynamoDBAttribute(attributeName = "RearView")
        public String getRearView() { return rearView; }
        public void setRearView(String rearView) { this.rearView = rearView; }

        @DynamoDBAttribute(attributeName = "SideView")
        public String getSideView() { return sideView; }
        public void setSideView(String sideView) { this.sideView = sideView; }

     }
}
```

You could then save a new `ProductCatalog` item, with `Pictures`, as shown in the following example\.

```
ProductCatalogItem item = new ProductCatalogItem();

Pictures pix = new Pictures();
pix.setFrontView("http://example.com/products/123_front.jpg");
pix.setRearView("http://example.com/products/123_rear.jpg");
pix.setSideView("http://example.com/products/123_left_side.jpg");
item.setPictures(pix);

item.setId(123);

mapper.save(item);
```

The resulting `ProductCatalog` item would look like the following \(in JSON format\)\.

```
{
  "Id" : 123
  "Pictures" : {
    "SideView" : "http://example.com/products/123_left_side.jpg",
    "RearView" : "http://example.com/products/123_rear.jpg",
    "FrontView" : "http://example.com/products/123_front.jpg"
  }
}
```

## DynamoDBHashKey<a name="DynamoDBMapper.Annotations.DynamoDBHashKey"></a>

Maps a class property to the partition key of the table\. The property must be one of the scalar string, number, or binary types\. The property can't be a collection type\. 

Assume that you have a table, `ProductCatalog`, that has `Id` as the primary key\. The following Java code defines a `CatalogItem` class and maps its `Id` property to the primary key of the `ProductCatalog` table using the `@DynamoDBHashKey` tag\.

```
@DynamoDBTable(tableName="ProductCatalog")
public class CatalogItem {
    private Integer Id;
   @DynamoDBHashKey(attributeName="Id")
   public Integer getId() {
        return Id;
   }
   public void setId(Integer Id) {
        this.Id = Id;
   }
   // Additional properties go here.
}
```

## DynamoDBIgnore<a name="DynamoDBMapper.Annotations.DynamoDBIgnore"></a>

Indicates to the `DynamoDBMapper` instance that the associated property should be ignored\. When saving data to the table, the `DynamoDBMapper` does not save this property to the table\.

 Applied to the getter method or the class field for a non\-modeled property\. If the annotation is applied directly to the class field, the corresponding getter and setter must be declared in the same class\. 

## DynamoDBIndexHashKey<a name="DynamoDBMapper.Annotations.DynamoDBIndexHashKey"></a>

Maps a class property to the partition key of a global secondary index\. The property must be one of the scalar string, number, or binary types\. The property can't be a collection type\. 

Use this annotation if you need to `Query` a global secondary index\. You must specify the index name \(`globalSecondaryIndexName`\)\. If the name of the class property is different from the index partition key, you also must specify the name of that index attribute \(`attributeName`\)\.

## DynamoDBIndexRangeKey<a name="DynamoDBMapper.Annotations.DynamoDBIndexRangeKey"></a>

Maps a class property to the sort key of a global secondary index or a local secondary index\. The property must be one of the scalar string, number, or binary types\. The property can't be a collection type\. 

Use this annotation if you need to `Query` a local secondary index or a global secondary index and want to refine your results using the index sort key\. You must specify the index name \(either `globalSecondaryIndexName` or `localSecondaryIndexName`\)\. If the name of the class property is different from the index sort key, you must also specify the name of that index attribute \(`attributeName`\)\.

## DynamoDBRangeKey<a name="DynamoDBMapper.Annotations.DynamoDBRangeKey"></a>

Maps a class property to the sort key of the table\. The property must be one of the scalar string, number, or binary types\. It cannot be a collection type\. 

If the primary key is composite \(partition key and sort key\), you can use this tag to map your class field to the sort key\. For example, assume that you have a `Reply` table that stores replies for forum threads\. Each thread can have many replies\. So the primary key of this table is both the `ThreadId` and `ReplyDateTime`\. The `ThreadId` is the partition key, and `ReplyDateTime` is the sort key\. 

The following Java code defines a `Reply` class and maps it to the `Reply` table\. It uses both the `@DynamoDBHashKey` and `@DynamoDBRangeKey` tags to identify class properties that map to the primary key\.

```
@DynamoDBTable(tableName="Reply")
public class Reply {
    private Integer id;
    private String replyDateTime;

    @DynamoDBHashKey(attributeName="Id")
    public Integer getId() { return id; }
    public void setId(Integer id) { this.id = id; }

    @DynamoDBRangeKey(attributeName="ReplyDateTime")
    public String getReplyDateTime() { return replyDateTime; }
    public void setReplyDateTime(String replyDateTime) { this.replyDateTime = replyDateTime; }

   // Additional properties go here.
}
```

## DynamoDBTable<a name="DynamoDBMapper.Annotations.DynamoDBTable"></a>

Identifies the target table in DynamoDB\. For example, the following Java code defines a class `Developer` and maps it to the `People` table in DynamoDB\. 

```
@DynamoDBTable(tableName="People")
public class Developer { ...}
```

The `@DynamoDBTable` annotation can be inherited\. Any new class that inherits from the `Developer` class also maps to the `People` table\. For example, assume that you create a `Lead` class that inherits from the `Developer` class\. Because you mapped the `Developer` class to the `People` table, the `Lead` class objects are also stored in the same table\.

The `@DynamoDBTable` can also be overridden\. Any new class that inherits from the `Developer` class by default maps to the same `People` table\. However, you can override this default mapping\. For example, if you create a class that inherits from the `Developer` class, you can explicitly map it to another table by adding the `@DynamoDBTable` annotation as shown in the following Java code example\.

```
@DynamoDBTable(tableName="Managers")
public class Manager extends Developer { ...}
```

## DynamoDBTypeConverted<a name="DynamoDBMapper.Annotations.DynamoDBTypeConverted"></a>

An annotation to mark a property as using a custom type converter\. Can be annotated on a user\-defined annotation to pass additional properties to the `DynamoDBTypeConverter`\. 

 The `DynamoDBTypeConverter` interface lets you map your own arbitrary data types to a data type that is natively supported by DynamoDB\. For more information, see [Mapping Arbitrary Data](DynamoDBMapper.ArbitraryDataMapping.md)\.

## DynamoDBTyped<a name="DynamoDBMapper.Annotations.DynamoDBTyped"></a>

An annotation to override the standard attribute type binding\. Standard types do not require the annotation if applying the default attribute binding for that type\. 

## DynamoDBVersionAttribute<a name="DynamoDBMapper.Annotations.DynamoDBVersionAttribute"></a>

Identifies a class property for storing an optimistic locking version number\. `DynamoDBMapper` assigns a version number to this property when it saves a new item, and increments it each time you update the item\. Only number scalar types are supported\. For more information about data types, see [Data Types](HowItWorks.NamingRulesDataTypes.md#HowItWorks.DataTypes)\. For more information about versioning, see [Optimistic Locking with Version Number](DynamoDBMapper.OptimisticLocking.md)\.
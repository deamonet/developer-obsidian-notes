## **1. Introduction**[](https://www.baeldung.com/gson-list#introduction)

In this tutorial, we'll explore a few advanced [serialization](https://www.baeldung.com/gson-serialization-guide) and [deserialization](https://www.baeldung.com/gson-deserialization-guide) cases for _List_ using [Google's Gson library](https://github.com/google/gson).

## **2. List of Objects** [](https://www.baeldung.com/gson-list#list-of-objects)

One common use case is to serialize and deserialize a list of POJOs.

Consider the class:

```java
public class MyClass {
    private int id;
    private String name;

    public MyClass(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // getters and setters
}
```

Here's how we would serialize _List<MyClass>_:

```java
@Test
public void givenListOfMyClass_whenSerializing_thenCorrect() {
    List<MyClass> list = Arrays.asList(new MyClass(1, "name1"), new MyClass(2, "name2"));

    Gson gson = new Gson();
    String jsonString = gson.toJson(list);
    String expectedString = "[{\"id\":1,\"name\":\"name1\"},{\"id\":2,\"name\":\"name2\"}]";

    assertEquals(expectedString, jsonString);
}
```

As we can see, serialization is fairly straightforward.

However, deserialization is tricky. Here's an incorrect way of doing it:

```java
@Test(expected = ClassCastException.class)
public void givenJsonString_whenIncorrectDeserializing_thenThrowClassCastException() {
    String inputString = "[{\"id\":1,\"name\":\"name1\"},{\"id\":2,\"name\":\"name2\"}]";

    Gson gson = new Gson();
    List<MyClass> outputList = gson.fromJson(inputString, ArrayList.class);

    assertEquals(1, outputList.get(0).getId());
}
```

Here, **although we would get a _List_ of size two, post-deserialization, it wouldn't be a _List_ of _MyClass_**. Therefore, line #6 throws _ClassCastException_.

**Gson can serialize a collection of arbitrary objects but can't deserialize the data without additional information. That's because there's no way for the user to indicate the type of the resulting object.** Instead, while deserializing, the _Collection_ must be of a specific, generic type.

The correct way to deserialize the _List_ would be:

```java
@Test
public void givenJsonString_whenDeserializing_thenReturnListOfMyClass() {
    String inputString = "[{\"id\":1,\"name\":\"name1\"},{\"id\":2,\"name\":\"name2\"}]";
    List<MyClass> inputList = Arrays.asList(new MyClass(1, "name1"), new MyClass(2, "name2"));

    Type listOfMyClassObject = new TypeToken<ArrayList<MyClass>>() {}.getType();

    Gson gson = new Gson();
    List<MyClass> outputList = gson.fromJson(inputString, listOfMyClassObject);

    assertEquals(inputList, outputList);
}
```

Here, **we use Gson's _TypeToken_ to determine the correct type to be deserialized – _ArrayList<MyClass>_**. The idiom used to get the _listOfMyClassObject_ actually defines an anonymous local inner class containing a method _getType()_ that returns the fully parameterized type.

## **3. List of Polymorphic Objects**[](https://www.baeldung.com/gson-list#list-of-polymorphic-objects)

### **3.1. The Problem**[](https://www.baeldung.com/gson-list#1-the-problem)

Consider an example class hierarchy of animals:

```java
public abstract class Animal {
    // ...
}

public class Dog extends Animal {
    // ...
}

public class Cow extends Animal {
    // ...
}
```

How do we serialize and deserialize _List<Animal>_? We could use _TypeToken<ArrayList<Animal>>_ like we used in the previous section. However, Gson still won't be able to figure out the concrete data type of the objects stored in the list.

### **3.2. Using Custom Deserializer** [](https://www.baeldung.com/gson-list#2-using-custom-deserializer)

One way to solve this is to add type information to the serialized JSON. We honor that type information during JSON deserialization. For this, we need to write our own custom serializer and deserializer.

Firstly, we'll introduce a new _String_ field called _type_ in the base class _Animal_. It stores the simple name of the class to which it belongs.

Let's take a look at our sample classes:

```java
public abstract class Animal {
    public String type = "Animal";
}
```

```java
public class Dog extends Animal {
    private String petName;

    public Dog() {
        petName = "Milo";
        type = "Dog";
    }

    // getters and setters
}
```

```java
public class Cow extends Animal {
    private String breed;

    public Cow() {
        breed = "Jersey";
        type = "Cow";
    }

    // getters and setters
}
```

Serialization will continue to work as before without any issues:

```java
@Test 
public void givenPolymorphicList_whenSerializeWithTypeAdapter_thenCorrect() {
    String expectedString
      = "[{\"petName\":\"Milo\",\"type\":\"Dog\"},{\"breed\":\"Jersey\",\"type\":\"Cow\"}]";

    List<Animal> inList = new ArrayList<>();
    inList.add(new Dog());
    inList.add(new Cow());

    String jsonString = new Gson().toJson(inList);

    assertEquals(expectedString, jsonString);
}
```

In order to deserialize the list, we'll have to provide a custom deserializer:

```java
public class AnimalDeserializer implements JsonDeserializer<Animal> {
    private String animalTypeElementName;
    private Gson gson;
    private Map<String, Class<? extends Animal>> animalTypeRegistry;

    public AnimalDeserializer(String animalTypeElementName) {
        this.animalTypeElementName = animalTypeElementName;
        this.gson = new Gson();
        this.animalTypeRegistry = new HashMap<>();
    }

    public void registerBarnType(String animalTypeName, Class<? extends Animal> animalType) {
        animalTypeRegistry.put(animalTypeName, animalType);
    }

    public Animal deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) {
        JsonObject animalObject = json.getAsJsonObject();
        JsonElement animalTypeElement = animalObject.get(animalTypeElementName);

        Class<? extends Animal> animalType = animalTypeRegistry.get(animalTypeElement.getAsString());
        return gson.fromJson(animalObject, animalType);
    }
}
```

Here, the _animalTypeRegistry_ map maintains the mapping between the class name and the class type.

During deserialization, we first extract out the newly added _type_ field. Using this value, we do a lookup on the _animalTypeRegistry_ map to get the concrete data type. This data type is then passed to _fromJson()_.

Let's see how to use our custom deserializer:

```java
@Test
public void givenPolymorphicList_whenDeserializeWithTypeAdapter_thenCorrect() {
    String inputString
      = "[{\"petName\":\"Milo\",\"type\":\"Dog\"},{\"breed\":\"Jersey\",\"type\":\"Cow\"}]";

    AnimalDeserializer deserializer = new AnimalDeserializer("type");
    deserializer.registerBarnType("Dog", Dog.class);
    deserializer.registerBarnType("Cow", Cow.class);
    Gson gson = new GsonBuilder()
      .registerTypeAdapter(Animal.class, deserializer)
      .create();

    List<Animal> outList = gson.fromJson(inputString, new TypeToken<List<Animal>>(){}.getType());

    assertEquals(2, outList.size());
    assertTrue(outList.get(0) instanceof Dog);
    assertTrue(outList.get(1) instanceof Cow);
}
```

### **3.3. Using _RuntimeTypeAdapterFactory_**[](https://www.baeldung.com/gson-list#3-using-runtimetypeadapterfactory)

An alternative to writing a custom deserializer is to use the _RuntimeTypeAdapterFactory_ class present in the [Gson source code](https://github.com/google/gson/blob/master/extras/src/main/java/com/google/gson/typeadapters/RuntimeTypeAdapterFactory.java). However, **it's not exposed by the library for the user to use**. Hence, we'll have to create a copy of the class in our Java project.

Once this is done, we can use it to deserialize our list:

```java
@Test
public void givenPolymorphicList_whenDeserializeWithRuntimeTypeAdapter_thenCorrect() {
    String inputString
      = "[{\"petName\":\"Milo\",\"type\":\"Dog\"},{\"breed\":\"Jersey\",\"type\":\"Cow\"}]";

    Type listOfAnimals = new TypeToken<ArrayList<Animal>>(){}.getType();

    RuntimeTypeAdapterFactory<Animal> adapter = RuntimeTypeAdapterFactory.of(Animal.class, "type")
      .registerSubtype(Dog.class)
      .registerSubtype(Cow.class);

    Gson gson = new GsonBuilder().registerTypeAdapterFactory(adapter).create();

    List<Animal> outList = gson.fromJson(inputString, listOfAnimals);

    assertEquals(2, outList.size());
    assertTrue(outList.get(0) instanceof Dog);
    assertTrue(outList.get(1) instanceof Cow);
}
```

Note that the underlying mechanism is still the same.

We still need to introduce the type information during serialization. The type information can later be used during deserialization. **Hence, the field _type_ is still required in every class for this solution to work.** We just don't have to write our own deserializer.

_RuntimeTypeAdapterFactory_ provides the correct type adapter based on the field name passed to it and the registered subtypes.

## **4. Conclusion**[](https://www.baeldung.com/gson-list#conclusion)

In this article, we saw how to serialize and deserialize a list of objects using Gson.

As usual, the code is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/json-modules/gson).
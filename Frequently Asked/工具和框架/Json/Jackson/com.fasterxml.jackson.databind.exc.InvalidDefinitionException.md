com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `me.deamonet.nar.transmit.UserRegistered` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)

原因是我在该实体类中添加了一个为了方便实例化该类用的构造函数，导致JVM不会添加默认的无参构造函数，而jackson的反序列化需要无参构造函数，因此报错。


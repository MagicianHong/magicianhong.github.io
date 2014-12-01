---
layout: post
title: MessagePack
category_name: Android
category: c_android
---


# [MessagePack](http://msgpack.org/)


It's like JSON.

but fast and small.

MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it's faster and smaller. Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves.

##How to use

[msgpack/msgpack-java](https://github.com/msgpack/msgpack-java/wiki/QuickStart)

###Make a message-packable class

The annotation @Message enables you to serialize "public" fields in objects of your own classes like this.

```
import org.msgpack.MessagePack;
import org.msgpack.annotation.Message;

public class Main1 {
    @Message // Annotation
    public static class MyMessage {
        // public fields are serialized.
        public String name;
        public double version;
    }

    public static void main(String[] args) throws Exception {
        MyMessage src = new MyMessage();
        src.name = "msgpack";
        src.version = 0.6;

        MessagePack msgpack = new MessagePack();
        // Serialize
        byte[] bytes = msgpack.write(src);
        // Deserialize
        MyMessage dst = msgpack.read(bytes, MyMessage.class);
    }
}
```
If you want to serialize multiple objects sequentially, you can use Packer and Unpacker objects. This is because MessagePack.write(Object) and read(byte[]) method invocations create Packer and Unpacker objects every times. To use Packer and Unpacker objects, call createPacker(OutputStream) and createUnpacker(InputStream).

```
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;

import org.msgpack.MessagePack;
import org.msgpack.annotation.Message;
import org.msgpack.packer.Packer;
import org.msgpack.unpacker.Unpacker;

public class Main2 {
    @Message
    public static class MyMessage {
        public String name;
        public double version;
    }

    public static void main(String[] args) throws Exception {
        MyMessage src1 = new MyMessage();
        src1.name = "msgpack";
        src1.version = 0.6;
        MyMessage src2 = new MyMessage();
        src2.name = "muga";
        src2.version = 10.0;
        MyMessage src3 = new MyMessage();
        src3.name = "frsyukik";
        src3.version = 1.0;

        MessagePack msgpack = new MessagePack();
        //
        // Serialize
        //
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        Packer packer = msgpack.createPacker(out);
        packer.write(src1);
        packer.write(src2);
        packer.write(src3);
        byte[] bytes = out.toByteArray();

        //
        // Deserialize
        //
        ByteArrayInputStream in = new ByteArrayInputStream(bytes);
        Unpacker unpacker = msgpack.createUnpacker(in);
        MyMessage dst1 = unpacker.read(MyMessage.class);
        MyMessage dst2 = unpacker.read(MyMessage.class);
        MyMessage dst3 = unpacker.read(MyMessage.class);
    }
}
```

###Various types of values serialization/deserialization

The classes Packer/Unpacker allows serializing/deserializing values of various types as follows. They enable serializing/deserializing values of various types like values of primitive types, values of primitive wrapper classes, String objects, byte[] objects, ByteBuffer objects and so on. As mentioned above, they also enable serializing/deserizing objects of your own classes annotated by @Message.

```
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.math.BigInteger;
import java.nio.ByteBuffer;

import org.msgpack.MessagePack;
import org.msgpack.packer.Packer;
import org.msgpack.unpacker.Unpacker;

public class Main3 {
    public static void main(String[] args) throws Exception {
        MessagePack msgpack = new MessagePack();

        //
        // Serialization
        //
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        Packer packer = msgpack.createPacker(out);

        // Serialize values of primitive types
        packer.write(true); // boolean value
        packer.write(10); // int value
        packer.write(10.5); // double value

        // Serialize objects of primitive wrapper types
        packer.write(Boolean.TRUE);
        packer.write(new Integer(10));
        packer.write(new Double(10.5));

        // Serialize various types of arrays
        packer.write(new int[] { 1, 2, 3, 4 });
        packer.write(new Double[] { 10.5, 20.5 });
        packer.write(new String[] { "msg", "pack", "for", "java" });
        packer.write(new byte[] { 0x30, 0x31, 0x32 }); // byte array

        // Serialize various types of other reference values
        packer.write("MesagePack"); // String object
        packer.write(ByteBuffer.wrap(new byte[] { 0x30, 0x31, 0x32 })); // ByteBuffer object
        packer.write(BigInteger.ONE); // BigInteger object

        //
        // Deserialization
        //
        byte[] bytes = out.toByteArray();
        ByteArrayInputStream in = new ByteArrayInputStream(bytes);
        Unpacker unpacker = msgpack.createUnpacker(in);

        // to primitive values
        boolean b = unpacker.readBoolean(); // boolean value
        int i = unpacker.readInt(); // int value
        double d = unpacker.readDouble(); // double value

        // to primitive wrapper value
        Boolean wb = unpacker.read(Boolean.class);
        Integer wi = unpacker.read(Integer.class);
        Double wd = unpacker.read(Double.class);

        // to arrays
        int[] ia = unpacker.read(int[].class);
        Double[] da = unpacker.read(Double[].class);
        String[] sa = unpacker.read(String[].class);
        byte[] ba = unpacker.read(byte[].class);

        // to String object, ByteBuffer object, BigInteger object, List object and Map object
        String ws = unpacker.read(String.class);
        ByteBuffer buf = unpacker.read(ByteBuffer.class);
        BigInteger bi = unpacker.read(BigInteger.class);
    }
}
```

The methods Packer#write() allow serializing various types of data.

The class Unpacker provides deserialization methods for deserializing binary to primitive values. For example, if you want to deserialize binary to value of boolean (or int) type, you can use readBoolean (or readInt) method in Unpacker. Unpacker also provides read methods for reference values. Its methods allow deserializing binary to values of references which types you specified as parameters. For example, if you want to deserialize binary to String (or byte[]) object, you have to describe a call of read(String.class) (or read(byte[].class)) method.

###List, Map objects serialization/deserialization

To serialize generic container objects like List and Map objects, you can use Template. Template objects are pairs of serializer and deserializer. For example, to serialize a List object that has Integer objects as elements, you create the Template object by the following way.

Template listTmpl = Templates.tList(Templates.TInteger);
The classes tList, TInteger are static method and field in Templates. A simple example of List and Map objects is shown in the following.

```
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.util.*;

import org.msgpack.MessagePack;
import org.msgpack.packer.Packer;
import org.msgpack.template.Template;
import org.msgpack.unpacker.Unpacker;
import static org.msgpack.template.Templates.tList;
import static org.msgpack.template.Templates.tMap;
import static org.msgpack.template.Templates.TString;

public class Main4 {
    public static void main(String[] args) throws Exception {
        MessagePack msgpack = new MessagePack();

        // Create templates for serializing/deserializing List and Map objects
        Template<List<String>> listTmpl = tList(TString);
        Template<Map<String, String>> mapTmpl = tMap(TString, TString);

        //
        // Serialization
        //

        ByteArrayOutputStream out = new ByteArrayOutputStream();
        Packer packer = msgpack.createPacker(out);

        // Serialize List object
        List<String> list = new ArrayList<String>();
        list.add("msgpack");
        list.add("for");
        list.add("java");
        packer.write(list); // List object

        // Serialize Map object
        Map<String, String> map = new HashMap<String, String>();
        map.put("sadayuki", "furuhashi");
        map.put("muga", "nishizawa");
        packer.write(map); // Map object

        //
        // Deserialization
        //

        byte[] bytes = out.toByteArray();
        ByteArrayInputStream in = new ByteArrayInputStream(bytes);
        Unpacker unpacker = msgpack.createUnpacker(in);

        // to List object
        List<String> dstList = unpacker.read(listTmpl);

        // to Map object
        Map<String, String> dstMap = unpacker.read(mapTmpl);
    }
}
```

###Serialization without annotations

If you cannot append @Message to classes representing objects that you want to serialize, register method enables you to serialize the objects of the classes.

```
MessagePack msgpack = new MessagePack();
msgpack.register(MyMessage2.class);
```

For example, if MyMessage2 class is included in external library, you cannot easily modify the class declaration and append @Message to it. register method allows to generate a pair of serializer and deserializer for MyMessage2 class automatically. You can serialize objects of MyMessage2 class after executing the method.

###Optional fields

You can add new fields maintaining the compatibility. Use the @Optional in the new fields.

```
@Message
public static class MyMessage {
    public String name;
    public double version;

    // new field
    @Optional
    public int flag = 0;
}
```

If you try to deserialize the old version data, optional fields will be ignored.

###Dynamic typing

As Java is a static typing language, MessagePack has achieved dynamic typing with Value. Value has methods that checks its own type (isIntegerType(), isArrayType(), etc ...) and also converts to its own type (asStringValue(), convert(Template)).

```
import java.util.*;

import org.msgpack.MessagePack;
import org.msgpack.type.Value;
import org.msgpack.unpacker.Converter;

import static org.msgpack.template.Templates.*;

public class Main5 {
    public static void main(String[] args) throws Exception {
        // Create serialize objects.
        List<String> src = new ArrayList<String>();
        src.add("msgpack");
        src.add("kumofs");
        src.add("viver");

        MessagePack msgpack = new MessagePack();
        // Serialize
        byte[] raw = msgpack.write(src);

        // Deserialize directly using a template
        List<String> dst1 = msgpack.read(raw, tList(TString));

        // Or, Deserialze to Value then convert type.
        Value dynamic = msgpack.read(raw);
        List<String> dst2 = new Converter(dynamic).read(tList(TString));
    }
}
```










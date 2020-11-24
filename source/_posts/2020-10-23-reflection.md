---
layout: post
title:  "Reflection technique"
date:   2020-10-23 11:38:54 +0900
categories: java
---

## Creating new instance

Creating new instance using Reflection technique means You would call the contructor of the class. 
First of all, you gotta set the parameters for the constructor.

### Create Class<?> by Class name

```java
Class<T> clazz = (Class<T>) Class.forName("CLASSNAME");
```

You could do either like this.

### Setting Parameters

```java
Class[] classArgs = {Map.class, Map.class};
```

### Setting Contruction method 

```java
Constructor constructor = clazz.getDeclaredConstructor(classArgs);
```

### Create new instance

```java
Object obj = constructor.newInstance(map1, map2);
```

## Access All fields

```java

    Field[] fields = object.getClass().getDeclaredFields();
    
    for(Field field : fields) {
        field.getName();        // It's the name of that field
        field.getType();        // It's the type of that field like String, int or some custom object
    }

```

## Getter/Setter Invoke

```java

private void callSetter(Object obj, String fieldName, Object value){
    PropertyDescriptor pd;
    
    try {
        pd = new PropertyDescriptor(fieldName, obj.getClass());
        pd.getWriteMethod().invoke(obj, value);
    } catch (IntrospectionException | IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
}

private void callGetter(Object obj, String fieldName){
    PropertyDescriptor pd;
    
    try {
        pd = new PropertyDescriptor(fieldName, obj.getClass());
        System.out.println("" + pd.getReadMethod().invoke(obj));
    } catch (IntrospectionException | IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
}

public static void main(String[] args) {
    GetterAndSetter gs = new GetterAndSetter();
    TestClass tc = new TestClass();
    
    gs.callSetter(tc, "name", "John");
    gs.callSetter(tc, "value", 12);
    gs.callSetter(tc, "flag", true);

    // Getting fields of the class
    Field[] fields = tc.getClass().getDeclaredFields();
    
    for(Field f : fields){
        String fieldName = f.getName();
        System.out.println("Field Name -- " + fieldName);
    }
    
    gs.callGetter(tc, "name");
    gs.callGetter(tc, "value");
    gs.callGetter(tc, "flag");
}

```

They are offered from PropertyDescriptor object. But i think it's too restrict. so You can invoke just a getter method via the getter name.

```java

    private static <T> String getter(T t, String fieldName) {
        try {
            String getterMethodName = "get" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1);
            Method method = t.getClass().getDeclaredMethod(getterMethodName);
            return String.valueOf(method.invoke(t));
        } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
            log.info(e.toString());
            e.printStackTrace();

            return "";
        }
    }

```

```java

    @SuppressWarnings("unchecked")
    public static <T, U> List<T> map(List<U> srcs, String fieldName, Class<T> clazz) throws NoSuchFieldException, IllegalAccessException {
        List<T> res = new ArrayList<>();

        for(U src : srcs) {
            Field field = src.getClass().getDeclaredField(fieldName);
            field.setAccessible(true);
            res.add((T) field.get(src));
        }

        return res;
    }
    
```
### Class 类的方法

```Java
 //将这个类转变为clazz表示的类的子类，作用可以将一个class<?> 类型的
 //Class对象转化成指定的对象
<U> Class<? extends U>	asSubclass(Class<U> clazz)  
    
//将obj强制转换为Class中保存的类型，若obj没有继承T，则抛出异常。
T	cast(Object obj) 

boolean	desiredAssertionStatus()
  
static Class<?>	forName(String className)
static Class<?>	forName(String name, boolean initialize, ClassLoader loader)

AnnotatedType[]	getAnnotatedInterfaces()
AnnotatedType	getAnnotatedSuperclass()
    
  //返回内部类
Class<?>[]	getClasses()
Class<?>[]	getDeclaredClasses()
    
    //返回当前内部类声明的外部类
Class<?>	getDeclaringClass()     
    
//返回直接继承父类
Class<? super T>	getSuperclass()  
Type	getGenericSuperclass()  
    
//返回直接实现接口
Type[]	getGenericInterfaces()
Class<?>[]	getInterfaces()
    
    
  //获取类的修饰符
int	getModifiers()
    
    //构造器
Constructor<T>	getConstructor(Class<?>... parameterTypes)
Constructor<?>[]	getConstructors()
Constructor<T>	getDeclaredConstructor(Class<?>... parameterTypes)
Constructor<?>[]	getDeclaredConstructors()    

       //属性  
Field	getField(String name)
Field[]	getFields()
Field	getDeclaredField(String name)
Field[]	getDeclaredFields()   
    
    //方法
Method	getDeclaredMethod(String name, Class<?>... parameterTypes)
Method[]	getDeclaredMethods()
Method	getMethod(String name, Class<?>... parameterTypes)
Method[]	getMethods()    
  
   //注解  
<A extends Annotation> A getAnnotation(Class<A> annotationClass)
<A extends Annotation>  A[]	getAnnotationsByType(Class<A> annotationClass)
Annotation[]	getAnnotations()
<A extends Annotation> A	getDeclaredAnnotation(Class<A> annotationClass)
<A extends Annotation> A[]	getDeclaredAnnotationsByType(Class<A> annotationClass)   
Annotation[]	getDeclaredAnnotations()    
    

    //获取数组类型的
Class<?>	getComponentType()   
    





//现在还不懂
Class<?>	getEnclosingClass()
Constructor<?>	getEnclosingConstructor()
Method	getEnclosingMethod()





 //获取类名，前一个是完整的类型，后一个是简答的类名
String	getName()
String	getSimpleName()
    
Package	getPackage()  
ProtectionDomain	getProtectionDomain()
    
URL	getResource(String name)
InputStream	getResourceAsStream(String name)
    
Object[]	getSigners()
String	getTypeName()
TypeVariable<Class<T>>[]	getTypeParameters()
ClassLoader	getClassLoader()
T[]	getEnumConstants()

    
    
boolean	isAnnotation()
boolean	isAnnotationPresent(Class<? extends Annotation> annotationClass)
boolean	isAnonymousClass()
boolean	isArray()
boolean	isAssignableFrom(Class<?> cls)
boolean	isEnum()
boolean	isInstance(Object obj)
boolean	isInterface()
boolean	isLocalClass()
boolean	isMemberClass()
boolean	isPrimitive()
    
// 这个类或方法或字段是编译器自动生成的，那返回值就是true，比如枚举中的values()方法！
boolean	isSynthetic()

    
    
T	newInstance()
String	toGenericString()
String	toString()


```

### getGenericInterfaces 和getInterfaces 区别

```Java


import java.lang.reflect.Type;

public class Test implements A<Test>,B{
	
	public static void main(String[] args) throws NoSuchMethodException, SecurityException {
		Class<?>[] a = Test.class.getInterfaces();
		for(Class aa : a)
			System.out.println(aa);
		System.out.println("-----------");
		Type[] tes = Test.class.getGenericInterfaces();
		for(Type te : tes)
			System.out.println(te);
	}
	
}

interface A<T>{}
interface B{}
interface C extends A{}
输出：

interface one.A
interface one.B
-----------
one.A<one.Test>
interface one.B
```

### getComponentType()

>获取数组类型的Class对象

```
public class Test {
	
	public static void main(String[] args) {
		Test[] te = new Test[3];
		Class cla = te.getClass();
		System.out.println(cla.getComponentType().getCanonicalName());
		Class test = Test.class;
		//报错，因为test.getComponentType()返回null，test不是数组
		//System.out.println(test.getComponentType().getCanonicalName());
		
	}
	
}
```

### getCanonicalName()

****

> 输出类的全限定名，看例子：
>
> ```
> public class Clazz {
>
> 	public static void main(String[] args) {
> 		HelloWorld hello = new HelloWorld();
> 		Class<HelloWorld> cla = (Class<HelloWorld>) hello.getClass();
> 		System.out.println(cla.getCanonicalName());
> 		int[] nums = {1,2,3,4,5,6};
> 		Class<? extends int[]> intClass = nums.getClass();
> 		System.out.println(intClass.getCanonicalName());
> 	}
>
> }
>
> 输出：
> wo.HelloWorld
> int[]
>
> ```
>
> 看其实现：
>
> ```
> public String getCanonicalName() {
>         if (isArray()) {
>             String canonicalName = getComponentType().getCanonicalName();
>             if (canonicalName != null)
>                 return canonicalName + "[]";
>             else
>                 return null;
>         }
>         if (isLocalOrAnonymousClass())
>             return null;
>         Class<?> enclosingClass = getEnclosingClass();
>         if (enclosingClass == null) { // top level class
>             return getName();
>         } else {
>             String enclosingName = enclosingClass.getCanonicalName();
>             if (enclosingName == null)
>                 return null;
>             return enclosingName + "." + getSimpleName();
>         }
>     }
> ```
>
> 若是数组的话，先获取组件类型 的名字，然后后面加上[]
>
> 若是匿名类，直接返回null
>
> 正常类的话先获取其包名，然后通过getSimpleName()方法获取其简单名。
>
> ```
> public String getSimpleName() {
>         if (isArray())
>             return getComponentType().getSimpleName()+"[]";
>
>         String simpleName = getSimpleBinaryName();
>         if (simpleName == null) { // top level class
>             simpleName = getName();
>             
>             // strip the package name
>             return simpleName.substring(simpleName.lastIndexOf('.')+1); 
>         }
>         return simpleName;
>     }
>     
> public String getName() {
>         String name = this.name;
>         if (name == null)
>             this.name = name = getName0();
>         return name;
> }
>
>     // cache the name to reduce the number of calls into the VM
>     private transient String name;
>     private native String getName0();
> ```
>
> 可以看到，最终还是调用的native方法，这是用C语言去方法区从Class字节流中获取到了类名。



# Construct 类

```java
boolean	equals(Object obj)

AnnotatedType	getAnnotatedReceiverType()
AnnotatedType	getAnnotatedReturnType()
<T extends Annotation> T	getAnnotation(Class<T> annotationClass)
Annotation[]	getDeclaredAnnotations()

    //返回当前构造器所表示的外部类
Class<T>	getDeclaringClass()
    
Class<?>[]	getExceptionTypes()
Type[]	getGenericExceptionTypes()
   
int	getParameterCount()
Class<?>[]	getParameterTypes()
Annotation[][]	getParameterAnnotations()
Type[]	getGenericParameterTypes()
TypeVariable<Constructor<T>>[]	getTypeParameters()

int	getModifiers()
int	hashCode()
//返回当前构造器是否是自动生成的
boolean	isSynthetic()
//判断构造器是否是可变参数
boolean	isVarArgs()
    
//返回当前构造器的类名
String	getName() 
 
    //初始化对象
T	newInstance(Object... initargs)
    
//返回当前构造器的类名和参数的类型名
String	toGenericString()
String	toString()
```


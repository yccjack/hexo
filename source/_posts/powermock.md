---
title: Mockito和PowerMock用法 
categories: java 
tags: [单元测试,java]
date: 2019-11-12 
cover: https://mysticalyu.gitee.io/pic/img/cavan-amp-danny-a1.jpg
description: 在单元测试中，我们往往想去独立地去测一个类中的某个方法，但是这个类可不是独立的，它会去调用一些其它类的方法和service，这也就导致了以下两个问题：外部服务可能无法在单元测试的环境中正常工作，因为它们可能需要访问数据库或者使用一些其它的外部系统。我们的测试关注点在于这个类的实现上，外部类的一些行为可能会影响到我们对本类的测试，那也就失去了我们进行单测的意义。

---

![](https://mysticalyu.gitee.io/pic/img/cavan-amp-danny-a1.jpg)

 在单元测试中，我们往往想去独立地去测一个类中的某个方法，但是这个类可不是独立的，它会去调用一些其它类的方法和service，这也就导致了以下两个问题：外部服务可能无法在单元测试的环境中正常工作，因为它们可能需要访问数据库或者使用一些其它的外部系统。我们的测试关注点在于这个类的实现上，外部类的一些行为可能会影响到我们对本类的测试，那也就失去了我们进行单测的意义。



<!-- more -->



## 一、mock测试和Mock对象

mock对象就是在调试期间用来作为真实对象的替代品
mock测试就是在测试过程中，对那些不容易构建的对象用一个虚拟对象来代替测试的方法就叫mock测试
## 二、Mockito和PowerMock
  PowerMock是Java开发中的一种Mock框架，用于单元模块测试。当你想要测试一个service接口，但service需要经过防火墙访问，防火墙不能为你打开或者你需要认证才能访问。遇到这样情况时，你可以在你能访问的地方使用MockService替代，模拟实现获取数据。
  PowerMock可以实现完成对private/static/final方法的Mock（模拟），而Mockito可以对普通的方法进行Mock，如：public等。

## 三、Mockito的使用
```java
// 1、模拟HttpServletRequest对象，不需要依赖web容器，模拟获得请求参数
HttpServletRequest request = mock(HttpServletRequest.class); 
when(request.getParameter("foo")).thenReturn("boo");
// 注意:mock()是Mockito的静态方法，可以用@mock注解替换
private @mock HttpServletRequest request
```
```java
// 2、Person person =mock(Person.class);
// 第一次调用返回"xiaoming"，第二次调用返回"xiaohong"
when(person.getName()).thenReturn("xiaoming").thenReturn("xiaohong"); 
when(person.getName()).thenReturn("xiaoming", "xiaohong"); 
when(person.getName()).thenReturn("xiaoming"); 
when(person.getName()).thenReturn("xiaohong");
```
```java
// 3、mockito模拟测试无返回值的方法
Person person =mock(Person.class);
doNothing().when(person).remove();
```
```java
// 4、mockito还能对被测试的方法强行抛出异常
Person person =mock(Person.class);
doThrow(new RuntimeException()).when(person).remove();
when(person.next()).thenThrow(new RuntimeException());
```
```java
// 5、//UserAppService用于参数匹配器的demo
参数匹配器
    UserApp app = new UserApp();
    app.setAppKey("q1w2e3r4t5y6u7i8o9p0");
    app.setAppSecret("q1w2e3r4t5y6u7i8o9p0");
    when(userAppMapper.getAppSecretByAppKey(argThat(new ArgumentMatcher<String>() {
        @Override
        public boolean matches(Object argument) {
            String arg = (String) argument;
            if (arg.equals("1234567890") || arg.equals("q1w2e3r4t5y6u7i8o9p0")) {
                return true;
            } else {
                throw new RuntimeException();
            }
        }
    }))).thenReturn(app);
```

```java
// 6、Answer接口模拟根据参数返回不同结果
    when(userAppMapper.getAppSecretByAppKey(anyString())).thenAnswer(
            (InvocationOnMock invocationOnMock) -> {
                String arg = (String) invocationOnMock.getArguments()[0];
                if (null == arg || arg.equals(null)) {
                    return null;
                } else if (arg.equals("q1w2e3r4t5y6u7i8o9p0")) {
                    UserApp app = new UserApp();
                    app.setAppKey("q1w2e3r4t5y6u7i8o9p0");
                    app.setAppSecret("q1w2e3r4t5y6u7i8o9p0");
                    return app;
                } else {
                    return null;
                }

            });

```

```java
// 7、Mock对象是能调用模拟方法，调用不了它真实的方法，但是spy() 或者@spy 可以监视一个真实的对象，对它进行方法调用时它将调用真实的方法，同时也可以设定这个对象的方法让它返回我们的期望值。同时，我们也可以用verify进行验证。
class A {
  public void goHome() {  
      System.out.println("I say go go go!!"); 
       return true; 
   }  
 }
  //  当需要整体Mock，只有少部分方法执行真正部分时，选用这种方式 
   A mockA = Mockito.mock(A.class);   
   Mockito.doCallRealMethod().when(mockA).goHome(); 
  // 当需要整体执行真正部分，只有少部分方法执行mock，选用这种方式  
   A spyA = Mockito.spy(new A());   
   Mockito.when(spyA.goHome()).thenReturn(false); 
```

**Demo演示**

```java
//目标测试类
@Service
public class UserAppService {

    @Autowired
    private UserAppMapper userAppMapper;

    /**
     * 通过appKey查询AppSecre
     * @return
     */
    public String getAppSecretByAppKey(String appKey){
        if (StringUtils.isEmpty(appKey)) {
            return null;
        }
        UserApp userApp = userAppMapper.getAppSecretByAppKey(appKey);
        if (null == userApp) {
            return null;
        }
        return userApp.getAppSecret();
    }
}
@RunWith(SpringJUnit4ClassRunner.class)
public class UserAppServiceTest {
    @InjectMocks //创建一个实例，其余用@Mock（或@Spy）注解创建的mock将被注入到用该实例中
    private UserAppService userAppService;
    @Mock
    private UserAppMapper userAppMapper;
    @Before
    public void setUp() { MockitoAnnotations.initMocks(this);  }//初始化Mock对象
    @Test
    public void getAppSecretByAppKey3() throws Exception {
        when(userAppMapper.getAppSecretByAppKey(anyString())).thenAnswer(
                (InvocationOnMock invocationOnMock) -> {
                    String arg = (String) invocationOnMock.getArguments()[0];
                    if (null == arg || arg.equals(null)) {
                        return null;
                    } else if (arg.equals("q1w2e3r4t5y6u7i8o9p0")) {
                        UserApp app = new UserApp();
                        app.setAppKey("q1w2e3r4t5y6u7i8o9p0");
                        app.setAppSecret("q1w2e3r4t5y6u7i8o9p0");
                        return app;
                    } else {
                        return null;
                    }

                });
        assertEquals(userAppService.getAppSecretByAppKey("q1w2e3r4t5y6u7i8o9p0"), "q1w2e3r4t5y6u7i8o9p0");
        assertEquals(userAppService.getAppSecretByAppKey("123456789"), null);
        assertEquals(userAppService.getAppSecretByAppKey(null), null);
        verify(userAppMapper, only()).getAppSecretByAppKey(anyString());
    }
// 注意：verify记录着这个模拟对象调用了什么方法，调用了多少次，never() 没有被调用，相当于 times(0)，atLeast(N) 至少被调用 N 次，atLeastOnce() 相当于 atLeast(1)，atMost(N) 最多被调用 N 次
// 参数匹配也可以为：verify(mock).someMethod(anyInt(), anyString()); 
```

## 四、PowerMock的使用

PowerMock基于Mockito开发，起语法规则与Mockito一致，主要区别在于使用方面，以实现完成对**private/static/fina**l等方法(也支持mock的对象是在方法内部new出来的)的Mock（模拟）。具体事例如下：

**依赖**

```xml
<dependency>
   <groupId>org.powermock</groupId>
   <artifactId>powermock-module-junit4</artifactId>
   <version>${powermock.version}</version>
   <scope>test</scope>
   <exclusions>
      <exclusion>
         <artifactId>objenesis</artifactId>
         <groupId>org.objenesis</groupId>
      </exclusion>
   </exclusions>
</dependency>
<dependency>
   <groupId>org.powermock</groupId>
   <artifactId>powermock-api-mockito</artifactId>
   <version>${powermock.version}</version>
   <scope>test</scope>
</dependency>
```

```html
//2、 PowerMock有两个重要的注解：
      –@RunWith(PowerMockRunner.class)
      –@PrepareForTest( { YourClassWithEgStaticMethod.class })
     // 如果你的测试用例里没有使用注解@PrepareForTest，那么可以不用加注解@RunWith(PowerMockRunner.class)，反之亦然。当你需要使用PowerMock强大功能（Mock静态、final、私有方法等）的时候，就需要加注解@PrepareForTest。

```

```java
//测试类
public class ClassUnderTest {  
    public boolean callArgumentInstance(File file) {  
        return file.exists();  
    }   
    public boolean callFinalMethod(ClassDependency refer) {  
        return refer.isAlive();  
    }  
    public boolean callStaticMethod() {  
        return ClassDependency.isExist();  
    }  
    public boolean callPrivateMethod() {  
       return ClassDependency.delete(); 
    }  
}  
//依赖类
public class ClassDependency {  
    public static boolean isExist() {   
        return false;  
    }  
    public final boolean isAlive() {  
        return false;  
    }  
    priavte final boolean delete() {  
        return false;  
    }  
}   
```

```java
// 2、Mock方法内部new出来的对象
public void testCallInternalInstance() throws Exception {  
    File file = PowerMockito.mock(File.class);  
    ClassUnderTest underTest = new ClassUnderTest();  
    PowerMockito.whenNew(File.class).withArguments("bbb").thenReturn(file);  
    PowerMockito.when(underTest.callArgumentInstance( new File("bbb"))).thenReturn(true);  
    PowerMockito.when(file.exists()).thenReturn(true); 
    Assert.assertTrue(file.exists(); 
}   
```

```java
// 3、Mock普通对象的final方法
public void testCallFinalMethod() {  
    ClassDependency depencency = PowerMockito.mock(ClassDependency.class);  
    ClassUnderTest underTest = new ClassUnderTest();  
    PowerMockito.when(depencency.isAlive()).thenReturn(true);  
    Assert.assertTrue(underTest.callFinalMethod(depencency)); 
}  
```

```java
// 4、Mock静态方法
public void testCallStaticMethod() {  
    ClassUnderTest underTest = new ClassUnderTest();  
    PowerMockito.mockStatic(ClassDependency.class);  
    PowerMockito.when(ClassDependency.isExist()).thenReturn(true);  
    Assert.assertTrue(underTest.callStaticMethod());  
}
```

```java
// 5、Mock私有方法
public void testCallPrivateMethod() throws Exception {  
    ClassUnderTest underTest = PowerMockito.mock(ClassUnderTest.class);  
    PowerMockito.when(underTest,"callPrivateMethod").thenCallRealMethod(); 
    Assert.assertTrue(underTest.callPrivateMethod());  
 }
```





---

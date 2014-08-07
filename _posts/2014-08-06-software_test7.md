---
layout: post
title: "单元测试工具对比"
description: "单元测试的定义、测试对象、时间、人员、内容、相关概念"
category: software_test
tags: [软件测试]
fullview: false
---

{% include JB/setup %}
#单元测试工具对比

##Junit or TestNg
TestNG和JUnit是针对Java语言的两个比较常用的测试框架。JUnit出现的比较早，但是早期的JUnit 3对测试代码有非常多的限制，使用起来很不方便，后来的JUnit 4得到很大的改进。TestNG的出现介于JUnit 3和JUnit 4，但是TestNG在很多方面还要优于JUnit 4。下面从整体上对TestNG和JUnit 4进行比较全面的比较。  
  
###TestNG与JUnit的相同点：
1. 使用annotation，且大部分annotation相同。
2. 都可以进行单元测试（Unit test）。
3. 都是针对Java测试的工具。

###TestNG与JUnit的不同点：
1. JUnit只能进行单元测试，TestNG可以进行单元测试，功能测试，端到端测试，集成测试等。
2. TestNG需要一个额外的xml配置文件，配置测试的class、method甚至package。
3. TestNG的运行方式更加灵活：命令行、ant和IDE，JUnit只能使用IDE。
4. TestNG的annotation更加丰富和易懂，比如@ExpectedExceptions、@DataProvider等。
5. 测试套件运行失败，JUnit 4会重新运行整个测试套件。TestNG运行失败时，会创建一个XML文件说明失败的测试，利用这个文件执行程序，就不会重复运行已经成功的测试。


###TestNG比JUnit 4灵活性的体现：
1. JUnit 4中必须把@BeforeClass修饰的方法声明为public static，这就限制了该方法中使用的变量必须是static。而TestNG中@BeforeClass修饰的方法可以跟普通函数完全一样。
2. JUnit 4测试的依赖性非常强，测试用例间有严格的先后顺序。前一个测试不成功，后续所有的依赖测试都会失败。TestNG 利用@Test 的dependsOnMethods属性来应对测试依赖性问题。某方法依赖的方法失败，它将被跳过，而不是标记为失败。
3. 对于n个不同参数组合的测试，JUnit 4要写n个测试用例。每个测试用例完成的任务基本是相同的，只是受测方法的参数有所改变。TestNG的参数化测试只需要一个测试用例，然后把所需要的参数加到TestNG的xml配置文件中。这样的好处是参数与测试代码分离，非程序员也可以修改参数，同时修改无需重新编译测试代码。
4. 为了测试无法用String或原语值表示的复杂参数化类型，TestNG提供的@DataProvider使它们映射到某个测试方法。
5. JUnit 4的测试结果通过Green/Red bar体现，TestNG的结果除了Green/Red bar，还有Console窗口和test-output文件夹，对测试结果的描述更加详细，方便定位错误。

####总结
在TestNG包含了JUnit4的核心功能，并且[迁移方便](http://blog.csdn.net/jmyue/article/details/9245947)的情况下，似乎没有理由不使用TestNg。


参考：    
<http://blog.csdn.net/jmyue/article/details/9041357>    
<http://www.ituring.com.cn/article/47829>

##Java or Groovy

###JAVA
Java语言自不用多说，我想看这篇文章的人，都是熟悉Java语言的码农了。

###Groovy
Groovy是一种基于JVM（Java虚拟机）的敏捷开发语言，它结合了Python、Ruby和Smalltalk的许多强大的特性，Groovy 代码能够与 Java 代码很好地结合，也能用于扩展现有代码。由于其运行在 JVM 上的特性，Groovy 可以使用其他 Java 语言编写的库。

####Groovy有何吸引力
1. 它与Java平台无缝的集成：运行在JVM上，可以调用JAVA语言编写的库，学习曲线很坡。
2. 它结合了Python、Ruby和Smalltalk的许多强大的特性，更适合小型的、非常特殊的、不是性能密集型的应用程序。对报表和单元测试来说，是近乎完美的选择。当然如果系统复杂，强调设计的话，乖乖用Java去。

####看一下Java和Groovy的对比
JAVA语言编写的测试用例：

        protected void test()
            throws Exception
        {
            Assert.assertNotNull( service );
            Context.set( test_kp );
            SmartDevice smartDevice = new SmartDevice();
            smartDevice.setApp_ver( "2014" );
            smartDevice.setBind_date( "2014" );
            smartDevice.setBind_status( "2014" );
            smartDevice.setDetail( "{}" );
            smartDevice.setDevice_id( test_device_id );
            smartDevice.setModel( "BMW" );
            smartDevice.setOs_ver( "2014" );
            smartDevice.setProduct_id( test_product_id );
            smartDevice.setProduct_type( "01" );
            smartDeviceBizService.bind( Context.get().getUid(), smartDevice );
            String table = ObdLocationService.TABLE;
            String data = getAddData();
            Object ret = service.add( Context.get().getUid(), table, data );
            Assert.assertNotNull( ret );
            // System.err.println( JSON.toJSONString( ret ) );
            JSONArray arrayResult = JSON.parseArray( JSON.toJSONString( ret ) );
            Assert.assertEquals( 6, arrayResult.size() );
            Assert.assertEquals( DataValidateEnum.SUCCESS.getCode(), arrayResult.getJSONObject( 0 ).get( "code" ) );
            Assert.assertEquals( DataValidateEnum.FAILURE.getCode(), arrayResult.getJSONObject( 1 ).get( "code" ) );
            Assert.assertEquals( DataValidateEnum.FAILURE.getCode(), arrayResult.getJSONObject( 2 ).get( "code" ) );
            Assert.assertEquals( DataValidateEnum.FAILURE.getCode(), arrayResult.getJSONObject( 3 ).get( "code" ) );
            Assert.assertEquals( DataValidateEnum.FAILURE.getCode(), arrayResult.getJSONObject( 4 ).get( "code" ) );
            Assert.assertEquals( DataValidateEnum.FAILURE.getCode(), arrayResult.getJSONObject( 5 ).get( "code" ) );
        }

使用groovy改写后：

        void test() {
            assertNotNull service
            Context.set(test_kp)
            SmartDevice smartDevice = new SmartDevice()
            smartDevice.with {
                app_ver = "2014"
                bind_date = "2014"
                detail = "{}"
                device_id = test_device_id
                model = "BMW"
                os_ver = "2014"
                product_id = test_product_id
                product_type = "01"
            }
            def uid = Context.get().uid
            smartDeviceBizService.bind(uid, smartDevice)
            def table = ObdLocationService.TABLE
            def data = getAddData()
            def ret = service.add(uid, table, data)
            def jsonarr = new JsonSlurper().parseText(JSON.toJSONString(ret))
            jsonarr.eachWithIndex {o, i ->
                if (i == 0)
                    assertEquals o["code"], DataValidateEnum.SUCCESS.code
                else
                    assertEquals o["code"], DataValidateEnum.FAILURE.code
            }
			}
####总结
在Groovy学起来简单，又可以优雅写测试代码的情况下，似乎没有理由不用Groovy.

##Spock

####使用groovy语言编写测试用例，强大的asserts。
def "subscribers receive published events at least once"() {
    when: publisher.send(event)
    then: (1.._) * subscriber.receive(event)
    where: event << ["started", "paused", "stopped"]
}
####自带mock

####强大的asserts构造

####比testNG生成的报告更加友好，信息更加详细

###总结
似乎没有理由不使用Spock。

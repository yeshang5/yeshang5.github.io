---
layout: post
title: '使用jxls实现基于模板导出Excel'
date: 2018-10-22
author: 白皓
cover: 'https://s1.ax1x.com/2018/10/22/iDB1vq.jpg'
tags: java excel poi jxls 导出
---


##  一.简介

在很多涉及到某种报表功能的Java程序中都需要生成Excel表格。目前通过Java来操作.xls文件最完整的类库是Apache POI类库，但是当需要创建多种自定义的复杂Excel报表的时候就会出现问题，这些Excel报表一般都带有多种格式和可扩展功能，在这种情况下，你就不得不写一大堆Java代码来创建报表的规则集（workbook），规则集一般包含所有要求的格式,公式,其他特定的设置和正确的Java对象集的数据出口。这些代码一般都是难以调试，任务也常常变得容易出错并且耗时。

另外一个问题是有很多Excel组件都没有提供的API。幸运的是POI API读取Excel文件，可以保持它原有的格式，然后根据需要进行修改。很明显，用一些Excel编辑工具来创建所有格式正确的报告模板然后指定真实的数据应该放置的地方，会容易很多。JXLS是实现这种方法并且只用几行代码就能创建极其复杂的Excel报表。你只需要用特定的标记来创建一个带有所有要求的格式,公式,宏等规则的.xls模板文件来指定数据放置的位置然后再写几行代码来调用JXLS引擎来传递.xls模板和导出的数据作为参数。

除了生成Excel报表功能，JXLS还提供了jxls-reader模块，jxls-reader模块会很有用，如果你需要解析一个预定义格式的Excel文件并在其中插入数据的话。jxls-reader允许你用一个简单的XML文件描述解析规则，读取Excel文件和你的各种JAVA对象（population of yourJava objects）的所有其他工作都会自动完成。



---
##  二.添加依赖

根据不同项目构建工具添加依赖

__Maven项目__：
```xml
    <dependency>
    <groupId>net.sf.jxls</groupId>
    <artifactId>jxls-core</artifactId>
    <version>1.0.6</version>
    </dependency>
```

__Gradle项目__
```java
    compile group: 'net.sf.jxls', name: 'jxls-core', version: '1.0.6'
```

    注意：当前JXLS版本可能无法正常地与较早的POI库版本工作，因此，如果你必须要使用较早版本的POI(prior3.2)使用较老版本的JXLS就行了

##  三．JXLS参考

### 1.简介

这部分描述在.xlsx模板文件的中的对象属性访问语法，如果想让JXLS引擎进行正确的处理.xlsx模板文件就必须使用规定的语法。

接下来的部分假设我们有两个相互依赖的JAVA beans，类型分别为Department和Employee，在代码中像这样被传递到XLSTransformer中：
```java
    Departmentdepartment;

    ...//initialization

    Map beans =new HashMap();

    beans.put("department",department);

    XLSTransformertransformer = new XLSTransformer(); 
    transformer.transformXLS(xlsTemplateFileName,beans, outputFileName);

```

### 2.属性访问
2.1 基本属性访问

使用下面的语句来访问Excel单元格中简单的bean属性：
```java
    ${department.name}
```

在上面这个语句中，JXLS引擎会通过关键字department在当前bean映射下搜索这个bean,然后会尝试获取这个bean的name属性的值并把它放到相应的Excel单元格中。

同理，我们可以访问更加复杂的属性，例如，要输出这个department中的属性chief中的name属性的值，我们可以用：

```java
    ${department.chief.name}
```
访问任何深度的对象属性都是可以的。例如:

```java
    ${bean.bean1.bean2.bean3.bean4.bean5.bean6.bean7.bean8.bean9.beanX.property1}
```
2.2 多个属性在一个单元格中

在一个单元格，我们可以连接几个属性。例如：

```java
    Employee:${employee.name} - ${employee.age} years
```
这样，我们得到的输出是：

```java
    Employee: John -35 years
```
其中${employee.name}的值是John,同理${employee.age}的值是35.

### 3.使用标签

JXLS允许在模板中使用预定义的XML标签来控制XLS转换行为。

3.1 jx:forEach标签

<jx:forEach>标签的典型用法如下：

```xml
<jx:forEach  items="${departments}"  var="department" >

       ${department.name}| ${department.chief}

 </jx:forEach>
 ```
jx标签可以相互嵌套使用

如果你把jx:forEach标签的开始标签和结束标签放在同一行的话，JXLS会在同一行上重复在jx:forEach标签的开始标签和结束标签之间的Excel单元格。

目前，如果你想要用jx:forEach标签重复Excel的行，那么你必须把jx:forEach标签的开始标签和结束标签放在不同的行，把要重复的行包含在中间，jx:forEach标签所在行的所有单元格都会被忽略。



实例：

创建模板如图 
！[模板](https://s1.ax1x.com/2018/10/22/iDBNaF.png)

后台代码如下：

```java
    private Resource templateResource;


    /**
     * 初始化文件
     */
    @PostConstruct
    public  void initResource()
    {
        templateResource=new ClassPathResource("/ExcelTemplates/userTemplate1.xlsx");
    }

    @ResponseBody
    @RequestMapping("/exportExcel")
    public String exportExcel(HttpServletResponse response)
    {
        try {

            User user1=new User("周旋",22,"男","光头","公司搬砖码农",2500.12);
            User user2=new User("朱凯欣",23,"男","长发","女装大佬",4000);
            User user3=new User("龚根华",23,"男","地中海","加班到猝死",1800);
            User user4=new User("伍嘉威",24,"男","发际线后移严重","回家继承鱼塘",500000);
            User user5=new User("李善文",24,"男","没有头发","裸贷大佬",3000);

            List<User> user=new ArrayList<User>();
            user.add(user1);
            user.add(user2);
            user.add(user3);
            user.add(user4);
            user.add(user5);

            Map<String,Object> data=new HashMap<>();
            data.put("userList",user1);
            //渲染数据到模板中
            Workbook workbook=new XLSTransformer().transformXLS(templateResource.getInputStream(),data);
            ByteArrayOutputStream outputStream=new ByteArrayOutputStream();

            workbook.setForceFormulaRecalculation(true);
            //输出文件流
            workbook.write(outputStream);

            String fileName="319统计情况表.xlsx";
            response.reset();
            response.setContentType(String.format("application/octet-stream; charset=utf-8"));
            response.setHeader("Content-Disposition", "attachment; filename="+new String(fileName.getBytes("UTF-8"),"iso8859-1"));
            response.getOutputStream().write(outputStream.toByteArray());

        } catch (InvalidFormatException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "下载成功";
    }
```


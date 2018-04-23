---
title: MyBatis批量操作
date: 2018-04-23 19:09:34
tags:
    - MyBatis
categories:
    - MyBatis
---
**Mapper.xml需要注意：**  
1.当parameterType为java.util.List时，foreach的collection值必须为list  
2.当parameterType为java.util.Map时，foreach的collection值必须与Map中的键值一样  
3.当parameterType为Bean时，foreach的collection值必须与Bean中的属性名一样

# 1. 批量添加
session.insert(String string, Object o)
mapper.batchInsertStudent(List<Student> stuList)
```
public void batchInsertStudent(){  
    List<Student> ls = new ArrayList<Student>();  
    for(int i = 5;i < 8;i++){  
        Student student = new Student();  
        student.setId(i);  
        student.setName("maoyuanjun" + i);  
        student.setSex("man" + i);  
        student.setTel("tel" + i);  
        student.setAddress("浙江省" + i);  
        ls.add(student);  
    }  
    SqlSession session = SessionFactoryUtil.getSqlSessionFactory().openSession();  
    session.insert("mybatisdemo.domain.Student.batchInsertStudent", ls);  
    session.commit();  
    session.close();  
}  

public void batchInsertStudent(List<Student> stuList);  

<!--List包裹Map或者Bean对象,此处collection必须填list-->
<insert id="batchInsertStudent" parameterType="java.util.List">  
    INSERT INTO STUDENT (id,name,sex,tel,address)  
    VALUES   
    <foreach collection="list" item="item" index="index" separator="," >  
        (#{item.id},#{item.name},#{item.sex},#{item.tel},#{item.address})  
    </foreach>  
</insert>
```

# 2. 批量修改  
## 实例1：  
session.update(String string, Object o)  
mapper.batchUpdateStudent(List<Long> idList)
```  
public void batchUpdateStudent(){  
    List<Long> ls = new ArrayList<Long>();  
    for(int i = 2;i < 8;i++){  
        ls.add(i);  
    }  
    SqlSession session = SessionFactoryUtil.getSqlSessionFactory().openSession();  
    session.update("mybatisdemo.domain.Student.batchUpdateStudent", ls);  
    session.commit();  
    session.close();  
}  

public void batchUpdateStudent(List<Long> idList);

<update id="batchUpdateStudent" parameterType="java.util.List">  
    UPDATE STUDENT SET name = "5566" WHERE id IN  
    <foreach collection="list" item="item" index="index" open="(" separator="," close=")" >  
        #{item}  
    </foreach>  
</update>  
```

## 实例2：
session.update(String string, Object o)  
mapper.batchUpdateStudentWithMap(Map<String, Object> map)
**注意：collection="idList"的值idList必须与Map中的键值一样**
```  
public void batchUpdateStudentWithMap(){  
    List<Long> ls = new ArrayList<Long>();  
    for(int i = 2;i < 8;i++){  
        ls.add(i);  
    }  
    Map<String,Object> map = new HashMap<String,Object>();  
    map.put("idList", ls);  
    map.put("name", "mmao789");  
    SqlSession session = SessionFactoryUtil.getSqlSessionFactory().openSession();  
    session.update("mybatisdemo.domain.Student.batchUpdateStudentWithMap", map);  
    session.commit();  
    session.close();  
}  

public void batchUpdateStudentWithMap(Map<String, Object> map);

<!--collection的值idList必须与Map中的属性值一样-->
<update id="batchUpdateStudentWithMap" parameterType="java.util.Map" >  
    UPDATE STUDENT SET name = #{name} WHERE id IN   
    <foreach collection="idList" index="index" item="item" open="(" separator="," close=")">   
        #{item}   
    </foreach>  
</update>
```

这里也可以将Map转换成bean对象，如：  
session.update(String string, Object o)  
mapper.batchUpdateStudentWithMap(StudentVO vo)  
**注意：collection="idList"的值idList必须与Bean中的属性名一样**
```
public Class StudentVO {
  private String name;
  private List<Long> idList;
  //get、set方法省略
}

public void batchUpdateStudentWithMap(StudentVO vo);

<!--collection的值idList必须与Bean中的属性名一样-->
<update id="batchUpdateStudentWithMap" parameterType="com.StudentVO" >  
    UPDATE STUDENT SET name = #{name} WHERE id IN   
    <foreach collection="idList" index="index" item="item" open="(" separator="," close=")">   
        #{item}   
    </foreach>  
</update>
```

# 3. 批量删除
session.delete(String string,Object o)   
mapper.batchDeleteStudent(List<Long> idList)
```
public void batchDeleteStudent(){  
    List<Long> ls = new ArrayList<Long>();  
    for(int i = 4;i < 8;i++){  
        ls.add(i);  
    }  
    SqlSession session = SessionFactoryUtil.getSqlSessionFactory().openSession();  
    session.delete("mybatisdemo.domain.Student.batchDeleteStudent",ls);  
    session.commit();  
    session.close();  
}  

public void batchDeleteStudent(List<Long> idList);

<delete id="batchDeleteStudent" parameterType="java.util.List">  
    DELETE FROM STUDENT WHERE id IN  
    <foreach collection="list" index="index" item="item" open="(" separator="," close=")">   
        #{item}   
    </foreach>  
</delete>
```

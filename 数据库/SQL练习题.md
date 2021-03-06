https://zhuanlan.zhihu.com/p/38354000

**表的关系图**

![image-20201119180036109](http://img.fosuchao.com/image-20201119180036109.png)



![image-20201119180118871](http://img.fosuchao.com/image-20201119180118871.png)

### 初始化表数据

```sql
# 初始化学生表
insert into student(study_num, student_name, birthday, gender)
values('0001' , '猴子' , '1989-01-01' , '男'), ('0002' , '猴子' , '1990-12-21' , '女'), ('0003' , '马云' , '1991-12-21' , '男'), ('0004' , '王思聪' , '1990-05-20' , '男');
# 初始化分数表
insert into score(study_num, course_num, score)
values('0001' , '0001' , 80),('0001' , '0002' , 90),('0001' , '0003' , 99),('0002' , '0002' , 60),('0002' , '0003' , 80),('0003' , '0001' , 80),('0003' , '0002' , 80),('0003' , '0003' , 80);
# 初始化课程表
insert into course(course_num, course_name, teacher_num)
values('0001' , '语文' , '0002'),('0002' , '数学' , '0001'),('0003' , '英语' , '0003');
# 初始化教师表
insert into teacher(teacher_num, teacher_name)
values('0001' , '孟扎扎'),('0002' , '马化腾'),('0003' , '李明');

```

### 汇总分析

##### 查询课程编号为“0002”的总成绩

```sql
select sum(t1.score) as score_sum from score t1 where t1.course_num = '0002'
```

![image-20201120093050474](http://img.fosuchao.com/image-20201120093050474.png)



##### 查询选了课程的学生人数

```sql
select count(distinct t1.study_num) as count from score t1;
```

![image-20201120093104611](http://img.fosuchao.com/image-20201120093104611.png)



##### 查询学生的总成绩并进行排名

```sql
select s.study_num, sum(s.score) as score_sum
from score s
group by s.study_num
order by score_sum DESC
```

![image-20201120102112876](http://img.fosuchao.com/image-20201120102112876.png)



##### 查询平均成绩大于60分的学生的学号和平均成绩

```sql
select s.study_num, avg(s.score) as avg_score
from score s
group by s.study_num
having avg_score > 60
```

![image-20201120102240074](http://img.fosuchao.com/image-20201120102240074.png)



### 分组 

##### 查询各科成绩最高和最低的分， 显示：课程号，最高分，最低分

```sql
select s.course_num, max(s.score) as max_score, min(s.score) as min_score
from score s
group by s.course_num;
```

![image-20201120093139285](http://img.fosuchao.com/image-20201120093139285.png)



##### 查询每门课程被选修的学生数，显示：课程号，选修人数

```sql
select s.course_num, count(s.study_num) as count
from score s
group by s.course_num;
```

![image-20201120093157911](http://img.fosuchao.com/image-20201120093157911.png)



##### 查询男生、女生人数

```sql
select s.gender as gender, count(*) as count
from student s
group by s.gender;
```

![image-20201120092934840](http://img.fosuchao.com/image-20201120092934840.png)



### 分组结果的条件

##### 查询平均成绩大于60分的学生的学号和平均成绩

```sql
select s.study_num as study_num, avg(s.score) as avg_score
from score s
group by s.study_num
having avg(s.score) > 60;
```

Gourd by 条件用having

![image-20201120093827193](http://img.fosuchao.com/image-20201120093827193.png)



##### 查询至少选修三门课程的学生学号

```sql
select s.study_num as study_num, count(s.course_num) as course_count
from score s
group by s.study_num
having count(course_num) >= 3;
```

![image-20201120095256808](http://img.fosuchao.com/image-20201120095256808.png)



##### 查询同名同姓学生名单并统计同名人数

```sql
select s.student_name as student_name ,count(*) as count
from student s
group by s.student_name;
```

![image-20201120095537926](http://img.fosuchao.com/image-20201120095537926.png)



##### 查询不及格的课程并按课程号从大到小排列

```sql
select s.course_num
from score s
where s.score < 60
order by s.course_num desc;
```

![image-20201120100029845](http://img.fosuchao.com/image-20201120100029845.png)



##### 查询每门课程的平均成绩，结果按平均成绩升序排序，平均成绩相同时，按课程号降序排列

```sql
select avg(s.score) as avg_score
from score s
group by s.course_num
order by avg_score, s.course_num DESC;
```

![image-20201120100417297](http://img.fosuchao.com/image-20201120100417297.png)



##### 检索课程编号为“0004”且分数小于60的学生学号，结果按按分数降序排列

```sql
select s.study_num as study_num , s.score as score
from score s
where s.course_num = '0003'
and s.score < 60
order by s.score DESC
```

![image-20201120100844350](http://img.fosuchao.com/image-20201120100844350.png)



##### 统计每门课程的学生选修人数(超过2人的课程才统计)

要求输出课程号和选修人数，查询结果按人数降序排序，若人数相同，按课程号升序排序

```sql
select s.course_num as course_num, count(s.study_num) as student_count
from score s
group by s.course_num
having count(s.study_num) > 2
order by student_count DESC, s.course_num;
```

![image-20201120101242058](http://img.fosuchao.com/image-20201120101242058.png)



##### 查询一门以上不及格课程的同学的学号及其平均成绩

```sql
select s.study_num, avg(s.score) as avg_score
from score s
where s.score < 60
group by s.study_num
having count(s.course_num) >= 2
```

![image-20201120101725797](http://img.fosuchao.com/image-20201120101725797.png)



### 复杂查询

##### 查询所有课程成绩小于60分学生的学号、姓名(子查询)

```sql
select std.study_num, std.student_name
    from student std
        where std.study_num in (select s.study_num as study_num
            from score s
            where s.score < 60);
```

![image-20201120103154361](http://img.fosuchao.com/image-20201120103154361.png)



##### 查询没有学全所有课的学生的学号、姓名

```sql
select std.study_num, std.student_name
from student std
where std.study_num
in (select s.study_num from score
    as s group by s.study_num
        having count(s.course_num) <
            (select count(*) from course))
```

![image-20201120104931977](http://img.fosuchao.com/image-20201120104931977.png)



##### 查询出只选修了两门课程的全部学生的学号和姓名

```sql
select std.study_num, std.student_name
from student std
where std.study_num in
    (select s.study_num from score as s group by s.study_num
    		having count(s.course_num) = 2)
```

<img src="http://img.fosuchao.com/image-20201120105144111.png" alt="image-20201120105144111"  />



##### 查询各科成绩前两名的记录

```sql
(select * from score where course_num = '0001' order by score  desc limit 2)
union all
(select * from score where course_num = '0002' order by score  desc limit 2)
union all
(select * from score where course_num = '0003' order by score  desc limit 2);
```

![image-20201120113353558](http://img.fosuchao.com/image-20201120113353558.png)



### 多表查询

##### 查询所有学生的学号、姓名、选课数、总成绩

```sql
select std.study_num, std.student_name, count(s.course_num) as course_count, sum(s.score) score_sum
from student as std
join score as s
on std.study_num = s.study_num
group by std.study_num
```

![image-20201120110059012](http://img.fosuchao.com/image-20201120110059012.png)



##### 查询平均成绩大于85的所有学生的学号、姓名和平均成绩

```sql
select std.study_num, std.student_name, avg(s.score) as avg_score
from student std
join score s on std.study_num = s.study_num
group by std.study_num
having avg_score > 85;
```

![image-20201120111125279](http://img.fosuchao.com/image-20201120111125279.png)



##### 查询学生的选课情况：学号，姓名，课程号，课程名称

```sql
select std.study_num, std.student_name, c.course_num, c.course_name
from student std
inner join score s on std.study_num = s.study_num
inner join course c on s.course_num = c.course_num
```

![image-20201120111628187](http://img.fosuchao.com/image-20201120111628187.png)



##### 查询出每门课程的及格人数和不及格人数

```sql
select s.course_num,
       sum(case when s.score >= 60 then 1 else 0 end) as pass,
       sum(case when s.score < 60 then 1 else 0 end)  as unpass
from score s
group by s.course_num;
```

![image-20201120112401016](http://img.fosuchao.com/image-20201120112401016.png)



##### 查询课程编号为0003且课程成绩在80分以上的学生的学号和姓名

```sql
select std.study_num, std.student_name
from student as std
join score s on std.study_num = s.study_num
where s.score > 80 and s.course_num = '0003'
```

![image-20201120112742635](http://img.fosuchao.com/image-20201120112742635.png)



##### 检索"0003"课程分数小于60，按分数降序排列的学生信息

```sql
select std.study_num, std.student_name, s.score as score
from student std
join score s on std.study_num = s.study_num
and s.course_num = '0003'
where s.score < 60
order by score DESC
```

![image-20201120161938103](http://img.fosuchao.com/image-20201120161938103.png)



##### 查询不同老师所教不同课程平均分从高到低显示

```sql
select t.teacher_name, c.course_num, avg(s.score) avg_score
from teacher t
join course c on t.teacher_num = c.teacher_num
join score s on c.course_num = s.course_num
group by t.teacher_name
order by avg_score DESC
```

![image-20201120163059737](http://img.fosuchao.com/image-20201120163059737.png)



##### 查询课程名称为"数学"，且分数低于60的学生姓名和分数

```sql
select std.student_name, s.score
from student std
join score s on std.study_num = s.study_num
join course c on s.course_num = c.course_num
where s.score < 60 and c.course_name = '数学'
```

![image-20201120163355987](http://img.fosuchao.com/image-20201120163355987.png)



##### 查询任何一门课程成绩在70分以上的学生姓名、课程名称和分数

```sql
select std.student_name, c.course_name, s.score
from student std
join score s on std.study_num = s.study_num
join course c on s.course_num = c.course_num
where s.score > 70
```

![image-20201120164003002](http://img.fosuchao.com/image-20201120164003002.png)



##### 查询一门及其以上不及格课程的同学的学号，姓名及其平均成绩

```sql
select std.study_num, std.student_name, avg(s.score) as avg_score
from student std
join score s on std.study_num = s.study_num
where s.score < 60
group by std.student_name
having count(s.course_num) >= 1
```

![image-20201120164421962](http://img.fosuchao.com/image-20201120164421962.png)



##### 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

```sql
select distinct s1.study_num, s1.course_num, s1.score
from score s1
join score s2 on s1.study_num = s2.study_num
where s1.course_num != s2.course_num and s1.score = s2.score
```

![image-20201120165433383](http://img.fosuchao.com/image-20201120165433383.png)



##### 查询课程编号为“0001”的课程比“0002”的课程成绩高的所有学生的学号

```sql
select distinct s1.study_num
from score s1
join score s2 on s1.study_num = s2.study_num
where s1.score > s2.score and s1.course_num = '0001' and s2.course_num = '0002'

select a.study_num
from
(select study_num, score from score where course_num = '0001') as a
join
(select study_num, score from score where course_num = '0002') as b
on a.study_num = b.study_num
join student c on a.study_num = c.study_num
where a.score > b.score
```

![image-20201120165829066](http://img.fosuchao.com/image-20201120165829066.png)



##### 查询学过编号为“0001”的课程并且也学过编号为“0002”的课程的学生的学号、姓名

```sql
select a.study_num
from
(select study_num, score from score where course_num = '0001') as a
join
(select study_num, score from score where course_num = '0002') as b
on a.study_num = b.study_num
join student std
on std.study_num = a.study_num
```

![image-20201120170409544](http://img.fosuchao.com/image-20201120170409544.png)



##### 查询学过“孟扎扎”老师所教的所有课的同学的学号、姓名

```sql
select std.study_num, std.student_name
from teacher t
join course c on t.teacher_num = c.teacher_num
join score s on c.course_num = s.course_num
join student std on s.study_num = std.study_num
where t.teacher_name = '孟扎扎'
```

![image-20201120170813677](http://img.fosuchao.com/image-20201120170813677.png)



##### 查询没学过"马化腾"老师讲授的任一门课程的学生姓名

与上题类似，"没学过"用not in来实现

```sql
select std.study_num, std.student_name
from student std
where std.study_num not in (
    select score.study_num from score where score.course_num = (
        select course.course_num from course where course.teacher_num = (
            select teacher.teacher_num from teacher where teacher.teacher_num = '马化腾'
            )
        )
    )
```


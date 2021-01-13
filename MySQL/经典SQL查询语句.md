#### 查询不重复

select distinct Depart from teacher;

#### 集合

select * from Score where Degree in (85,86,88);

#### 按照顺序查询

select * from Score order by Cno asc,Degree desc;  (sql默认是升序)

#### 查询Score表中的最高分的学生学号和课程号。（子查询或者排序）

select Sno,Cno from Score where Degree = ( select max(Degree) from Score); 

or

select Sno,Cno from Score order by Degree desc limit 0,1;

#### 查询分数大于70，小于90的Sno列

此处需要与上面的区间比较：这里不是标准的区间，而是>70 and <90，因此需要写成

select Sno from Score where Degree >70 and Degree < 90;

#### 多表连接

- 内连接 inner join

  select Sname,Cno,Degree from Student inner join Score on Student.Sno=Score.Sno;

  select Sname,Cname,Degree from Student inner join Score on Student.Sno=Score.Sno inner join  Course on Score.Cno=Course.Cno;

- 外连接 left join

  select Sname,Cno,Degree from Student left join Score on Student.Sno=Score.Sno;

- [外连接与内连接的区别在于](https://www.cnblogs.com/tinyj/p/10035143.html)

- where

  select Sname,Cno,Degree from Student,Score where Student.Sno=Score.Sno;

  select Sname,Cname,Degree from student,course,score where student.Sno=score.Sno and course.Cno=score.Cno;

#### 查询成绩比该课程平均成绩低的同学的成绩表

select * from score a  where degree < ( select avg(degree) from score b where b.cno=a.cno);

#### 查询所有任课教师的Tname和Depart

select Tname,Depart from Teacher where Tno in ( select Tno from Course);

#### 查询所有未讲课的教师的Tname和Depart

select Tname,Depart from Teacher where Tno not in ( select Tno from Course where Cno in ( select Cno from Score ));

#### 查询至少有2名男生的班号

select Class from Student where Ssex = '男' group by Class  having count(Ssex) >1;

#### 查询Student表中每个学生的姓名和年龄

select Sname,year(now())-year(Sbirthday) from Student;




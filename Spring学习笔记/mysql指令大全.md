##### 字符串转日期

```mysql
select * from shoppingCar where MONTH(str_to_date(date, '%Y/%m/%d'))>8;
```


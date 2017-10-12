# practice
#假设你获取了用户输入的日期和时间如2015-1-21 9:01:30，以及一个时区信息如UTC+5:00，均是str，请编写一个函数将其转换为timestamp：
#期望输出
t1 = to_timestamp('2015-6-1 08:10:30', 'UTC+7:00')
assert t1 == 1433121030.0, t1

t2 = to_timestamp('2015-5-31 16:10:30', 'UTC-09:00')
assert t2 == 1433121030.0, t2

print('Pass')


# -*- coding:utf-8 -*-

import re
from datetime import datetime, timezone, timedelta

# -*- coding:utf-8 -*-
import re
from datetime import datetime,timezone,timedelta
def to_timestamp(dt_str,tz_str):
    #cday=datetime.strptime(dt_str,'%Y-%m-%d %H:%M:%S')
    stz = re.match(r'([0-9]+)\-([0-9]+)\-([0-9]+)\s(\d{2})\:(\d{2})',dt_str)
    s1 = int(stz.group(1))
    s2 = int(stz.group(2))
    s3 = int(stz.group(3))
    s4 = int(stz.group(4))
    s5 = int(stz.group(5))


    rtz=re.match(r'UTC(\+|\-)([0-9]+)\:00',tz_str)
    if rtz.group(1)=='+':
        p=int(rtz.group(2))
    if rtz.group(1)=='-':
        p=-int(rtz.group(2))
    if p<8:
            a=8-p
            s4=s4+a
            if s4>24:
                s4=s4-24
                s3=s3+1
                if s3>31:
                    s2+=1
                    s3=s3-31

            dt=datetime(s1,s2,s3,s4,s5)
            print(dt.timestamp())
    else:
        a=int(rtz.group(2))-8
        s4=s4-a
        dt = datetime(s1, s2, s3, s4, s5)
        print(dt.timestamp())
t1 = to_timestamp('2015-6-1 08:10:30', 'UTC+7:00')
t2 = to_timestamp('2015-5-31 16:10:30', 'UTC-09:00')
#以上代码仅考虑了有31天的月份，还不完善

#coding: utf-8
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column,String,Integer
engine=create_engine('mysql+mysqldb://root@localhost:3306/blog?charest=utf8')
Base=declarative_base()
class User(Base):
  __tablename__='users'
  id=Column(Integer,primary_key=True)
  username=Column(String(64),nullable=False,index=True)
  password=Column(String(64),nullable=False)
  email=Column(String(64),nullable=False,index=True)
  def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, self.username)
Base.metadata.create_all(engine)


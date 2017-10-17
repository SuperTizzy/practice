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
# coding: utf-8

import random
from faker import Factory

from sqlalchemy import create_engine, Table
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import ForeignKey
from sqlalchemy import Column, String, Integer, Text
from sqlalchemy.orm import sessionmaker, relationship


engine = create_engine('mysql+mysqldb://root@localhost:3306/blog?charset=utf8')
Base = declarative_base()


class User(Base):

    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    username = Column(String(64), nullable=False, index=True)
    password = Column(String(64), nullable=False)
    email = Column(String(64), nullable=False, index=True)
    articles = relationship('Article', backref='author')
    userinfo = relationship('UserInfo', backref='user', uselist=False)

    def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, self.username)


class UserInfo(Base):

    __tablename__ = 'userinfos'

    id = Column(Integer, primary_key=True)
    name = Column(String(64))
    qq = Column(String(11))
    phone = Column(String(11))
    link = Column(String(64))
    user_id = Column(Integer, ForeignKey('users.id'))


class Article(Base):

    __tablename__ = 'articles'

    id = Column(Integer, primary_key=True)
    title = Column(String(255), nullable=False, index=True)
    content = Column(Text)
    user_id = Column(Integer, ForeignKey('users.id'))
    cate_id = Column(Integer, ForeignKey('categories.id'))
    tags = relationship('Tag', secondary='article_tag', backref='articles')

    def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, self.title)


class Category(Base):

    __tablename__ = 'categories'

    id = Column(Integer, primary_key=True)
    name = Column(String(64), nullable=False, index=True)
    articles = relationship('Article', backref='category')

    def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, self.name)


article_tag = Table(
    'article_tag', Base.metadata,
    Column('article_id', Integer, ForeignKey('articles.id')),
    Column('tag_id', Integer, ForeignKey('tags.id'))
)


class Tag(Base):

    __tablename__ = 'tags'

    id = Column(Integer, primary_key=True)
    name = Column(String(64), nullable=False, index=True)

    def __repr__(self):
        return '%s(%r)' % (self.__class__.__name__, self.name)


if __name__ == '__main__':
    Base.metadata.create_all(engine)

    faker = Factory.create()
    Session = sessionmaker(bind=engine)
    session = Session()

    faker_users = [User(
        username=faker.name(),
        password=faker.word(),
        email=faker.email(),
    ) for i in range(10)]
    session.add_all(faker_users)

    faker_categories = [Category(name=faker.word()) for i in range(5)]
    session.add_all(faker_categories)

    faker_tags= [Tag(name=faker.word()) for i in range(20)]
    session.add_all(faker_tags)

    for i in range(100):
        article = Article(
            title=faker.sentence(),
            content=' '.join(faker.sentences(nb=random.randint(10, 20))),
            author=random.choice(faker_users),
            category=random.choice(faker_categories)
        )
        for tag in random.sample(faker_tags, random.randint(2, 5)):
            article.tags.append(tag)
        session.add(article)

    session.commit()

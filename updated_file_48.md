# Text2GQL语法提示手册
本手册列出了GQL常见的语法点以及参考的提示方法，用户可参考提问示例让模型生成对应的语法的GQL语句。

| 语法 | 提问示例 | 结果 |
| --- | --- | --- |
| 查找点 | 请找出的类型为person**的点** | match(a:person) return a |
| 查找边 | 返回图中所有的标签为knows**的边** | match(a)-[e:knows]->(b) return e |
| 查找关系 | 查询10个属于北京的大学<br />找出5个和小张老师**有关系**的学生 | match(a:city where a.name = '北京')<-[:belong]-(b:university) return b limit 10<br />match(a:teacher where a.name='小张')-[e]-(b:student) return b limit 5 |
| 查找多度关系 | 查找小王同学的朋友认识的人<br /><br />请帮我找出和标签为大学的点有拥有关系的部门,并且和这些部门有拥有关系的学生,并且和这些学生有选择关系的课程<br /><br />请查找腾讯和谷歌公司共同创建的软件的名称，返回5个。 | match(a:student where a.name = '小王')-[e:friend]->(b)-[e2:knows]->(c:person) return c<br /><br />match(a:university)-[e:has]->(b:department)-[e2:has]->(c:student)-[e3:selects]->(d:course) return d<br /><br />match(a:company where a.name='腾讯')-[e:creates]->(b:software)<-[e2:creates]-(c:company where c.name='谷歌') return b.name limit 5 |
| Loop | 查询从person张思琪出发，经过类型为支付的边，**走2到4度到达的点** | match(a:person where a.name='张思琪')-[e:pay]->{2,4}(b:person) return b |
| Loop(环) | 找出与person李红有knows关系的**3跳以内的环** | match(a:person where name = '李红')-[e:knows]->{1,2}(b)->(a) return a.id, b.id as b_id<br /> |
| 过滤条件 | 找出小红认识的年龄大于20，工资大于5000的人<br /><br /><br />请帮我找出10个性别不是famale且身高小于160,或id大于5的节点 | match(a:person where a.name='小红')-[e:knows]->(b:person where b.age > 20 and b.salary > 5000) return b<br /><br />match(a where (a.gender <> 'female' and a.height < 160) or a.id > 5) return a limit 10 |
| Let单值 | 查询蚂蚁集团创造的软件, **令**软件的minPrice**等于**软件的价格的最小值, 返回公司的id, 软件的minPrice | match(a:company where a.name = '蚂蚁集团')-[e:creates]->(b:software) let b.minPrice = MIN(b.price) return a.id, b.minPrice |
| Let子查询 | 请帮我查找公司蚂蚁集团雇佣的person, **令**这个person的countSalary**的值等于**认识他的人的薪资的和, 再查找他购买的软件<br /><br />给出id为10的城市所属于的国家, 并将国家有关的公司的人数的平均值**赋值**给国家的avgCnt | match(a:company where a.name = '蚂蚁集团')-[e:employee]->(b:person) let b.countSalary = SUM((b:person)<-[e2:knows]-(c:person) => c.salary) match(b:person)-[e3:buy]->(d:software) return b.countSalary, d<br /><br />match(a:city where id = '10')-[e:belong]->(b:country)<-[e2:belong]-(c:company) let b.avgCnt = AVG(c.peopleNumber) return b |
| 函数调用 | **调用SSSP函数**,以'arg1', 10**作为输入**,返回id, distance | match(a:person) call sssp(a, 10) yield (id, distance) return id, distance |
| order | 返回公司创造的软件，并根据公司的规格从大到小、软件的价格**从小到大排序** | match(a:company)-[e:creates]->(b:software) return a.scale,b.price order by a.scale desc, b.price asc |
| group by  | 找出小红认识的人，**根据**这些人的性别**分组**，返回工资的最大值<br /><br />帮我找出与北京大学有关联的公司，返回公司**以**规模**进行分组**的人数的平均值 | match(a:person where person.name = '小红')-[e:knows]->(b:person) return MAX(b.salary) group by b.gender<br /><br />match(a:university where a.name='北京大学')-[e]-(b:company) return AVG(b.peopleNumber) group by b.scale |
| join | 找出郑伟喜欢的所有人，以及认识郑伟的所有人，并将它们**一起返回**<br /><br />请帮我找出和person alice有关联的学校,**称为X**,再帮我找出**和这个X有关联**的其他公司,以及和X有关联的person | match(a:person where a.name = '郑伟')-[e:likes]->(b:person),(a:person where a.name = '郑伟')<-[e2:knows]-(c:person) return a, b, c<br /><br />match(a:person where a.name = 'alice')-[e]-(b:school), (b:school)-[e2]-(c:company),(b:school)-[e3]-(d:person) return a, b, c, d |
| 带图的schema单条查询<br />**（在Console中开启“附带图schema”开关会自动拼接，不需要用户自己输入**） | 使用这个图:CREATE GRAPH g ( Vertex film ( id int ID, name varchar, category varchar, value int ), Vertex cinema ( id int ID, name varchar, address varchar, size int ), Vertex person ( id int ID, name varchar, age int, gender varchar, height int, salary int ), Vertex comment ( id int ID, name varchar, createTime bigint, wordCount int ), Vertex tag ( id int ID, name varchar, value int ), Edge person_likes_comment ( srcId int FROM person SOURCE ID, targetId int FROM comment DESTINATION ID, weight double, f0 int, f1 boolean ), Edge cinema_releases_film ( srcId int FROM cinema SOURCE ID, targetId int FROM film DESTINATION ID, weight double, f0 int, f1 boolean ), Edge person_watch_film ( srcId int FROM person SOURCE ID, targetId int FROM film DESTINATION ID, weight double, f0 int, f1 boolean, timeStamp bigint ), Edge film_has_tag ( srcId int FROM film SOURCE ID, targetId int FROM tag DESTINATION ID, weight double, f0 int, f1 boolean ), Edge person_creates_comment ( srcId int FROM person SOURCE ID, targetId int FROM comment DESTINATION ID, weight double, f0 int, f1 boolean, timeStamp bigint ), Edge comment_belong_film ( srcId int FROM comment SOURCE ID, targetId int FROM film DESTINATION ID, weight double, f0 int, f1 boolean ));找出和person孙梅有person_creates_comment关系的comment,以及和person孙建聪有person_likes_comment关系的comment,将它们都返回 | atch(a:person where a.name = '孙梅')-[e:person_creates_comment]->(b:comment),(c:person where c.name = '孙建聪')-[e2:person_likes_comment]->(d:comment)return a, b, c, d |
| 带图的schema多条查询 | 使用这个图:CREATE GRAPH g ( Vertex book ( id int ID, name varchar, id int ID, name varchar, category varchar, price int, wordCount int, createTime bigint ), Vertex publisher ( id int ID, name varchar, age int, gender varchar, height int, salary int ), Vertex reader ( id int ID, name varchar, age int, gender varchar, height int, salary int ), Vertex author ( id int ID, name varchar, age int, gender varchar, height int, salary int ), Edge author_write_book ( srcId int FROM author SOURCE ID, targetId int FROM book DESTINATION ID, weight double, f0 int, f1 boolean, timeStamp bigint ), Edge publisher_publish_book ( srcId int FROM publisher SOURCE ID, targetId int FROM book DESTINATION ID, weight double, f0 int, f1 boolean, timeStamp bigint ), Edge book_refers_book ( srcId int FROM book SOURCE ID, targetId int FROM book DESTINATION ID, weight double, f0 int, f1 boolean ), Edge reader_likes_book ( srcId int FROM reader SOURCE ID, targetId int FROM book DESTINATION ID, weight double, f0 int, f1 boolean ), Edge author_knows_author ( srcId int FROM author SOURCE ID, targetId int FROM author DESTINATION ID, weight double, f0 int, f1 boolean ));执行以下4个查询:1: 找出被作家黄建聪认识的作家;2: 返回源点标签为作家,目标点标签为作家,标签为author_knows_author的边;3: 请帮我查找所有与book计算机网络有关系的book的id;4: 请帮我查找152个与何雪和张建聪都存在关系的书节点; | 查询语句为:1: match(a:author)<-[e:author_knows_author]-(b:author where b.name='黄建聪') return a, b;2: match(a:author)-[e:author_knows_author]->(b:author) return e;3: match(a:book where a.name='计算机网络')-[e]-(b:book) return b.id;4: match(a where a.name='何雪')-[e]->(b:book)<-[e2]-(c where c.name='张建聪') return b limit 152; |
表格内容描述:
该表格包含三列，分别为“语法”、“提问示例”和“结果”。

1. 第一行数据表示查找点的语法，其中提问示例为“请找出的类型为person的点”，结果为“match(a:person) return a”。
2. 第二行数据表示查找边的语法，提问示例为“返回图中所有的标签为knows的边”，结果为“match(a)-[e:knows]->(b) return e”。
3. 第三行数据包含查找关系的语法，提问示例为“查询10个属于北京的大学，找出5个和小张老师有关系的学生”，结果为“match(a:city where a.name = '北京')<-[:belong]-(b:university) return b limit 10; match(a:teacher where a.name='小张')-[e]-(b:student) return b limit 5”。
4. 第四行数据表示查找多度关系的语法，提问示例涉及查找小王的朋友及与大学相关的关系，结果为相应的查询语句。
5. 第五行数据涵盖循环查询的语法，提问示例要求从张思琪出发通过支付关系查找相应的点，结果为“match(a:person where a.name='张思琪')-[e:pay]->{2,4}(b:person) return b”。
6. 第六行数据处理环的查询，提问示例要求找到与李红有knows关系的3跳以内的环，结果为相关查询。
7. 后续行数据均包含相应的提问示例和结果，逐条描述过滤条件、Let语句、函数调用、排序、分组、连接以及带图的schema查询等。

总结：该表格通过多种语法和提问示例展示了图数据库中常见的查询方式，每一行数据均给出了相应的提问及其对应的查询结果。这些示例涵盖了基础的查找、过滤、分组、连接以及复杂的子查询等操作，帮助用户理解如何使用图数据库进行信息检索。





